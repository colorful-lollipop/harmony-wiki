# Kit API 参考

## 概述

本文档列出 Cangjie SDK 所有 Kit 的 API 声明信息，包括 Kit 导入方式、核心模块和关键类型。

## Kit 清单

| Kit | 包名 | 模块数 | 描述 |
|-----|------|--------|------|
| AbilityKit | `kit.AbilityKit` | 18+ | 应用生命周期、Ability、Want、Bundle |
| ArkData | `kit.ArkData` | 10+ | RDB 存储、KV 存储、Preferences |
| ArkGraphics2D | `kit.ArkGraphics2D` | 4+ | 2D 图形绘制 |
| ArkUI | `kit.ArkUI` | 60+ | UI 组件、状态管理、窗口 |
| ArkWeb | `kit.ArkWeb` | 5+ | Web 组件 |
| BasicServicesKit | `kit.BasicServicesKit` | 9+ | 设备信息、设置、事件 |
| CameraKit | `kit.CameraKit` | 3+ | 相机拍照 |
| CangjieKit | `kit.CangjieKit` | 47+ | 标准库、互操作、FFI |
| ConnectivityKit | `kit.ConnectivityKit` | 10+ | 蓝牙、WiFi、NFC |
| CoreFileKit | `kit.CoreFileKit` | 5+ | 文件系统操作 |
| CryptoArchitectureKit | `kit.CryptoArchitectureKit` | 3+ | 加密框架 |
| IPCKit | `kit.IPCKit` | 3+ | IPC、RPC |
| ImageKit | `kit.ImageKit` | 3+ | 图像处理 |
| LocalizationKit | `kit.LocalizationKit` | 5+ | 本地化 |
| LocationKit | `kit.LocationKit` | 4+ | 定位 |
| MediaKit | `kit.MediaKit` | 5+ | 音频、视频 |
| MediaLibraryKit | `kit.MediaLibraryKit` | 5+ | 媒体库 |
| NetworkKit | `kit.NetworkKit` | 10+ | HTTP、Socket |
| PerformanceAnalysisKit | `kit.PerformanceAnalysisKit` | 6+ | 日志、追踪 |
| SensorServiceKit | `kit.SensorServiceKit` | 3+ | 传感器 |
| TelephonyKit | `kit.TelephonyKit` | 4+ | 电话、短信 |
| TestKit | `kit.TestKit` | 3+ | 测试框架 |
| UniversalKeystoreKit | `kit.UniversalKeystoreKit` | 4+ | 密钥库 |

## Kit API 详细清单

### AbilityKit

**导入**: `import kit.AbilityKit`

**核心模块**:
```
ohos.app.ability          # Ability 基础
ohos.app.ability.ui_ability    # UIAbility
ohos.app.ability.ability_stage # AbilityStage
ohos.app.ability.ability_constant # 常量
ohos.app.ability.want          # Want
ohos.app.ability.want_constant # Want 常量
ohos.app.ability.configuration  # 配置
ohos.app.ability.context_constant # 上下文常量
ohos.app.ability.start_options # 启动选项
ohos.bundle.bundle_manager     # Bundle 管理
ohos.ability_access_ctrl       # 权限管理
```

**关键类型**:
```cangjie
public class UIAbility {
    public func onCreate(want: Want): Unit
    public func onWindowStageCreate(windowConfig: Configuration): Unit
    public func onForeground(): Unit
    public func onDestroy(): Unit
}

public class Want {
    public let bundleName: String
    public let abilityName: String
    public let entities: Array<String>
}

public class BundleInfo {
    public let name: String
    public let permissions: Array<String>
    public let signatureInfo: SignatureInfo
}
```

**权限要求**:
- 无需额外权限的基础 Ability 能力
- 敏感操作需要 `ohos.permission.XXX` 权限

> **证据**: `kits/kit.AbilityKit.cj.d`, `api/AbilityKit/ohos.app.ability.ui_ability.cj.d`

### ArkUI

**导入**: `import kit.ArkUI`

