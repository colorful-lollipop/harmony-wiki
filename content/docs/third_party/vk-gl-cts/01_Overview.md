# VK-GL-CTS 原始库简介与 OpenHarmony 定位

## 1. 原始库基本信息

### 1.1 库概述

**VK-GL-CTS** (Vulkan GL Conformance Test Suite) 是由 Khronos Group 维护的官方 GPU 一致性测试套件，起源于 dEQP (drawElements Quality Program)。

### 1.2 版本信息

| 属性 | 值 |
|------|-----|
| **当前版本** | vulkan-cts-1.3.7.3 |
| **上游地址** | https://github.com/KhronosGroup/VK-GL-CTS |
| **许可证** | Apache-2.0 |
| **维护组织** | Khronos Group |

### 1.3 支持的图形 API

VK-GL-CTS 包含以下图形 API 的一致性测试：

| API | 版本 | 测试目录 |
|-----|------|----------|
| **OpenGL** | 4.x | `external/openglcts/modules/gl/` |
| **OpenGL ES** | 2.0, 3.0, 3.1, 3.2 | `external/openglcts/modules/gles*/` |
| **EGL** | 1.x | `modules/egl/` |
| **Vulkan** | 1.0-1.3 | `external/vulkancts/` |
| **Vulkan SC** | 1.0 | `external/vulkancts/` (SC 变体) |

### 1.4 原始功能

VK-GL-CTS 提供以下测试类型：

1. **功能测试 (Functional Tests)**
   - 验证 API 功能正确性
   - 覆盖所有 API 入口点

2. **压力测试 (Stress Tests)**
   - 长时间运行测试
   - 资源耗尽测试
   - 多线程测试

3. **性能测试 (Performance Tests)**
   - 渲染性能基准
   - 带宽测试

4. **精确度测试 (Accuracy Tests)**
   - 浮点精度验证
   - 纹理采样精度
   - 着色器计算精度

## 2. OpenHarmony 中的定位

### 2.1 系统架构中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Applications)                    │
├─────────────────────────────────────────────────────────────┤
│                   ArkUI / 图形框架                             │
├─────────────────────────────────────────────────────────────┤
│                  Rosen 图形服务                                │
│         (Render Service / 合成 / 窗口管理)                    │
├─────────────────────────────────────────────────────────────┤
│                  GPU 驱动层                                    │
│    ┌──────────────┬──────────────┬──────────────┐           │
│    │ OpenGL ES    │   Vulkan     │   EGL        │           │
│    │ 驱动实现      │   驱动实现    │   实现       │           │
│    └──────────────┴──────────────┴──────────────┘           │
├─────────────────────────────────────────────────────────────┤
│                   内核 / DRM / GPU 硬件                       │
└─────────────────────────────────────────────────────────────┘
                              ↑
                    ┌─────────┴─────────┐
                    │   VK-GL-CTS       │
                    │  (一致性测试套件)  │
                    └───────────────────┘
```

### 2.2 在 OH 中的核心价值

#### 2.2.1 GPU 驱动一致性验证

VK-GL-CTS 是 OpenHarmony 验证 GPU 驱动正确性的核心工具：

- **合规性验证**：确保 GPU 驱动符合 Khronos 标准
- **回归测试**：驱动升级后验证兼容性
- **跨平台一致性**：不同 GPU 厂商实现的一致性验证

#### 2.2.2 XTS 自动化测试集成

作为 XTS (eXtended Test Suite) 的重要组成部分：

| 测试套件 | 路径 | 用途 |
|----------|------|------|
| gltest | `test/xts/acts/graphic/gltest/` | OpenGL ES 一致性测试 |
| vkgl | `test/xts/acts/graphic/vkgl/` | Vulkan/OpenGL 综合测试 |
| vktest | `test/xts/acts/graphic/vktest/` | Vulkan 专项测试 |

#### 2.2.3 图形栈质量保障

在 OpenHarmony 图形栈中的角色：

1. **驱动开发阶段**：GPU 厂商开发驱动时使用
2. **系统集成阶段**：集成到 OH 时验证兼容性
3. **发布测试阶段**：XTS 自动化测试执行
4. **持续集成**：CI/CD 流程中自动运行

### 2.3 使用场景

#### 场景 1：GPU 厂商驱动开发

GPU 厂商（如 Mali、Adreno、PowerVR）在移植驱动到 OpenHarmony 时：

```
1. 实现 OpenGL ES / Vulkan 驱动
2. 使用 VK-GL-CTS 验证实现正确性
3. 修复测试失败项
4. 达到 Khronos 一致性标准
```

#### 场景 2：OpenHarmony 设备认证

设备厂商在申请 OpenHarmony 认证时：

```
1. 运行 XTS 测试套件
2. 图形测试部分调用 VK-GL-CTS
3. 必须达到一定通过率才能获得认证
```

#### 场景 3：图形框架开发

Rosen 图形框架开发时：

```
1. 修改合成器/窗口管理器
2. 使用 VK-GL-CTS 验证渲染正确性
3. 确保不影响 GPU 驱动接口
```

## 3. OpenHarmony 特有适配

### 3.1 适配策略对比

| 方面 | 传统方式 | OpenHarmony 方式 |
|------|----------|------------------|
| **Patch 文件** | 使用 .patch 修改上游 | 无 Patch，平台层完全重写 |
| **平台抽象** | 修改现有平台代码 | 新增 `framework/platform/ohos/` |
| **构建系统** | CMake | GN (OpenHarmony 标准) |
| **系统集成** | 通用适配 | Rosen 图形框架深度集成 |

### 3.2 适配代码分布

| 目录 | 文件数 | 说明 |
|------|--------|------|
| `framework/platform/ohos/` | 30+ | OHOS 平台适配核心 |
| `build/external/vulkancts/` | 生成文件 | Vulkan 扩展代码 |

### 3.3 与上游的差异

OpenHarmony 版本与上游 VK-GL-CTS 的主要差异：

1. **新增平台实现**：完全独立的 OHOS 平台层
2. **Vulkan 扩展**：添加 `VK_OpenHarmony_*` 系列扩展
3. **构建系统**：从 CMake 迁移到 GN
4. **系统集成**：与 Rosen 框架深度耦合

## 4. 版本说明

### 4.1 当前版本

- **上游版本**：vulkan-cts-1.3.7.3
- **OH 版本**：3.2
- **最后更新**：需查看 commit 历史

### 4.2 升级建议

由于没有 Patch 文件，升级上游版本相对简单：

1. 替换上游源码
2. 验证 `framework/platform/ohos/` 兼容性
3. 检查 Vulkan 扩展定义是否变化
4. 重新运行测试验证

## 5. 参考文档

- [VK-GL-CTS 官方 Wiki](https://github.com/KhronosGroup/VK-GL-CTS/wiki)
- [Khronos 一致性测试说明](https://www.khronos.org/conformance/)
- [OpenHarmony 图形子系统架构](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/graphics)
