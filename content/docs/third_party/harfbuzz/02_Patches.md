# HarfBuzz Patch 详细分析

本文档详细分析 OpenHarmony 对 HarfBuzz 11.0.0 所做的所有 Patch，解释每个 Patch 的原始问题、修改内容以及在 OpenHarmony 中的价值。

## 1 Patch 清单总览

| Patch 文件 | 修改文件数 | 新增代码行 | 主要修改类型 |
|-----------|-----------|-----------|-------------|
| huawei_harfbuzz.patch | 10 | 872 | ICCARM 适配 + COLR 增强 + glyf 支持 |

### 1.1 Patch 元数据

- **Patch 文件**: `huawei_harfbuzz.patch`
- **作者**: Zacoh (kouzhenrong@h-partners.com)
- **创建日期**: 2024年5月14日
- **对应上游版本**: HarfBuzz 11.0.0
- **Patch 路径**: `/Volumes/lexar/code/d/work/oh/third_party/harfbuzz/huawei_harfbuzz.patch`

---

## 2 ICCARM 编译器适配

这是 Patch 的核心目的之一。华为设备大量使用 IAR Embedded Workbench（ICCARM）编译器，需要对 HarfBuzz 代码进行特定适配才能正确编译和运行。

### 2.1 CoverageFormat2.hh 类型溢出修复

**修改文件**: `src/OT/Layout/Common/CoverageFormat2.hh`

**原始问题**: ICCARM 编译器在处理 `large_int` 类型时存在返回值溢出问题，导致 `get_population()` 函数返回不正确的结果。

**修改内容**:

```cpp
// 原代码
template <typename Types>
typename Types::large_int get_population () const
{
  typename Types::large_int ret = 0;
  for (const auto &r : rangeRecord)
    ret += r.get_population ();
  return ret > UINT_MAX ? UINT_MAX : (unsigned) ret;  // ICCARM 问题
}

// 修改后
template <typename Types>
typename Types::large_int get_population () const
{
  typename Types::large_int ret = 0;
  for (const auto &r : rangeRecord)
    ret += r.get_population ();
#ifdef ENABLE_ICCARM
  return ret;  // 直接返回，避免溢出
#else
  return ret > UINT_MAX ? UINT_MAX : (unsigned) ret;
#endif
}
```

**OH 价值**: 确保在 ICCARM 编译器下正确计算 OpenType Coverage 表的字形数量，这对于字形查找和字体子集化至关重要。

**回归风险**: 低。这是一个兼容性修改，不影响 GCC/Clang 的行为。

### 2.2 hb-open-type.hh 模板转换符修复

**修改文件**: `src/hb-open-type.hh`

**原始问题**: ICCARM 编译器对隐式类型转换的处理与 GCC/Clang 不同，导致 `SortedUnsizedArrayOf` 的模板实例化失败。

**修改内容**:

```cpp
// 原代码
struct SortedUnsizedArrayOf : UnsizedArrayOf<Type>
{
  operator hb_sorted_array_t<Type> ()             { return as_array (); }
  operator hb_sorted_array_t<const Type> () const { return as_array (); }
};

// 修改后
struct SortedUnsizedArrayOf : UnsizedArrayOf<Type>
{
#ifdef ENABLE_ICCARM
  operator hb_sorted_array_t<Type> ()             { return as_array ( 0 ); }
  operator hb_sorted_array_t<const Type> () const { return as_array ( 0 ); }
#else
  operator hb_sorted_array_t<Type> ()             { return as_array (); }
  operator hb_sorted_array_t<const Type> () const { return as_array (); }
#endif
};
```

**OH 价值**: 解决 ICCARM 编译器上的数组转换问题，确保 `hb_sorted_array_t` 正确工作。

### 2.3 hb-static.cc 静态初始化修复

**修改文件**: `src/hb-static.cc`

**原始问题**: ICCARM 编译器对全局变量的空初始化（`{}`）处理可能导致静态初始化顺序问题。

**修改内容**:

