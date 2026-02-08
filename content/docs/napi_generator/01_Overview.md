# 项目概览

> napi-generator 定位、边界、核心能力与运行环境

## 项目定位

**napi-generator** 是 OpenHarmony 生态的 N-API 代码生成工具集，为 Native 应用开发者提供自动化代码生成能力。

### 核心定位

| 维度 | 定位 |
|------|------|
| **产品类型** | 代码生成工具集 (非运行时库) |
| **目标用户** | OpenHarmony Native 应用开发者、系统框架开发者 |
| **核心价值** | 减少 N-API 框架代码编写，提升开发效率 |
| **技术栈** | Node.js/TypeScript (工具) + C++ (生成产物) |

### 与同类工具的差异

| 工具 | 定位 | 差异点 |
|------|------|--------|
| **napi-generator** | 代码生成工具集 | 面向 OpenHarmony 生态，完整框架生成 |
| **node-addon-api** | N-API 封装库 | 提供 C++ 封装层，不生成代码 |
| **ffi-napi** | FFI 调用 | 运行时绑定，非代码生成 |

## 项目边界

### 包含范围 ✅

```
src/cli/                          # 6 个代码生成工具
├── dts2cpp/                      # TypeScript → N-API (核心)
├── h2sa/                         # SA 服务框架生成
├── h2dtscpp/                     # C++ → TS + NAPI + 测试
├── h2dts/                        # C++ → TypeScript
├── cmake2gn/                     # CMake → GN
└── h2hdf/                        # HDF 驱动生成

examples/                         # 完整示例项目
docs/                             # 工具使用文档
```

### 不包含范围 ❌

```
❌ 运行时 N-API 实现 (属于 arkui_napi 仓库)
❌ 系统能力实现 (属于 safwk/samgr 仓库)
❌ 硬件驱动实现 (属于 hdf 仓库)
❌ 应用业务逻辑 (由开发者实现)
```

## 核心能力详解

### 1. dts2cpp - TypeScript 到 N-API

**功能**: 将 TypeScript 声明文件 (.d.ts) 转换为完整的 N-API C++ 框架代码

**证据**: `src/cli/dts2cpp/src/gen/cmd_gen.js`

```
输入: @ohos.mylib.d.ts
       │
       ▼ [dts2cpp]
       │
输出: mylib_middle.h/cpp    # N-API 中间层
     mylib.h/cpp            # 业务框架
     tool_utility.h/cpp     # N-API 工具类
     BUILD.gn              # 构建脚本
```

**支持的 TypeScript 特性**:
- 基本类型: `string`, `number`, `boolean`, `any`
- 复合类型: `Array<T>`, `Map<K,V>`, 联合类型
- 函数类型: 同步、异步(Promise)、回调、on/off 事件
- 接口与类: 支持继承、属性、方法
- 枚举: 数字和字符串枚举

### 2. h2sa - Service Ability 框架生成

**功能**: 根据 .h 头文件生成完整的 System Ability 框架代码

**证据**: `src/cli/h2sa/src/gen/generate.js`

```
输入: IMyService.h (定义远程方法)
       │
       ▼ [h2sa]
       │
输出: IMyService_proxy.cpp/h    # 客户端代理
     IMyService_stub.cpp/h      # 服务端存根
     MyService.cpp/h            # 服务实现
     sa_profile/                # SA 配置
     BUILD.gn                   # 构建配置
```

**生成的 IPC 框架**:
- `IRemoteProxy` - 客户端调用代理
- `IRemoteStub` - 服务端接收处理
- `MessageParcel` - 数据序列化/反序列化
- `REGISTER_SYSTEM_ABILITY_BY_ID` - 服务注册

### 3. h2dtscpp - 完整开发链

**功能**: 从 C++ 头文件生成 TypeScript 声明 + N-API 实现 + 测试用例

**证据**: `src/cli/h2dtscpp/src/src/tsGen/tsMain.js`

```
输入: mylib.h (C++ 接口)
       │
       ▼ [h2dtscpp]
       │
输出: tsout/index.d.ts         # TypeScript 声明
     cppout/*.cpp               # N-API 实现
     testout/*.Ability.test.ets # 自动化测试
```

### 4. 其他工具

| 工具 | 功能 | 证据 |
|------|------|------|
| h2dts | C++ 头文件 → TypeScript 声明 | `src/cli/h2dts/src/` |
| cmake2gn | CMakeLists.txt → BUILD.gn | `src/cli/cmake2gn/src/` |
| h2hdf | HDF 驱动代码生成 | `src/cli/h2hdf/` |
| scan | 三方库 API 依赖扫描 | `src/tool/api/src/` |

## 运行环境

### 工具运行环境

| 工具 | 操作系统 | 依赖版本 |
|------|----------|----------|
| dts2cpp | Linux/Windows/macOS | Node.js 14+, npm |
| h2sa | Linux/Windows | VS Code 1.62.0+ |
| h2dtscpp | Windows 10 (推荐) | Node.js, header_parser.exe |
| cmake2gn | Linux | CMake, Python |
| h2dts | Windows 10 | VS Code 插件 |
| scan | Linux | Node.js, xlsx |

### 生成产物的运行环境

**目标平台**: OpenHarmony 标准系统

**最小系统版本**:
- N-API: OpenHarmony 4.0+
- SA Framework: OpenHarmony 3.2+

### 依赖关系

```
napi-generator (工具)
    │
    ├── Node.js / npm          # 工具运行
    ├── TypeScript             # .d.ts 解析
    ├── C++ 编译器             # 生成产物编译
    └── OpenHarmony SDK       # N-API 头文件
            │
            └── arkui_napi     # N-API 运行时
```

## 关键概念

### N-API (Node-API)

N-API 是 Node.js/OpenHarmony 提供的 C/C++ 原生模块接口，用于实现 JS 与 Native 代码的互操作。

```cpp
// 标准 N-API 模块注册
static napi_module myModule = {
    .nm_version = 1,
    .nm_register_func = init,
    .nm_modname = "mylib",
};

napi_module_register(&myModule);
```

### Service Ability (SA)

SA 是 OpenHarmony 的系统能力机制，支持跨进程通信。

```cpp
// SA 服务注册
REGISTER_SYSTEM_ABILITY_BY_ID(MyService, MY_SERVICE_ID, true);
```

### AKI (Alpha Kernel Intertering)

AKI 是简化 N-API 开发的 C++ 封装框架。

```cpp
// AKI 绑定示例
JSBIND_GLOBAL() {
    JSBIND_FUNCTION(myFunction);
}
```

## 版本与发布

| 组件 | 当前版本 | 发布状态 |
|------|----------|----------|
| dts2cpp | V1.4.1 | ✅ 活跃开发 |
| h2sa | V1.0.0 | ✅ 活跃开发 |
| h2dtscpp | V1.0.0 | ✅ 活跃开发 |
| h2dts | V1.0.0 | ✅ 活跃开发 |
| cmake2gn | V1.0.0 | ✅ 活跃开发 |

---

## 相关章节

- 目录结构: [02_Directory_Structure.md](02_Directory_Structure.md)
- 架构设计: [03_Architecture.md](03_Architecture.md)
- API 参考: [04_NAPI_Reference.md](04_NAPI_Reference.md)

---

[返回 SUMMARY.md](SUMMARY.md)
