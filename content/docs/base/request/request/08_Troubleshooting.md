# 常见构建/运行/调试问题

## 目的

本文档提供 Request 服务常见问题的诊断和解决方案。

## 适用范围

- GN 构建问题
- 运行时错误
- 调试和定位方法

## 关键结论

1. **主要错误来源**: 参数校验失败、权限拒绝、网络错误
2. **调试工具**: HiLog 日志、系统事件、GDB
3. **常见问题**: 大部分可通过正确的配置或权限设置解决

## 构建问题

### 问题 1: 找不到 N-API 模块

**现象**:
```
JavaScript:
import request from '@ohos.request';
// Error: Module not found
```

**原因**: 模块未正确注册或构建

**诊断**:
1. 检查 `BUILD.gn` 是否包含 N-API target
2. 检查 `napi_module_register` 是否被调用

**解决**:
```bash
# 检查构建输出
find . -name "librequest.so"
find . -name "libcachedownload.so"

# 检查模块注册
grep -r "napi_module_register" frameworks/js/napi/
```

**证据**: `request_module.cpp:276-287`

---

### 问题 2: SA 启动失败

**现象**:
```
dmesg: RequestService failed to start
```

**原因**: System Ability 注册失败

**诊断**:
1. 检查 SA 配置文件
2. 检查 SA ID 是否冲突
3. 查看系统日志

**解决**:
```bash
# 检查 SA profile
cat /system/profile/3706.json

# 查看 SA 状态
hidumper -s RequestService
```

**证据**: `etc/sa_profile/3706.json`, `services/src/ability.rs:177-187`

---

### 问题 3: Rust CXX 绑定错误

**现象**:
```
error: failed to generate CXX bindings
```

**原因**: Rust 类型与 C++ 不匹配

**解决**:
```bash
# 清理并重新构建
rm -rf out/
./build.py --build-target ohos
```

**证据**: `services/BUILD.gn:8` (download_server_cxx_gen)

---

## 运行时问题

### 问题 1: 权限拒绝 (201)

**现象**:
```
JavaScript:
request.download(config);
// Error: EXCEPTION_PERMISSION (201)
// Message: "The permissions check fails"
```

**原因**: 应用缺少必要的权限

**诊断**:
1. 检查 module.json5 中的权限配置
2. 使用开发者工具验证权限

**解决**:
```json
// 在 module.json5 中添加权限
{
  "requestPermissions": [
    "ohos.permission.INTERNET"
  ]
}
```

**证据**: `preload_module.cpp:318-329`, `bundle.json:47-48`

---

### 问题 2: 参数校验失败 (202)

**现象**:
```
JavaScript:
request.download({ url: "invalid" });
// Error: EXCEPTION_PARAMCHECK (202)
// Message: "Parameter verification failed"
```

**常见原因**:

| 错误 | 原因 | 解决 |
|-------|------|------|
| URL 格式错误 | URL 不以 `http://` 或 `https://` 开头 | 修正 URL 格式 |
| URL 过长 | 超过 8192 字符 | 缩短 URL |
| 路径无效 | 包含 `..` 或绝对路径 | 使用相对路径 |
| 文件数过多 | 超过限制 | 减少文件数 |

**证据**: `js_initialize.cpp:736-761`

---

### 问题 3: 文件 I/O 错误 (204/205)

**现象**:
```
JavaScript:
request.download({ url: "...", filePath: "/invalid/path" });
// Error: EXCEPTION_FILEIO (204) or EXCEPTION_FILEPATH (205)
```

**原因**: 路径不可访问、权限不足、磁盘满

**诊断**:
1. 检查文件路径是否在沙箱内
2. 检查应用是否有文件访问权限
3. 检查磁盘空间

**解决**:
```javascript
// 使用正确的路径格式
const config = {
  url: "http://example.com/file",
  filePath: "internal://cache/file.txt"  // 正确
};

// 或使用后台模式（需要额外权限）
const config = {
  url: "http://example.com/file",
  filePath: "/data/storage/file.txt",
  mode: request.agent.Mode.BACKGROUND
};
```

**证据**: `js_initialize.cpp:1274-1323`

---

### 问题 4: 网络错误

**现象**:
```
JavaScript:
task.on('fail', (error) => {
  console.log(error);  // ErrorCode::DNS, TCP, SSL, HTTP
});
```

**错误码含义**:

