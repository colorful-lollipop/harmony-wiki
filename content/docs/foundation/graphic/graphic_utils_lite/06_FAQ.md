# 常见问题（FAQ）

本文档收集 `graphic_utils_lite` 的常见构建、运行和调试问题及其解决方案。

## 构建问题

### Q1：编译提示找不到头文件

**问题描述**：
```
fatal error: 'gfx_utils/color.h' file not found
```

**原因分析**：include 目录未正确配置。

**解决方案**：

1. 确保在 `BUILD.gn` 中正确声明依赖：

```gn
deps += [ "//foundation/graphic/graphic_utils_lite:graphic_utils_lite" ]
```

2. 或直接包含项目的 include 路径：

```gn
include_dirs += [
    "//foundation/graphic/graphic_utils_lite/interfaces/kits",
    "//foundation/graphic/graphic_utils_lite/interfaces/innerkits",
]
```

**证据来源**：`utils.gni` 第 14-17 行

---

### Q2：liteos_m 版本编译失败

**问题描述**：
```
error: undefined reference to 'hilog_lite' functions
```

**原因分析**：`liteos_m` 内核版本使用不同的日志库配置。

**解决方案**：

确保 `BUILD.gn` 正确配置日志依赖：

```gn
if (ohos_kernel_type == "liteos_m") {
    deps = [ "//third_party/bounds_checking_function:libsec_static" ]
    public_deps = [ "//base/hiviewdfx/hilog_lite/frameworks/mini:hilog_lite" ]
} else {
    deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
    public_deps = [ "//base/hiviewdfx/hilog_lite/frameworks/featured:hilog_shared" ]
}
```

**证据来源**：`BUILD.gn` 第 93-101 行

---

### Q3：编译产物版本不匹配

**问题描述**：
```
Library 'libgraphic_utils.so' version mismatch
```

**原因分析**：`bundle.json` 中定义的版本与实际编译版本不一致。

**解决方案**：

1. 检查 `bundle.json` 版本配置：

```json
{
    "version": "3.1",
    "component": {
        "name": "graphic_utils_lite",
        "subsystem": "graphic"
    }
}
```

2. 确保使用正确的编译命令：

```bash
hb build graphic_utils_lite --product {product_name} --ccache
```

---

### Q4：编译时提示内存函数未定义

**问题描述**：
```
undefined reference to '__wrap_malloc'
undefined reference to '__wrap_free'
```

**原因分析**：`bounds_checking_function` 依赖未正确链接。

**解决方案**：

确保链接了内存安全库：

```gn
deps = [ "//third_party/bounds_checking_function:libsec_static" ]
# 或
deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
```

**证据来源**：`BUILD.gn` 第 94 行、第 98 行

---

## 运行问题

### Q5：图形渲染显示异常

**问题描述**：绘制内容未正确显示或显示位置偏移。

**排查步骤**：

1. 检查 `BufferInfo` 配置是否正确：

```cpp
BufferInfo bufferInfo;
bufferInfo.virAddr = mappedAddr;  // 确认映射地址正确
bufferInfo.stride = width * bytesPerPixel;  // 确认 stride 计算
bufferInfo.rect = {0, 0, width, height};  // 确认矩形范围
```

2. 检查 `ColorMode` 是否与实际缓冲区格式匹配。

**证据来源**：`interfaces/kits/gfx_utils/graphic_buffer.h`

---

### Q6：HAL 初始化失败

**问题描述**：
```
GfxEngines init failed: driver not found
HiFbdev init failed
```

**排查步骤**：

1. 确认设备支持显示驱动：

```bash
ls -la /dev/fb0
```

2. 检查 `hdi_display` 驱动是否正常加载。

3. 确认非 `liteos_m` 系统编译了 `graphic_hals` 模块。

**证据来源**：`BUILD.gn` 第 129 行（条件编译）

---

### Q7：内存泄漏

**问题描述**：长时间运行后内存持续增长。

**排查步骤**：

1. 检查 `UIMalloc`/`UIFree` 是否配对使用。

2. 检查 `ImageCacheMalloc`/`ImageCacheFree` 是否配对使用。

3. 使用内存追踪工具验证释放路径。

**证据来源**：`interfaces/kits/gfx_utils/mem_api.h`

---

## 调试问题

### Q8：如何启用详细日志

**解决方案**：

确保 `hilog_lite` 日志已集成，日志宏使用方式：

