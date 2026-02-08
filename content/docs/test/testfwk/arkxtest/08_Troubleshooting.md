# 故障排查指南

本文档汇总 ArkXtest 框架在构建、部署和运行时常见的故障及其解决方案。

## 构建问题

### GN 构建失败

#### 问题: GN 未找到

**错误信息**:
```
ERROR: gn not found. Please set the GN path.
```

**解决方案**:
```bash
# 确保 OpenHarmony 构建环境已配置
source build.sh

# 或手动设置 GN 路径
export GN_PATH=/path/to/gn
```

#### 问题: build.sh 未找到

**错误信息**:
```
./build.sh: No such file or directory
```

**解决方案**:
```bash
# 必须从 OpenHarmony 根目录执行
cd /path/to/openharmony

# 然后运行构建命令
./build.sh --product-name rk3568 --build-target uitestkit
```

**证据**: AGENTS.md 明确要求从 OpenHarmony 根目录执行构建命令

#### 问题: 权限被拒绝

**错误信息**:
```
permission denied: out/rk3568/testfwk/arkxtest/uitest
```

**解决方案**:
```bash
# 检查输出目录权限
chmod -R 755 out/

# 或清理后重新构建
rm -rf out/rk3568
./build.sh --product-name rk3568 --build-target uitestkit
```

### 编译错误

#### 问题: 缺少头文件

**错误信息**:
```
fatal error: 'xxx.h' file not found
```

**解决方案**:
1. 检查 `include_dirs` 配置
2. 确认依赖子系统已构建
3. 清理并重新构建

```bash
# 清理构建产物
rm -rf out/rk3568/obj/test/testfwk/arkxtest/*
rm -rf out/rk3568/testfwk/arkxtest/*

# 重新构建
./build.sh --product-name rk3568 --build-target uitestkit
```

#### 问题: 链接失败

**错误信息**:
```
undefined reference to `xxx'
```

**解决方案**:
1. 检查 `deps` 和 `external_deps` 配置
2. 确认依赖库已构建

```bash
# 先构建依赖
./build.sh --product-name rk3568 --build-target test_server_service
./build.sh --product-name rk3568 --build-target test_server_client

# 再构建目标
./build.sh --product-name rk3568 --build-target uitestkit
```

---

## 部署问题

### 设备连接问题

#### 问题: HDC 连接失败

**错误信息**:
```
error: device not found
```

**解决方案**:
```bash
# 列出连接的设备
hdc list targets

# 如果设备未识别，检查 USB 调试
adb devices

# 重新连接设备
hdc kill
hdc start
```

#### 问题: 挂载失败

**错误信息**:
```
mount: 'overlayfs' not supported or missing required kernel support
```

**解决方案**:
```bash
# 使用 target mount
hdc target mount

# 如果失败，尝试手动挂载
hdc shell mount -o rw,remount /
```

#### 问题: 权限被拒绝

**错误信息**:
```
permission denied: /system/bin/uitest
```

**解决方案**:
```bash
# 确保系统分区可写
hdc target mount
hdc shell mount -o rw,remount /

# 设置执行权限
hdc shell chmod +x /system/bin/uitest
hdc shell chmod +x /system/bin/perftest
```

### 文件推送问题

#### 问题: 文件不存在

**错误信息**:
```
error: file not found: out/rk3568/testfwk/arkxtest/uitest
```

**解决方案**:
```bash
# 确认产物存在
ls -la out/rk3568/testfwk/arkxtest/

# 如果不存在，先构建
./build.sh --product-name rk3568 --build-target uitestkit
```

#### 问题: 产物路径错误

**解决方案**:
产物路径格式为: `out/<product>/testfwk/arkxtest/<产物>`

```bash
# 确认产品名
./build.sh --product-name rk3568 --help

# 正确路径示例
hdc file send out/rk3568/testfwk/arkxtest/uitest /system/bin/uitest
hdc file send out/rk3568/testfwk/arkxtest/libuitest.z.so /system/lib/module/libuitest.z.so
```

---

## 运行时问题

### UiTest 服务端问题

#### 问题: 服务端无法启动

**排查步骤**:
```bash
# 1. 检查二进制文件是否存在
ls -la /system/bin/uitest