```cpp
// 原代码
uint64_t const _hb_NullPool[(HB_NULL_POOL_SIZE + sizeof (uint64_t) - 1) / sizeof (uint64_t)] = {};
/*thread_local*/ uint64_t _hb_CrapPool[(HB_NULL_POOL_SIZE + sizeof (uint64_t) - 1) / sizeof (uint64_t)] = {};

// 修改后
#ifdef ENABLE_ICCARM
uint64_t const _hb_NullPool[(HB_NULL_POOL_SIZE + sizeof (uint64_t) - 1) / sizeof (uint64_t)] = { 0 };
/*thread_local*/ uint64_t _hb_CrapPool[(HB_NULL_POOL_SIZE + sizeof (uint64_t) - 1) / sizeof (uint64_t)] = { 0 };
#else
uint64_t const _hb_NullPool[(HB_NULL_POOL_SIZE + sizeof (uint64_t) - 1) / sizeof (uint64_t)] = {};
/*thread_local*/ uint64_t _hb_CrapPool[(HB_NULL_POOL_SIZE + sizeof (uint64_t) - 1) / sizeof (uint64_t)] = {};
#endif
```

**OH 价值**: 解决 ICCARM 上的静态初始化问题，确保 HarfBuzz 的内存池正确初始化。

---

## 3 COLR 颜色字体支持增强

COLR（Color）是 OpenType 2.0 规范中引入的彩色字体扩展，允许字体包含彩色字形（如 Emoji）。Patch 大幅增强了 HarfBuzz 对 COLR 表的处理能力。

### 3.1 PaintGlyph 增强

**修改文件**: `src/OT/Color/COLR/COLR.hh`

**原始问题**: 上游版本的 `PaintGlyph` 类内联实现过于复杂，在 ICCARM 编译器上可能导致内联展开问题或代码膨胀。

**修改内容**:

将 `PaintGlyph` 类的三个核心方法改为声明分离模式：

```cpp
#ifdef ENABLE_ICCARM
  bool subset (hb_subset_context_t *c,
               const ItemVarStoreInstancer &instancer) const;
  bool sanitize (hb_sanitize_context_t *c) const;
  void paint_glyph (hb_paint_context_t *c) const;
#else
  // 原有内联实现保持不变
  bool subset (hb_subset_context_t *c,
               const ItemVarStoreInstancer &instancer) const
  {
    // ... 内联实现
  }
  // ...
#endif
```

**OH 价值**: 支持 ICCARM 编译，同时为彩色字体渲染提供完整的子集化（subsetting）和绘制支持。

### 3.2 PaintTranslate 变换支持

**修改文件**: `src/OT/Color/COLR/COLR.hh`

**功能说明**: `PaintTranslate` 用于在绘制彩色字形时进行坐标平移变换。

**新增方法**:

```cpp
inline bool PaintTranslate::subset (hb_subset_context_t *c,
              const ItemVarStoreInstancer &instancer,
              uint32_t varIdxBase) const
{
  TRACE_SUBSET (this);
  auto *out = c->serializer->embed (this);
  if (unlikely (!out)) return_trace (false);

  if (instancer && !c->plan->pinned_at_default && varIdxBase != VarIdx::NO_VARIATION)
  {
    out->dx = dx + (int) roundf (instancer (varIdxBase, 0));
    out->dy = dy + (int) roundf (instancer (varIdxBase, 1));
  }

  if (format == 15 && c->plan->all_axes_pinned)
      out->format = 14;

  return_trace (out->src.serialize_subset (c, src, this, instancer));
}
```

**OH 价值**: 支持彩色字体的坐标变换，这对于正确渲染带位置的彩色 Emoji 至关重要。

### 3.3 PaintScale 系列变换支持

**修改文件**: `src/OT/Color/COLR/COLR.hh`

**包含类**:
- `PaintScale`: 独立缩放（X 和 Y 方向可不同）
- `PaintScaleAroundCenter`: 围绕中心点缩放
- `PaintScaleUniform`: 均匀缩放
- `PaintScaleUniformAroundCenter`: 围绕中心点均匀缩放

