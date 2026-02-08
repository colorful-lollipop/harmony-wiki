# 常见问题

> 目的：汇总设备互信认证模块的常见构建、运行和调试问题，提供定位路径和解决方案。
>
> 适用范围：使用本模块过程中遇到问题的开发者。

---

## 1. 构建问题

### 1.1 GN 构建失败

**问题表现**：
```
ERROR at //base/security/device_auth/BUILD.gn:16
Expected a label.
```

**可能原因**：
- `os_level` 参数未正确设置
- 依赖组件未拉取

**解决方案**：
```bash
# 设置正确的 os_level
gn gen out/standard --args='os_level="standard"'

# 或检查依赖
hb set
hb build
```

**证据**：`BUILD.gn:16-39`

---

### 1.2 N-API 编译缺失

**问题表现**：
```
error: 'napi/native_api.h' file not found
```

**可能原因**：
- `os_level` 不是 `standard`
- napi 依赖未配置

**解决方案**：
```bash
# 确认 os_level 为 standard
gn gen out/standard --args='os_level="standard"'

# 或检查 bundle.json 依赖
cat bundle.json | grep napi
```

---

### 1.3 依赖组件缺失

**问题表现**：
```
ninja: error: unknown target '//base/security/device_auth:deviceauth_napi_build'
```

**可能原因**：
- N-API 仅在 standard 级别支持
- 构建配置未包含

**解决方案**：
```bash
# 查看可用的构建目标
gn ls out/standard //base/security/device_auth/

# 编译 N-API（仅 standard）
hb build deviceauth_napi -p
```

---

## 2. 运行问题

### 2.1 SA 启动失败

**问题表现**：
```
E0010/SA: DeviceAuthAbility: OnStart failed
```

**日志位置**：
```
hilog | grep DeviceAuthAbility
```

**可能原因**：
1. SA 配置文件缺失
2. 依赖服务未就绪
3. 权限不足

**解决方案**：
```bash
# 检查 SA 配置
cat /system/etc/init/deviceauth_service.cfg

# 检查权限
dumpsys app devauth
```

---

### 2.2 获取实例失败

**问题表现**：
```
Error: InitDeviceAuthService failed
```

**代码位置**：
- N-API：`credmgr_napi.cpp:124-128`
- C++：`device_auth.h`

**解决方案**：
```typescript
import deviceauth from '@ohos.deviceauth';

// 重试机制
let retryCount = 3;
while (retryCount > 0) {
  try {
    const credManager = deviceauth.getCredMgrInstance();
    break;
  } catch (e) {
    retryCount--;
    await new Promise(r => setTimeout(r, 1000));
  }
}
```

---

### 2.3 认证超时

**问题表现**：
```
HC_ERR_TIME_OUT: Authentication timeout
```

**日志位置**：
```
hilog | grep "HC_ERR_TIME_OUT"
```

**可能原因**：
1. 对端设备无响应
2. 网络连接中断
3. 协议握手卡住

**排查步骤**：
```bash
# 1. 检查 SoftBus 连接
hdc shell "param get | grep softbus"

# 2. 检查设备状态
hdc shell "hidumper -s DeviceAuthService"

# 3. 检查网络
hdc shell "ping <peer_ip>"
```

---

## 3. 调试方法

### 3.1 日志查看

**日志标签**：
| 标签 | 说明 |
|------|------|
| `DeviceAuth` | 设备认证主日志 |
| `GroupManager` | 群组管理日志 |
| `SessionMgr` | 会话管理日志 |
| `Protocol` | 协议日志 |

**命令**：
```bash
# 查看所有设备认证日志
hilog | grep "DeviceAuth"

# 查看详细日志
hilog -D all | grep "DeviceAuth"

# 查看错误日志
hilog | grep -E "Error|FAILED"
```

**证据**：`deps_adapter/os_adapter/interfaces/linux/hc_log.h`

---

### 3.2 调试宏

**可用调试级别**：
```cpp
LOGI("Info message");   // 信息
LOGE("Error message");   // 错误
LOGW("Warn message");   // 警告
LOGD("Debug message");  // 调试（仅 Debug 版本）
```

**启用调试日志**：
```bash
# 编译时添加
--args='build_type="debug"'
```

---

### 3.3 状态 dump

**SA 状态查询**：
```bash
hdc shell "hidumper -s DeviceAuthAbility"

# 或
hdc shell "hidumper -s 4701"
```

**数据库查询**：
```bash
# 群组数据
hdc shell "cat /data/service/el/device_auth/group.db"

# 凭证数据
hdc shell "cat /data/service/el/device_auth/cred.db"
```

---

### 3.4 抓包分析

**SoftBus 数据抓包**：
```bash
# 启用调试
hdc shell "param set debug.dsoftbus 1"

# 导出日志
hdc shell "hilog > dsoftbus.log"
```

---

## 4. 性能问题

### 4.1 认证耗时过长

**排查方法**：
```bash
# 1. 查看耗时统计
hdc shell "hidumper -s DeviceAuthService | grep -i time"

# 2. 检查 CPU 占用
hdc shell "top -n 1 | grep deviceauth"

# 3. 检查内存
hdc shell "cat /proc/meminfo | grep -i slab"
```

**常见原因**：
1. 密钥生成使用软件实现
2. 未启用硬件加速
3. 大量磁盘 I/O

**解决方案**：
```bash
# 使用硬件密钥库
--args='device_auth_use_hardware_key=true'
```

---

### 4.2 内存占用过高

**排查方法**：
```bash
# 内存使用统计
hdc shell "cat /proc/<pid>/status | grep VmRSS"

# 内存泄漏检测
hdc shell "hidumper -s DeviceAuthService | grep -i memory"
```

---

## 5. 常见错误码

### 5.1 通用错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 0 | 成功 | - |
| 2 | 参数错误 | 检查输入参数 |
| 4 | 空指针 | 检查对象初始化 |
| 5 | 内存分配失败 | 检查系统内存 |
| 9 | 超时 | 检查网络连接 |

### 5.2 IPC 错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 12289 | IPC 内部错误 | 检查 SA 状态 |
| 12297 | 获取服务失败 | 检查 SA 是否启动 |
| 12306 | 服务已死亡 | 重启服务 |

### 5.3 群组错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 20481 | 访问拒绝 | 检查权限 |
| 20483 | 服务需重启 | 重启设备认证服务 |
| 20484 | 无候选群组 | 检查群组配置 |

### 5.4 身份服务错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 65537 | HUKS 密钥生成失败 | 检查 HUKS 服务 |
| 65545 | 本地凭据不存在 | 检查凭据是否导入 |
| 65551 | PIN 码不匹配 | 检查 PIN 码 |

---

## 6. 相关跳转

| 内容 | 文档 |
|------|------|
| API 接口 | [03_API_Reference.md](./03_API_Reference.md) |
| Inner API | [04_Inner_API.md](./04_Inner_API.md) |
| 构建配置 | [05_Build_Config.md](./05_Build_Config.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |

---

*本文档最后更新：2026-02-06*
