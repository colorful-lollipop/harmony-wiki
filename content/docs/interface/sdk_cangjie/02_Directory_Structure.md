# 目录结构

## 整体目录树

```
interface/sdk_cangjie/
├── api/                                    # [Kit API 声明目录] 25 Kit, 221+ 模块
│   ├── AbilityKit/                         # 应用能力
│   ├── ArkData/                            # 数据存储
│   ├── ArkGraphics2D/                      # 2D 图形
│   ├── ArkUI/                              # UI 框架
│   │   └── component/                      # UI 组件子目录
│   ├── ArkWeb/                             # Web 组件
│   ├── BasicServicesKit/                   # 基础服务
│   ├── CameraKit/                          # 相机
│   ├── Cangjie/                            # Cangjie 标准库
│   │   └── third_party/std/                # 标准库模块
│   ├── ConnectivityKit/                    # 连接（蓝牙/WiFi/NFC）
│   ├── CoreFileKit/                        # 文件系统
│   ├── CryptoArchitectureKit/              # 加密架构
│   ├── IPCKit/                             # 进程间通信
│   ├── ImageKit/                           # 图像处理
│   ├── LocalizationKit/                    # 本地化
│   ├── LocationKit/                        # 定位
│   ├── MediaKit/                           # 媒体
│   ├── MediaLibraryKit/                    # 媒体库
│   ├── NetworkKit/                         # 网络
│   ├── PerformanceAnalysisKit/             # 性能分析
│   ├── SensorServiceKit/                   # 传感器
│   ├── TelephonyKit/                       # 电话
│   ├── TestKit/                            # 测试
│   └── UniversalKeystoreKit/               # 密钥库
│
├── kits/                                   # [Kit 统一外部声明] 23 Kit 声明
│   ├── kit.AbilityKit.cj.d
│   ├── kit.ArkData.cj.d
│   ├── kit.ArkGraphics2D.cj.d
│   ├── kit.ArkUI.cj.d
│   ├── kit.ArkWeb.cj.d
│   ├── kit.BasicServicesKit.cj.d
│   ├── kit.CameraKit.cj.d
│   ├── kit.CangjieKit.cj.d
│   ├── kit.ConnectivityKit.cj.d
│   ├── kit.CoreFileKit.cj.d
│   ├── kit.CryptoArchitectureKit.cj.d
│   ├── kit.IPCKit.cj.d
│   ├── kit.ImageKit.cj.d
│   ├── kit.LocalizationKit.cj.d
│   ├── kit.LocationKit.cj.d
│   ├── kit.MediaKit.cj.d
│   ├── kit.MediaLibraryKit.cj.d
│   ├── kit.NetworkKit.cj.d
│   ├── kit.PerformanceAnalysisKit.cj.d
│   ├── kit.SensorServiceKit.cj.d
│   ├── kit.TelephonyKit.cj.d
│   ├── kit.TestKit.cj.d
│   └── kit.UniversalKeystoreKit.cj.d
│
├── build-tools/                            # [SDK 构建工具]
│   ├── lib/
│   │   └── mocks/                         # Mock 库生成工具
│   │       ├── BUILD.gn
│   │       ├── sdk_mock.gni
│   │       ├── mock_stub.cpp
│   │       ├── mock_stub.h
│   │       ├── generate_mock.py
│   │       └── mock/                       # ohos.mock 模块
│   └── script/                            # 构建脚本
│       ├── copy_cangjie_headers.py
│       ├── copy_and_prue.py
│       └── process_libs.py
│
├── docs/                                  # 文档
│   ├── cangjie_sdk_build_guide.md
│   └── cangjie_cjo_serialization_and_deserialization_guide.md
│
├── figures/                               # README 图片
│   ├── interface_sdk_cangjie_architecture.png
│   └── interface_sdk_cangjie_delivery_view.png
│
├── wiki/                                  # 本 Wiki 文档
│   ├── README.md
│   ├── SUMMARY.md
│   ├── 01_Overview.md
│   ├── 02_Directory_Structure.md
│   ├── 03_Architecture.md
│   ├── 04_Kit_API.md
│   ├── 05_Inner_API.md
│   ├── 06_GN_Build.md
│   ├── 07_Build_Artifacts.md
│   ├── 08_Security_Review.md
│   ├── 09_FAQ.md
│   └── _work/                             # 工作区
│       ├── NOTES.md
│       └── PLAN.md
│
├── BUILD.gn                               # [根构建入口] GN 构建配置
├── bundle.json                            # [组件配置] 组件元数据
├── sdk_cangjie.gni                        # [构建模板] SDK 构建模板定义
├── README.md                               # 项目说明
└── LICENSE                                 # 许可证
```