| 错误码 | 含义 | 常见原因 | 解决 |
|-------|------|-----------|------|
| DNS (1) | DNS 解析失败 | 域名错误、网络不可达 | 检查 URL、网络连接 |
| TCP (2) | TCP 连接失败 | 防火墙阻断、端口关闭 | 检查网络配置 |
| SSL (3) | SSL/TLS 握手失败 | 证书无效、协议不匹配 | 检查证书、URL |
| HTTP (4) | HTTP 错误 | 4xx/5xx 状态码 | 检查服务器状态 |
| OTHERS (0) | 其他错误 | 网络断开、超时 | 重试、检查网络 |

**证据**: `common/request_core/src/error_code.rs:29-42`

---

### 问题 5: 任务暂停 (PAUSED_*)

**现象**:
```
JavaScript:
task.on('pause', () => {
  console.log("Task paused");
});
```

**暂停原因**:

| 原因码 | 含义 | 触发条件 | 解决 |
|---------|------|-----------|------|
| PAUSED_QUEUED_FOR_WIFI (0) | 等待 WiFi | 网络类型不匹配 | 连接 WiFi 或修改网络设置 |
| PAUSED_WAITING_FOR_NETWORK (1) | 等待网络 | 无可用网络 | 连接网络 |
| PAUSED_WAITING_TO_RETRY (2) | 等待重试 | 下载失败、网络波动 | 等待自动重试 |
| PAUSED_BY_USER (3) | 用户暂停 | 调用 `task.pause()` | 调用 `task.resume()` |

---

## 调试方法

### 1. 启用详细日志

**修改构建配置**:
```bash
# 编译时启用调试日志
./build.py --build-target ohos --gn-args "is_debug=true"
```

**运行时日志**:
```cpp
// 查找日志位置
// 通常在 /data/log/request/
```

**日志级别**:
- `REQUEST_HILOGD` - Debug 级别
- `REQUEST_HILOGI` - Info 级别
- `REQUEST_HILOGW` - Warning 级别
- `REQUEST_HILOGE` - Error 级别

**证据**: `common/include/log.h`

---

### 2. 使用 HiSysEvent

**查看系统事件**:
```bash
# 使用 shell 命令
hilog -x RequestService
hilog -x DownloadTask

# 或使用 HiDumper
hidumper -s RequestService -v
```

**相关事件**:
- 任务创建/删除
- 进度更新
- 错误发生
- 权限检查失败

**证据**: `common/sys_event/src/cxx/common_event.cpp`, `hisysevent.yaml`

---

### 3. GDB 调试

**附加到下载服务**:
```bash
# 查找进程 PID
ps -ef | grep download_server

# 附加 GDB
gdb -p <PID>

# 设置断点
(gdb) b services/src/service/command/construct.rs:53
(gdb) c
```

---

### 4. 网络抓包

**使用 tcpdump**:
```bash
# 抓取 HTTP 流量
tcpdump -i any port 80 -w capture.pcap

# 分析 Wireshark
wireshark capture.pcap
```

---

## 定位路径

### 1. 参数问题

```
JavaScript 调用
  ↓
N-API 层 (js_initialize.cpp)
  ↓ 参数校验失败？
    ├─ 是 → 返回 EXCEPTION_PARAMCHECK (202)
    └─ 否 → 继续
```

### 2. 权限问题

```
JavaScript 调用
  ↓
N-API 层 (preload_module.cpp)
  ↓ CheckInternetPermission()
    ├─ 失败 → 返回 EXCEPTION_PERMISSION (201)
    └─ 成功 → 继续
```

### 3. IPC 问题

```
N-API 层
  ↓
RequestServiceProxy::Create()
  ↓ IPC 调用
    ├─ 失败 → 返回错误码
    └─ 成功 → 继续处理
```

### 4. 任务执行问题

```
RequestServiceStub::on_remote_request()
  ↓
TaskManager::add_task()
  ↓ 网络请求
    ├─ 成功 → 更新进度
    └─ 失败 → 更新状态、触发失败回调
```

---

## 性能优化建议

### 1. 网络优化

- 使用合适的超时值
- 启用连接重用
- 并发限制（最多 5-10 个同时任务）

### 2. 文件 I/O 优化

- 使用缓冲读写
- 减少小文件写入操作
- 异步写入

### 3. 内存优化

- 及时释放任务资源
- 限制任务队列大小
- 使用共享缓冲区

---

## 相关跳转

- [对外 N-API](03_NAPI_JS_API.md) - API 错误码
- [安全风险评审](07_Security_Review.md) - 安全相关问题
- [目录结构](01_Directory_Structure.md) - 定位模块
- [架构说明](02_Architecture.md) - 理解数据流
