# Communication Sample Wiki - 目录导航

> 新人建议按推荐顺序阅读本文档

---

## 新人阅读路线（推荐顺序）

1. **[项目概览](index.md)** - 了解项目整体定位和目标
2. **[项目定位与边界](01_Project_Position.md)** - 明确项目边界和核心能力
3. **[目录结构与模块职责](02_Directory_Structure.md)** - 理解代码组织方式
4. **[架构说明](03_Architecture.md)** - 掌握组件交互和数据流
5. **[GN Targets](06_GN_Targets.md)** - 了解构建系统和依赖关系
6. **[编译产物](07_Build_Artifacts.md)** - 理解输出和部署
7. **[安全风险评审](08_Security_Audit.md)** - 了解安全考虑和风险

如遇问题，参考：
- **[常见问题与调试](09_Troubleshooting.md)** - 常见问题和定位方法

---

## 文档索引

### 概览类

| 文档 | 说明 |
|------|------|
| [index.md](index.md) | 项目概览与快速入门 |
| [01_Project_Position.md](01_Project_Position.md) | 项目定位、边界、核心能力、运行环境、关键概念 |

### 架构类

| 文档 | 说明 |
|------|------|
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 |
| [03_Architecture.md](03_Architecture.md) | 架构说明：组件图 / 数据流 / 线程模型 / 关键时序 |

### API 类

| 文档 | 说明 |
|------|------|
| [04_External_API.md](04_External_API.md) | 对外 API：N-API、导出符号、权限/参数/错误码 |
| [05_Internal_API.md](05_Internal_API.md) | 内部 API：模块接口、依赖方向、稳定性、可替换点 |

### 构建类

| 文档 | 说明 |
|------|------|
| [06_GN_Targets.md](06_GN_Targets.md) | GN 目标梳理：targets 列表、类型、依赖、产物、开关 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物：.so/.a/.hap/可执行文件等；安装路径；运行时加载关系 |

### 安全类

| 文档 | 说明 |
|------|------|
| [08_Security_Audit.md](08_Security_Audit.md) | 安全风险评审：攻击面、信任边界、可被利用点、修复建议 |

### 实用类

| 文档 | 说明 |
|------|------|
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见构建/运行/调试问题与定位路径 |

---

## 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏/feature flags |

---

## 文档约定

### 代码引用格式

本文档中的代码引用遵循以下格式：

```
文件路径:行号
```

例如：
- `hostapd/src/hostapd_sample.c:56` - 表示 hostapd_sample.c 第 56 行
- `BUILD.gn:16` - 表示根目录 BUILD.gn 第 16 行

### 证据标记

- ✅ 已有代码证据
- ⚠️ 需进一步确认
- 📝 代码中未发现（不适用）

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本，基于 v3.1 代码生成 |
