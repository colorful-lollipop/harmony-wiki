# OpenHarmony SDK C 项目概览

> **目的**: 快速理解本项目在整个 OpenHarmony 系统中的定位、核心能力与边界  
> **适用范围**: 初次接触本项目的新人、架构设计者、技术决策者  
> **生成时间**: 2025-02-06

---

## 1. 项目定位

### 1.1 是什么

**OpenHarmony SDK C**（仓库名：`interface_sdk_c`）是 **OpenHarmony 操作系统提供给应用使用的 C/C++ 语言生态库的系统 C 接口声明仓库**。

它是**系统与应用之间的 C 能力契约**，所有接口需要保持足够的前向兼容性。

### 1.2 核心职责

| 职责 | 说明 |
|------|------|
| **API 声明** | 提供 C/C++ 头文件（.h）声明系统能力 |
| **符号定义** | 通过 .ndk.json 定义导出的 API 符号列表 |
| **构建配置** | 提供 GN 构建脚本（BUILD.gn）定义编译目标 |
| **版本管理** | 通过 first_introduced 字段管理 API 版本 |

### 1.3 与相关仓库的关系

```
OpenHarmony 系统架构（简化）
┌────────────────────────────────────────────────────────────────┐
│                        应用层 (Applications)                    │
│     ArkTS 应用 / C++ 原生应用 / 第三方框架 (Unity/Electron)      │
├────────────────────────────────────────────────────────────────┤
│                        框架层 (Framework)                       │
│     ArkUI 框架 ◄─── 本仓库声明的 C API ───► NAPI 运行时          │
├────────────────────────────────────────────────────────────────┤
│                        服务层 (Services)                        │
│     多媒体服务 / 图形服务 / 安全服务 / 网络服务 ...              │
├────────────────────────────────────────────────────────────────┤
│                        内核层 (Kernel)                          │
│     Linux 内核 / 鸿蒙内核 / 驱动框架                            │
└────────────────────────────────────────────────────────────────┘
                            ▲
                            │ 调用
                            ▼
┌────────────────────────────────────────────────────────────────┐
│                   interface_sdk_c (本仓库)                     │
│  C API 头文件 + .ndk.json 符号定义 + BUILD.gn 构建配置          │
└────────────────────────────────────────────────────────────────┘
```

### 1.4 非本仓库职责

| 不属于本仓库的内容 | 实际所在仓库 |
|-------------------|-------------|
| C API 实现代码 | 各子系统实现仓库（如 `arkui_ace_engine`、`multimedia_player_framework` 等） |
| NAPI 绑定实现 | `arkui_napi` 等运行时仓库 |
| 构建工具链 | `build` 仓库（GN 模板定义） |
| 完整 SDK 包 | 通过编译 `//build/ohos/ndk` 生成 |

---

## 2. 项目边界

### 2.1 接口开放策略（基于代码证据）

根据 `docs/user_guide.md`（路径: `docs/user_guide.md`）定义的规则：

**❌ 不开放原则**
1. 应用生态核心特性只开放 ArkTS API，不开放 C API
2. 不开发 ArkUI 的 C API（但提供 N-API 绑定）
3. 不开发硬件底层接口（HDI）C API
4. 不开放命令行接口 C API

**✅ 开放原则**
1. **高性能场景**: 需要高性能的 IO、CPU 密集计算、音视频编解码、图形计算
2. **应用生态依赖**: 对标竞品的高阶媒体 C API
3. **框架依赖**: Unity/electron/CFE 等框架依赖的 C API
4. **行业标准**: 金融/安全行业特定算法（独立发布）

### 2.2 接口稳定性保证

根据 `docs/user_guide.md:72-74`：

- **不允许接口原型变更** → 通过新增接口、废弃老接口实现演进
- **不允许接口语义变更** → 通过 XTS 用例保持兼容性
- **结构体成员、枚举值可废弃但不应删除**

---

## 3. 核心能力

### 3.1 能力域分布

