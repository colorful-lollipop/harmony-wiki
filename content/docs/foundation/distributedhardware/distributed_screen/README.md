# OpenHarmony 分布式屏幕 (distributed_screen) 工程Wiki

## 简介

本Wiki为OpenHarmony分布式屏幕组件的工程文档，涵盖架构设计、接口定义、构建系统、安全风险等方面，旨在帮助开发者快速理解项目结构和实现细节。

## 覆盖范围

- **项目定位**: 分布式硬件子系统 - 屏幕虚拟化能力
- **核心功能**: 系统投屏、屏幕镜像、屏幕分割
- **实现语言**: C++
- **架构模式**: SystemAbility + IPC Proxy-Stub

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── 00_Overview.md         # 项目概览
├── 01_Architecture.md     # 架构设计
├── 02_Directory_Structure.md  # 目录结构
├── 03_Interfaces.md       # 对外接口 (Native SDK)
├── 04_Inner_APIs.md       # 内部接口
├── 05_Build_System.md     # GN构建系统
├── 06_Products.md         # 编译产物
├── 07_Security.md         # 安全风险评审
├── 08_Troubleshooting.md  # 常见问题
└── appendix/
    ├── Callgraphs.md      # 关键调用链
    └── Config_Flags.md    # 配置参数
```

## 更新方式

本文档基于代码自动生成，最后更新时间：**2025-02-06**

如需更新文档：
1. 修改 `/foundation/distributedhardware/distributed_screen/wiki/` 目录下的Markdown文件
2. 同步更新 `SUMMARY.md` 中的导航链接

## 快速开始

### 新人学习路线
👉 [项目概览](00_Overview.md) → [架构设计](01_Architecture.md) → [目录结构](02_Directory_Structure.md) → [对外接口](03_Interfaces.md)

### 安全研究路线
👉 [安全风险](07_Security.md) - 包含攻击面分析、可利用点详情、修复建议

**关键安全发现**:
- ⚠️ **高危**: ConfigDistributedHardware 接口缺少权限检查 (`dscreen_source_stub.cpp:160`)
- ⚠️ **高危**: 多处内存分配未使用 nothrow (`image_sink_decoder.cpp:88`)
- ✅ 编译安全选项完整 (CFI/UBSan/边界检查/栈保护)

---

## 注意事项

- 本文档不包含测试代码分析
- 所有关键结论均基于代码证据（文件路径+行号）
- 未发现N-API接口，项目为纯Native C++实现
- **最后更新**: 2026-02-07 (新增安全风险评估)

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony)
- [分布式硬件子系统](https://gitee.com/openharmony/distributedhardware_distributed_hardware_fwk)
- [项目源码](https://gitee.com/openharmony/distributedhardware_distributed_screen)

---

*本Wiki遵循 Apache License 2.0 开源协议*