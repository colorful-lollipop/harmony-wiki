# 系统架构 - graphic_cangjie_wrapper

## 架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                    应用层 (Cangjie/ArkTS)                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              kit.ArkGraphics2D                          │   │
│  │              ohos.graphics.color_space_manager          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              ↓ FFI
┌─────────────────────────────────────────────────────────────────┐
│              graphic_cangjie_wrapper (Cangjie 封装)              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  color_space_manager.cj                                │   │
│  │    - ColorSpaceManager                                 │   │
│  │    - create() 工厂方法                                  │   │
│  │    - 外部函数绑定 (CJ_ColorMgr*)                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  color_space_primaries.cj                              │   │
│  │    - ColorSpace 枚举                                    │   │
│  │    - ColorSpacePrimaries 类                            │   │
│  │    - CJColorSpacePrimaries (C结构映射)                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  cj_color_manager_utils.cj                             │   │
│  │    - 错误码映射                                          │   │
│  │    - checkRet() 函数                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│              graphic_2d (Native 实现)                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  cj_color_manager_ffi                                   │   │
│  │    - CJ_ColorMgrCreateByColorSpace                      │   │
│  │    - CJ_ColorMgrCreate                                  │   │
│  │    - CJ_ColorMgrGetColorSpaceName                       │   │
│  │    - CJ_ColorMgrGetWhitePoint                           │   │
│  │    - CJ_ColorMgrGetGamma                                │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

> **证据**: `kit/ArkGraphics2D/index.cj:18-20`, `color_space_manager.cj:24-34`

## 模块职责

| 模块 | 职责 | 证据来源 |
|-----|-----|---------|
| **kit.ArkGraphics2D** | Kit 层入口，重导出 color_space_manager | `index.cj:20` |
| **ohos.graphics** | ohos.graphics 包定义 | `graphics.cj:18` |
| **color_space_manager** | 核心 API 实现、FFI 绑定 | `color_space_manager.cj:18` |
| **color_space_primaries** | ColorSpace 枚举、ColorSpacePrimaries 类 | `color_space_primaries.cj:18` |
| **cj_color_manager_utils** | 错误码映射与校验 | `cj_color_manager_utils.cj:18` |
| **cj_color_manager_log** | 日志通道封装 | `cj_color_manager_log.cj:22` |

## 数据流

### 创建色域流程

```
1. 用户调用 create(ColorSpace) 或 create(ColorSpacePrimaries, gamma)
          ↓
2. ColorSpaceManager 构造函数创建实例
          ↓
3. FFI 调用 CJ_ColorMgrCreateByColorSpace 或 CJ_ColorMgrCreate
          ↓
4. graphic_2d: cj_color_manager_ffi 实现创建
          ↓
5. 返回 Int64 类型的 FFI data ID
          ↓
6. ColorSpaceManager 包装为 RemoteDataLite
```

### 查询色域流程

```
1. 用户调用 getColorSpaceType() / getWhitePoint() / getGamma()
          ↓
2. FFI 调用 CJ_ColorMgrGet* 函数
          ↓
3. graphic_2d 返回结果
          ↓
4. Cangjie 侧解析并返回
```

> **证据**: `color_space_manager.cj:48-165`

## 线程模型

| 操作 | 线程要求 | 说明 |
|-----|---------|-----|
| create() | 主线程 | 需要创建 FFI data ID |
| getColorSpaceType() | 主线程 | FFI 调用 |
| getWhitePoint() | 主线程 | FFI 调用，内存需手动释放 |
| getGamma() | 主线程 | FFI 调用 |
| ~init() (析构) | 主线程 | 调用 releaseFFIData |

> **证据**: `color_space_manager.cj:96-103` - ColorSpaceManager 继承 RemoteDataLite

## 资源生命周期

```
┌────────────────┐
│ create() 调用   │──→ 生成 FFI data ID (Int64)
└────────────────┘
        ↓
┌────────────────┐
│ 使用 ColorSpaceManager │
└────────────────┘
        ↓
┌────────────────┐
│ ~init() 调用   │──→ 调用 releaseFFIData(myDataId) 释放
└────────────────┘
```

> **证据**: `color_space_manager.cj:101-103`

## 依赖方向

```
kit.ArkGraphics2D
    │
    ├──→ ohos.graphics.color_space_manager
    │         │
    │         ├──→ cangjie_ark_interop:ohos.ffi (FFI 支持)
    │         ├──→ cangjie_ark_interop:ohos.labels (API 注解)
    │         ├──→ cangjie_ark_interop:ohos.business_exception (异常)
    │         ├──→ hiviewdfx_cangjie_wrapper:ohos.hilog (日志)
    │         │
    │         └──→ graphic_2d:cj_color_manager_ffi (native 实现)
    │                   │
    │                   └──→ graphic_2d 内部实现
```

> **证据**: `kit/ArkGraphics2D/BUILD.gn:22`, `ohos/graphics/color_space_manager/BUILD.gn:31-38`

## 相关文档

- [Overview](01_Overview.md)
- [N-API](03_N-API.md)
- [Build](05_Build.md)
