# 附录：配置标志

> 本文档整理 multimedia_cangjie_wrapper 的关键宏、配置开关与 Feature Flags

---

## 目的与适用范围

**目的**：汇总项目中的配置标志，帮助理解条件编译和功能开关

**适用范围**：框架开发者、构建工程师

---

## GN 构建条件

### 平台适配

| 条件 | 说明 | 使用位置 |
|------|------|----------|
| `is_mingw` | Windows 平台 | 所有 BUILD.gn |
| `is_mac` | macOS 平台 | 所有 BUILD.gn |
| `is_linux` | Linux 平台（隐式） | - |

**使用示例**：
```gn
# ohos/multimedia/camera/BUILD.gn:20-36
if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.multimedia.camera.cj"]
} else {
    sources = [
        "camera.cj",
        "camera_ability.cj",
        // ...
    ]
}
```

---

## API 级别配置

### @!APILevel 属性

用于标记接口的 API 级别、系统能力、权限等：

| 属性 | 说明 | 示例 |
|------|------|------|
| `since` | API 引入版本 | `"22"` |
| `syscap` | 系统能力要求 | `"SystemCapability.Multimedia.Camera.Core"` |
| `permission` | 所需权限 | `"ohos.permission.CAMERA"` |
| `throwexception` | 是否抛出异常 | `true` |
| `workerthread` | Worker 线程执行 | `true` |

**使用示例**：
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.CAMERA",
    syscap: "SystemCapability.Multimedia.Camera.Core",
    throwexception: true
]
public func createCameraInput(camera: CameraDevice): CameraInput
```

---

## 权限常量

### 相机权限

```cangjie
// 声明在 camera_manager.cj
permission: "ohos.permission.CAMERA"
```

### 相册权限

```cangjie
// photo_accesshelper.cj:33
const INVALID_ARGUMENT_ERROR: Int32 = 13900020

// 读权限
permission: "ohos.permission.READ_IMAGEVIDEO"

// 写权限
permission: "ohos.permission.WRITE_IMAGEVIDEO"
```

---

## 错误码常量

### Camera 模块错误码

来源：`ohos/multimedia/camera/camera_common.cj:36-50`

```cangjie
let ERROR_CODE_MAP = HashMap<Int32, String>(
    [
        (7400101, "Parameter missing or parameter type incorrect."),
        (7400102, "Operation not allowed."),
        (7400103, "Session not config."),
        (7400104, "Session not running."),
        (7400105, "Session config locked."),
        (7400106, "Device setting locked."),
        (7400107, "Can not use camera cause of conflict."),
        (7400108, "Camera disabled cause of security reason."),
        (7400109, "Can not use camera cause of preempted."),
        (7400110, "Unresolved conflicts with current configurations."),
        (7400201, "Camera service fatal error.")
    ]
)
```

### Image 模块错误码

来源：`ohos/multimedia/image/cj_image_common.cj`

| 错误码 | 含义 |
|--------|------|
| 62980001 | 未知错误 |
| 62980002 | 参数错误 |
| 62980003 | 初始化失败 |
| 62980004 | 内存不足 |
| 62980006 | 解码失败 |
| 62980011 | 格式不支持 |
| 62980104 | 内部对象失败 |

### MediaLibrary 模块错误码

来源：`ohos/file/photo_access_helper/photo_accesshelper_utils.cj`

| 错误码 | 含义 |
|--------|------|
| 13900011 | 内存不足 |
| 13900012 | 权限被拒绝 |
| 13900020 | 参数无效 |
| 14000011 | 系统内部错误 |
| 201 | 权限被拒绝 |

---

## 日志配置

### 日志域

| 模块 | 日志域 | 标签 |
|------|--------|------|
| Camera | `0xD002B00` | `CJ-Camera` |
| Image | `0xD002B00` | `CJ-Image` |
| Media | `0xD002B00` | `CJ-Media` |
| PhotoAccessHelper | `0xD002B00` | `CJ-PhotoAccessHelper` |

### 日志初始化

```cangjie
// camera_common.cj:29-30
const LOG_DOMAIN: UInt32 = 0xD002B00
let CAMERA_LOG = HilogChannel(0, LOG_DOMAIN, "CJ-Camera")
```

---

## 隐藏功能标记

### @!Hide 属性

用于标记内部功能，不对外暴露：

```cangjie
@!Hide[isChecked: true]
internal enum ResolutionQuality {
    @!Hide[isChecked: true]
    Low | Medium | High
}
```

### 隐藏功能列表

详见 [内部 API 文档](../04_Inner_API.md) 中的"隐藏功能列表"章节。

---

## 枚举值定义

### CameraFormat

| 枚举值 | 数值 |
|--------|------|
| CameraFormatRgba8888 | 3 |
| CameraFormatYuv420Sp | 1003 |
| CameraFormatJpeg | 2000 |
| CameraFormatYcbcrP010 | 2001 |
| CameraFormatYcrcbP010 | 2002 |
| CameraFormatHeic | 2003 |

### SceneMode

| 枚举值 | 数值 |
|--------|------|
| NormalPhoto | 1 |
| NormalVideo | 2 |
| SecurePhoto | 12 |

### CameraPosition

| 枚举值 | 数值 |
|--------|------|
| CameraPositionUnspecified | 0 |
| CameraPositionBack | 1 |
| CameraPositionFront | 2 |

---

## Bundle 配置

### 组件配置 (bundle.json)

| 属性 | 值 |
|------|-----|
| name | `@ohos/multimedia_cangjie_wrapper` |
| version | `6.1` |
| subsystem | `multimedia` |
| rom | `1300KB` |
| ram | `1212KB` |

### 适配系统类型

```json
"adapted_system_type": ["standard"]
```

---

## 运行时开关

### 当前无运行时开关

本项目当前没有通过宏或配置开关控制的功能，所有功能编译时确定。

---

## 相关跳转

- [GN Targets](../05_GN_Targets.md)
- [内部 API](../04_Inner_API.md)
- [FAQ 与调试](../08_FAQ_Debugging.md)
