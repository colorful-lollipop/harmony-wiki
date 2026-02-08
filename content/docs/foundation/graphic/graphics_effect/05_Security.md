# graphics_effect 安全风险评审

## 评审概述

本章节对 graphics_effect 进行安全风险评审，基于代码证据（`BUILD.gn`, `include/*.h`, `src/*.cpp`）分析潜在攻击面和风险点。

**评审范围**: graphics_effect 源代码（不含测试）
**评审时间**: 2026-02-06

---

## 攻击面分析

### 1. 数据输入

| 输入源 | 类型 | 说明 |
|-------|------|------|
| `GEVisualEffectContainer` | 效果配置 | 上层传入的效果链 |
| `GEVisualEffect::SetParam()` | 参数 | 字符串标签 + 数值/对象 |
| `Drawing::Image` | 图像数据 | GPU 处理 |
| Shader 代码 | GLSL/SkSL | 内联 shader 字符串 |
| 外部 SO 库 | 动态加载 | `libgraphics_effect_ext.z.so` |

### 2. API 暴露

| 暴露接口 | 访问层级 | 风险 |
|---------|---------|------|
| `GEVisualEffectContainer` | C++ | 中（依赖上游校验） |
| `GERender` | C++ | 中（依赖上游校验） |
| N-API | 无 | 低（本模块无 N-API） |

### 3. 依赖安全

| 依赖 | 信任程度 | 说明 |
|-----|---------|------|
| `graphic_2d` | 高 | 系统图形库 |
| `bounds_checking_function` | 高 | 安全函数库 |
| `hilog` | 高 | 系统日志 |
| `libgraphics_effect_ext.z.so` | 中 | 外部动态加载库，需完整性校验 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  graphics_effect 内部                                     │   │
│  │  • Shader 代码 (内联，编译时固定)                         │   │
│  │  • 效果参数处理                                           │   │
│  │  • 图像数据处理                                           │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
           ▲                                     ▼
    上层调用方 (ArkUI/                    图形渲染输出
    EffectKit, 已校验)                    (Canvas/Image)
```

**关键点**: graphics_effect 假设调用方已进行必要的输入校验。

---

## 安全机制

### 1. 编译时安全

| 机制 | 状态 | 配置 |
|-----|------|------|
| FORTIFY_SOURCE | ✅ 启用 | `-D_FORTIFY_SOURCE=2` |
| Integer Overflow Check | ✅ 启用 | `-ftrapv` |
| Control Flow Integrity | ✅ 启用 | `cfi = true` |
| Pointer Authentication | ✅ 启用 | `branch_protector_ret = "pac_ret"` |
| ASan/MSan | ❌ 未启用 | `debug = false` |

**证据来源**: `BUILD.gn:24-51`

### 2. 运行时安全

| 机制 | 状态 | 说明 |
|-----|------|------|
| 边界检查 | ✅ 依赖 libsec_shared | `bounds_checking_function` |
| 空指针检查 | ⚠️ 部分实现 | 需上层保证 |
| 范围校验 | ⚠️ 部分实现 | 效果参数需校验 |

### 3. 日志与审计

| 机制 | 状态 | 说明 |
|-----|------|------|
| 日志 | ✅ 启用 | `GE_LOG_*` (hilog) |
| 追踪 | ✅ 依赖 hitrace | 性能追踪 |

---

## 潜在风险与修复建议

### 风险 1: Shader 参数范围未校验

| 属性 | 值 |
|-----|------|
| **严重程度** | 中 |
| **可利用性** | 中 |
| **证据** | `include/ge_visual_effect.h:48-72` - `SetParam()` 无范围检查 |
| **触发条件** | 传入极端数值参数 |
| **影响** | GPU 渲染异常、崩溃 |

**代码证据**:
```cpp
// include/ge_visual_effect.h
void SetParam(const std::string& tag, float param);  // 无范围校验
void SetParam(const std::string& tag, int32_t param); // 无范围校验
```

**修复建议**:
```cpp
// 添加参数范围校验
void GEVisualEffect::SetParam(const std::string& tag, float param) {
    constexpr float MAX_RADIUS = 1000.0f;
    constexpr float MAX_INTENSITY = 10.0f;
    
    if (tag == "radius" && (param < 0.0f || param > MAX_RADIUS)) {
        GE_LOGW("Invalid radius value: %{public}f, clamping to range [0, %{public}f]", 
                param, MAX_RADIUS);
        param = std::clamp(param, 0.0f, MAX_RADIUS);
    }
    // ... 设置参数
}
```

---

### 风险 2: 效果链过长可能导致资源耗尽

| 属性 | 值 |
|-----|------|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **证据** | `include/ge_visual_effect_container.h:58` - `filterVec_` 无长度限制 |
| **触发条件** | 添加大量效果到容器 |
| **影响** | 内存耗尽、渲染超时 |

**代码证据**:
```cpp
// include/ge_visual_effect_container.h
private:
    std::vector<std::shared_ptr<GEVisualEffect>> filterVec_;  // 无限制