**OH 价值**: 完整的缩放变换支持使得彩色字体的视觉呈现更加灵活，支持不同尺寸和比例的彩色图标。

### 3.4 PaintRotate 旋转支持

**修改文件**: `src/OT/Color/COLR/COLR.hh`

**包含类**:
- `PaintRotate`: 基础旋转
- `PaintRotateAroundCenter`: 围绕中心点旋转

**OH 价值**: 支持彩色字体的旋转变换，增强字体的视觉表现力。

### 3.5 PaintSkew 倾斜支持

**修改文件**: `src/OT/Color/COLR/COLR.hh`

**包含类**:
- `PaintSkew`: 基础倾斜
- `PaintSkewAroundCenter`: 围绕中心点倾斜

**OH 价值**: 支持彩色字体的倾斜变换，用于实现特殊字体效果。

### 3.6 PaintComposite 复合操作支持

**修改文件**: `src/OT/Color/COLR/COLR.hh`

**功能说明**: `PaintComposite` 支持多个彩色绘制操作的复合，支持 32 种混合模式。

**OH 价值**: 高级彩色字体效果的基础，支持复杂的颜色混合和叠加。

---

## 4 glyf 字形表支持增强

### 4.1 Glyph::get_points() ICCARM 适配

**修改文件**: `src/OT/glyf/Glyph.hh`

**原始问题**: `Glyph::get_points()` 模板函数是一个复杂的内联函数，包含大量的字形点提取逻辑。在 ICCARM 编译器上可能导致编译时间过长或代码膨胀。

**修改内容**: 将完整的实现移至 `ENABLE_ICCARM` 条件分支内：

```cpp
template <typename accelerator_t>
#ifdef ENABLE_ICCARM
inline bool Glyph::get_points (...) const
#else
bool Glyph::get_points (...) const
#endif
{
  // 完整的字形点提取实现
  // 包含：
  // - 简单字形的轮廓点提取
  // - 复合字形的组件点提取
  // - 变体（variation）处理
  // - 幻像点（phantom points）添加
  // ...
}
```

**OH 价值**: 确保在 ICCARM 编译器上正确提取 TrueType 字形的轮廓点和组件信息。

### 4.2 GlyphHeader::get_extents_without_var_scaled() ICCARM 适配

**修改文件**: `src/OT/glyf/GlyphHeader.hh`

**功能说明**: 获取字形边界框信息（不含变体缩放）。

**OH 价值**: 正确的边界框计算对于字形定位和文本布局至关重要。

---

## 5 OpenType 布局相关修复

### 5.1 hb-ot-layout-common.hh 条件处理修复

**修改文件**: `src/hb-ot-layout-common.hh`

**修改内容**: 将多个条件类（ConditionAnd、ConditionOr、ConditionNegate）的 `sanitize()` 方法改为声明分离模式：

```cpp
#ifdef ENABLE_ICCARM
  bool sanitize (hb_sanitize_context_t *c) const;
#else
  bool sanitize (hb_sanitize_context_t *c) const
  {
    TRACE_SANITIZE (this);
    return_trace (conditions.sanitize (c, this));
  }
#endif
```

**OH 价值**: 确保 ICCARM 编译器上 OpenType 布局引擎的条件处理正确工作。

### 5.2 hb-ot-layout-gsubgpos.hh 模板分离

**修改文件**: `src/hb-ot-layout-gsubgpos.hh`

**修改内容**: `GSUBGPOS::closure_lookups` 模板函数声明与实现分离。

**OH 价值**: 解决 ICCARM 编译器上的模板链接问题。

### 5.3 hb-ot-layout-gpos-table.hh 实现补充

**修改文件**: `src/hb-ot-layout-gpos-table.hh`

**修改内容**: 为 ICCARM 提供 `GSUBGPOS::closure_lookups` 的模板实现：

