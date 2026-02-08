# UniProton 内核 Wiki 文档

## 文档概述

本文档是 **UniProton** 实时操作系统内核的工程 Wiki，旨在帮助开发者快速理解项目架构、API 接口、构建系统以及安全考量。

## 覆盖范围

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目定位与核心能力 | ✅ 已覆盖 | 详见 [概览](./01_Overview.md) |
| 目录结构与模块职责 | ✅ 已覆盖 | 详见 [目录结构](./02_Directory_Structure.md) |
| C 原生 API 参考 | ✅ 已覆盖 | 详见 [API 参考](./03_API_Reference.md) |
| 架构说明 | ✅ 已覆盖 | 详见 [架构设计](./04_Architecture.md) |
| GN 构建系统 | ✅ 已覆盖 | 详见 [构建系统](./05_Build_System.md) |
| 攻击面分析 | ✅ 已覆盖 | 详见 [攻击面分析](./05_AttackSurface.md) |
| 安全风险评审 | ✅ 已覆盖 | 详见 [安全评审](./06_Security_Review.md) |
| 常见问题 | ✅ 已覆盖 | 详见 [故障排查](./07_Troubleshooting.md) |

## 未覆盖范围

- **N-API / JavaScript 绑定**: UniProton 是裸机 RTOS 内核，无 JavaScript/Node.js 绑定层
- **OpenHarmony 上层框架**: 本仓库仅为内核实现，上层框架位于其他仓库
- **完整硬件驱动**: 仅提供 GIC 中断控制器驱动，设备驱动由 HDF 框架提供

## 关键结论

1. **UniProton 是纯 C 内核**: 无 C++ 异常支持，无 N-API，无 JavaScript 绑定
2. **IPC 为内核级**: 任务间通信通过信号量/队列/事件/读写锁实现，非跨进程 IPC
3. **无权限框架**: 单地址空间 RTOS，安全依赖硬件 (MMU/MPU)
4. **支持双架构**: ARMv7-M (Cortex-M4) 和 ARMv8 (A/M 系列)

## 文档更新

**生成时间**: 2026-02-06

**更新方式**:
```bash
# 重新生成 Wiki
cd /Volumes/lexar/code/d/work/oh/kernel/uniproton
python3 scripts/generate_wiki.py  # 如有脚本
```

或手动更新 `wiki/` 目录下的对应文档。

## 阅读建议

新手推荐阅读顺序：

1. [概览](./01_Overview.md) → 理解项目定位
2. [目录结构](./02_Directory_Structure.md) → 了解代码组织
3. [API 参考](./03_API_Reference.md) → 学习核心接口
4. [架构设计](./04_Architecture.md) → 深入实现原理
5. [构建系统](./05_Build_System.md) → 掌握编译流程

## 相关链接

- [UniProton 官方仓库](https://gitee.com/openeuler/UniProton)
- [OpenHarmony 文档](https://gitee.com/openharmony/docs)
- [GNU Arm Embedded Toolchain](https://developer.arm.com/downloads/-/gnu-rm)

---

*本文档基于 UniProton 源码自动生成*