```

**修复建议**:
```cpp
constexpr size_t MAX_FILTER_COUNT = 64;

void GEVisualEffectContainer::AddToChainedFilter(
    std::shared_ptr<GEVisualEffect> visualEffect) {
    if (filterVec_.size() >= MAX_FILTER_COUNT) {
        GE_LOGE("Filter chain too long, maximum %{public}zu filters allowed", 
                MAX_FILTER_COUNT);
        return;
    }
    filterVec_.push_back(visualEffect);
}
```

---

### 风险 3: Shader 编译错误未处理

| 属性 | 值 |
|-----|------|
| **严重程度** | 中 |
| **可利用性** | 低 |
| **证据** | `include/ge_render.h` - `GenerateShaderFilter()` 返回空可能未处理 |
| **触发条件** | Shader 编译失败 |
| **影响** | 渲染失败、空指针访问 |

**代码证据**:
```cpp
// include/ge_render.h:218
std::shared_ptr<GEShaderFilter> GenerateShaderFilter(
    const std::shared_ptr<Drawing::GEVisualEffect>& ve);
```

**修复建议**:
```cpp
std::shared_ptr<GEShaderFilter> GERender::GenerateShaderFilter(
    const std::shared_ptr<Drawing::GEVisualEffect>& ve) {
    if (!ve) {
        GE_LOGE("Cannot generate shader filter from null visual effect");
        return nullptr;
    }
    
    auto shaderFilter = /* ... */;
    if (!shaderFilter) {
        GE_LOGE("Failed to generate shader filter for effect: %{public}s", 
                ve->GetName().c_str());
        return nullptr;
    }
    return shaderFilter;
}
```

---

### 风险 4: 图像尺寸未校验

| 属性 | 值 |
|-----|------|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **证据** | `include/ge_render.h:48-50` - `DrawImageEffect()` 图像尺寸未校验 |
| **触发条件** | 传入超大/异常尺寸图像 |
| **影响** | 内存峰值过高、渲染超时 |

**修复建议**:
```cpp
constexpr int64_t MAX_IMAGE_AREA = 4096 * 4096; // 16M 像素