## 目录职责说明

### api/ - Kit API 声明目录

**职责**: 存储各 Kit 的完整 API 声明（类型定义、函数、类、枚举等）

**特点**:
- 按 Kit 组织，每个 Kit 一个子目录
- 包含 `.cj.d` 格式的接口声明文件
- 声明使用 `@!APILevel` 注解标注版本和能力要求

| 子目录 | 描述 | 模块数 |
|--------|------|--------|
| `AbilityKit/` | 应用生命周期、Ability、Want、Bundle 等 | 18+ |
| `ArkData/` | RDB、KV Store、Preferences 等 | 10+ |
| `ArkUI/` | 组件、状态管理、窗口等 | 60+ |
| `BasicServicesKit/` | 设备信息、设置、事件等 | 9+ |
| `Cangjie/` | 标准库、互操作、FFI 等 | 47+ |
| `NetworkKit/` | HTTP、Socket、连接管理 | 10+ |
| `IPCKit/` | IPC、RPC、共享内存 | 3+ |
| ... | 其他 Kit | - |

> **证据**: `api/` 目录 glob 结果显示 100+ `.cj.d` 文件

### kits/ - Kit 统一外部声明目录

**职责**: 提供 Kit 级别的统一导入声明

**特点**:
- 每个 Kit 一个 `kit.{KitName}.cj.d` 文件
- 仅包含 `package` 声明和 `public import` 语句
- 作为 Kit 的对外暴露点

**示例**:
```cangjie
package kit.NetworkKit
public import ohos.net.*
public import ohos.net.http.*
public import ohos.net.connection.*
```

> **证据**: `kits/kit.AbilityKit.cj.d` 文件结构

### build-tools/ - SDK 构建工具目录

**职责**: 提供 SDK 构建所需的工具和脚本

| 子目录/文件 | 职责 |
|-------------|------|
| `lib/mocks/` | 生成 API mock 库，用于 SDK 构建 |
| `script/` | 包含头文件复制、库处理等构建脚本 |

### wiki/ - 工程文档目录

**职责**: 本 Wiki 文档，详见 [README](README.md)

## 模块职责划分（按功能）

### 应用开发层

| Kit | 职责 | 核心模块 |
|-----|------|----------|
| AbilityKit | 应用能力管理 | UIAbility, Context, Want |
| ArkUI | UI 开发 | Component, State Management |
| ArkWeb | Web 渲染 | Webview |
| MediaKit | 媒体播放 | Audio, Video |
| CameraKit | 相机拍照 | Camera, Session |

### 数据存储层

| Kit | 职责 | 核心模块 |
|-----|------|----------|
| ArkData | 结构化数据 | RDB, KV Store |
| CoreFileKit | 文件操作 | File, Directory |

### 系统服务层

| Kit | 职责 | 核心模块 |
|-----|------|----------|
| NetworkKit | 网络通信 | Http, Socket |
| IPCKit | 进程通信 | MessageParcel, RemoteObject |
| ConnectivityKit | 连接管理 | Bluetooth, WiFi, NFC |

### 安全层

| Kit | 职责 | 核心模块 |
|-----|------|----------|
| UniversalKeystoreKit | 密钥管理 | KeyStore, KeyPair |
| CryptoArchitectureKit | 加密框架 | Cipher, Digest |

### 工具层

| Kit | 职责 | 核心模块 |
|-----|------|----------|
| TestKit | 测试框架 | TestRunner |
| PerformanceAnalysisKit | 性能分析 | HiTrace, HiLog |

## 模块依赖关系（简化）

```
kit.AbilityKit
    ├── kit.CoreFileKit（应用文件访问）
    ├── kit.NetworkKit（网络请求）
    └── kit.UniversalKeystoreKit（数据加密）

kit.ArkUI
    └── kit.AbilityKit（上下文获取）

kit.MediaKit
    ├── kit.CameraKit（相机采集）
    └── kit.CoreFileKit（媒体文件存储）

kit.NetworkKit
    └── kit.IPCKit（底层通信）
```

> **注意**: 实际依赖关系以 `.cj.d` 文件中的 `import` 语句为准

## 忽略的目录

以下目录不包含在 Wiki 分析范围内：

| 目录模式 | 说明 |
|---------|------|
| `test/` | 测试目录 |
| `tests/` | 测试目录 |
| `unittest/` | 单元测试 |
| `*_test.*` | 测试文件 |
| `*_fuzzer.*` | 模糊测试 |
