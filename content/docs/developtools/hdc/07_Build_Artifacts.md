# 编译产物

## 目的

描述 hdc 项目编译生成的产物、安装路径、运行时加载关系。

## 适用范围

本文档适用于：
- 了解编译输出
- 部署和安装
- 运行时库依赖
- 故障排查（产物缺失）

## 相关跳转

- [GN 目标梳理](./06_GN_Targets.md) - 构建系统详细说明

---

## 主要编译产物

### PC 端产物

| 产物 | Target 类型 | 安装位置 | 说明 |
|--------|------------|----------|------|
| `hdc` | ohos_executable | SDK 工具链目录 | PC 端命令行工具（Windows/Linux/Mac）|
| `libhdc_register.so` | ohos_shared_library | /usr/lib/ 或系统库路径 | JDWP 注册库 |

**安装路径**：
- Windows SDK：`toolchain/windows/ohos-toolchains/native/llvm/bin/hdc.exe`
- Linux SDK：`toolchain/linux/ohos-toolchains/native/llvm/bin/hdc`
- macOS SDK：`toolchain/darwin/ohos-toolchains/native/llvm/bin/hdc`

### Device 端产物（System 镜像）

| 产物 | Target 类型 | 安装位置 | 说明 |
|--------|------------|----------|------|
| `hdcd` | ohos_prebuilt_executable | `/system/bin/hdcd` | Device 守护进程（系统镜像）|

**配置文件**（`src/daemon/etc/`）：
- `/system/param/hdc.para` - 系统参数
- `/system/param/hdc.para.dac` - DAC 权限参数
- `/system/init/hdcd.cfg` - Daemon 配置（非 root）或 `/system/init/hdcd.root.cfg`（root）

**额外组件**（条件编译）：
- `/system/bin/hdcd_user_permit` - 用户权限助手（如果 `support_hdcd_user_permit`）
- `/system/bin/hdc_credential` - 凭证管理进程（如果 `hdc_feature_support_credential`）
- `/system/bin/sudo` - sudo 工具（如果 `hdc_feature_support_sudo`）

### Device 端产物（Updater 镜像）

| 产物 | Target 类型 | 安装位置 | 说明 |
|--------|------------|----------|------|
| `hdcd` | ohos_prebuilt_executable | `/updater/bin/hdcd` | Device 守护进程（升级镜像）|

**配置文件**：
- `/updater/param/hdc.para` - 系统参数（升级镜像）
- `/updater/param/hdc.para.dac` - DAC 权限参数
- `/updater/init/hdcd.cfg` - Daemon 配置

---

## 运行时库依赖

### PC 端运行时依赖

**hdc 可执行文件依赖**：

| 库名称 | 来源 | 链接方式 |
|--------|--------|----------|
| libusb | third_party | 动态链接（Linux/macOS），静态链接（Windows MinGW）|
| libuv | third_party | 静态链接（`uv_static`）|
| libcrypto + libssl | third_party | 静态链接（`libcrypto_static`, `libssl_static`）|
| lz4 | third_party | 静态链接（`liblz4_static`）|
| bounds_checking_function | third_party | 静态链接（`libsec_static`）|

**OpenHarmony 构建**（如果 `is_ohos`）：

| 库名称 | 来源 | 链接方式 |
|--------|--------|----------|
| libhukssdk | ohos/huks | 动态链接 |
| c_utils | ohos/c_utils | 动态链接 |
| init (libbegetutil) | ohos/startup_init | 动态链接 |

**证据**：`BUILD.gn:422-449`

### Device 端运行时依赖

**hdcd 可执行文件依赖**：

| 库名称 | 来源 | 链接方式 |
|--------|--------|----------|
| libsec_shared | bounds_checking_function | 动态链接 |
| libhilog | ohos/hilog | 动态链接 |
| libbegetutil | ohos/startup_init | 动态链接 |
| libuv | ohos/libuv | 动态链接 |
| liblz4_static | third_party | 静态链接 |
| libcrypto_shared | third_party | 动态链接 |
| libssl_shared | third_party | 动态链接 |
| libhicollie | ohos/hicollie | 动态链接（仅 system 镜像）|
| libselinux | ohos/selinux | 动态链接（如果 `build_selinux`）|
| hitrace_meter | ohos/hitrace | 动态链接（如果 `build_selinux && system`）|
| libhisysevent | ohos/hisysevent | 动态链接（如果 `build_selinux && system`）|

**证据**：`BUILD.gn:145-174`

---

## 运行时加载关系

### Device 端启动流程

```
/init (init 进程)
  │
  ├─> 启动 hdcd [/system/bin/hdcd]
  │
  ├─> 加载配置 [/system/param/hdc.para]
  │
  ├─> 动态链接依赖库
  │   ├─ libhilog
  │   ├─ libbegetutil
  │   ├─ libuv
  │   ├─ liblz4_static
  │   ├─ libcrypto_shared
  │   └─ libssl_shared
  │
  └─> 开始服务
       ├─> USB 监听（FunctionFS）
       ├─> TCP 监听（如果启用）
       ├─> UART 监听（如果启用）
       └─> 处理连接请求
```