void GERender::DrawImageEffect(/* ... */, 
    const std::shared_ptr<Drawing::Image>& image, /* ... */) {
    if (!image) {
        GE_LOGE("Null image passed to DrawImageEffect");
        return;
    }
    
    auto dimensions = image->GetDimensions();
    int64_t area = static_cast<int64_t>(dimensions.width) * dimensions.height;
    if (area > MAX_IMAGE_AREA) {
        GE_LOGW("Image area too large: %{public}lld, maximum allowed: %{public}lld",
                area, MAX_IMAGE_AREA);
        // 返回或降级处理
    }
}
```

---

### 风险 5: 缓存资源未释放

| 属性 | 值 |
|-----|------|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **证据** | `include/ge_visual_effect_container.h:45` - `UpdateCachedBlurImage()` |
| **触发条件** | 频繁更新缓存 |
| **影响** | 内存泄漏 |

---

### 风险 6: 外部动态加载潜在风险

| 属性 | 值 |
|-----|------|
| **严重程度** | 中 |
| **可利用性** | 低 |
| **证据** | `src/ge_external_dynamic_loader.cpp:29-33` - 硬编码库路径 |
| **触发条件** | 外部 SO 库被篡改或替换 |
| **影响** | 代码执行、权限提升 |

**代码证据**:
```cpp
// 硬编码的库路径
#if (defined(__aarch64__) || defined(__x86_64__))
const std::string GRAPHICS_EFFECT_EXT_LIB_PATH = "/system/lib64/libgraphics_effect_ext.z.so";
#else
const std::string GRAPHICS_EFFECT_EXT_LIB_PATH = "/system/lib/libgraphics_effect_ext.z.so";
#endif

// 使用 dlopen 加载
libHandle_ = dlopen(GRAPHICS_EFFECT_EXT_LIB_PATH.c_str(), RTLD_LAZY);
```

**缓解因素**:
- 路径为系统受保护目录 (`/system/lib*/`)
- 使用 `RTLD_LAZY` 延迟绑定
- 可通过 `rosen.graphic.gex.enable` 参数禁用

**修复建议**:
```cpp
// 1. 添加 SO 完整性校验
bool VerifyLibraryIntegrity(const std::string& path) {
    // 验证签名或 checksum
    return system::VerifySignature(path);
}

// 2. 使用 dlopen 前校验
libHandle_ = dlopen(path.c_str(), RTLD_LAZY);
if (libHandle_ && !VerifyLibraryIntegrity(path)) {
    dlclose(libHandle_);
    libHandle_ = nullptr;
    GE_LOGE("Library integrity verification failed");
}
```

---

## 安全总结

### 风险评估汇总

| 风险 | 严重程度 | 可能性 | 风险等级 |
|-----|---------|--------|---------|
| Shader 参数范围未校验 | 中 | 中 | 🟡 中 |
| 效果链过长 | 低 | 低 | 🟢 低 |
| Shader 编译错误未处理 | 中 | 低 | 🟢 低 |
| 图像尺寸未校验 | 低 | 低 | 🟢 低 |
| 缓存资源未释放 | 低 | 低 | 🟢 低 |
| 外部动态加载风险 | 中 | 低 | 🟡 中 |

### 安全优势

1. **编译时保护完善**: FORTIFY_SOURCE, CFI, PAC/RET 全启用
2. **依赖安全**: 使用系统安全库 (`libsec_shared`)
3. **无外部数据暴露**: 不涉及网络/文件 I/O
4. **无 N-API 暴露**: 减少攻击面

### 改进建议

1. ✅ 添加效果参数范围校验
2. ✅ 添加效果链长度限制
3. ✅ 增强 Shader 编译错误处理
4. ⚠️ 考虑添加图像尺寸限制
5. ⚠️ 增强缓存资源管理
6. ⚠️ 添加外部 SO 库完整性校验

---

## 检查范围声明

### 已检查范围

| 范围 | 状态 |
|-----|------|
| `include/` 头文件 | ✅ 完整检查 |
| `src/` 源文件 | ✅ 完整检查 |
| `BUILD.gn` | ✅ 完整检查 |
| `bundle.json` | ✅ 完整检查 |

### 未检查范围

| 范围 | 原因 |
|-----|------|
| 测试代码 | 按规范忽略 |
| 第三方依赖 (Skia) | 依赖上游审计 |
| 上层调用 (ArkUI) | 超出本模块范围 |

---

## 相关文档

- [项目概述](01_Overview.md) - 定位和核心能力
- [架构说明](02_Architecture.md) - 详细架构设计
- [内部 API](03_InnerAPIs.md) - API 参考
