# SUMMARY - auth_widget Wiki 导航

## 阅读路线图

### 新人入门（建议顺序）

1. **[概览](00_Overview.md)** - 了解项目定位、核心能力、运行环境
2. **[架构说明](01_Architecture.md)** - 理解组件关系、数据流、线程模型
3. **[认证框架 API](02_UserAuth_API.md)** - 掌握与 user_auth_framework 的集成方式
4. **[构建配置](10_GN_Build.md)** - 了解编译流程与产物

### 进阶主题

5. **[模块配置](11_Module_Config.md)** - 深入理解 bundle/module 配置
6. **[组件 API](03_Components_API.md)** - 内部组件接口详情
7. **[安全风险评审](20_Security_Review.md)** - 安全分析与建议

### 参考资料

8. **[常见问题](90_FAQ.md)** - 构建/运行/调试问题汇总
9. **[术语表](91_Glossary.md)** - 关键术语中英文对照

## 文档索引

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| README | Wiki 使用指南 | 更新方式、生成时间 |
| 00_Overview | 项目概览 | 定位、能力、运行环境 |
| 01_Architecture | 架构说明 | 组件图、数据流、时序 |
| 02_UserAuth_API | 对外 API | user_auth_framework 集成 |
| 03_Components_API | 内部 API | 组件接口、工具类 |
| 10_GN_Build | 构建配置 | targets、产物、依赖 |
| 11_Module_Config | 模块配置 | bundle/module.json |
| 20_Security_Review | 安全评审 | 攻击面、风险、建议 |
| 90_FAQ | 常见问题 | 问题定位、解决方案 |
| 91_Glossary | 术语表 | 术语解释 |

## 代码证据索引

所有关键结论均可追溯到源码证据：

- **UserAuthAbility**: `entry/src/main/ets/extensionability/UserAuthAbility.ts`
- **Index 页面**: `entry/src/main/ets/pages/Index.ets`
- **认证工具**: `entry/src/main/ets/common/utils/AuthUtils.ts`
- **常量定义**: `entry/src/main/ets/common/vm/Constants.ts`
- **构建配置**: `BUILD.gn`
- **模块配置**: `bundle.json`, `module.json`
