# mtdev OpenHarmony 集成文档

> **快速导航**: [阅读路线建议](SUMMARY.md) | [评估报告](wiki/_work/ASSESSMENT.md)

---

## 概述

本文档记录了 `mtdev` 库在 OpenHarmony 中的集成与适配情况，重点说明 OpenHarmony 对该库的 Patch、特殊适配、以及在系统中的使用方式。

### 库简介

**mtdev**（Multitouch Protocol Translation Library）是一个用于将 Linux 内核各种多点触控（MT）事件转换为统一的 Type B 协议的库。

- **上游版本**: 1.1.7
- **OH 组件**: @ohos/mtdev
- **所属子系统**: multimodalinput (多模态输入)
- **许可证**: MIT License

### OH 集成特点

| 特点 | 说明 |
|-----|------|
| **Patch 机制** | 通过 1 个 Patch 文件修改核心事件处理逻辑 |
| **强制 X/Y 事件** | 禁用去重，强制发送所有 X/Y 坐标事件 |
| **禁用数据过滤** | 跳过 EWMA 滤波，保持原始触摸精度 |
| **GN 构建集成** | 完整的 GN 构建系统适配 |

---

## 文档导航

### 核心文档

| 文档 | 内容 | 适合读者 |
|-----|------|---------|
| [01_Overview.md](01_Overview.md) | 原始库简介、OH 定位、快速开始 | 所有人 |
| [02_Patches.md](02_Patches.md) | **Patch 详细分析**（核心文档） | 开发者、维护者 |
| [03_Build_Integration.md](03_Build_Integration.md) | GN 构建系统、Patch 应用机制 | 构建工程师 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系、使用场景、运行时行为 | 架构师、开发者 |

### 工作文档

| 文档 | 内容 |
|-----|------|
| [wiki/_work/ASSESSMENT.md](wiki/_work/ASSESSMENT.md) | 项目评估结果（Phase 0） |
| [wiki/_work/NOTES.md](wiki/_work/NOTES.md) | 分析过程记录 |
| [wiki/_work/PLAN.md](wiki/_work/PLAN.md) | 任务进度跟踪 |

---

## 快速开始

### 我需要了解 mtdev 是什么？

→ 阅读 [01_Overview.md](01_Overview.md)

### 我需要知道 OH 对 mtdev 做了哪些修改？

→ 阅读 [02_Patches.md](02_Patches.md)（核心文档）

### 我需要理解 OH 的构建流程？

→ 阅读 [03_Build_Integration.md](03_Build_Integration.md)

### 我需要知道 mtdev 在 OH 中如何被使用？

→ 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md)

### 我需要全面评估 OH 集成情况？

→ 阅读 [wiki/_work/ASSESSMENT.md](wiki/_work/ASSESSMENT.md)

---

## 关键信息速查

### OH 特定修改总结

| 修改类型 | 文件 | 关键内容 |
|---------|------|---------|
| Patch | `patch/diff_libmtdev_mmi/mtdev/mtdev_0000.diff` | 强制发送 X/Y 事件，禁用数据过滤 |
| 编译选项 | `BUILD.gn` | `-DDISABLE_FILTER` 宏 |
| 构建集成 | `patch/apply_patch.sh` | Patch 应用脚本 |

### 依赖关系

```
mtdev (libmtdev-third-mmi)
  └── libinput (libinput-third-mmi)
       └── MMI 服务 (multimodalinput/input)
            └── 270+ 子模块
```

### 资源占用

| 资源 | 占用 |
|-----|------|
| ROM | 400KB |
| RAM | 800KB |

---

## 维护建议

### 升级上游版本

1. 检查 `mtdev-1.1.7/` 源码版本
2. 手动验证 `mtdev_0000.diff` 是否兼容
3. 如不兼容，更新 Patch 文件
4. 参考 [02_Patches.md](02_Patches.md) 的"回归风险"章节

### 性能监控

- 监控触摸事件处理频率
- 测量 CPU 占用率
- 检查事件延迟

详见 [04_Usage_in_OH.md](04_Usage_in_OH.md) 的"性能影响"章节。

---

## 参考资源

### 官方文档

- [mtdev 官方文档](http://bitmath.org/code/mtdev/)
- [mtdev 上游仓库](http://bitmath.org/git/mtdev)
- [Linux MT 协议规范](https://www.kernel.org/doc/html/latest/input/multi-touch-protocol.html)

### OpenHarmony 相关

- [libinput 集成文档](../libinput/)
- [MMI 服务文档](https://gitee.com/openharmony/multimodalinput_input)

---

## 贡献指南

### 如何更新文档

1. 修改对应的 `.md` 文件
2. 确保引用的代码路径正确
3. 更新 `wiki/_work/ASSESSMENT.md` 中的相关部分
4. 提交 Pull Request

### 如何新增 Patch 说明

1. 在 `02_Patches.md` 中添加 Patch 分析
2. 更新 `ASSESSMENT.md` 的 Patch 统计
3. 如影响构建，更新 `03_Build_Integration.md`

---

## 联系方式

**组件维护者**: gaoshangqi1@huawei.com

**反馈渠道**: [OpenHarmony Gitee Issue](https://gitee.com/openharmony/third_party_mtdev/issues)

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
