# 文档导航

## 推荐阅读路径

### 新人学习路线 ⭐

建议按以下顺序阅读，快速掌握 i18n 模块：

1. **[概览](index.md)** - 了解项目定位、核心能力、运行环境
2. **[代码地图](03_CodeMap.md)** - 快速定位功能对应的代码文件
3. **[架构说明](Architecture.md)** - 理解三层架构设计
4. **[N-API 接口](N-API.md)** - 掌握对外提供的 API
5. **[构建与产物](Build.md)** - 了解编译配置和产物

⏱️ **预计时间**: 30-45 分钟

### 安全研究路线 🔒

针对安全研究员的快速上手路径：

1. **[概览](index.md)** - 了解项目基本定位
2. **[架构说明](Architecture.md)** - 理解信任边界和数据流
3. **[攻击面分析](05_AttackSurface.md)** - 识别所有外部输入入口
4. **[安全风险评估](06_SecurityReview.md)** - 深入分析具体漏洞
5. **[代码地图](03_CodeMap.md)** - 定位关键代码位置

⏱️ **预计时间**: 20-30 分钟

---

## 完整文档目录

### 核心文档

| 文档 | 说明 | 适合读者 |
|------|------|---------|
| [概览](index.md) | 项目定位、能力边界、快速示例 | 所有人 |
| [代码地图](03_CodeMap.md) | 功能-文件映射表、核心路径速查 | 开发者 |
| [架构说明](Architecture.md) | 三层架构、调用链、线程模型 | 架构师 |
| [N-API 接口](N-API.md) | JS API 清单、参数说明 | 应用开发者 |
| [攻击面分析](05_AttackSurface.md) | 输入入口、敏感操作、信任边界 | 安全研究员 |
| [安全风险评估](06_SecurityReview.md) | 漏洞分析、修复建议 | 安全研究员 |
| [构建与产物](Build.md) | GN targets、依赖关系、产物清单 | 构建工程师 |

### 附录

| 文档 | 说明 |
|------|------|
| [关键调用链](appendix/Callgraphs.md) | 序列图形式的调用流程 |
| [配置选项](appendix/Config_Flags.md) | Feature 开关和宏定义 |

---

## 代码导航

### N-API 入口

| 模块 | 文件 | 注册函数 | 行号 |
|------|------|----------|------|
| `@ohos/i18n` | `interfaces/js/kits/src/i18n_addon.cpp` | `I18nAddon::Init` | 1900-1956 |
| `@ohos/intl` | `interfaces/js/innerkits/intl/src/intl_module.cpp` | `IntlAddon::Init` | 25-34 |

**N-API 注册点**:

```cpp
// interfaces/js/kits/src/i18n_addon.cpp:1970
extern "C" __attribute__((constructor)) void I18nRegister()
{
    napi_module_register(&g_i18nModule);
}
```

### SA 服务

| 服务 | 文件 | System Ability ID | 行号 |
|------|------|-------------------|------|
| I18nServiceAbility | `services/src/i18n_service_ability.cpp` | `I18N_SA_ID` (5296) | 35 |

**SA 注册点**:

```cpp
// services/src/i18n_service_ability.cpp:35
REGISTER_SYSTEM_ABILITY_BY_ID(I18nServiceAbility, I18N_SA_ID, false);
```

### 关键类

| 类名 | 头文件 | 实现文件 | 职责 |
|------|--------|---------|------|
| `DateTimeFormat` | `date_time_format.h` | `date_time_format.cpp` | 日期时间格式化 |
| `NumberFormat` | `number_format.h` | `number_format.cpp` | 数字格式化 |
| `LocaleConfig` | `locale_config.h` | `locale_config.cpp` | 系统区域配置 |
| `I18nServiceAbility` | `i18n_service_ability.h` | `i18n_service_ability.cpp` | 系统服务 |

---

## 快速参考

### 常用搜索模式

```bash
# 查找 N-API 函数实现
grep -r "napi_value.*GetSystemLocale" interfaces/js/

# 查找 IPC 接口
grep -r "SetSystemLanguage\|SetSystemRegion" services/

# 查找权限检查
grep -r "CheckPermission\|VerifyAccessToken" services/

# 查找 ICU 调用
grep -r "icu::" frameworks/intl/src/
```

### 关键常量

| 常量 | 值 | 位置 |
|------|-----|------|
| `I18N_SA_ID` | 5296 | `i18n_types.h` |
| `UPDATE_CONFIGURATION` | 权限名 | `i18n_service_ability.cpp` |

---

*导航更新时间: 2026-02-07*
