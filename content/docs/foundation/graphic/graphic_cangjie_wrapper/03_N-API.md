# Cangjie API 参考 - graphic_cangjie_wrapper

## API 概览

| 命名空间 | 导出类/枚举/函数 | 描述 |
|---------|-----------------|-----|
| `kit.ArkGraphics2D` | ColorSpace, ColorSpacePrimaries, ColorSpaceManager, create() | 色彩管理 Kit |

> **证据**: `kit/ArkGraphics2D/index.cj:20`

---

## ColorSpace 枚举

**包**: `ohos.graphics.color_space_manager`
**描述**: 预定义的色彩空间类型枚举

### 枚举值列表

| 枚举值 | 值 | 描述 | evidence来源 |
|-------|-----|-----|-------------|
| `Unknown` | 0 | 未知色域 | `color_space_primaries.cj:37-41` |
| `AdobeRgb1998` | 1 | Adobe RGB (1998) | `color_space_primaries.cj:45-50` |
| `DciP3` | 2 | DCI-P3 电影级色域 | `color_space_primaries.cj:55-59` |
| `DisplayP3` | 3 | Display P3 | `color_space_primaries.cj:64-68` |
| `Srgb` | 4 | sRGB IEC 61966-2.1 | `color_space_primaries.cj:73-77` |
| `Custom` | 5 | 自定义色域 | `color_space_primaries.cj:318-322` |
| `Bt709` | 6 | ITU-R BT.709 | `color_space_primaries.cj:83-87` |
| `Bt601Ebu` | 7 | BT.601 (EBU) | `color_space_primaries.cj:93-97` |
| `Bt601SmpteC` | 8 | BT.601 (SMPTE-C) | `color_space_primaries.cj:103-107` |
| `Bt2020Hlg` | 9 | BT.2020 HLG | `color_space_primaries.cj:113-117` |
| `Bt2020Pq` | 10 | BT.2020 PQ | `color_space_primaries.cj:123-127` |
| `P3Hlg` | 11 | P3 HLG | `color_space_primaries.cj:132-136` |
| `P3Pq` | 12 | P3 PQ | `color_space_primaries.cj:141-145` |
| `AdobeRgb1998Limit` | 13 | Adobe RGB Limited | `color_space_primaries.cj:150-154` |
| `DisplayP3Limit` | 14 | Display P3 Limited | `color_space_primaries.cj:159-163` |
| `SrgbLimit` | 15 | sRGB Limited | `color_space_primaries.cj:168-172` |
| `Bt709Limit` | 16 | BT.709 Limited | `color_space_primaries.cj:177-181` |
| `Bt601EbuLimit` | 17 | BT.601 (EBU) Limited | `color_space_primaries.cj:186-190` |
| `Bt601SmpteCLimit` | 18 | BT.601 (SMPTE-C) Limited | `color_space_primaries.cj:195-199` |
| `Bt2020HlgLimit` | 19 | BT.2020 HLG Limited | `color_space_primaries.cj:204-208` |
| `Bt2020PqLimit` | 20 | BT.2020 PQ Limited | `color_space_primaries.cj:213-217` |
| `P3HlgLimit` | 21 | P3 HLG Limited | `color_space_primaries.cj:222-226` |
| `P3PqLimit` | 22 | P3 PQ Limited | `color_space_primaries.cj:231-235` |
| `LinearP3` | 23 | Linear P3 | `color_space_primaries.cj:240-244` |
| `LinearSrgb` | 24 | Linear sRGB | `color_space_primaries.cj:248-252` |
| `LinearBt709` | 25 | Linear BT.709 | `color_space_primaries.cj:257-261` |
| `LinearBt2020` | 26 | Linear BT.2020 | `color_space_primaries.cj:266-270` |
| `DisplaySrgb` | 4 | Display sRGB | `color_space_primaries.cj:276-280` |
| `DisplayP3Srgb` | 3 | Display P3 sRGB | `color_space_primaries.cj:285-289` |
| `DisplayP3Hlg` | 11 | Display P3 HLG | `color_space_primaries.cj:294-298` |
| `DisplayP3Pq` | 12 | Display P3 PQ | `color_space_primaries.cj:302-307` |

### 方法

| 方法 | 返回值 | 描述 | evidence来源 |
|-----|-------|-----|-------------|
| `getValue()` | UInt32 | 获取枚举对应的整数值（protected） | `color_space_primaries.cj:325-360` |
| `parse(UInt32)` | ColorSpace | 从整数值解析枚举（static, protected） | `color_space_primaries.cj:362-392` |

---

## ColorSpacePrimaries 类

**包**: `ohos.graphics.color_space_manager`
**描述**: 自定义色域的基色参数类

### 类定义

```cangjie
public class ColorSpacePrimaries {
    public var redX: Float32
    public var redY: Float32
    public var greenX: Float32
    public var greenY: Float32
    public var blueX: Float32
    public var blueY: Float32
    public var whitePointX: Float32
    public var whitePointY: Float32

    public init(
        redX: Float32,
        redY: Float32,
        greenX: Float32,
        greenY: Float32,
        blueX: Float32,
        blueY: Float32,
        whitePointX: Float32,
        whitePointY: Float32
    )
}
```

> **证据**: `color_space_primaries.cj:403-502`

### 属性说明

