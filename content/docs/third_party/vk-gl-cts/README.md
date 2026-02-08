# VK-GL-CTS OpenHarmony Wiki

## 库概览

**VK-GL-CTS** (Vulkan GL Conformance Test Suite) 是 OpenHarmony 第三方库中的重要图形测试组件，用于验证 GPU 驱动的 Khronos 标准一致性。

### 核心信息

| 属性 | 值 |
|------|-----|
| **上游版本** | vulkan-cts-1.3.7.3 |
| **上游地址** | https://github.com/KhronosGroup/VK-GL-CTS |
| **许可证** | Apache-2.0 |
| **OH 组件名** | @ohos/vk-gl-cts |

### OpenHarmony 适配亮点

不同于传统 Patch 修改方式，VK-GL-CTS 在 OpenHarmony 中采用了**平台层完全重写**的适配策略：

1. **无 Patch 文件** - 零侵入式适配，便于上游版本升级
2. **全新平台实现** - `framework/platform/ohos/` 目录下实现 OHOS 专用平台层
3. **Vulkan 扩展** - 添加 OHOS 特有的 Vulkan 扩展支持 (`VK_OpenHarmony_*`)
4. **Rosen 集成** - 深度集成 OpenHarmony Rosen 图形框架

### 在 OH 中的作用

- **GPU 驱动一致性验证** - 确保设备 GPU 符合 Khronos 标准
- **XTS 测试集成** - 自动化图形测试的核心组件
- **图形质量保障** - 覆盖 OpenGL ES 2.0/3.0/3.1/3.2 和 Vulkan

## 文档导航

| 文档 | 内容 | 建议阅读顺序 |
|------|------|--------------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 | 1 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（无 Patch 的特殊情况） | 2 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配详解 | 3 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | 4 |
| [05_API_Differences.md](./05_API_Differences.md) | OH 特有 Vulkan 扩展 | 5 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 6 |

## 快速参考

### 主要 BUILD.gn 目标

```gn
# 主入口
deqp - 包含所有测试模块的 group

# 可执行文件
glcts - OpenGL ES 测试可执行文件
glcts_app - App 模式测试
glcts_app_mock - Mock 测试

# 平台库
libdeqp_ohos_platform - OHOS 平台实现
rosen_context - Rosen 图形框架集成
```

### 关键目录

```
third_party/vk-gl-cts/
├── framework/platform/ohos/      # OHOS 平台适配（核心适配代码）
├── external/vulkancts/           # Vulkan CTS 测试
├── external/openglcts/           # OpenGL CTS 测试
├── modules/                      # 测试模块
├── build/                        # 构建输出（含 Vulkan 生成代码）
└── wiki/                         # 本文档
```

## 注意事项

1. **Rosen 依赖**：该库高度依赖 OHOS Rosen 图形框架的内部接口
2. **无传统 Patch**：适配通过新增平台层实现，而非修改上游代码
3. **测试数据**：运行测试需要配套的数据文件，详见 `04_Usage_in_OH.md`

## 参考链接

- [Khronos VK-GL-CTS 官方 Wiki](https://github.com/KhronosGroup/VK-GL-CTS/wiki)
- [OpenHarmony 图形子系统文档](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/graphics)
