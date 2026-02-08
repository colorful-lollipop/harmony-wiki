# 目录结构与模块职责

> 代码组织与职责划分（不含测试）

## 目的与适用范围

### 目的
本文档描述 JSVM 项目的目录结构和各模块的职责，帮助新人快速定位代码。

### 适用范围
- 需要理解代码组织的开发者
- 需要进行代码审查的开发者
- 需要定位 bug 的开发者

---

## 顶层目录结构

### 完整目录树

```
arkcompiler/jsvm/
├── interface/              # 接口层
│   └── kits/              # 公共 API（应用开发者使用）
│       ├── jsvm.h          # JSVM API 主头文件
│       └── jsvm_types.h    # JSVM API 类型定义
├── src/                   # 源代码
│   ├── inspector/          # Inspector 调试支持
│   └── platform/           # 平台抽象层
├── test/                  # 测试套（本文档不涉及）
├── BUILD.gn               # GN 构建脚本
├── jsvm.gni               # 源文件定义
├── bundle.json            # 部件配置
└── wiki/                  # 文档目录
```

**证据位置**:
- 项目根目录: `README.md:10-23`
- `ls /Volumes/lexar/code/d/work/oh/arkcompiler/jsvm` - 实际目录结构

---

## 接口层（interface/）

### kits/ - 公共 API

**路径**: `interface/kits/`

**职责**: 提供给应用开发者使用的公共 API 头文件。

**文件列表**:

| 文件 | 行数 | 职责 |
|------|------|------|
| jsvm.h | ~2500+ | JSVM API 函数声明 |
| jsvm_types.h | ~1020 | JSVM API 类型定义（结构体、枚举） |

**证据位置**:
- `bundle.json:44-54` - inner_kits 定义
- `BUILD.gn:97-99` - public_jsvm_config

#### jsvm.h

**职责**: 定义所有 JSVM API 函数。

**关键内容**:
- VM 生命周期管理 API
- Environment 管理 API
- 代码编译与执行 API
- JS/C++ 互操作 API
- 调试与性能分析 API
- 错误处理 API

**证据位置**: `interface/kits/jsvm.h:16-2500+`

#### jsvm_types.h

**职责**: 定义 JSVM API 所需的所有类型。

**关键内容**:
- 不透明类型（JSVM_VM, JSVM_Env, JSVM_Value 等）
- 枚举（JSVM_Status, JSVM_ValueType 等）
- 结构体（JSVM_InitOptions, JSVM_CreateVMOptions 等）
- 回调函数指针类型

**证据位置**: `interface/kits/jsvm_types.h:16-1020`

---

## 源代码层（src/）

### 核心实现文件

#### js_native_api_v8.cpp

**路径**: `src/js_native_api_v8.cpp`

**行数**: 6140 行

**职责**: 实现 JSVM API 到 V8 引擎的桥接。

**关键功能**:
- VM 生命周期管理
- Environment 管理
- 代码编译与执行
- JS/C++ 互操作
- 快照（Snapshot）管理
- Inspector 集成

**证据位置**:
- `jsvm.gni:17-20` - 源文件列表
- `grep OH_JSVM_ src/js_native_api_v8.cpp | wc -l` - 约 200+ API 函数实现

#### jsvm_env.cpp

**路径**: `src/jsvm_env.cpp`

**行数**: 137 行

**职责**: Environment 相关的辅助实现。

**证据位置**: `jsvm.gni:18`

#### jsvm_reference.cpp

**路径**: `src/jsvm_reference.cpp`

**行数**: 195 行

**职责**: 引用计数管理。

**证据位置**: `jsvm.gni:19`

---

## Inspector 模块（src/inspector/）

### 模块职责

提供 JS 调试支持，包括：
- WebSocket 服务器（Inspector 协议）
- Chrome DevTools 协议实现
- V8 Inspector 集成

### 文件列表

| 文件 | 行数 | 职责 |
|------|------|------|
| inspector_socket.cpp | ~700 | WebSocket 连接管理 |
| inspector_socket_server.cpp | ~550 | WebSocket 服务器实现 |
| inspector_utils.cpp | ~300 | 工具函数 |
| js_native_api_v8_inspector.cpp | ~1100 | JSVM Inspector API 实现 |
| inspector_socket.h | ~60 | WebSocket 连接头文件 |
| inspector_socket_server.h | ~130 | WebSocket 服务器头文件 |
| inspector_utils.h | ~250 | 工具函数头文件 |
| js_native_api_v8_inspector.h | ~120 | Inspector API 头文件 |
| jsvm_host_port.h | ~60 | 主机端口配置 |
| jsvm_mutex.h | ~280 | 互斥锁封装 |
| v8_inspector_protocol_json.h | ~2300 | Inspector 协议 JSON 定义 |

**证据位置**:
- `jsvm.gni:22-27` - inspector 源文件列表
- `ls -la src/inspector/` - 实际文件列表

### 关键类与函数

#### InspectorSocket

**路径**: `src/inspector/inspector_socket.cpp`

**职责**: 管理 WebSocket 连接。

**证据位置**: `src/inspector/inspector_socket.cpp` - InspectorSocket 类定义

#### InspectorSocketServer

**路径**: `src/inspector/inspector_socket_server.cpp`

**职责**: 监听 WebSocket 连接，管理多个客户端连接。

**证据位置**: `src/inspector/inspector_socket_server.cpp` - InspectorSocketServer 类定义

#### JSVM Inspector API

**路径**: `src/inspector/js_native_api_v8_inspector.cpp`

**职责**: 实现 JSVM Inspector API，与 V8 Inspector 集成。

**关键 API**:
- `OH_JSVM_OpenInspector()` - 打开 Inspector
- `OH_JSVM_CloseInspector()` - 关闭 Inspector
- `OH_JSVM_WaitForDebugger()` - 等待调试器连接

