# 01 - 项目概览 (Overview)

## 1. 项目定位

### 一句话定义

**OpenHarmony IDL Tool** 是一个编译时代码生成器，将接口定义语言（IDL）文件转换为跨进程/跨设备通信所需的代理（Proxy）和桩（Stub）代码。

### 解决的问题

在 OpenHarmony 系统中：
- 客户端和服务端进行 IPC（进程间通信）或 RPC（远程过程调用）时
- 需要定义双方都认可的接口，以保障成功通信
- IDL 工具将接口定义转换为操作系统能够理解的基本类型封装代码

### 能力边界

| 能做什么 | 不能做什么 |
|---------|-----------|
| 解析 .idl 接口定义文件 | 不实现 IPC/RPC 运行时 |
| 生成 C/C++ Proxy/Stub 代码 | 不生成服务端业务逻辑 |
| 生成 TypeScript/Rust/Java 代码 | 不提供 N-API/JS 接口 |
| 生成 HDI 硬件接口代码 | 不管理硬件设备 |
| 生成 SA 系统能力接口代码 | 不实现系统服务 |

---

## 2. 核心概念

### 2.1 IDL（Interface Definition Language）

IDL 是一种用于定义软件组件接口的语言，特点：
- **接口形式定义服务**：专注于接口定义，隐藏实现细节
- **支持跨进程/跨设备调用**：简化分布式通信接口实现
- **语言无关**：同一接口可生成多种编程语言代码

### 2.2 IPC/RPC 通信模型

```
┌─────────────┐          ┌─────────────┐
│   Client    │ ───────→ │   Proxy     │
│  (应用进程)  │          │  (代理对象)  │
└─────────────┘          └──────┬──────┘
                                │ 跨进程/跨设备
                                ↓
                         ┌─────────────┐
                         │ IPC Driver  │
                         └──────┬──────┘
                                │
                                ↓
┌─────────────┐          ┌─────────────┐
│   Server    │ ←─────── │    Stub     │
│  (系统服务)  │          │  (桩对象)    │
└─────────────┘          └─────────────┘
```

### 2.3 HDI vs SA

| 特性 | HDI（Hardware Device Interface）| SA（System Ability）|
|------|--------------------------------|---------------------|
| **用途** | 硬件设备抽象接口 | 系统能力服务接口 |
| **运行域** | 内核态/驱动层 | 系统服务层 |
| **支持语言** | C, C++, Java | C++, TypeScript, Rust |
| **代码生成** | Proxy + Driver + Stub + Service | Proxy + Stub + Interface |
| **典型场景** | Camera, Audio, Sensor 驱动接口 | Ability, Service 能力接口 |

---

## 3. 快速开始

### 3.1 创建 IDL 文件

创建 `IIdlTestService.idl`：

```idl
interface OHOS.IIdlTestService {
    int TestIntTransaction([in] int data);
    void TestStringTransaction([in] String data);
}
```

### 3.2 生成 C++ 代码

```bash
idl -gen-cpp -d output_dir -c IIdlTestService.idl
```

**输出文件** (`output_dir/`)：
```
iidl_test_service.h         # 接口定义头文件
idl_test_service_proxy.h    # Proxy 类头文件
idl_test_service_proxy.cpp  # Proxy 类实现
idl_test_service_stub.h     # Stub 类头文件
idl_test_service_stub.cpp   # Stub 类实现
```

### 3.3 生成 TypeScript 代码

```bash
idl -gen-ts -d output_dir -c IIdlTestService.idl --intf-type sa
```

### 3.4 常用命令选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `-c <file>` | 编译 IDL 文件 | `-c IIdlTestService.idl` |
| `-d <dir>` | 输出目录 | `-d output_dir` |
| `--gen-cpp` | 生成 C++ 代码 | `--gen-cpp` |
| `--gen-ts` | 生成 TypeScript 代码 | `--gen-ts` |
| `--gen-rust` | 生成 Rust 代码 | `--gen-rust` |
| `--intf-type` | 接口类型 (sa/hdi) | `--intf-type sa` |
| `--dump-ast` | 显示 AST | `--dump-ast` |
| `-h` | 显示帮助 | `-h` |

---

## 4. 运行环境

### 4.1 编译时工具

IDL 工具是**编译时工具**，不运行在设备上：
- 运行在编译主机（Linux/macOS）
- 输入：开发者编写的 .idl 文件
- 输出：生成的源代码文件

### 4.2 依赖关系

**编译依赖**（`bundle.json:22-31`）：
```json
"deps": {
  "components": [
    "hilog",      // 日志
    "ipc",        // IPC 框架
    "samgr",      // 系统能力管理器
    "safwk",      // 系统服务框架
    "c_utils",    // C 工具库
    "bounds_checking_function"  // 边界检查
  ]
}
```

**运行时依赖**（生成的代码）：
- OpenHarmony IPC 框架
- 系统服务管理器（SAMGR）

---

## 5. 项目相关仓

- **本仓库**: [ability_idl_tool](https://gitee.com/openharmony/ability_idl_tool)
- **元能力子系统相关**:
  - [ability_base](https://gitee.com/openharmony/ability_ability_base)
  - [ability_runtime](https://gitee.com/openharmony/ability_ability_runtime)
  - [dmsfwk](https://gitee.com/openharmony/ability_dmsfwk)
  - [form_fwk](https://gitee.com/openharmony/ability_form_fwk)

---

## 6. 延伸阅读

- [02_Architecture.md](02_Architecture.md) - 架构与数据流详解
- [04_Interface.md](04_Interface.md) - 完整命令行接口说明
- [IDL 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/IDL/)

---

**文档证据**: `README.md:1-392`, `bundle.json:1-54`
