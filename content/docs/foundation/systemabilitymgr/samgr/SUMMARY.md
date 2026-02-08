# SUMMARY - Samgr Wiki 导航

## 新人阅读路线

推荐按以下顺序阅读，快速掌握 Samgr 项目：

### 第一步：项目概览（5分钟）
1. [项目概览](00_Overview.md) - 了解项目定位、核心能力和运行环境

### 第二步：代码结构（10分钟）
2. [目录结构](01_Directory_Structure.md) - 理解代码组织和模块职责

### 第三步：架构理解（20分钟）
3. [架构设计](02_Architecture.md) - 掌握组件关系、数据流和线程模型

### 第四步：接口学习（30分钟）
4. [对外 API](03_Public_API.md) - 学习 C++ 接口和 Rust 绑定

### 第五步：深度开发（按需阅读）
5. [内部实现](04_Internal_Architecture.md) - 核心模块实现细节
6. [GN 构建](05_GN_Targets.md) - 构建系统和配置选项
7. [编译产物](06_Build_Artifacts.md) - 输出文件和运行时加载

### 第六步：安全和排错（参考查阅）
8. [安全分析](07_Security_Analysis.md) - 安全风险分析和修复建议
9. [问题排查](08_Troubleshooting.md) - 常见问题解决方案

---

## 快速参考

### 关键文件索引

| 类别 | 文件路径 | 说明 |
|------|----------|------|
| 主接口 | `interfaces/innerkits/samgr_proxy/include/if_system_ability_manager.h` | ISystemAbilityManager 定义 |
| SA ID 定义 | `interfaces/innerkits/samgr_proxy/include/system_ability_definition.h` | 所有 SA ID 常量 |
| 错误码 | `interfaces/innerkits/common/include/samgr_err_code.h` | 错误码定义 |
| 主实现 | `services/samgr/native/include/system_ability_manager.h` | SystemAbilityManager 类 |
| 组件配置 | `bundle.json` | 组件描述和构建配置 |

### 构建目标索引

| 目标 | 类型 | 输出 | 路径 |
|------|------|------|------|
| samgr | executable | samgr | `services/samgr/native/BUILD.gn` |
| samgr_proxy | shared_library | libsamgr_proxy.so | `interfaces/innerkits/samgr_proxy/BUILD.gn` |
| samgr_common | source_set | - | `interfaces/innerkits/common/BUILD.gn` |
| dynamic_cache | source_set | - | `interfaces/innerkits/dynamic_cache/BUILD.gn` |

---

## 附录

- [调用链分析](appendix/Callgraphs.md)
- [配置选项](appendix/Config_Flags.md)

---

*最后更新: 2025-02-06*