**核心模块**:
```
ohos.arkui.component      # UI 组件基类
ohos.arkui.component.text # Text 组件
ohos.arkui.component.button # Button 组件
ohos.arkui.component.column # Column 布局
ohos.arkui.component.row  # Row 布局
ohos.arkui.component.flex  # Flex 布局
ohos.arkui.state_management # 状态管理
ohos.arkui.ui_context     # UI 上下文
ohos.window               # 窗口管理
ohos.display              # 显示管理
```

**关键类型**:
```cangjie
// 状态管理
public class State<T> { ... }
public class Prop<T> { ... }
public class Link<T> { ... }

// UI 组件
public class Component {
    public func build(): Unit
    public func onClick(handler: () => Unit): Component
}

// 窗口
public class Window {
    public static func createWindow(config: Configuration): Window
    public func setContent(content: Component): Unit
}
```

> **证据**: `kits/kit.ArkUI.cj.d`, `api/ArkUI/ohos.arkui.component.cj.d`

### NetworkKit

**导入**: `import kit.NetworkKit`

**核心模块**:
```
ohos.net.http           # HTTP 请求
ohos.net.connection     # 连接管理
ohos.net.socket        # Socket 通信
ohos.net.netAddress    # 网络地址
```

**关键类型**:
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.INTERNET",
    syscap: "SystemCapability.Communication.NetManager.Core"
]
public class HttpRequest {
    public static func create(): HttpRequest
    public func request(url: String): Promise<HttpResponse>
    public func destroy(): Unit
}

public class HttpResponse {
    public let resultType: number
    public let headers: Record<string, string>
    public let result: Uint8Array
}
```

**权限要求**:
- `ohos.permission.INTERNET` - 访问互联网
- `ohos.permission.GET_NETWORK_INFO` - 获取网络信息

> **证据**: `kits/kit.NetworkKit.cj.d`, `api/NetworkKit/ohos.net.http.cj.d`

### IPCKit

**导入**: `import kit.IPCKit`

**核心模块**:
```
ohos.rpc              # IPC/RPC
ohos.ipc              # IPC 基础
```

**关键类型**:
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
]
public class Ashmem {
    public static const PROT_READ: UInt32 = 1
    public static const PROT_WRITE: UInt32 = 2
    public static const PROT_EXEC: UInt32 = 4
    public static const PROT_NONE: UInt32 = 0

    public static func create(name: String, size: Int32): Ashmem
    public func closeAshmem(): Unit
    public func mapReadWriteAshmem(): Unit
}

public class MessageParcel {
    public static func create(): MessageParcel
    public func writeInt32(value: Int32): Bool
    public func readInt32(): Int32
    public func writeRemoteInterfaceToken(token: String): Bool
}
```

**系统能力**:
- `SystemCapability.Communication.IPC.Core`

> **证据**: `kits/kit.IPCKit.cj.d`, `api/IPCKit/ohos.rpc.cj.d`

### UniversalKeystoreKit

**导入**: `import kit.UniversalKeystoreKit`

**权限相关**:
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Security.Keystore"
]
public class KeyStore {
    public static func getInstance(): KeyStore
    public func getKey(alias: String): Key
    public func createKey(spec: KeySpec): Key
}
```

**系统能力**:
- `SystemCapability.Security.Keystore`

> **证据**: `api/UniversalKeystoreKit/ohos.security.keystore.cj.d`

### CoreFileKit

**导入**: `import kit.CoreFileKit`

**核心模块**:
```
ohos.file              # 文件操作
ohos.fileio           # 文件 I/O
ohos.storage           # 存储管理
```

**关键类型**:
```cangjie
public class File {
    public static func open(path: String, flags: OpenFlags): File
    public func read(buffer: Uint8Array): Int64
    public func write(buffer: Uint8Array): Int64
    public func close(): Unit
}