```
OpenHarmony SDK C 能力域（30+ 模块）

┌─────────────────────────────────────────────────────────────┐
│  🎨 UI 框架层                                                │
│  arkui/              - ArkUI 原生引擎、NAPI、窗口管理         │
│  ani/                - Ark Native Interface                 │
├─────────────────────────────────────────────────────────────┤
│  🎬 多媒体                                                   │
│  multimedia/         - 音视频编解码、相机、图像、播放         │
│  ├─ audio_framework  - 音频采集/渲染/管理                    │
│  ├─ av_codec        - 音视频编解码                           │
│  ├─ camera_framework - 相机控制                              │
│  ├─ image_framework - 图像处理（PixelMap/ImageSource）       │
│  └─ player_framework - 播放器/录制器/屏幕录制                │
├─────────────────────────────────────────────────────────────┤
│  🖼️ 图形与渲染                                               │
│  graphic/graphic_2d/ - 2D 图形、OpenGL/GLES、Vulkan          │
│  ├─ native_drawing   - 原生 2D 绘制（50+ API）               │
│  ├─ native_window    - 原生窗口                              │
│  ├─ EGL/GLES2/GLES3  - OpenGL 接口                           │
│  └─ vulkan           - Vulkan 图形 API                       │
├─────────────────────────────────────────────────────────────┤
│  🔒 安全                                                     │
│  security/           - 密钥管理、证书、访问控制               │
│  CryptoArchitectureKit - 加密架构（摘要/签名/加解密）         │
├─────────────────────────────────────────────────────────────┤
│  🌐 网络与连接                                               │
│  network/            - HTTP、WebSocket、网络连接              │
│  ConnectivityKit/    - WiFi、蓝牙                            │
├─────────────────────────────────────────────────────────────┤
│  💾 数据管理                                                 │
│  distributeddatamgr/ - RDB、Preferences、剪贴板、UDMF         │
│  filemanagement/     - 文件 IO、文件分享、云盘               │
├─────────────────────────────────────────────────────────────┤
│  🎮 输入与外设                                               │
│  multimodalinput/    - 多模态输入（键盘/鼠标/触摸）           │
│  GameControllerKit/  - 游戏手柄                              │
│  drivers/            - USB、HID 设备驱动接口                 │
├─────────────────────────────────────────────────────────────┤
│  🔧 基础服务                                                 │
│  hiviewdfx/          - 日志（hilog）、调试（hidebug）         │
│  resourceschedule/   - 并行运行时（ffrt）、QoS               │
│  global/             - 国际化（i18n）、资源管理               │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 API 规模统计

| 类别 | 数量 | 说明 |
|------|------|------|
| 主要模块 | 30+ | 一级目录模块 |
| 公开头文件 | 394+ | 排除 third_party |
| .ndk.json 文件 | 110+ | API 符号定义文件 |
| GN 构建目标 | 287+ | ndk_targets.gni 中定义 |
| N-API 符号 | 306 | libnapi.ndk.json |
| ArkUI API | 900+ | libace.ndk.json |
| 音频 API | 130+ | ohaudio.ndk.json |

---

## 4. 运行环境

### 4.1 目标平台

| 平台 | 架构 | 说明 |
|------|------|------|
| OpenHarmony | arm64 | 主要目标平台 |
| Linux | x86_64, arm64 | SDK 开发环境 |
| macOS | x86_64, arm64 (M1/M2) | SDK 开发环境 |
| Windows | x86_64 | SDK 开发环境 |

### 4.2 编译产物路径

根据 `docs/howto_add.md:24-26`：

```
out/sdk-native/
├── os-specific/        # 平台相关（工具链、cmake）
│   └── darwin|windows|linux/
└── os-irrelevant/      # 平台无关
    └── sysroot/
        └── usr/
            ├── include/    # 头文件（按模块组织）
            └── lib/        # 库文件（.so）

out/packages/ohos-sdk-native/
└── [darwin|windows|linux]/
    └── native-[platform]-x64-[version].zip
```

### 4.3 运行时库路径

根据 `docs/user_guide.md:64`：

- **C API 实现动态库路径**: `/system/{lib|lib64}/ndk/`
- **库命名**: `lib<name>.z.so`（带版本号）或 `lib<name>.so`

---

## 5. 关键概念

### 5.1 NDK（Native Development Kit）

**定义**: OpenHarmony 原生开发工具包，提供 C/C++ 接口访问系统能力。

**组成**: 
- 头文件（`sysroot/usr/include/`）
- 库文件（`sysroot/usr/lib/`）
- 工具链（LLVM/Clang）
- CMake 配置

### 5.2 N-API

**定义**: Node.js N-API 的 OpenHarmony 扩展，用于 C/C++ 代码与 ArkTS 之间的互操作。

**关键文件**: 
- `arkui/napi/native_api.h` - OpenHarmony 扩展 API
- `third_party/node/src/node_api.h` - Node.js 兼容层

**核心机制**:
- `NAPI_MODULE` 宏注册模块
- `napi_property_descriptor` 定义导出属性
- `napi_define_properties` 批量导出到 JS

### 5.3 .ndk.json

**定义**: NDK 接口符号描述文件，声明库导出的 API 符号列表。

**格式**:
```json
[
    {"name": "API_Function_Name"},
    {
        "first_introduced": "12",
        "name": "New_API_Function"
    }
]
```

**作用**:
- 构建时生成符号版本脚本
- 运行时控制符号可见性
- API 版本管理

### 5.4 SysCap（System Capability）

**定义**: 系统能力，用于声明 API 所需的系统功能。

**示例**: `SystemCapability.ArkUI.ArkUI.Napi`

**关联**: 每个 .ndk.json 对应一个 SysCap，头文件与 SysCap 一一对应。

### 5.5 GN 与 BUILD.gn

**定义**: GN（Generate Ninja）是构建系统，BUILD.gn 是构建脚本。

**关键模板**:
- `ohos_ndk_library` - 构建 NDK 共享库
- `ohos_ndk_headers` - 安装头文件

**全局配置**: `ndk_targets.gni` 定义所有 NDK 构建目标。

---

## 6. 代码证据索引

| 结论 | 证据文件 | 关键位置 |
|------|----------|----------|
| 项目定位 | `README.md` | 全文 |
| 接口开放策略 | `docs/user_guide.md` | 第 7-22 行 |
| 兼容性规则 | `docs/user_guide.md` | 第 71-74 行 |
| 构建产物路径 | `docs/howto_add.md` | 第 24-43 行 |
| C API 编码规范 | `docs/capi_naming.md` | 全文 |
| NDK 目标列表 | `ndk_targets.gni` | 第 17-287 行 |
| N-API 定义 | `arkui/napi/libnapi.ndk.json` | 全文（306 个符号）|

---

## 7. 相关跳转

- **下一章**: [目录结构与模块职责](./01_Directory_Structure.md)
- **API 文档**: [N-API 接口文档](./03_NAPI_Reference.md)
- **构建文档**: [GN 构建目标](./05_GN_Build.md)
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**OpenHarmony SDK C 概览文档 - 基于代码生成**
