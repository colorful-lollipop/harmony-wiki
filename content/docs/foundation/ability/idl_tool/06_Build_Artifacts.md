# OpenHarmony IDL Tool - 编译产物

**目的**：详细说明 IDL Tool 的编译产物、安装路径和运行时加载关系。

---

## 适用范围

本文档适用于：
- 需要理解 IDL Tool 构建输出的开发者
- 需要调试代码加载问题的工程师
- 需要分析二进制体积的工程师

---

## 主编译产物

### idl 可执行文件

**GN 目标**: `:idl`
**证据**: `BUILD.gn:377-392`

| 构建类型 | 产物路径 | 文件名 |
|---------|---------|--------|
| Debug | `out/riscv64/ability/idl_tool/clang_x64/` | `idl` |
| Release | `out/riscv64/ability/idl_tool/clang_x64/` | `idl` |
| Host (x86_64) | `out/host/linux-x86_64/...` | `idl` |

**特性**:
- 静态链接（无 .so 依赖）
- 可执行文件（ELF 格式）
- 依赖 `bounds_checking_function:libsec_static`（边届检查）
- 代码大小：约 1-2 MB（取决于编译器和优化级别）

---

## 代码生成产物

### 产物类型与命名规范

**证据**: `idl_tool_2/codegen/code_emitter.h:88-108`

### SA 模式产物

#### C++ 代码

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| **接口头文件** | `<package>_<interface_name>.h` | `OHOS_System_ITestService.h` |
| **客户端代理头** | `<interface_name>_proxy.h` | `test_service_proxy.h` |
| **服务桩头** | `<interface_name>_stub.h` | `test_service_stub.h` |
| **客户端代理实现** | `<interface_name>_proxy.cpp` | `test_service_proxy.cpp` |
| **服务桩实现** | `<interface_name>_stub.cpp` | `test_service_stub.cpp` |

#### TypeScript 代码

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| **接口定义** | `i_<interface_name>.d.ts` | `i_test_service.d.ts` |
| **客户端代理** | `<interface_name>_proxy.ts` | `test_service_proxy.ts` |
| **服务桩** | `<interface_name>_stub.ts` | `test_service_stub.ts` |

#### Rust 代码

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| **模块文件** | `mod.rs` | `mod.rs` |
| **接口定义** | `<interface_name>.rs` | `test_service.rs` |

### HDI 模式产物

#### C 代码

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| **接口头文件** | `<interface_name>.h` | `i_camera_device.h` |
| **服务驱动头** | `<interface_name>_service.h` | `camera_device_service.h` |
| **驱动实现头** | `<interface_name>_driver.h` | `camera_device_driver.h` |

#### C++ 代码

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| **接口头文件** | `<package>_<interface_name>.h` | `OHOS_Hardware_Camera_ICameraDevice.h` |
| **客户端代理头** | `<interface_name>_proxy.h` | `camera_device_proxy.h` |
| **服务桩头** | `<interface_name>_stub.h` | `camera_device_stub.h` |
| **客户端代理实现** | `<interface_name>_proxy.cpp` | `camera_device_proxy.cpp` |
| **服务桩实现** | `<interface_name>_stub.cpp` | `camera_device_stub.cpp` |
| **自定义类型头** | `<interface_name>_types.h` | `camera_device_types.h` |
| **自定义类型实现** | `<interface_name>_types.cpp` | `camera_device_types.cpp` |

#### Java 代码

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| **接口文件** | `I<InterfaceName>.java` | `ICameraDevice.java` |
| **客户端代理** | `<InterfaceName>Proxy.java` | `CameraDeviceProxy.java` |

---

## 安装路径

### 不安装到系统镜像

**证据**: `BUILD.gn:389`

```gn
install_enable = false
```

**说明**：
- IDL Tool 作为**开发工具**，不安装到 OpenHarmony 系统镜像
- 用户需要手动将 `idl` 可执行文件复制到开发环境