```cpp
#include "graphic_log.h"

GRAPHIC_LOG_DEBUG("Debug message");
GRAPHIC_LOG_INFO("Info message");
GRAPHIC_LOG_ERROR("Error message");
```

**证据来源**：`interfaces/kits/gfx_utils/graphic_log.h`

---

### Q9：如何调试图形渲染问题

**解决方案**：

1. 启用性能统计：

```cpp
GraphicPerformance::GetInstance()->StartTrace("Render");
    // 渲染逻辑
GraphicPerformance::GetInstance()->EndTrace("Render");
```

2. 检查 `GraphicTimer` 计时是否异常。

**证据来源**：`interfaces/innerkits/graphic_performance.h`

---

### Q10：如何定位 GFX 驱动问题

**解决方案**：

1. 检查 `GfxEngines` 返回值：

```cpp
GfxEngines* gfx = GfxEngines::GetInstance();
if (!gfx->InitDriver()) {
    // 打印驱动初始化失败原因
}
```

2. 检查 `HiFbdevInit` 状态：

```cpp
HiFbdevInit();
if (GetDevSurfaceData() == nullptr) {
    // FrameBuffer 未正确初始化
}
```

**证据来源**：`interfaces/innerkits/hals/gfx_engines.h`、`hi_fbdev.h`

---

## 性能问题

### Q11：渲染性能低

**排查方向**：

1. 检查是否启用了 NEON 优化（ARM 设备）：

```cpp
#include "graphic_neon_utils.h"
```

2. 检查 `TransformAlgorithm` 选择是否合适：

```cpp
Transform::SetAlgorithm(TransformAlgorithm::BILINEAR);  // 或 NEAREST_NEIGHBOR
```

3. 检查缓冲区格式是否优化（如使用 `RGB565` 替代 `ARGB8888`）。

**证据来源**：`interfaces/kits/gfx_utils/graphic_types.h`

---

### Q12：内存占用过高

**解决方案**：

1. 检查是否启用了不需要的 Feature Flags，禁用以减小 ROM：

```gn
# 在 BUILD.gn 中注释不需要的功能
# defines -= [ "GRAPHIC_ENABLE_BLUR_EFFECT_FLAG" ]
# defines -= [ "GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG" ]
```

2. 优化 `BufferInfo` 分配策略，避免分配过大缓冲区。

**证据来源**：`BUILD.gn` 第 111-125 行

---

## 移植问题

### Q13：如何移植到新平台

**步骤**：

1. 确保 `hdi_display` 驱动已适配新平台。

2. 配置 `graphic_config.h` 中的平台参数。

3. 如需自定义内存分配，实现 `MemApi` 接口。

4. 编译测试并验证 GFX 功能。

**证据来源**：`interfaces/innerkits/graphic_config.h`

---

### Q14：liteos_m 与非 liteos_m 差异

| 差异点 | liteos_m | 非 liteos_m |
|--------|----------|-------------|
| 库类型 | 静态库 `.a` | 动态库 `.so` |
| HAL 模块 | 不编译 | 编译 `graphic_hals` |
| 日志库 | `hilog_lite:mini` | `hilog_lite:featured` |
| 内存安全 | `libsec_static` | `libsec_shared` |

**证据来源**：`BUILD.gn` 第 39-43 行、第 93-101 行

---

## 其他问题

### Q15：如何贡献代码

**步骤**：

1. Fork 项目仓库。

2. 创建功能分支：`git checkout -b feature/xxx`。

3. 遵循代码风格（`.clang-format`）。

4. 确保新增 API 有完整的头文件注释（Doxygen 风格）。

5. 提交 Pull Request。

**代码风格配置**：`/foundation/graphic/graphic_utils_lite/.clang-format`

---

### Q16：API 兼容性如何保证

**说明**：

- Kits API（`interfaces/kits/`）相对稳定，向后兼容。
- InnerKits API（`interfaces/innerkits/`）可能在次要版本中调整。
- Feature Flags 可能随版本变化。

**建议**：
- 使用稳定版本的 API。
- 关注 Release Notes 中的 API 变更公告。

---

## 相关资源

| 资源 | 链接 |
|------|------|
| 项目源码 | `//foundation/graphic/graphic_utils_lite` |
| OpenHarmony 文档 | https://gitee.com/openharmony/docs |
| 构建指南 | 参考 [GN 构建配置](04_Build.md) |
| 安全指南 | 参考 [安全风险评估](05_Security.md) |

---

*最后更新时间：2026-02-06*
