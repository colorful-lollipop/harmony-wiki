# 内部模块与接口 - graphic_cangjie_wrapper

## 模块概览

| 模块路径 | 包名 | 稳定性 | 职责 |
|---------|-----|-------|-----|
| `kit/ArkGraphics2D` | `kit.ArkGraphics2D` | Stable (Kit) | Kit 层入口，重导出 |
| `ohos/graphics` | `ohos.graphics` | Stable | 包定义 |
| `ohos/graphics/color_space_manager` | `ohos.graphics.color_space_manager` | Stable | 核心 API 实现 |

---

## 模块职责详解

### kit.ArkGraphics2D

**路径**: `kit/ArkGraphics2D/`
**包**: `kit.ArkGraphics2D`
**类型**: Kit 层（对外暴露）

**职责**:
- 作为色彩管理 Kit 的入口模块
- 重导出 `ohos.graphics.color_space_manager` 包

**导出内容**:
```cangjie
package kit.ArkGraphics2D

public import ohos.graphics.color_space_manager.*
```

> **证据**: `kit/ArkGraphics2D/index.cj:18-20`

### ohos.graphics

**路径**: `ohos/graphics/`
**包**: `ohos.graphics`
**类型**: 框架层（Inner API）

**职责**:
- 定义 `ohos.graphics` 包
- 在 mingw/mac 平台使用 mock 实现

**构建配置**:
```gn
ohos_cangjie_shared_library("ohos.graphics") {
  if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.graphics.cj" ]
  } else {
    sources = [ "graphics.cj" ]
  }
}
```

> **证据**: `ohos/graphics/BUILD.gn:21-25`

### color_space_manager

**路径**: `ohos/graphics/color_space_manager/`
**包**: `ohos.graphics.color_space_manager`
**类型**: 核心模块（Inner API + 内部 FFI）

**职责**:
- 实现 ColorSpaceManager 核心类
- 定义 ColorSpace 枚举
- 定义 ColorSpacePrimaries 类
- 绑定 graphic_2d 的 FFI 函数
- 处理错误码与日志

---

## 依赖方向

```
kit.ArkGraphics2D (Kit, 对外)
    │
    └──→ ohos.graphics.color_space_manager (框架层, Inner)
            │
            ├──→ cangjie_ark_interop:ohos.ffi
            │       │
            │       └──→ FFI 类型定义 (RetDataCString, CPointer, etc.)
            │
            ├──→ cangjie_ark_interop:ohos.labels
            │       │
            │       └──→ @APILevel, @Hide 注解支持
            │
            ├──→ cangjie_ark_interop:ohos.business_exception
            │       │
            │       └──→ BusinessException 异常类
            │
            ├──→ hiviewdfx_cangjie_wrapper:ohos.hilog
            │       │
            │       └──→ HilogChannel 日志
            │
            └──→ graphic_2d:cj_color_manager_ffi (Native FFI)
                    │
                    └──→ CJ_ColorMgr* FFI 函数实现
```

> **证据**: `ohos/graphics/color_space_manager/BUILD.gn:31-38`

---

## 内部类型

### CJColorSpacePrimaries（C 结构映射）

**定义**: `@C struct CJColorSpacePrimaries`

**用途**: 与 Native FFI 交互的 C 语言结构体映射

```cangjie
@C
struct CJColorSpacePrimaries {
    CJColorSpacePrimaries(
        let redX!: Float32,
        let redY!: Float32,
        let greenX!: Float32,
        let greenY!: Float32,
        let blueX!: Float32,
        let blueY!: Float32,
        let whitePointX!: Float32,
        let whitePointY!: Float32
    ) {}

    init(primaries: ColorSpacePrimaries) { ... }
}
```

> **证据**: `color_space_primaries.cj:504-527`

---

## FFI 外部函数

| 函数名 | 来源 | 描述 |
|-------|-----|-----|
| `CJ_ColorMgrCreateByColorSpace` | graphic_2d:cj_color_manager_ffi | 按预设类型创建 |
| `CJ_ColorMgrCreate` | graphic_2d:cj_color_manager_ffi | 自定义创建 |
| `CJ_ColorMgrGetColorSpaceName` | graphic_2d:cj_color_manager_ffi | 获取色域类型名 |
| `CJ_ColorMgrGetWhitePoint` | graphic_2d:cj_color_manager_ffi | 获取白点 |
| `CJ_ColorMgrGetGamma` | graphic_2d:cj_color_manager_ffi | 获取伽马值 |

> **证据**: `color_space_manager.cj:24-34`

---

## 稳定性标注

| 模块 | 稳定性 | 依据 |
|-----|-------|-----|
| kit.ArkGraphics2D | Stable | Kit 层，对外暴露 |
| ohos.graphics | Stable | 框架层标准包 |
| ohos.graphics.color_space_manager | Stable | 核心功能模块 |

---

## 可替换点

| 可替换组件 | 替换方式 | 影响范围 |
|-----------|---------|---------|
| graphic_2d:cj_color_manager_ffi | 替换 Native 实现 | ColorSpaceManager FFI 调用结果 |

> **说明**: graphic_2d 是独立仓库，cj_color_manager_ffi 的替换不影响 Cangjie 封装层代码

---

## 相关文档

- [Architecture](02_Architecture.md)
- [N-API](03_N-API.md)
- [Build](05_Build.md)
