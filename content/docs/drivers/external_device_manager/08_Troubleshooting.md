# 常见问题与调试

本文档汇总了扩展外部设备管理模块在构建、运行和调试过程中可能遇到的常见问题及其解决方案。文档按问题类型分类，便于。

## 构建快速定位和解决问题问题

### 问题 1：编译找不到头文件

**错误现象**：

```
fatal error: 'xxx.h' file not found
```

**可能原因**：

头文件路径未正确配置。模块的 include_dirs 分散在多个 BUILD.gn 文件中，定制编译时可能遗漏部分路径。

**解决方案**：

首先确认构建环境是否完整初始化。执行以下命令设置环境变量：

```bash
source build/build.sh
```

如果使用独立编译，确保 `--gn-args` 参数包含正确的 include 路径。检查目标 BUILD.gn 文件的 `include_dirs` 配置是否完整。模块的通用配置路径通常在 `utils/BUILD.gn` 中定义。

### 问题 2：外部依赖缺失

**错误现象**：

```
ninja: error: 'xxx' referenced but not defined
```

**可能原因**：

`external_deps` 中声明的外部依赖未在构建环境中配置，或依赖版本不匹配。

**解决方案**：

模块依赖的外部组件在 `bundle.json` 中定义，包括 `hilog`、`ipc`、`safwk`、`samgr`、`ability_runtime`、`access_token` 等。确认构建环境是否包含这些组件的源码或预编译库。检查 `external_deps` 版本标识是否与构建环境匹配。

### 问题 3：32 位架构编译错误

**错误现象**：

```
error: binder ipc requires 64-bit build for this architecture
```

**可能原因**：

在 32 位 ARM 架构上编译时未启用 32 位 Binder IPC 支持。

**解决方案**：

在 BUILD.gn 中添加 32 位 Binder 编译标志：

```gn
if (target_cpu == "arm") {
  cflags += [ "-DBINDER_IPC_32BIT" ]
}
```

或者在构建命令中指定正确的目标架构。

### 问题 4：CFI 编译警告

**错误现象**：

```
warning: control flow integrity check failed
```

**可能原因**：

启用了 CFI（Control Flow Integrity） sanitizer，但代码中存在不符合 CFI 规范的虚函数调用或函数指针调用。

**解决方案**：

