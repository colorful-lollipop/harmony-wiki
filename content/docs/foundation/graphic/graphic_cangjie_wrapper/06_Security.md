# 安全风险评审 - graphic_cangjie_wrapper

## 评审范围

| 范围 | 说明 |
|-----|-----|
| 代码路径 | `kit/`, `ohos/graphics/color_space_manager/` |
| 排除范围 | `test/`, `mock/`, `figures/` |
| FFI 依赖 | `graphic_2d:cj_color_manager_ffi` (不直接评审) |

---

## 攻击面清单

| 攻击面 | 类型 | 说明 |
|-------|-----|-----|
| **create() API** | 输入验证 | 接收 ColorSpace 枚举或 ColorSpacePrimaries 参数 |
| **getColorSpaceType()** | 信息泄露 | 返回色域类型 |
| **getWhitePoint()** | 信息泄露 | 返回白点坐标数组 |
| **getGamma()** | 信息泄露 | 返回伽马值 |
| **FFI 调用** | Native 交互 | 通过 FFI 调用 graphic_2d 实现 |
| **错误码** | 异常信息 | BusinessException 可能泄露内部状态 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────┐
│  Cangjie 用户代码 (可信域)                            │
│    create(), getColorSpaceType(), getWhitePoint()   │
└─────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────┐
│  graphic_cangjie_wrapper (边界)                     │
│    - 参数校验                                        │
│    - 错误码映射                                      │
│    - FFI 调用转发                                    │
└─────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────┐
│  graphic_2d:cj_color_manager_ffi (不可信域)           │
│    - Native 实现                                     │
│    - 实际的色彩空间管理                              │
└─────────────────────────────────────────────────────┘
```

---

## 风险分析与修复建议

### 风险 1：ColorSpacePrimaries 参数未做范围校验

| 属性 | 值 |
|-----|-----|
| **风险 ID** | SEC-001 |
| **严重程度** | 中 |
| **可利用性** | 低 |
| **证据位置** | `color_space_primaries.cj:492-501` |

**描述**:
`ColorSpacePrimaries` 构造函数接收 8 个 Float32 参数（色度坐标），但代码未对这些值进行范围校验。有效的 CIE xy 色度坐标应满足 x ∈ (0, 1), y ∈ (0, 1) 且 x + y ≤ 1。

**可利用路径**:
```cangjie
// 构造非法色度坐标
let maliciousPrimaries = ColorSpacePrimaries(0.9, 0.9, 0.5, 0.5, 0.5, 0.5, 0.9, 0.9)
// → 传递给 CJ_ColorMgrCreate FFI
```

**潜在影响**:
- 传递给 Native 层可能导致未定义行为
- 可能触发 graphic_2d 内部的断言失败或异常
- 渲染输出异常或色彩计算错误

**修复建议**:
```cangjie
public init(redX: Float32, redY: Float32, ...) {
    // 添加范围校验
    if (redX <= 0 || redX >= 1 || redY <= 0 || redY >= 1) {
        throw BusinessException(18600001, "redX/redY out of range")
    }
    // 校验 x + y <= 1 (xy 色度图约束)
    if (redX + redY >= 1) {
        throw BusinessException(18600001, "Invalid xy chromaticity")
    }
    // ... 其他参数校验
}
```

---

### 风险 2：gamma 参数未做正值校验

| 属性 | 值 |
|-----|-----|
| **风险 ID** | SEC-002 |
| **严重程度** | 中 |
| **可利用性** | 中 |
| **证据位置** | `color_space_manager.cj:74-87` |

**描述**:
`create(primaries, gamma)` 方法接收 gamma 参数，但未校验 gamma > 0。负值或零的伽马值会导致渲染计算异常。

**可利用路径**:
```cangjie
let badManager = create(primaries, -1.0)  // 负伽马值
```

**潜在影响**:
- 图像渲染亮度异常
- 可能触发除零错误

**修复建议**:
```cangjie
public func create(primaries: ColorSpacePrimaries, gamma: Float32): ColorSpaceManager {
    if (gamma <= 0) {
        throw BusinessException(18600001, "gamma must be positive")
    }
    // ... 现有逻辑
}
```

---

### 风险 3：FFI 返回指针未完全校验

| 属性 | 值 |
|-----|-----|
| **风险 ID** | SEC-003 |
| **严重程度** | 低 |
| **可利用性** | 低 |
| **证据位置** | `color_space_manager.cj:51-52, 139-142` |

**描述**:
`CJ_ColorMgrGetWhitePoint` 返回 `CPointer<Float32>`，代码在获取结果后调用 `LibC.free(res)` 释放内存。但若 FFI 返回空指针，可能导致程序崩溃。

**现有防护**:
- 使用 `unsafe` 块
- `checkRet(errCode, ...)` 校验错误码

**修复建议**:
```cangjie
let res = CJ_ColorMgrGetWhitePoint(getID(), inout errCode)
if (res == nullptr) {  // 添加空指针检查
    throw BusinessException(18600001, "Failed to get white point")
}
```

---

### 风险 4：BusinessException 泄露内部错误码

| 属性 | 值 |
|-----|-----|
| **风险 ID** | SEC-004 |
| **严重程度** | 低 |
| **可利用性** | 低 |
| **证据位置** | `cj_color_manager_utils.cj:26-40` |

**描述**:
错误处理通过 `getUniversalErrorMsg(errCode)` 获取错误信息，若 FFI 返回未知错误码，可能泄露 graphic_2d 内部状态。

**现有防护**:
```cangjie
if (let Some(v) <- getUniversalErrorMsg(errCode)) {
    msg = message + v
} else if (ERROR_CODE_MAP.contains(errCode)) {
    msg = message + ERROR_CODE_MAP[errCode]
} else {
    msg = message + "Unrecognized error code: ${errCode}"
}
```

**评估**: 风险较低，代码已有防护机制

---

### 风险 5：ColorSpaceManager ID 暴露

| 属性 | 值 |
|-----|-----|
| **风险 ID** | SEC-005 |
| **严重程度** | 低 |
| **可利用性** | 极低 |
| **证据位置** | `color_space_manager.cj:97-99` |

**描述**:
`ColorSpaceManager` 内部存储 `myDataId`（Int64 类型 FFI 数据 ID）。虽然 ID 是内部状态，但若被恶意利用可能访问非法 FFI 数据。

**现有防护**:
- `init(id: Int64)` 为 protected 方法
- 只能通过 `create()` 工厂方法创建实例

**评估**: 风险较低，封装设计合理

---

## 已验证安全特性

| 特性 | 状态 | 说明 |
|-----|-----|-----|
| 输入范围校验 | ⚠️ 部分实现 | ColorSpacePrimaries 和 gamma 缺少校验 |
| 空指针防护 | ⚠️ 部分实现 | FFI 返回值校验不完整 |
| 错误信息脱敏 | ✅ 已实现 | 错误码映射机制 |
| 实例创建管控 | ✅ 已实现 | protected 构造函数 |
| 资源释放 | ✅ 已实现 | ~init() 调用 releaseFFIData |

---

## 风险汇总

| 风险 ID | 风险描述 | 严重程度 | 修复优先级 |
|--------|---------|---------|-----------|
| SEC-001 | ColorSpacePrimaries 参数范围未校验 | 中 | 高 |
| SEC-002 | gamma 参数未做正值校验 | 中 | 高 |
| SEC-003 | FFI 返回指针空指针检查缺失 | 低 | 中 |
| SEC-004 | BusinessException 可能泄露内部状态 | 低 | 低 |
| SEC-005 | ColorSpaceManager ID 暴露风险 | 低 | 低 |

---

## 安全加固建议

### 短期（高优先级）

1. **添加 ColorSpacePrimaries 参数校验**
   - 校验 xy 坐标范围 (0, 1)
   - 校验 x + y ≤ 1 色度约束
   - 抛出 BusinessException(18600001)

2. **添加 gamma 参数校验**
   - 校验 gamma > 0
   - 建议 gamma 范围 (0.1, 10.0)

### 中期（中优先级）

3. **完善 FFI 空指针校验**
   - 所有 FFI 返回指针添加 nullptr 检查

### 长期（低优先级）

4. **考虑添加参数白名单**
   - 对 ColorSpace 枚举值进行白名单校验
5. **增加安全测试用例**
   - 边界值测试 (NaN, Infinity)
   - 异常输入测试

---

## 相关文档

- [Overview](01_Overview.md)
- [Architecture](02_Architecture.md)
- [N-API](03_N-API.md)
- [Build](05_Build.md)