# 2. 检查权限
ls -la /system/bin/uitest

# 3. 尝试手动启动
hdc shell uitest

# 4. 查看日志
hdc shell hilog | grep -i uitest
```

**常见原因**:
- 权限不足
- 依赖库缺失
- 开发者模式未启用

#### 问题: 控件查找失败

**排查步骤**:
```bash
# 1. 导出布局信息
hdc shell uitest dumpLayout -p /data/local/tmp/layout.json

# 2. 查看布局
cat /data/local/tmp/layout.json

# 3. 检查无障碍服务
hdc shell hidumper -s Accessibility
```

**代码证据**: `uitest/server/dump_handler.cpp` 实现了布局导出

#### 问题: IPC 超时

**排查步骤**:
```bash
# 1. 检查服务端是否运行
ps -ef | grep uitest

# 2. 检查日志
hdc shell hilog | grep -i uitest

# 3. 重启服务端
hdc shell killall uitest
hdc shell uitest start-daemon test@1234@1000@0
```

**当前代码**: `uitest/napi/uitest_napi.cpp:67-84` 实现了连接建立

### PerfTest 服务端问题

#### 问题: 性能数据采集失败

**排查步骤**:
```bash
# 1. 检查 TestServer SA
hdc shell smgr list | grep 5502

# 2. 查看日志
hdc shell hilog | grep -i perf

# 3. 检查 HiSysEvent 服务
hdc shell hidumper -s HiSysEvent
```

**常见原因**:
- TestServer 未运行
- 权限不足
- 进程不存在

#### 问题: 回调超时

**错误信息**:
```
ERR_CALLBACK_TIMEOUT: Callback execution timeout
```

**解决方案**:
```javascript
// 增加超时时间
let strategy = new PerfTestStrategy();
strategy.timeout = 60000; // 60秒
```

**代码证据**: `perftest/napi/src/callback_code_napi.cpp` 实现了回调执行

### TestServer 问题

#### 问题: SA 未注册

**排查步骤**:
```bash
# 1. 检查 SA 列表
hdc shell smgr list | grep 5502

# 2. 检查配置文件
cat /system/profile/testserver.json
cat /system/etc/init/testserver.cfg

# 3. 查看启动日志
hdc shell hilog | grep -i "C03110"  # TestServer 日志域
```

**配置文件证据**:
```json
// testserver/sa_profile/5502.json
{
  "name": 5502,
  "libpath": "libtest_server_service.z.so",
  "run-on-create": false
}
```

#### 问题: 权限错误

**错误码**:
- `TEST_SERVER_GET_INTERFACE_FAILED` (-1)
- `TEST_SERVER_ADD_DEATH_RECIPIENT_FAILED` (19000001)

**解决方案**:
```bash
# 1. 检查开发者模式
hdc shell param get const.security.developermode.state

# 2. 如果返回 false，需要启用开发者模式
# 开发者模式需在设备设置中启用