### 典型使用场景

```bash
# 场景 1：在本地开发环境使用
cp out/riscv64/ability/idl_tool/clang_x64/idl ~/bin/
chmod +x ~/bin/idl

# 场景 2：在 CI/CD 中使用
# 使用构建产物中的 idl
./build.sh --product-name rk3568
```

---

## 运行时加载关系

### 生成的代码依赖

**证据**: `bundle.json:24-27`, `codegen/cpp_code_emitter.cpp:67`

#### SA C++ 代码依赖

| 依赖 | 头文件 | 用途 |
|------|--------|------|
| **iremote_stub.h** | IPC 框架 | `IRemoteStub<IInterface>` |
| **iremote_object.h** | IPC 基类 | IPC 机制 |
| **iremote_request.h** | IPC 请求处理 | OnRemoteRequest 方法 |
| **message_parcel.h** | IPC 数据序列化 | 数据读写 |
| **ipc_skeleton.h** | IPC 骨架基础 | 服务端基础类 |

#### SA TS 代码依赖

| 依赖 | N-API 模块 | 用途 |
|------|------------|------|
| **@ohos.rpc** | RPC 模块 | `import rpc from "@ohos.rpc"` |
| **@ohos.hilog** | 日志模块 | `import hilog from "@ohos.hilog"` |

#### HDI 代码依赖

| 依赖 | 头文件 | 用途 |
|------|--------|------|
| **hdf_base.h** | HDF 基类 | HDI 基础机制 |
| **hdf_log.h** | HDF 日志 | HDF 日志接口 |
| **hdf_sbuf.h** | HDF 缓冲区 | 数据传输 |
| **osal/hdf_osal.h** | HDF 操作系统抽象层 | HDF OSAL |

---

## 元数据文件

### 产物（仅 SA 模式）

**证据**: `idl_tool_2/main.cpp:90-106`

**生成命令**:
```bash
idl -c <idl_file> -s <metadata_file>
```

**元数据文件特性**:
- **格式**: 二进制格式（位置无关）
- **魔数**: `0x1DF02ED1`（用于识别）
- **内容**: 接口类型信息、方法签名、类型映射
- **大小**: 取决于接口复杂度（通常几 KB）
- **文件扩展**: 无固定扩展（由用户指定）

**元数据用途**:
1. **运行时类型信息（RTTI）**
   - 动态接口发现
   - 类型反射
   - 版本兼容性检查

2. **跨语言绑定**
   - C++ 服务与 TS 客户端通信
   - 类型安全的数据序列化

---

## 构建变量影响

### Debug vs Release

| 变量 | Debug | Release |
|--------|-------|---------|
| 符号表 | 包含 | 不包含或压缩 |
| 优化级别 | -O0 或 -O1 | -O2 或 -O3 |
| 调试信息 | 包含 | 不包含 |
| 代码大小 | 较大 | 较小 |
| 性能 | 较慢 | 较快 |

### 目标 CPU

| CPU | 说明 | 典型用途 |
|-----|------|---------|
| riscv64 | RISC-V 64 位 | OpenHarmony 标准架构 |
| x86_64 | Intel/AMD 64 位 | 宿主机交叉编译 |

---

## 关键结论

1. **主要产物是 `idl` 可执行文件** - 静态链接，无 .so 依赖
2. **不安装到系统镜像** - 作为开发工具使用
3. **生成的代码依赖系统组件** - IPC、SAMGR、HiLog 等
4. **SA 模式支持元数据** - 二进制格式，用于运行时类型信息
5. **支持多种目标语言** - C/C++/Java/TS/Rust，每种语言有不同的产物命名规则

---

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览与支持的后端
- [03_External_API.md](03_External_API.md) - 命令行接口
- [05_GN_Targets.md](05_GN_Targets.md) - GN 构建配置

---

**最后更新**: 2026-02-06