CFI 配置在 `services/BUILD.gn` 中定义：

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  cfi_no_nvcall = true
  cfi_vcall_icall_only = true
  debug = false
}
```

如果 CFI 检查失败，需要检查并修复代码中的非规范调用。如果无法修复，可以考虑在调试阶段禁用 CFI。

### 问题 5：NDK 库版本不匹配

**错误现象**：

```
symbol version mismatch for library libxxx.z.so
```

**可能原因**：

NDK 库的 ABI 版本与使用方代码不兼容。

**解决方案**：

确保 NDK 库和使用方代码使用相同的 NDK 版本编译。检查 `external_deps` 中的 NDK 组件版本标识。NDK 库的安装路径为 `system/lib/ndk/`。

## 运行问题

### 问题 6：SA 启动失败

**错误现象**：

```
Failed to start system ability 5110
hdf_ext_devmgr: initialize failed
```

**可能原因**：

SA 依赖的底层服务未就绪，或权限配置不正确。

**解决方案**：

首先检查 SA 配置文件 `sa_profile/5110.json` 是否正确部署。验证依赖的 HDI 服务是否正常运行：

```bash
hdc shell hidumper -sa
```

检查权限配置。服务启动需要访问 USB HDI、IPC 等系统服务，确保 `hdf_ext_devmgr` 进程具有相应权限。查看系统日志获取详细错误信息：

```bash
hdc shell hilog | grep -E "EDM|ExtDevice|DriverExt"
```

### 问题 7：设备枚举失败

**错误现象**：

```
queryDevices() returns empty array
```

**可能原因**：

USB 总线扩展未正确初始化，或设备未被内核识别。

**解决方案**：

首先确认 USB 设备是否被操作系统识别：

```bash
hdc shell lsusb
```

检查 USB 服务状态：

```bash
hdc shell hidumper -s Usb
```

查看设备管理器日志：

```bash
hdc shell hilog | grep -E "UsbBus|UsbDev"
```

确认 `UsbBusExtension` 是否正确获取 USB 接口实例。检查 `UsbDevSubscriber` 是否成功注册热插拔监听。

### 问题 8：设备绑定失败

**错误现象**：

```
bindDevice() error code: 201 (Permission denied)
```

**可能原因**：

应用缺少必要的系统权限。

**解决方案**：

调用设备绑定 API 需要 `ohos.permission.ACCESS_EXTENSIONAL_DEVICE_DRIVER` 权限。权限是系统级别，仅系统应用可申请。检查应用的权限配置文件是否包含该权限声明。如果权限正确，检查权限验证是否被正确调用（`driver_ext_mgr.cpp:167`）。

### 问题 9：驱动加载超时

**错误现象**：

```
bindDevice() timeout, callback not called
```

**可能原因**：

驱动扩展 Ability 启动超时，或回调注册失败。

**解决方案**：

检查驱动扩展 Ability 是否正确定义在应用的配置文件中。验证 AMS 是否能正常启动 Ability：

```bash
hdc shell aa dump -a
```

检查 `DriverExtensionController` 的日志，确认启动请求是否正确发送：

```bash
hdc shell hilog | grep -E "DriverExtension|Connect"
```

### 问题 10：DDK 调用失败

**错误现象**：

```
OH_Usb_Init() returns error code 27400002 (INVALID_OPERATION)
```

**可能原因**：

DDK 初始化失败，或设备句柄无效。

**解决方案**：

确认在使用 DDK 前正确调用初始化函数。USB DDK 调用流程：

```c
int32_t ret = OH_Usb_Init();
if (ret != USB_DDK_SUCCESS) {
    // 处理初始化失败
}
```

检查设备 ID 是否有效。设备 ID 由设备枚举流程获取，确认 `queryDevices()` 是否成功返回设备。对于已关闭的设备，需要重新枚举获取新的设备 ID。

### 问题 11：权限错误码区分

**错误现象**：

```
bindDevice() returns error code: 26300002 (SERVICE_NOT_ALLOW_ACCESS)
```

**可能原因**：

权限验证通过，但服务级别拒绝访问。

**解决方案**：

该错误发生在权限检查通过后，服务内部根据业务规则拒绝访问。可能原因包括：应用不在驱动的可访问名单中、驱动状态异常、用户已取消绑定等。查看服务日志获取详细拒绝原因。

## 调试方法

### 日志查看

模块使用 `hilog` 输出日志，日志标签为 `MODULE_*` 系列常量。

**查看所有 EDM 日志**：

```bash
hdc shell hilog | grep -E "EDM|DriverExtMgr"
```

**按模块过滤**：

| 模块标签 | 说明 |
|---------|------|
| `MODULE_DEV_MGR` | 设备管理器 |
| `MODULE_PKG_MGR` | 包管理器 |
| `MODULE_BUS_USB` | USB 总线 |
| `MODULE_BASE_DDK` | 基础 DDK |

**日志级别控制**：

日志级别通过 `hilog_wrapper.h` 中的宏控制：

```cpp
#define EDM_LOGD(tag, format, ...)  // Debug
#define EDM_LOGI(tag, format, ...)  // Info
#define EDM_LOGW(tag, format, ...)  // Warning
#define EDM_LOGE(tag, format, ...)  // Error
```

### 服务状态诊断

**查看 SA 状态**：

```bash
hdc shell hidumper -sa 5110
```

**查看设备列表**：

```bash
hdc shell hidumper | grep -A 20 "ExternalDevice"
```

### 调试方法

**启用详细日志**：

修改代码中的日志级别宏，从 `LOGE` 改为 `LOGD`。重新编译后运行。

**GDB 调试**：

对于 native 代码，可以使用 GDB 进行调试：

```bash
hdc gdb
(gdb) target remote :1234
```

**Native 崩溃分析**：

当服务崩溃时，查看崩溃堆栈：

```bash
hdc shell backtrace
```

### 常见调试场景

**场景 1：设备枚举正常但绑定失败**

排查步骤：首先确认权限配置正确；然后检查驱动是否已安装并通过系统认证；最后验证驱动扩展 Ability 的配置是否正确，特别是 `bundleName` 和 `abilityName` 是否与应用配置一致。

**场景 2：DDK 读写失败**

排查步骤：首先确认设备句柄是否有效；然后检查设备是否仍连接；最后验证传输参数（缓冲区大小、端点地址）是否正确。

**场景 3：回调未被调用**

排查步骤：首先确认回调是否正确注册；然后检查回调对象是否在调用期间保持有效；最后验证异步操作是否正常完成。

## 性能问题

### 问题 12：设备枚举耗时过长

**排查方向**：

USB 设备枚举涉及 USB 描述符读取和解析。耗时可能原因包括：USB 设备响应慢、描述符解析效率低、大量设备同时枚举。优化建议：考虑实现增量枚举机制；对枚举操作添加超时控制；在非主线程执行耗时枚举。

### 问题 13：内存占用过高

**排查方向**：

模块 RAM 占用约 8000KB。过高可能原因包括：设备列表未及时清理、共享内存泄漏、数据库连接未释放。优化建议：定期清理断开连接的设备记录；检查 Ashmem 使用是否正确释放；确认数据库操作的生命周期管理。

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
| [06_Build_System.md](./06_Build_System.md) | GN 构建系统说明 |
| [07_Security_Review.md](./07_Security_Review.md) | 安全风险分析 |