# 3. 检查 SELinux 上下文
hdc shell ls -Z /system/lib/libtest_server_service.z.so
```

**代码证据**: `testserver/src/service/test_server_service.cpp:92-104`

---

## 常见错误码

### UiTest 错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 0 | 成功 | - |
| 401 | 参数错误 | 检查 API 参数 |
| 10200001 | 无效选择器 | 检查 On/By 选择器 |
| 10200002 | 控件未找到 | 检查控件是否存在 |
| 10200003 | 操作失败 | 检查控件状态 |

### PerfTest 错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `NO_ERROR` | 成功 |
| 32400001 | `ERR_INITIALIZE_FAILED` | 初始化失败 |
| 32400002 | `ERR_INTERNAL` | 系统错误 |
| 32400003 | `ERR_INVALID_INPUT` | 无效输入 |
| 32400004 | `ERR_CALLBACK_FAILED` | 回调执行失败 |
| 32400005 | `ERR_DATA_COLLECTION_FAILED` | 数据采集失败 |
| 32400006 | `ERR_GET_RESULT_FAILED` | 获取结果失败 |
| 32400007 | `ERR_API_USAGE` | API 使用错误 |

**证据**: `perftest/core/include/frontend_api_defines.h:33-50`

### TestServer 错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `TEST_SERVER_OK` | 成功 |
| -1 | `TEST_SERVER_GET_INTERFACE_FAILED` | 获取 SA 接口失败 |
| 19000001 | `TEST_SERVER_ADD_DEATH_RECIPIENT_FAILED` | 添加死亡回调失败 |
| 19000002 | `TEST_SERVER_CREATE_PASTE_DATA_FAILED` | 创建剪贴板数据失败 |
| 19000003 | `TEST_SERVER_SET_PASTE_DATA_FAILED` | 设置剪贴板数据失败 |
| 19000004 | `TEST_SERVER_PUBLISH_EVENT_FAILED` | 发布事件失败 |
| 19000005 | `TEST_SERVER_SPDAEMON_PROCESS_FAILED` | SmartPerf 操作失败 |
| 19000006 | `TEST_SERVER_COLLECT_PROCESS_INFO_FAILED` | 采集进程信息失败 |
| 19000007 | `TEST_SERVER_OPERATE_WINDOW_FAILED` | 窗口操作失败 |
| 19000008 | `TEST_SERVER_DATASHARE_FAILED` | DataShare 操作失败 |

**证据**: `testserver/src/utils/test_server_error_code.h`

---

## 日志查看

### UiTest 日志

```bash
# 查看所有 UiTest 日志
hdc shell hilog | grep -i uitest

# 查看详细日志
hdc shell hilog -v detail | grep -i uitest

# 过滤特定标签
hdc shell hilog | grep "UiTestKit"
```

### PerfTest 日志

```bash
# 查看 PerfTest 日志
hdc shell hilog | grep -i perf

# 查看 TestServer 日志
hdc shell hilog | grep "C03110"
```

### 日志级别

| 级别 | 说明 |
|------|------|
| DEBUG | 调试信息 |
| INFO | 普通信息 |
| WARN | 警告 |
| ERROR | 错误 |
| FATAL | 严重错误 |

---

## 调试技巧

### 1. 布局调试

```bash
# 导出完整布局
hdc shell uitest dumpLayout -p /data/local/tmp/layout.json -a

# 导出特定窗口
hdc shell uitest dumpLayout -p /data/local/tmp/app.json -b com.example.app

# 查看层级
hdc shell cat /data/local/tmp/layout.json | jq '.[].children'
```

### 2. 截图调试

```bash
# 全屏截图
hdc shell uitest screenCap -p /data/local/tmp/screenshot.png

# 指定显示截图
hdc shell uitest screenCap -p /data/local/tmp/screenshot.png -d 0
```

### 3. 输入调试

```bash
# 点击坐标
hdc shell uitest uiInput click 500 1000

# 滑动
hdc shell uitest uiInput swipe 100 1000 900 1000 600

# 输入文本
hdc shell uitest uiInput inputText 500 1000 "Hello"
```

### 4. 服务端调试

```bash
# 启动服务端（带参数）
hdc shell uitest start-daemon token

# 查看服务端日志
hdc shell "cat /data/log/uitest.log"

# 杀掉服务端
hdc shell killall uitest
```

---

## 回退与恢复

### 恢复系统状态

```bash
# 恢复原始二进制
hdc shell rm /system/bin/uitest
hdc shell rm /system/lib/module/libuitest.z.so
hdc shell rm /system/lib/libuitest_ani.so
hdc shell rm /system/bin/perftest
hdc shell rm /system/lib/module/test/libperftest.z.so

# 重启设备
hdc shell reboot
```

### 清理构建缓存

```bash
# 清理 GN 缓存
rm -rf out/rk3568/.gn

# 清理 Ninja 缓存
rm -rf out/rk3568/.ninja*

# 完整清理
rm -rf out/rk3568
```

---

## 相关文档链接

- [OpenHarmony 测试框架指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/application-test/uitest-guidelines.md)
- [UiTest API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-test-kit/js-apis-uitest.md)
- [PerfTest API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-test-kit/js-apis-perftest.md)
