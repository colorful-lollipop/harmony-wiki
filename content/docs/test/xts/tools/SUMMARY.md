# SUMMARY.md

本文档提供 XTS Tools Wiki 的导航与新人阅读顺序。所有链接指向内部页面。

---

## 新人快速入门（推荐阅读顺序）

1. [00_Overview.md](00_Overview.md) — 项目定位、边界、核心能力、运行环境、关键概念
2. [01_Directory_Structure.md](01_Directory_Structure.md) — 目录结构与模块职责（不含测试）
3. [02_Architecture.md](02_Architecture.md) — 架构说明：组件图、数据流、线程模型、关键时序
4. [03_NAPI_Surfaces.md](03_NAPI_Surfaces.md) — 对外 N-API 清单（如有）
5. [04_Inner_APIs.md](04_Inner_APIs.md) — 内部 API：模块接口、依赖方向、稳定性、可替换点
6. [05_GN_Targets.md](05_GN_Targets.md) — GN 目标梳理：target 列表、类型、依赖、产物、开关
7. [06_Build_Artifacts.md](06_Build_Artifacts.md) — 编译产物：.so/.a/.hap/可执行文件；安装路径；运行时加载关系
8. [07_Security_Review.md](07_Security_Review.md) — 安全风险评审：攻击面、信任边界、可被利用点、修复建议
9. [08_Common_Troubleshooting.md](08_Common_Troubleshooting.md) — 常见构建/运行/调试问题与定位路径

## 附录（可选）

- [appendix/Callgraphs.md](appendix/Callgraphs.md) — 关键调用链（入口→核心逻辑）
- [appendix/Config_Flags.md](appendix/Config_Flags.md) — 关键宏/feature flags

---

## 文档约定

- 术语统一：所有名词在各章节保持一致（见 00_Overview.md）。
- 证据链：关键结论附带路径、符号名或行号。
- 不引用测试：不包含 test/tests/unittest/fuzz 等测试相关内容作为业务证据。

---

## 反馈与维护

如发现不一致或缺失，请优先在对应页面补充证据（路径/符号/代码片段），并更新 SUMMARY 链接。