**证据位置**:
- `interface/kits/jsvm.h:1719,1736,1747` - API 声明
- `src/inspector/js_native_api_v8_inspector.cpp` - API 实现

---

## Platform 模块（src/platform/）

### 模块职责

提供平台抽象层，封装操作系统特定的功能，包括：
- 线程管理
- 定时器
- 文件 I/O
- 网络 I/O

### 文件列表

| 文件 | 行数 | 职责 |
|------|------|------|
| platform.cpp | ~50 | 平台抽象接口 |
| platform.h | ~50 | 平台抽象头文件 |
| platform_ohos.cpp | ~300 | OpenHarmony 平台实现 |
| platform_ohos.h | ~40 | OpenHarmony 平台头文件 |

**证据位置**:
- `ls -la src/platform/` - 实际文件列表

### 平台抽象设计

Platform 层采用抽象工厂模式，支持多平台：

```cpp
// platform.h - 抽象接口
class Platform {
public:
    virtual void RunTask(std::function<void()> task) = 0;
    virtual void* CreateTimer(uint32_t delay, std::function<void()> callback) = 0;
    virtual void DeleteTimer(void* timer) = 0;
    // ... 其他平台相关接口
};

// platform_ohos.cpp - OpenHarmony 实现
class OhosPlatform : public Platform {
    // 实现 OpenHarmony 特定的线程、定时器等
};
```

**证据位置**:
- `src/platform/platform.h` - Platform 抽象接口
- `src/platform/platform_ohos.cpp` - OpenHarmony 实现

### 平台相关功能

#### 线程管理

**职责**: 提供跨平台的线程池和任务队列。

**证据位置**: `src/platform/platform_ohos.cpp` - 线程相关实现

#### 定时器

**职责**: 提供 setTimeout/setInterval 类似的定时器功能。

**证据位置**: `src/platform/platform_ohos.cpp` - 定时器相关实现

#### 文件 I/O

**职责**: 提供跨平台的文件读写接口。

**证据位置**: `src/platform/platform_ohos.cpp` - 文件 I/O 相关实现

#### 网络 I/O

**职责**: 提供跨平台的网络通信接口。

**证据位置**:
- `BUILD.gn:45-60` - llhttp 集成
- `src/platform/platform_ohos.cpp` - 网络 I/O 相关实现

---

## 构建配置文件

### BUILD.gn

**路径**: `BUILD.gn`

**行数**: 186 行

**职责**: 定义 GN 构建目标和依赖关系。

**关键 Targets**:
1. `jit_enable_list_appid` - JIT 配置文件
2. `copy_v8` - 复制 V8 库
3. `copy_llhttp` - 复制 llhttp 源码
4. `libv8` - V8 预编译库
5. `llhttp` - llhttp 静态库
6. `build_libjsvm` - 构建主库
7. `libjsvm` - JSVM 共享库（主产物）
8. `jsvm_packages` - 顶层打包目标

**证据位置**: `BUILD.gn:18-185`

### jsvm.gni

**路径**: `jsvm.gni`

**行数**: 36 行

**职责**: 定义源文件列表和编译参数。

**关键内容**:
- jsvm_sources - 核心源文件列表
- jsvm_inspector_sources - Inspector 源文件列表
- declare_args() - 编译参数声明

**证据位置**: `jsvm.gni:14-36`

### bundle.json

**路径**: `bundle.json`

**行数**: 62 行

**职责**: 定义部件配置，包括依赖、系统能力、资源占用等。

**关键信息**:
- 部件名称: @ohos/jsvm
- 子系统: arkcompiler
- 系统能力: SystemCapability.ArkCompiler.JSVM
- 内部 Kit: libjsvm + 头文件
- 外部依赖: 12 个组件
- ROM/RAM 占用

**证据位置**: `bundle.json:1-62`

---

## 编译脚本

### build_jsvm.sh

**路径**: `build_jsvm.sh`

**职责**: 使用 CMake 构建 libjsvm.so。

**关键步骤**:
1. 配置 CMake 参数
2. 编译源代码
3. 链接依赖库
4. 生成 libjsvm.so

**证据位置**: `BUILD.gn:142-174` - build_libjsvm action

### build_jsvm_inter.sh

**路径**: `build_jsvm_inter.sh`

**职责**: 交互式构建脚本。

**证据位置**: `build_jsvm_inter.sh` - 脚本内容

### copy_v8.sh

**路径**: `copy_v8.sh`

**职责**: 从预编译目录复制 V8 库到构建目录。

**证据位置**: `BUILD.gn:28-43` - copy_v8 action

### copy_llhttp.sh

**路径**: `copy_llhttp.sh`

**职责**: 复制 llhttp 源码到构建目录。

**证据位置**: `BUILD.gn:45-60` - copy_llhttp action

---

## 配置文件

### jit_enable_list_appid.conf

**路径**: `jit_enable_list_appid.conf`

**职责**: 配置启用 JIT 的应用 ID 列表。

**安装路径**: `/system/etc/jsvm/`

**证据位置**: `BUILD.gn:18-23` - jit_enable_list_appid target

### CMakeLists.txt

**路径**: `CMakeLists.txt`

**职责**: CMake 构建配置。

**证据位置**: `CMakeLists.txt` - CMake 配置内容

---

## 资源占用

根据 `bundle.json`，JSVM 的资源占用：

- **ROM**: 5120 KB
- **RAM**: 10240 KB

**证据位置**: `bundle.json:22-23`

---

## 相关链接

- [概览](./01_Overview.md) - 了解项目定位
- [架构说明](./04_Architecture.md) - 理解系统架构
- [GN Targets 与编译产物](./07_GN_Targets.md) - 构建系统详解