| 参数 | 类型 | 范围 | 描述 |
|-----|-----|------|-----|
| `redX` | Float32 | (0, 1) | 红色色度坐标 x |
| `redY` | Float32 | (0, 1) | 红色色度坐标 y |
| `greenX` | Float32 | (0, 1) | 绿色色度坐标 x |
| `greenY` | Float32 | (0, 1) | 绿色色度坐标 y |
| `blueX` | Float32 | (0, 1) | 蓝色色度坐标 x |
| `blueY` | Float32 | (0, 1) | 蓝色色度坐标 y |
| `whitePointX` | Float32 | (0, 1) | 白点色度坐标 x |
| `whitePointY` | Float32 | (0, 1) | 白点色度坐标 y |

> **证据**: `color_space_primaries.cj:407-474`

---

## ColorSpaceManager 类

**包**: `ohos.graphics.color_space_manager`
**继承**: `RemoteDataLite`
**描述**: 色域管理器类，用于创建和管理色彩空间

### 类定义

```cangjie
public class ColorSpaceManager <: RemoteDataLite {
    protected init(id: Int64)

    ~init()

    public func getColorSpaceType(): ColorSpace
    public func getWhitePoint(): Array<Float32>
    public func getGamma(): Float32
}
```

> **证据**: `color_space_manager.cj:96-166`

### 继承说明

- 继承自 `RemoteDataLite`，提供 FFI data ID 管理能力
- 构造函数为 protected，只能通过 `create()` 工厂方法实例化
- 析构函数调用 `releaseFFIData(myDataId)` 释放 FFI 资源

---

## create() 工厂方法

### 方法重载 1：按预设类型创建

```cangjie
public func create(colorSpaceType: ColorSpace): ColorSpaceManager
```

| 参数 | 类型 | 描述 |
|-----|-----|-----|
| `colorSpaceType` | ColorSpace | 预设的色域类型 |

**返回值**: ColorSpaceManager 实例

**异常**: `BusinessException(18600001)` - 参数值异常

**示例**:
```cangjie
let srgbManager = create(ColorSpace.Srgb)
```

> **证据**: `color_space_manager.cj:43-59`

### 方法重载 2：自定义创建

```cangjie
public func create(primaries: ColorSpacePrimaries, gamma: Float32): ColorSpaceManager
```

| 参数 | 类型 | 描述 |
|-----|-----|-----|
| `primaries` | ColorSpacePrimaries | 自定义色域基色 |
| `gamma` | Float32 | 伽马值 (> 0) |

**返回值**: ColorSpaceManager 实例

**异常**: `BusinessException(18600001)` - 参数值异常

**示例**:
```cangjie
let primaries = ColorSpacePrimaries(0.64, 0.33, 0.30, 0.60, 0.15, 0.06, 0.3127, 0.3290)
let customManager = create(primaries, 2.2)
```

> **证据**: `color_space_manager.cj:69-87`

---

## 实例方法

### getColorSpaceType()

```cangjie
public func getColorSpaceType(): ColorSpace
```

**描述**: 获取当前色域的类型

**返回值**: ColorSpace 枚举值

**异常**: `BusinessException(18600001)` - 参数值异常

> **证据**: `color_space_manager.cj:111-123`

### getWhitePoint()

```cangjie
public func getWhitePoint(): Array<Float32>
```

**描述**: 获取色域的白点坐标

**返回值**: 包含 2 个元素的 Float32 数组 `[whitePointX, whitePointY]`

**异常**: `BusinessException(18600001)` - 参数值异常

**内存管理**: 内部调用 `LibC.free()` 释放 FFI 返回的指针

> **证据**: `color_space_manager.cj:131-145`

### getGamma()

```cangjie
public func getGamma(): Float32
```

**描述**: 获取色域的伽马值

**返回值**: Float32 类型的伽马值

**异常**: `BusinessException(18600001)` - 参数值异常

> **证据**: `color_space_manager.cj:153-165`

---

## 错误码

| 错误码 | 含义 | 处理建议 |
|-------|-----|---------|
| 18600001 | 参数值异常 | 检查传入参数是否在有效范围内 |

**错误码映射**: `cj_color_manager_utils.cj:23-24`

```cangjie
let ERROR_CODE_MAP = HashMap<Int32, String>(
    [(18600001, "Parameter value is abnormal.")])
```

---

## API 调用链

```
Cangjie 用户代码
    │
    ↓ 调用 create()
┌──────────────────────────────────────────────┐
│  color_space_manager.cj                       │
│  ├─ CJ_ColorMgrCreateByColorSpace (FFI)      │
│  └─ CJ_ColorMgrCreate (FFI)                  │
└──────────────────────────────────────────────┘
    │
    ↓ external_deps
┌──────────────────────────────────────────────┐
│  graphic_2d:cj_color_manager_ffi              │
│  (Native C++ 实现)                            │
└──────────────────────────────────────────────┘
```

---

## API 注解

所有公开 API 均包含以下注解：

| 注解 | 值 | 说明 |
|-----|-----|-----|
| `@!APILevel[since]` | "22" | API 起始版本 |
| `@!APILevel[syscap]` | "SystemCapability.Graphic.Graphic2D.ColorManager.Core" | 系统能力要求 |

> **证据**: `color_space_manager.cj:43-47`, `color_space_primaries.cj:29-31`

---

## 相关文档

- [Overview](01_Overview.md)
- [Architecture](02_Architecture.md)
- [Appendix: Callgraphs](appendix/Callgraphs.md)
