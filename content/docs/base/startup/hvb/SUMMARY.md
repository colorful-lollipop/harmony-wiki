# HVB Wiki 全站导航

## 新人推荐阅读顺序

1. **[项目概览](00_Overview.md)** - 先了解 HVB 是什么，解决什么问题
2. **[项目定位与边界](01_Project_Scope.md)** - 了解核心能力、运行环境
3. **[目录结构](02_Directory_Structure.md)** - 熟悉代码组织方式
4. **[架构说明](03_Architecture.md)** - 理解组件关系和数据流
5. **[对外 API](04_Public_API.md)** - 学习如何调用 libhvb（Bootloader/init 开发者）
6. **[内部 API](05_Internal_API.md)** - 深入理解模块实现（贡献者）
7. **[GN Targets](06_GN_Targets.md)** - 了解构建系统（构建工程师）
8. **[编译产物](07_Build_Artifacts.md)** - 知道输出什么、在哪里
9. **[安全分析](08_Security_Analysis.md)** - 识别潜在风险和安全最佳实践
10. **[常见问题](09_FAQ.md)** - 快速定位和解决问题

---

## 文档索引

### 核心文档
- [README](README.md) - 文档覆盖范围、更新方式
- [项目概览](00_Overview.md) - HVB 组件介绍
- [项目定位与边界](01_Project_Scope.md) - 能力范围和约束
- [目录结构](02_Directory_Structure.md) - 代码组织
- [架构说明](03_Architecture.md) - 系统架构与数据流

### API 文档
- [对外 API](04_Public_API.md) - C 接口清单与使用指南
- [内部 API](05_Internal_API.md) - 模块接口与依赖关系

### 构建文档
- [GN Targets](06_GN_Targets.md) - GN 构建配置详解
- [编译产物](07_Build_Artifacts.md) - 输出文件与集成方式

### 安全与运维
- [安全分析](08_Security_Analysis.md) - 攻击面与风险评估
- [常见问题](09_FAQ.md) - 构建、运行、调试问题

### 附录
- [调用链分析](appendix/Callgraphs.md) - 关键调用链
- [配置选项](appendix/Config_Flags.md) - 关键宏与编译选项

---

## 按角色查找文档

### Bootloader 开发者
- [对外 API](04_Public_API.md) - 如何集成 libhvb
- [项目定位与边界](01_Project_Scope.md) - Bootloader 适配要求
- [常见问题](09_FAQ.md) - 平台适配问题

### Init 开发者
- [对外 API](04_Public_API.md) - 如何在 init 中使能 dm-verity
- [架构说明](03_Architecture.md) - init → libhvb → 内核数据流
- [常见问题](09_FAQ.md) - fstab 配置问题

### 构建工程师
- [GN Targets](06_GN_Targets.md) - 如何集成 hvbtool
- [编译产物](07_Build_Artifacts.md) - 构建输出说明
- [项目定位与边界](01_Project_Scope.md) - 适配系统类型要求

### 安全审计人员
- [安全分析](08_Security_Analysis.md) - 威胁模型与风险评估
- [架构说明](03_Architecture.md) - 信任边界与数据流
- [对外 API](04_Public_API.md) - 接口安全约束

### 代码贡献者
- [目录结构](02_Directory_Structure.md) - 模块划分
- [内部 API](05_Internal_API.md) - 模块接口与依赖
- [架构说明](03_Architecture.md) - 组件设计原则

---

## 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 安全启动 | Verified Boot | 确保系统镜像来源合法、未被篡改、无法回滚 |
| 镜像校验 | Image Verification | 验证镜像完整性和签名 |
| 整包校验 | Hash Verification | 对整个镜像计算 hash 值并校验 |
| 按需校验 | Hashtree Verification | 通过哈希树按需校验数据块 |
| 根校验表 | Root Verity Table (RVT) | 存储多个镜像的公钥和校验信息 |
| Verity Footer | Verity Footer | 镜像末尾的校验信息（签名、哈希、公钥） |
| 防回滚 | Anti-Rollback | 防止设备回滚到有漏洞的历史版本 |
| Bootloader | Bootloader | 引导加载程序，设备上电后第一个运行的程序 |
| dm-verity | dm-verity | Linux 内核的设备映射器完整性校验模块 |

---

*最后更新: 2026-02-06*
