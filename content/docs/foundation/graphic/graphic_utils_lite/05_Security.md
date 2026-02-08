# 安全风险评估（Security）

本文档对 `graphic_utils_lite` 进行安全风险评估，基于代码证据识别潜在的安全问题并提出修复建议。

## 评估范围

### 已评估代码

| 模块 | 路径 | 评估状态 |
|------|------|----------|
| Utils | `frameworks/*.cpp` | 已评估 |
| Diagram | `frameworks/diagram/**` | 已评估 |
| Hals | `frameworks/hals/**` | 已评估 |
| API 层 | `interfaces/` | 已评估 |

### 未评估代码

| 模块 | 未评估原因 |
|------|------------|
| 测试代码 | 根据规范，测试代码不纳入评估范围 |

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界图                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   外部（不可信）                                                  │
│   ┌─────────────┐                                                 │
│   │ 用户输入    │ ──┐                                             │
│   │ 文件路径    │ ──┼──→ graphic_utils_lite ──→ 驱动/硬件        │
│   │ 网络数据    │ ──┘            │                │            │
│   └─────────────┘                 │                ↓            │
│                                    │        ┌─────────────┐       │
│                                    │        │ FrameBuffer │       │
│                                    │        │ GPU/显示驱动│       │
│                                    │        └─────────────┘       │
│                                    ↓                              │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    信任边界内（可信）                    │   │
│   │   - 模块内部数据                                          │   │
│   │   - 框架层调用（window_manager, surface, arkui）         │   │
│   │   - HAL 层                                              │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 攻击面分析

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| 内存分配接口 | `UIMalloc`, `ImageCacheMalloc` | 中 |
| 图形缓冲区 | `BufferInfo.phyAddr`, `BufferInfo.virAddr` | 高 |
| 路径遍历 | 文件路径参数处理 | 低（本库不直接访问文件系统） |
| 整数溢出 | 尺寸/坐标计算 | 中 |
| 指针操作 | 缓冲区指针解引用 | 高 |
| 驱动接口 | `GfxEngines` 调用驱动 | 中 |

## 已识别风险

### 风险 1：缓冲区指针未校验（高风险）

**风险描述**：`BufferInfo` 结构体包含物理地址和虚拟地址指针，若传入无效指针可能导致内核崩溃或安全漏洞。

**证据位置**：`interfaces/kits/gfx_utils/graphic_buffer.h` 第 49-58 行

```cpp
struct BufferInfo {
    Rect rect;
    uint32_t stride;
    void* phyAddr;  // 物理地址指针
    void* virAddr;   // 虚拟地址指针
    uint16_t width;
    uint16_t height;
    ColorMode mode;
    uint32_t color;
};
```

**利用路径**：

```
攻击者控制输入 → 获取 BufferInfo → 传入无效 phyAddr/virAddr →
GfxFillArea/GfxBlit 执行 → 驱动解引用 → 系统崩溃或越权访问
```

**影响**：
- 系统拒绝服务（DoS）
- 潜在的内存越权访问

**修复建议**：
1. 在 `GfxEngines::GfxFillArea` 和 `GfxEngines::GfxBlit` 中添加指针校验
2. 校验 `phyAddr` 和 `virAddr` 是否为有效地址范围
3. 使用页面对齐校验

**建议代码**：

```cpp
bool GfxEngines::GfxFillArea(const LiteSurfaceData& dstSurfaceData,
                             const Rect& fillArea,
                             const ColorType& color,
                             const OpacityType& opa) {
    // 校验缓冲区指针
    if (dstSurfaceData.virAddr == nullptr ||
        !IsValidUserAddress(dstSurfaceData.virAddr, dstSurfaceData.size)) {
        GRAPHIC_LOG_ERROR("Invalid buffer address");
        return false;
    }
    // ... 后续处理
}
```

### 风险 2：内存分配缺少边界检查（中风险）

**风险描述**：`UIMalloc` 和 `ImageCacheMalloc` 未见明显的单次分配上限检查，可能导致内存耗尽。

**证据位置**：`interfaces/kits/gfx_utils/mem_api.h` 第 51 行、第 70 行

```cpp
void* UIMalloc(uint32_t size);  // size 参数无显式检查
void* ImageCacheMalloc(const ImageInfo& info);
```

**利用路径**：
```
循环调用 UIMalloc(size=max_uint32) → 内存耗尽 → 系统不稳定
```

**影响**：
- 内存耗尽导致系统 OOM
- 拒绝服务

**修复建议**：
1. 在 `UIMalloc` 中添加最大分配限制
2. 添加进程级内存使用统计和上限
3. 参考 `bundle.json` 中 RAM 占用（~50KB）设置合理上限

**建议代码**：

```cpp
void* UIMalloc(uint32_t size) {
    constexpr uint32_t MAX_ALLOC_SIZE = 32 * 1024;  // 32KB
    if (size == 0 || size > MAX_ALLOC_SIZE) {
        GRAPHIC_LOG_ERROR("UIMalloc: invalid size %u", size);
        return nullptr;
    }
    // ... 实际分配逻辑
}
```

### 风险 3：整数溢出风险（中风险）

**风险描述**：坐标和尺寸计算可能发生整数溢出，导致渲染越界或缓冲区溢出。

**证据位置**：`interfaces/kits/gfx_utils/graphic_types.h` 第 137-142 行

```cpp
struct Point {
    int16_t x;
    int16_t y;
};
```

**利用路径**：
```
设置异常大的坐标值 → 坐标计算溢出 → 渲染到错误内存区域
```

**影响**：
- 渲染越界
- 潜在的安全边界突破

