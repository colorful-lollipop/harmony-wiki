# 项目概览

## 目的

本文档提供 OpenHarmony Pasteboard（剪贴板）组件的整体概览，包括项目定位、核心能力、运行环境和关键概念。

## 适用范围

- 新加入 Pasteboard 团队的开发者
- 需要集成剪贴板功能的应用开发者
- 进行代码评审或安全审计的人员

## 项目定位

Pasteboard 是 OpenHarmony **分布式数据管理子系统**的核心组件，提供系统级剪贴板服务，支持：

- **本地剪贴板**: 设备内的复制/粘贴功能
- **分布式剪贴板**: 跨设备的剪贴板数据同步
- **数据类型支持**: 文本、HTML、URI、PixelMap、Want、自定义 MIME 类型
- **安全控制**: DLP（数据防泄漏）、权限管控、分享范围控制

### 子系统位置

```
distributeddatamgr/         # 分布式数据管理子系统
├── pasteboard/            # 剪贴板服务 (本文档)
├── udmf/                  # 统一数据管理框架
└── data_share/            # 数据共享
```

## 核心能力

### 1. 数据管理

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 数据创建 | 支持多种 MIME 类型创建 PasteData | `framework/innerkits/src/paste_data.cpp` |
| 数据存储 | 进程内数据缓存 + 跨进程服务存储 | `services/core/src/pasteboard_service.cpp` |
| 数据序列化 | TLV 格式序列化，支持 Ashmem 大数据传输 | `framework/tlv/` |
| 数据加密 | 分布式传输加密 | `adapter/security_level/` |

### 2. 接口支持

| 接口类型 | 说明 | 入口文件 |
|----------|------|----------|
| **N-API** | JavaScript/TypeScript 接口，`@ohos.pasteboard` | `interfaces/kits/napi/src/napi_init.cpp:55` |
| **NDK** | C/C++ 原生接口，`oh_pasteboard.h` | `interfaces/ndk/include/oh_pasteboard.h` |
| **CJ FFI** | Cangjie 语言接口 | `interfaces/cj/src/pasteboard_ffi.cpp` |
| **ANI** | ArkTS Native Interface | `interfaces/ani/src/pasteboard_ani.cpp` |
| **Taihe** | 现代 TypeScript 绑定 | `interfaces/taihe/src/ohos.pasteboard.pasteboard.impl.cpp` |
| **InnerKit** | 系统内部 C++ 接口 | `framework/innerkits/include/pasteboard_client.h` |

### 3. 安全特性

| 特性 | 说明 | 代码位置 |
|------|------|----------|
| 权限控制 | `ohos.permission.READ_PASTEBOARD` | `framework/framework/permission/permission_utils.cpp:23` |
| DLP 支持 | 敏感数据防泄漏 | `services/core/src/pasteboard_service.cpp:28` |
| 分享控制 | ShareOption: InApp/LocalDevice/CrossDevice | `interfaces/kits/napi/src/napi_pasteboard.cpp:284` |
| URI 授权 | URI 权限委托校验 | `services/core/src/pasteboard_service.cpp:2134` |
| 数据分类 | 数据安全等级标记 | `pasteboard.gni:38` |

### 4. 分布式能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 设备发现 | DeviceManager 集成 | `framework/framework/device/dm_adapter.cpp` |
| 设备画像 | DeviceProfile 同步 | `adapter/src/device_profile_adapter.cpp` |
| P2P 传输 | 点对点数据传输 | `services/core/src/pasteboard_service.cpp` |
| 延迟加载 | 跨设备延迟数据获取 | `services/core/src/pasteboard_delay_manager.cpp` |

## 运行环境

### 系统要求

- **OS**: OpenHarmony 3.0+
- **系统类型**: Standard（标准系统）
- **进程**: 独立 SA 进程，SA ID = **3701**

### 资源占用

根据 `bundle.json`:
- **ROM**: 300KB
- **RAM**: 1024KB

### 关键依赖

```json
// 来自 bundle.json
deps: [
  "ability_runtime",    // Ability 框架
  "access_token",       // 权限管理
  "device_manager",     // 设备管理
  "dlp_permission_service", // DLP 服务
  "ipc",               // IPC 通信
  "samgr",             // SA 管理
  "udmf",              // 统一数据管理
  ...
]
```

