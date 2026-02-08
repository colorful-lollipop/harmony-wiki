# SUMMARY - 阅读路线建议

## 按角色阅读

### 我是第一次接触这个库

1. **必读**：[01_Overview.md](01_Overview.md)
   - 了解 OpenMAX IL 是什么
   - 理解它在 OH 中的作用

2. **推荐**：[04_Usage_in_OH.md](04_Usage_in_OH.md)
   - 看看谁在用这个库
   - 理解它在系统中的位置

### 我是多媒体子系统开发者

1. **必读**：[05_API_Differences.md](05_API_Differences.md)
   - OH 扩展 API 详细参考
   - `codec_omx_ext.h` 使用指南

2. **参考**：[03_Build_Integration.md](03_Build_Integration.md)
   - 如何在 BUILD.gn 中引用
   - 头文件路径配置

3. **了解**：[04_Usage_in_OH.md](04_Usage_in_OH.md)
   - av_codec 和 media_foundation 的使用模式

### 我是芯片厂商/驱动开发者

1. **必读**：[05_API_Differences.md](05_API_Differences.md)
   - 必须实现的 OH 扩展接口
   - Buffer 类型支持要求

2. **必读**：[04_Usage_in_OH.md](04_Usage_in_OH.md)
   - 理解 OMX IL 实现在 OH 架构中的位置
   - 参考 Rockchip 实现案例

3. **参考**：[02_Patches.md](02_Patches.md)
   - 了解为什么不要修改上游头文件

### 我是维护者/升级负责人

1. **必读**：[02_Patches.md](02_Patches.md)
   - Patch 策略说明（本库无 Patch）
   - 升级注意事项

2. **必读**：[06_Security.md](06_Security.md)
   - 安全风险评估
   - 维护建议

3. **参考**：[_work/ASSESSMENT.md](_work/ASSESSMENT.md)
   - 完整的项目评估信息

### 我是架构师/技术管理者

1. **速览**：[README.md](README.md) - 库概览
2. **重点**：[04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系图
3. **评估**：[06_Security.md](06_Security.md) - 风险与建议

## 按场景阅读

### 场景：添加新的编解码器格式支持

1. [05_API_Differences.md](05_API_Differences.md) - 在 `codec_omx_ext.h` 中添加新的格式定义
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 更新相关模块引用
3. 参考 av_codec 中的使用模式

### 场景：升级 OpenMAX IL 版本

1. [02_Patches.md](02_Patches.md) - 确认无 Patch 需要迁移
2. [05_API_Differences.md](05_API_Differences.md) - 检查扩展 API 兼容性
3. [06_Security.md](06_Security.md) - 评估版本升级的安全影响

### 场景：移植到新芯片平台

1. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解 Rockchip 实现架构
2. [05_API_Differences.md](05_API_Differences.md) - 实现必需的 OH 扩展接口
3. [03_Build_Integration.md](03_Build_Integration.md) - 配置 BUILD.gn 依赖

## 文档依赖关系图

```
                    README.md (入口)
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   01_Overview    04_Usage_in_OH    03_Build_Integration
        │                │                │
        └────────────────┼────────────────┘
                         │
              05_API_Differences (核心扩展)
                         │
        ┌────────────────┴────────────────┐
        │                                 │
   02_Patches                       06_Security
   (维护策略)                        (风险评估)
```

## 阅读时间估算

| 文档 | 深度阅读 | 快速浏览 |
|------|---------|---------|
| README.md | 5 min | 2 min |
| 01_Overview.md | 10 min | 3 min |
| 02_Patches.md | 5 min | 2 min |
| 03_Build_Integration.md | 10 min | 3 min |
| 04_Usage_in_OH.md | 15 min | 5 min |
| 05_API_Differences.md | 20 min | 5 min |
| 06_Security.md | 10 min | 3 min |
| **总计** | **~75 min** | **~23 min** |
