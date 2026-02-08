# i18n 模块 Wiki 文档

## 覆盖范围

本 Wiki 覆盖 OpenHarmony `base/global/i18n` 模块的完整技术文档，包括：

- **对外 API**: JS N-API ( `@ohos/i18n`, `@ohos/intl` ), NDK, ETS/ANI
- **内部架构**: 核心框架模块 (`frameworks/intl`, `frameworks/zone`), SA 服务
- **构建系统**: GN 构建配置, 编译产物, 依赖关系
- **安全评审**: 攻击面分析, 权限检查, 信任边界, 风险评估

**不包含**:
- 测试代码 (`test/`, `*_test.*`, `fuzztest/`)
- 第三方库内部实现 (ICU 等)

## 文档结构

### 核心文档

| 文档 | 说明 | 目标读者 |
|------|------|---------|
| [概览](index.md) | 项目定位、核心能力、快速示例 | 所有人 |
| [代码地图](03_CodeMap.md) | 功能-文件映射表、核心路径速查 | 开发者 |
| [架构说明](Architecture.md) | 三层架构、调用链、线程模型 | 架构师 |
| [N-API 接口](N-API.md) | JS API 清单、参数说明 | 应用开发者 |
| [攻击面分析](05_AttackSurface.md) | 输入入口、敏感操作、信任边界 | 安全研究员 |
| [安全风险评估](06_SecurityReview.md) | 漏洞分析、修复建议 | 安全研究员 |
| [构建与产物](Build.md) | GN targets、依赖关系、产物清单 | 构建工程师 |
| [内部实现细节](08_Internals.md) | 核心类职责、生命周期、API 契约 | 开发者 |

### 附录

- [关键调用链](appendix/Callgraphs.md) - 序列图形式的调用流程
- [配置选项](appendix/Config_Flags.md) - Feature 开关和宏定义

### 导航

- [SUMMARY.md](SUMMARY.md) - 完整文档导航，含新人/安全双路线

## 受众路线

### 新人学习路线 ⭐

建议阅读顺序：
1. [概览](index.md) - 了解项目定位
2. [代码地图](03_CodeMap.md) - 快速定位代码
3. [架构说明](Architecture.md) - 理解架构设计
4. [N-API 接口](N-API.md) - 掌握 API
5. [构建与产物](Build.md) - 了解编译

⏱️ 预计时间: 30-45 分钟

### 安全研究路线 🔒

建议阅读顺序：
1. [概览](index.md) - 了解基本定位
2. [架构说明](Architecture.md) - 理解信任边界
3. [攻击面分析](05_AttackSurface.md) - 识别攻击入口
4. [安全风险评估](06_SecurityReview.md) - 深入漏洞分析
5. [代码地图](03_CodeMap.md) - 定位关键代码

⏱️ 预计时间: 20-30 分钟

## 更新方式

当代码发生变更时，需同步更新对应文档：

1. **API 变更**: 更新 `N-API.md` 中的 API 清单表
2. **架构变更**: 更新 `Architecture.md` 和调用链图
3. **构建变更**: 更新 `Build.md` 中的 targets 和产物
4. **权限变更**: 更新 `05_AttackSurface.md` 和 `06_SecurityReview.md`

建议使用代码搜索验证文档准确性：
- N-API 注册点: `NAPI_MODULE`, `napi_define_properties`
- SA 服务: `REGISTER_SYSTEM_ABILITY_BY_ID(I18nServiceAbility`
- 构建目标: `BUILD.gn`, `.gni`

## 质量标准

- ✅ 所有技术结论都有代码路径支撑
- ✅ 每个风险都有具体行号定位
- ✅ 文档间交叉引用有效
- ✅ 支持新人/安全双路线阅读

## 生成信息

- **生成时间**: 2026-02-07
- **版本**: OpenHarmony i18n 1.0.0
- **维护者**: i18n 模块代码所有者

## 相关链接

- [OpenHarmony i18n 源码](https://gitee.com/openharmony/global_i18n)
- [API 参考](https://developer.harmonyos.com)
- [SystemCapability.Global.I18n](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-i18n.md)