## 关键概念

### 1. PasteData

剪贴板数据对象，包含多条记录（PasteDataRecord）。

```cpp
// framework/innerkits/include/paste_data.h
class PasteData {
    std::vector<std::shared_ptr<PasteDataRecord>> records_;
    PasteDataProperty property_;
};
```

### 2. PasteDataRecord

单条数据记录，包含特定 MIME 类型的数据。

```cpp
// framework/innerkits/include/paste_data_record.h
class PasteDataRecord {
    std::string mimeType_;
    std::shared_ptr<EntryValue> entryValue_;
};
```

### 3. ShareOption

数据分享范围控制：

- **InApp**: 仅同应用内粘贴
- **LocalDevice**: 仅本设备粘贴
- **CrossDevice**: 允许跨设备粘贴

```cpp
// interfaces/kits/napi/src/napi_pasteboard.cpp:290
enum class ShareOption {
    InApp = 0,
    LocalDevice = 1,
    CrossDevice = 2
};
```

### 4. 延迟加载 (Delay Getter)

支持在数据粘贴时才获取实际内容，用于大文件或跨设备场景。

```cpp
// framework/innerkits/include/pasteboard_delay_getter.h
class PasteboardDelayGetter {
    virtual std::shared_ptr<PasteData> GetDelayPasteData() = 0;
};
```

### 5. SystemAbility (SA)

剪贴板服务作为系统能力运行：

```cpp
// services/core/src/pasteboard_service.cpp:133
class PasteboardService : public SystemAbility, public PasteboardServiceStub {
    static constexpr int32_t PASTEBOARD_SERVICE_ID = 3701;
};
```

## 架构总览

```
┌─────────────────────────────────────────────────────────────┐
│                      Application Layer                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │    JS    │ │    C     │ │   CJ     │ │  ArkTS   │       │
│  │  (@ohos) │ │  (NDK)   │ │  (FFI)   │ │  (ANI)   │       │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘       │
└───────┼────────────┼────────────┼────────────┼─────────────┘
        │            │            │            │
        ▼            ▼            ▼            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Interface Layer                         │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │    N-API    │ │     NDK     │ │  CJ/ANI/    │           │
│  │   (kits)    │ │  (interfaces/ndk)│  Taihe    │           │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘           │
└─────────┼───────────────┼───────────────┼───────────────────┘
          │               │               │
          ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│                      Framework Layer                         │
│  ┌─────────────────────────────────────────┐                │
│  │         PasteboardClient (InnerKit)      │                │
│  │    framework/innerkits/src/...           │                │
│  └──────────────────┬──────────────────────┘                │
│                     │                                        │
│  ┌──────────────────┴──────────────────────┐                │
│  │      PasteboardServiceLoader            │                │
│  │    (Service proxy management)           │                │
│  └──────────────────┬──────────────────────┘                │
└─────────────────────┼───────────────────────────────────────┘
                      │ IPC
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                      Service Layer                           │
│  ┌─────────────────────────────────────────┐                │
│  │       PasteboardService (SA ID: 3701)    │                │
│  │    services/core/src/pasteboard_...      │                │
│  │                                         │                │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐   │                │
│  │  │  Core   │ │  DFX    │ │ Account │   │                │
│  │  │ Manager │ │ Reporter│ │ Manager │   │                │
│  │  └─────────┘ └─────────┘ └─────────┘   │                │
│  └─────────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

## 关键结论

1. **多语言支持**: Pasteboard 提供 N-API、NDK、CJ FFI、ANI、Taihe 五种接口，覆盖主流开发语言。

2. **安全优先**: 内置 DLP、权限控制、ShareOption、数据分类等多重安全机制。

3. **分布式架构**: 支持跨设备剪贴板，集成 DeviceManager 和 DeviceProfile。

4. **高效传输**: 使用 TLV 序列化和 Ashmem 共享内存，支持大文件传输。

5. **模块化设计**: 清晰的层次结构（Interface → Framework → Service），便于维护和扩展。

## 相关链接

- [架构详情 → 01_Architecture.md](01_Architecture.md)
- [N-API 详情 → 03_NAPI_Reference.md](03_NAPI_Reference.md)
- [安全评审 → 06_Security.md](06_Security.md)
