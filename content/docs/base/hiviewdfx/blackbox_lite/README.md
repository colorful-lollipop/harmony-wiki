# blackbox_lite Wiki

## 文档覆盖范围

本文档是 OpenHarmony DFX 子系统 `blackbox_lite` 模块的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 接口及安全风险。

### 已覆盖内容

| 文档 | 内容概要 |
|------|----------|
| [README](README.md) | 本文档，覆盖范围与更新说明 |
| [SUMMARY](SUMMARY.md) | 全站导航与新人阅读路线 |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [03_API_Reference](03_API_Reference.md) | 内部 C API 接口清单与调用链 |
| [04_Build_Configuration](04_Build_Configuration.md) | GN 构建配置与编译产物 |
| [05_Security_Review](05_Security_Review.md) | 安全风险评审与威胁模型 |
| [06_Troubleshooting](06_Troubleshooting.md) | 常见构建/运行/调试问题 |

### 未覆盖内容

- **N-API / JS API**：本模块为纯 C 内核态实现，不提供 JS 接口
- **IPC/RPC 机制**：模块运行于内核态，不涉及进程间通信
- **权限系统**：内核态模块不适用用户态权限模型
- **测试用例引用**：本文档不引用测试代码作为业务证据

## 代码证据原则

本文档所有关键结论均可追溯到源代码证据：

- **文件路径**：包含必要的行号引用
- **符号名称**：函数、宏、结构体均有路径标注
- **调用链描述**：关键逻辑提供文字版调用链

示例格式：
> 初始化函数在 `blackbox_core.c:318` 定义，使用 `CORE_INIT_PRI` 宏注册

## 更新方式

当代码发生变更时：

1. 更新对应模块的 `NOTES.md` 事实记录
2. 同步修改相关 Wiki 章节
3. 更新 `SUMMARY.md` 的章节链接
4. 更新本文档的"最后更新"时间戳

## 相关资源

- **源代码**：`//base/hiviewdfx/blackbox_lite/`
- **官方 README**：[README_zh.md](../../README_zh.md)
- **组件配置**：[bundle.json](../../bundle.json)
- **构建配置**：[BUILD.gn](../../BUILD.gn)

## 贡献指南

如需修改本文档：

1. 确保修改符合[证据原则](#代码证据原则)
2. 保持术语一致性（见各文档术语表）
3. 更新 SUMMARY.md 的导航链接
4. 不添加测试代码引用

---

*最后更新：2024-02 基于代码版本 3.1*