```cpp
#ifdef ENABLE_ICCARM
template <typename TLookup>
void GSUBGPOS::closure_lookups (hb_face_t      *face,
    const hb_set_t *glyphs,
    hb_set_t       *lookup_indexes /* IN/OUT */) const
{
  hb_set_t visited_lookups, inactive_lookups;
  hb_closure_lookups_context_t c (face, glyphs, &visited_lookups, &inactive_lookups);

  c.set_recurse_func (TLookup::template dispatch_recurse_func<hb_closure_lookups_context_t>);

  for (unsigned lookup_index : *lookup_indexes)
    reinterpret_cast<const TLookup &> (get_lookup (lookup_index)).closure_lookups (&c, lookup_index);

  hb_set_union (lookup_indexes, &visited_lookups);
  hb_set_subtract (lookup_indexes, &inactive_lookups);
}
#endif
```

**OH 价值**: 支持 ICCARM 编译器上的 OpenType 布局查找闭包计算。

---

## 6 cmap 表修复

### 6.1 SubtableUnicodesCache 构造函数重载

**修改文件**: `src/hb-ot-cmap-table.hh`

**原始问题**: ICCARM 编译器对模板构造函数的选择存在差异。

**修改内容**:

```cpp
#ifdef ENABLE_ICCARM
  SubtableUnicodesCache(const void* cmap_base);
  SubtableUnicodesCache(hb_blob_ptr_t<cmap> base_blob_);
#else
  SubtableUnicodesCache(const void* cmap_base)
      : base_blob(),
        base ((const char*) cmap_base),
        // ...
  // ...
#endif
```

**OH 价值**: 确保 ICCARM 编译器上 cmap 表的缓存机制正常工作。

---

## 7 Patch 升级建议

### 7.1 可推向上游的修改

以下修改具有通用性，建议向上游 HarfBuzz 项目提交：

1. **COLR Paint* 类的完整实现**: 这些实现质量较高，对所有用户都有价值
2. **CoverageFormat2.hh 溢出修复**: 虽然主要针对 ICCARM，但 `large_int` 处理逻辑本身有改进空间
3. **模板声明与实现的合理分离**: 提高代码可维护性

### 7.2 OH 特有修改

以下修改与 ICCARM 编译器紧密耦合，建议保留在 OH Patch 中：

1. **所有 `ENABLE_ICCARM` 条件分支**: 这些是编译器特定适配
2. **hb-static.cc 初始化方式**: 静态初始化顺序问题具有编译器特定性
3. **hb-open-type.hh 转换符修改**: 模板实例化差异

### 7.3 升级注意事项

升级 HarfBuzz 新版本时需要：

1. **首先应用 ICCARM 相关修改**: 确保代码能够编译
2. **然后合并 COLR 增强**: 逐步验证功能
3. **最后测试所有字形渲染**: 确保没有回归
4. **特别关注**: `src/OT/Color/COLR/` 目录的变更

---

## 8 风险评估

| 风险项 | 等级 | 说明 |
|-------|------|------|
| ICCARM 兼容性 | 低 | 通过条件编译保护，不影响其他编译器 |
| 功能回归 | 低 | Patch 主要添加功能，不修改现有行为 |
| 上游合并困难 | 中 | COLR 部分可以尝试合并，ICCARM 部分较困难 |
| 性能影响 | 无 | 条件编译在非 ICCARM 环境下无额外开销 |

---

## 9 验证建议

### 9.1 编译验证

```bash
# ICCARM 编译测试
hb焚烧CC=iariccarm make

# GCC/Clang 编译测试
hb焚烧CC=gcc make
hb焚烧CC=clang make
```

### 9.2 功能验证

1. **彩色字体渲染**: 测试带 COLR 表的 Emoji 字体
2. **复合字形**: 测试 TrueType 复合字形渲染
3. **文本布局**: 测试各种语言的文本排版
4. **字形子集化**: 测试字体子集化工具

### 9.3 性能验证

1. **内存占用**: 对比启用/禁用 HB_TINY 的内存差异
2. **渲染速度**: 测试大段文本的渲染性能
3. **启动时间**: 测试库加载和初始化时间

---

*本文档最后更新: 2024年*
