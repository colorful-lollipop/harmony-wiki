# Skia - OpenHarmony 2D 图形库

> **维护者**: yangguangyu6@huawei.com
> **所属子系统**: thirdparty
> **开源协议**: BSD 3-Clause / MIT (OH 版本)
> **上游地址**: https://skia.googlesource.com/skia.git

---

## 快速概览

**Skia** 是 Google 开发的完整 2D 图形库，提供文本、几何图形和图像的绘制能力。在 OpenHarmony 中，Skia 作为核心图形渲染引擎，被 ArkUI、graphic_2d 等关键子系统广泛使用。

### 关键信息

| 项目 | 信息 |
|------|------|
| **当前版本** | m133 |
| **OH 组件版本** | 3.1 |
| **主要功能** | 2D 图形渲染、文本排版、图像编解码、滤镜特效 |
| **核心依赖** | harfbuzz, icu, libjpeg-turbo, libpng, libwebp |
| **主要使用者** | graphic_2d, ace_engine, graphics_effect |

---

## OpenHarmony 适配概述

Skia 在 OpenHarmony 中的适配主要通过以下方式实现：

### 1. 平台独立实现
- **字体管理**：`src/ports/skia_ohos/` 实现 OHOS 字体管理器（fontmgr_ohos）
- **日志输出**：`src/ports/SkDebug_ohos.cpp` 集成 HiLog 日志系统
- **配置系统**：JSON 格式的字体配置文件

### 2. 编译宏控制
通过大量 OH 特定宏进行条件编译：
```c
SK_OHOS_EXTENSION        // OHOS 扩展功能
SKIA_OHOS               // OHOS 平台标识
SK_ENABLE_OHOS_CODEC    // OHOS 编解码器
SKIA_OHOS_SINGLE_OWNER  // 单所有者模式
```

### 3. 构建系统集成
- **GN 构建适配**：完整的 BUILD.gn 配置
- **性能优化**：PGO、LTO、SIMD 优化
- **特性开关**：通过 bundle.json features 控制功能

### 4. 子库 Patch
Skia 内嵌的第三方库包含 36 个 Patch：
- **zlib**: 17 个（SIMD 优化、平台适配、Bug 修复）
- **icu**: 17 个（本地化优化、编码修复）
- **expat**: 2 个（系统调用禁用）

---

## 核心功能

### 2D 图形渲染
- Canvas 绘制 API
- 路径（Path）和几何图形
- 画笔（Paint）和着色器（Shader）
- 混合模式和合成
- GPU 加速（OpenGL/Vulkan）

### 文本渲染
- 字体管理（fontmgr_ohos）
- 文本排版（Paragraph）
- 复杂文本支持（ bidi、shaping）
- HarmonyOS Symbol 支持

### 图像处理
- 编解码：JPEG、PNG、WebP、HEIF、BMP、WBMP
- 图像滤镜：模糊、阴影、渐变
- 色彩空间转换（CMS）
- SVG 渲染

### 特效系统
- 图像滤镜（ImageFilter）
- 路径特效（PathEffect）
- 颜色滤镜（ColorFilter）
- 遮罩（MaskFilter）

---

## OpenHarmony 使用场景

### 主要依赖模块

| 模块 | 用途 |
|------|------|
| **graphic_2d/2d_graphics** | 2D 图形核心，Drawing API 封装 |
| **skia_libtxt** | 文本渲染引擎（基于 skparagraph） |
| **graphics_effect** | 图形特效库（模糊、滤镜等） |
| **color_manager** | 颜色空间管理 |
| **ace_engine** | ArkUI 框架，UI 组件渲染 |
| **video_processing** | 视频处理引擎 |

### 典型使用流程

```
应用层
  ↓ 调用 Drawing API
graphic_2d (Drawing)
  ↓ 封装 Skia API
Skia (third_party/skia)
  ↓ 使用
fontmgr_ohos / HiLog / hitrace
```

---

## 文档导航