**修复建议**：
1. 在坐标运算前添加范围校验
2. 使用更宽的数据类型进行中间计算
3. 在 `Rect` 运算中添加溢出检测

**建议代码**：

```cpp
bool Rect::Union(const Rect& other) {
    // 使用 32 位临时变量避免溢出
    int32_t newLeft = std::min(static_cast<int32_t>(left_),
                               static_cast<int32_t>(other.left_));
    int32_t newTop = std::min(static_cast<int32_t>(top_),
                              static_cast<int32_t>(other.top_));
    // ... 边界检查
}
```

### 风险 4：ColorMode 枚举越界（中风险）

**风险描述**：`ColorMode` 枚举用于索引或数组访问时，若传入非法值可能导致越界访问。

**证据位置**：`interfaces/kits/gfx_utils/graphic_types.h` 第 67-116 行

```cpp
enum ColorMode : uint8_t {
    ARGB8888 = 0,
    // ... 多个枚举值
    UNKNOWN,  // 最后一个枚举值
};
```

**利用路径**：
```
传入 ColorMode::UNKNOWN → 像素格式转换数组越界访问
```

**影响**：
- 未定义行为
- 潜在的内存越界读取

**修复建议**：
1. 在所有 ColorMode 使用处添加范围校验
2. 将 UNKNOWN 移到枚举末尾单独处理

**建议代码**：

```cpp
ColorMode ValidateColorMode(uint8_t mode) {
    if (mode >= ColorMode::UNKNOWN) {
        GRAPHIC_LOG_ERROR("Invalid color mode: %u", mode);
        return ColorMode::UNKNOWN;  // 或返回默认安全值
    }
    return static_cast<ColorMode>(mode);
}
```

### 风险 5：驱动接口缺乏校验（中风险）

**风险描述**：`GfxEngines` 封装对驱动层的调用，但传入的参数直接透传给驱动，可能触发驱动层安全问题。

**证据位置**：`interfaces/innerkits/hals/gfx_engines.h` 第 33-42 行

```cpp
bool GfxFillArea(const LiteSurfaceData& dstSurfaceData,
                 const Rect& fillArea,
                 const ColorType& color,
                 const OpacityType& opa);

bool GfxBlit(const LiteSurfaceData& srcSurfaceData,
             const Rect& srcRect,
             const LiteSurfaceData& dstSurfaceData,
             int16_t x,
             int16_t y);
```

**利用路径**：
```
构造异常参数的 LiteSurfaceData → 传入 GfxFillArea/GfxBlit →
驱动处理异常 → 驱动崩溃或行为异常
```

**影响**：
- 驱动层拒绝服务
- 渲染异常

**修复建议**：
1. 添加 `LiteSurfaceData` 完整性校验
2. 校验 `Rect` 区域是否在 Buffer 范围内
3. 记录所有驱动调用参数便于调试

**建议代码**：

```cpp
bool GfxEngines::GfxFillArea(const LiteSurfaceData& dstSurfaceData,
                             const Rect& fillArea,
                             const ColorType& color,
                             const OpacityType& opa) {
    // 校验 Rect 在 Buffer 范围内
    if (fillArea.left < 0 || fillArea.top < 0 ||
        fillArea.right > dstSurfaceData.width ||
        fillArea.bottom > dstSurfaceData.height) {
        GRAPHIC_LOG_ERROR("Fill area out of bounds");
        return false;
    }
    // ... 继续处理
}
```

## 安全最佳实践

### 已采用的安全措施

| 措施 | 说明 |
|------|------|
| 内存安全库 | 使用 `bounds_checking_function` 替代标准内存函数 |
| 单例模式 | `GfxEngines` 使用单例，避免重复初始化 |
| 日志记录 | 通过 `GRAPHIC_LOG_*` 记录错误信息 |

### 建议加强的安全措施

| 优先级 | 措施 | 适用模块 |
|--------|------|----------|
| 高 | 缓冲区指针校验 | Hals |
| 高 | 坐标范围校验 | Utils, Diagram |
| 中 | 内存分配上限 | MemApi |
| 中 | ColorMode 校验 | PixelFormatUtils |
| 低 | 驱动调用参数记录 | Hals |

## 安全相关配置

### Feature Flags 与安全

| Flag | 安全影响 |
|------|----------|
| `GRAPHIC_ENABLE_BLUR_EFFECT_FLAG` | 模糊算法涉及卷积运算，需确保边界安全 |
| `GRAPHIC_ENABLE_GRADIENT_FILL_FLAG` | 渐变填充涉及内存读写，需校验步长 |

### 构建时安全选项

**证据来源**：`BUILD.gn` 第 48-59 行

| 工具链 | 安全相关配置 |
|--------|--------------|
| ICCARM | 抑制特定编译器警告（`Pe068`, `Pa089` 等） |
| Clang | `-Wno-float-equal` |

## 审计结论

### 总体评估

| 维度 | 评级 | 说明 |
|------|------|------|
| 输入校验 | 中 | 部分接口缺乏充分校验 |
| 内存安全 | 中 | 使用 bounds_checking_function，但仍有风险点 |
| 访问控制 | 低 | 无显式权限检查，依赖调用方信任 |
| 驱动安全 | 中 | 缺乏对驱动参数的安全校验 |

### 需重点关注

1. **高优先级**：缓冲区指针校验机制
2. **高优先级**：坐标运算溢出防护
3. **中优先级**：内存分配上限控制
4. **中优先级**：ColorMode 枚举越界防护

### 局限性说明

- 本评估基于静态代码分析，未进行动态模糊测试
- 未覆盖所有边界情况
- 驱动层安全性由 `hdi_display` 组件保证，本评估未涉及

---

*最后更新时间：2026-02-06*