### 配置文件说明

#### hdc.para - 系统参数

**路径**：`/system/param/hdc.para` (或 `/updater/param/hdc.para`)

**作用**：定义 hdcd 的系统参数

**证据**：`src/daemon/etc/hdc.para`

可能的参数：
- USB 模式配置
- TCP 模式配置
- UART 模式配置
- 调试日志级别

#### hdcd.para.dac - DAC 权限

**路径**：`/system/param/hdc.para.dac`

**作用**：定义 hdcd 的 DAC（Discretionary Access Control）权限

**证据**：`src/daemon/etc/hdc.para.dac`

可能的权限：
- 文件系统访问权限
- 设备访问权限
- 用户 ID 约束

#### hdcd.cfg - Daemon 配置

**路径**：`/system/init/hdcd.cfg` (user 模式) 或 `/system/init/hdcd.root.cfg` (root 模式)

**作用**：hdcd 守护进程的配置文件

**可能的配置项**：
- 日志级别
- 最大连接数
- 超时设置

---

## Rust 实现产物

### Rust Daemon

| 产物 | Target 类型 | 说明 |
|--------|------------|------|
| `hdcd` | ohos_rust_executable | Rust 版本的 hdcd（如果 `product_name != "ohos-sdk"`）|

**特殊配置**：
- 特性：`emulator`（如果 `is_emulator`）
- 配置：`-Cforce-frame-pointers=yes`

### CFFI 桥接库

| 产物 | Target 类型 | 说明 |
|--------|------------|------|
| `libserialize_structs.a` | ohos_static_library | C++ 到 Rust 的桥接库 |

**作用**：提供 C++ 数据结构和函数的 Rust 绑定，用于 Rust 实现与 C++ 代码互操作

---

## 产物大小估算

### ROM/RAM 占用

**来源**：`bundle.json:24-25`

```json
"rom": "1725KB",
"ram": "1599KB"
```

**说明**：
- ROM：hdcd 可执行文件和库的代码段大小（不包括数据）
- RAM：运行时内存占用（包括数据段、栈、堆）

**注意**：实际占用可能因编译配置和平台而异。

---

## 安装路径总结

### PC 端

| 平台 | SDK 路径 | 产物位置 |
|--------|----------|----------|
| Windows | `toolchain/windows/ohos-toolchains/native/llvm/bin/` | `hdc.exe`, `libhdc_register.dll` |
| Linux | `toolchain/linux/ohos-toolchains/native/llvm/bin/` | `hdc`, `libhdc_register.so` |
| macOS | `toolchain/darwin/ohos-toolchains/native/llvm/bin/` | `hdc`, `libhdc_register.dylib` |

### Device 端

| 镜像 | 安装路径 | 产物位置 |
|--------|----------|----------|
| System | `/system/bin/`, `/system/param/`, `/system/init/` | `hdcd` 及配置文件 |
| Updater | `/updater/bin/`, `/updater/param/`, `/updater/init/` | `hdcd` 及配置文件 |

---

## 故障排查

### 产物缺失问题

**问题**：hdcd 运行时找不到依赖库

**可能原因**：
1. 编译配置问题（依赖未正确链接）
2. 安装路径问题（库未安装到预期位置）
3. SELinux 权限问题（访问库文件被拒绝）

**排查步骤**：
1. 使用 `ldd hdcd` 查看依赖库（Linux）
2. 使用 `readelf -d hdcd` 查看 NEEDED entries
3. 检查 SELinux 日志：`dmesg | grep avc`
4. 确认库文件存在：`ls -l /system/lib/`

### 配置文件问题

**问题**：hdcd 无法读取配置文件

**可能原因**：
1. 文件权限不正确
2. 文件不存在
3. 文件格式错误

**排查步骤**：
1. 检查文件权限：`ls -l /system/param/hdc.para`
2. 查看配置内容：`cat /system/param/hdc.para`
3. 检查 SELinux 上下文：`ls -Z /system/param/`

---

## 关键结论

1. **三部分输出清晰**：
   - PC 端：`hdc` 可执行文件 + `libhdc_register.so`
   - Device 端 System 镜像：`hdcd` + 配置文件
   - Device 端 Updater 镜像：`hdcd` + 配置文件

2. **静态链接 vs 动态链接**：
   - PC 端 `hdc` 主要静态链接库（减小部署依赖）
   - Device 端 `hdcd` 动态链接系统库（减少 ROM 占用）

3. **配置文件分离**：
   - `*.para` 文件：系统参数（可通过参数框架修改）
   - `*.cfg` 文件：进程配置（只读）
   - `*.dac` 文件：DAC 权限定义

4. **Rust 迁移**：
   - 提供 Rust 版本的 `hdcd` 可执行文件
   - 提供 CFFI 桥接库 `libserialize_structs.a` 用于互操作

---

## 待确认事项

**TODO(需确认)**：
1. 各产物在不同平台的实际文件大小
2. 运行时库加载失败的具体错误码
3. 配置文件的完整参数列表