### 核心文档
- [阅读路线建议](SUMMARY.md) - 根据需求选择阅读路径
- [01_Overview.md](01_Overview.md) - Skia 库详细概览
- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [03_Build_Integration.md](03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系与使用
- [05_API_Differences.md](05_API_Differences.md) - API/接口差异
- [06_Security.md](06_Security.md) - 安全风险分析

### 工作文档
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估报告（Phase 0）

---

## 特殊适配说明

### OH 特有功能

1. **字体管理器（fontmgr_ohos）**
   - 支持 OHOS 字体系统
   - JSON 配置文件驱动
   - 手机和穿戴设备分别配置

2. **性能优化**
   - PGO（Profile Guided Optimization）
   - LTO（Link Time Optimization）
   - SIMD 优化（ARM、x86）
   - JPEG/PNG 解码优化

3. **调试支持**
   - HiLog 日志集成
   - hitrace 性能追踪
   - faultloggerd 故障日志
   - 单所有者模式（Render Service 专用）

4. **跨平台支持**
   - ArkUI-X 静态库版本
   - CROSS_PLATFORM 宏支持

---

## 版本管理

### 版本结构
```
third_party/skia/
├── BUILD.gn              # OHOS 主构建入口
├── m133/                 # M133 版本（当前使用）
│   ├── BUILD.gn          # Skia 原始构建配置
│   ├── gn/               # 构建配置
│   └── src/ports/skia_ohos/  # OHOS 实现
└── third_party/          # 内嵌第三方库
```

### 版本切换
通过 `skia_feature_upgrade` 控制：
```gn
# build_overrides/skia.gni
if (skia_feature_upgrade) {
  skia_root_dir = "//third_party/skia/m133"
} else {
  skia_root_dir = "//third_party/skia"
}
```

---

## 构建选项

### 关键特性（bundle.json）

| 特性 | 说明 |
|------|------|
| skia_feature_upgrade | 启用 M133 版本 |
| skia_feature_enable_pgo | 启用 PGO 优化 |
| skia_feature_enable_codemerge | 启用代码合并优化 |
| skia_feature_hispeed_plugin | 启用高速插件 |
| skia_feature_use_vulkan | 启用 Vulkan 支持 |
| skia_feature_ace_enable_gpu | 启用 ACE GPU 加速 |

### 编译变量示例

```bash
# 启用 PGO 优化
skia_feature_enable_pgo=true
skia_feature_pgo_path="/path/to/pgo/data"

# 启用 Vulkan
skia_feature_use_vulkan=true

# 升级到 M133
skia_feature_upgrade=true
```

---

## 常见问题

### Q: 为什么 Skia 没有直接的 Patch 文件？
A: Skia 在 OH 中的适配主要通过以下方式实现：
- 独立实现 OHOS 特定代码（`src/ports/skia_ohos/`）
- 编译宏控制（`is_ohos`、`SKIA_OHOS_*`）
- 构建系统配置（BUILD.gn、.gni 文件）

子库（zlib、icu、expat）的 Patch 主要是为了性能优化和平台适配。

### Q: fontmgr_ohos 与系统字体系统的关系？
A: fontmgr_ohos 是 Skia 的字体管理适配层，通过 JSON 配置文件映射 OHOS 字体系统路径，支持 HarmonyOS Symbol 和多设备形态（手机、穿戴）。

### Q: 如何启用 Skia 性能优化？
A: 通过 bundle.json 特性开关：
- `skia_feature_enable_pgo`: 启用 PGO 优化
- `skia_feature_enable_codemerge`: 启用代码合并优化
- `skia_feature_hispeed_plugin`: 启用高速插件

### Q: Skia 版本升级如何评估影响？
A: 参考文档：
1. [02_Patches.md](02_Patches.md) - 了解子库 Patch 状态
2. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 评估报告
3. 重点关注 OHOS 特定实现（fontmgr_ohos、SkDebug_ohos.cpp）

---

## 参考资源

### 官方文档
- [Skia 官方网站](https://skia.org/)
- [Skia 用户指南](https://skia.org/docs/user/)
- [Skia API 文档](https://api.skia.org/)

### OpenHarmony 相关
- [OpenHarmony 图形子系统文档](https://docs.openharmony.cn/)
- [Graphic 2D 组件文档](https://docs.openharmony.cn/docs/application/dev/graphics/)

### 社区资源
- [Skia 上游仓库](https://skia.googlesource.com/skia.git)
- [Chromium Skia 使用](https://www.chromium.org/developers/design-documents/graphics/skia)

---

## 贡献

### 联系方式
- **维护者**: yangguangyu6@huawei.com
- **组件**: third_party/skia
- **子系统**: thirdparty

### 问题反馈
如发现 Skia 在 OpenHarmony 中的问题，请：
1. 在 OpenHarmony Issue Tracker 提交问题
2. 标注组件：third_party/skia
3. 提供复现步骤和环境信息

---

## 许可证

| 组件 | 许可证 |
|------|--------|
| Skia 上游 | BSD 3-Clause License |
| OH 组件版本 | MIT License |
| 内嵌第三方库 | 各自的许可证（见各子库 LICENSE 文件） |

---

## 更新日志

- **2026-02-08**: 初始 Wiki 创建，完成 Phase 0 评估
- 待更新...

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