public struct OpenFlags {
    public static const RDONLY: OpenFlags
    public static const WRONLY: OpenFlags
    public static const RDWR: OpenFlags
    public static const CREAT: OpenFlags
}
```

> **证据**: `kits/kit.CoreFileKit.cj.d`

### MediaKit

**导入**: `import kit.MediaKit`

**核心模块**:
```
ohos.multimedia.audio       # 音频
ohos.multimedia.video       # 视频
ohos.multimedia.player      # 播放器
ohos.multimedia.recorder    # 录音器
```

**权限要求**:
- `ohos.permission.MICROPHONE` - 录音
- `ohos.permission.CAMERA` - 视频录制

> **证据**: `kits/kit.MediaKit.cj.d`

### LocationKit

**导入**: `import kit.LocationKit`

**权限要求**:
- `ohos.permission.APPROXIMATELY_LOCATION` - 大致位置
- `ohos.permission.LOCATION` - 精确位置（需要特殊授权）

**关键类型**:
```cangjie
public class GeoLocation {
    public let latitude: Double
    public let longitude: Double
    public let altitude: Double
    public let accuracy: Float
}

public class LocationManager {
    public static func getLocation(): Promise<GeoLocation>
}
```

> **证据**: `api/LocationKit/ohos.geo_location_manager.cj.d`

### CameraKit

**导入**: `import kit.CameraKit`

**权限要求**:
- `ohos.permission.CAMERA` - 访问相机
- `ohos.permission.MICROPHONE` - 录制音频

**关键类型**:
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.CAMERA"
]
public class Camera {
    public static func createCamera(): Camera
    public func takePhoto(): Promise<Image>
    public func startVideoRecording(): Unit
    public func stopVideoRecording(): Unit
}
```

> **证据**: `kits/kit.CameraKit.cj.d`, `api/CameraKit/ohos.multimedia.camera.cj.d`

## API 版本与能力

### API Level 22+

所有在 Cangjie SDK 中声明的 API 均使用 `@!APILevel` 注解，标记为 `since: "22"`。

### 系统能力 (syscap) 映射

| syscap | 用途 | 涉及 Kit |
|--------|------|----------|
| `SystemCapability.Communication.IPC.Core` | IPC 核心能力 | IPCKit |
| `SystemCapability.Communication.NetManager.Core` | 网络管理 | NetworkKit |
| `SystemCapability.Security.AccessToken` | 权限管理 | AbilityKit |
| `SystemCapability.Security.Keystore` | 密钥库 | UniversalKeystoreKit |
| `SystemCapability.Location.LBS` | 定位服务 | LocationKit |
| `SystemCapability.Multimedia.Camera` | 相机 | CameraKit |

## 权限声明模式

### API 级别权限

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.XXX"
]
public func sensitiveApi(): ReturnType
```

### 常见权限列表

| 权限 | 用途 | 错误码 |
|------|------|--------|
| `ohos.permission.CAMERA` | 访问相机 | 201 |
| `ohos.permission.MICROPHONE` | 访问麦克风 | 201 |
| `ohos.permission.INTERNET` | 访问互联网 | - |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | 201 |
| `ohos.permission.ACCESS_LOCATION` | 访问位置 | 201 |
| `ohos.permission.READ_CONTACTS` | 读取联系人 | 201 |
| `ohos.permission.WRITE_CONTACTS` | 写入联系人 | 201 |

> **证据**: `api/AbilityKit/ohos.ability_access_ctrl.cj.d`

## API 使用示例

### 基本导入

```cangjie
// 导入单个 Kit
import kit.NetworkKit

// 使用 API
let http = HttpRequest.create()
let response = await http.request("https://example.com")
```

### 权限请求

```cangjie
import kit.AbilityKit

let atManager = AtManager.create()
let result = await atManager.requestPermissionsFromUser(
    context,
    ["ohos.permission.CAMERA", "ohos.permission.MICROPHONE"]
)
```

### 错误处理

```cangjie
@!APILevel[
    since: "22",
    throwexception: true
]
public func getCamera(): Camera

// 调用
try {
    let camera = getCamera()
} catch (e: BusinessException) {
    // 处理权限错误: e.code == 201
}
```

## 相关文档

- [系统架构](03_Architecture.md)
- [权限管理 API](api/AbilityKit/ohos.ability_access_ctrl.cj.d)
