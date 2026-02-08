# mtdev 文档阅读路线建议

## 基于角色的阅读指南

### 如果你是... 开发者

**目标**: 理解 mtdev 在 OpenHarmony 中的实现和使用

**推荐路径**:
1. [01_Overview.md](01_Overview.md) - 了解 mtdev 是什么，OH 定位
2. [02_Patches.md](02_Patches.md) - 理解 OH 对上游的修改（重要！）
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解依赖关系和使用场景

**可选补充**:
- [03_Build_Integration.md](03_Build_Integration.md) - 如果需要修改构建逻辑

**预期时间**: 30-45 分钟

---

### 如果你是... 构建工程师

**目标**: 理解 OH 的构建流程和 Patch 机制

**推荐路径**:
1. [01_Overview.md](01_Overview.md) - 快速了解库的背景
2. [03_Build_Integration.md](03_Build_Integration.md) - 重点阅读构建流程图和 Patch 应用机制
3. [02_Patches.md](02_Patches.md) - 了解 Patch 内容，便于调试构建问题

**可选补充**:
- [wiki/_work/ASSESSMENT.md](wiki/_work/ASSESSMENT.md) - 评估报告中的构建配置部分

**预期时间**: 45-60 分钟

---

### 如果你是... 架构师 / 技术评审

**目标**: 全面了解 OH 集成方案，评估技术合理性

**推荐路径**:
1. [wiki/_work/ASSESSMENT.md](wiki/_work/ASSESSMENT.md) - 评估报告（核心文档）
2. [01_Overview.md](01_Overview.md) - 库的功能和定位
3. [02_Patches.md](02_Patches.md) - 修改的技术合理性分析
4. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖链和影响范围

**重点关注**:
- ASSESSMENT.md 中的"风险评估"和"建议后续工作"
- 02_Patches.md 中的"升级建议"
- 04_Usage_in_OH.md 中的"性能影响"

**预期时间**: 60-90 分钟

---

### 如果你是... 维护者

**目标**: 掌握 OH 集成的所有细节，便于维护和升级

**推荐路径**: **完整阅读**

1. [wiki/_work/ASSESSMENT.md](wiki/_work/ASSESSMENT.md) - 评估报告
2. [01_Overview.md](01_Overview.md) - 原始库信息
3. [02_Patches.md](02_Patches.md) - Patch 详细分析
4. [03_Build_Integration.md](03_Build_Integration.md) - 构建系统
5. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用情况和故障排查
6. [wiki/_work/NOTES.md](wiki/_work/NOTES.md) - 分析过程记录

**必读章节**:
- 02_Patches.md 的"回归风险"
- 03_Build_Integration.md 的"常见问题"
- 04_Usage_in_OH.md 的"故障排查"

**预期时间**: 2-3 小时

---

## 基于任务的阅读指南

### 任务 1: 升级 mtdev 上游版本

**阅读清单**:
- ✅ [02_Patches.md](02_Patches.md) - "回归风险"和"升级建议"
- ✅ [03_Build_Integration.md](03_Build_Integration.md) - "常见问题"
- ✅ [wiki/_work/ASSESSMENT.md](wiki/_work/ASSESSMENT.md) - "风险评估"

**关键步骤**:
1. 检查 Patch 兼容性
2. 验证 BUILD.gn 配置
3. 运行测试套件
4. 监控性能指标

---

### 任务 2: 修改 mtdev 行为

**阅读清单**:
- ✅ [02_Patches.md](02_Patches.md) - "修改点 1/2/3"和"修改目的分析"
- ✅ [03_Build_Integration.md](03_Build_Integration.md) - "编译选项"
- ✅ [04_Usage_in_OH.md](04_Usage_in_OH.md) - "OH 特定行为"

**关键步骤**:
1. 理解现有修改的原因
2. 评估修改的影响范围
3. 更新 Patch 或编译选项
4. 测试和验证

---

### 任务 3: 调试触摸事件问题

**阅读清单**:
- ✅ [01_Overview.md](01_Overview.md) - "快速开始"
- ✅ [04_Usage_in_OH.md](04_Usage_in_OH.md) - "调试和诊断"和"故障排查"

**关键步骤**:
1. 使用 libinput 工具查看事件流
2. 检查日志中的 mtdev 相关信息
3. 验证 Patch 是否生效

---

### 任务 4: 评估性能影响

**阅读清单**:
- ✅ [02_Patches.md](02_Patches.md) - "修改目的分析"
- ✅ [04_Usage_in_OH.md](04_Usage_in_OH.md) - "性能影响"

**关键指标**:
- CPU 占用率
- 事件量变化
- 触摸延迟

---

## 文档结构图

```
wiki/
├── README.md                          # 📋 入口和导航
├── SUMMARY.md                         # 📖 本文件 - 阅读路线
├── 01_Overview.md                    # 📚 库简介（必读）
├── 02_Patches.md                     # ⭐ Patch 分析（核心）
├── 03_Build_Integration.md           # 🔧 构建系统
├── 04_Usage_in_OH.md                # 🚀 使用和依赖
└── _work/
    ├── ASSESSMENT.md                 # 📊 评估报告
    ├── NOTES.md                      # 📝 分析记录
    └── PLAN.md                      # 📋 任务进度
```

---

## 重点关注事项

### ⭐ 核心 Patch 修改

**文件**: `patch/diff_libmtdev_mmi/mtdev/mtdev_0000.diff`

**关键点**:
1. 强制发送 X/Y 坐标事件（即使值不变）
2. 禁用数据过滤（通过 `-DDISABLE_FILTER` 宏）

**原因**: 详见 [02_Patches.md](02_Patches.md) 的"修改目的分析"

---

### 🏗️ 构建系统要点

**关键配置**:
- `libmtdev-third_config`: 编译选项（禁用过滤）
- `patch_gen_libmtdev-third-mmi`: Patch 后的源码集合
- `libmtdev-third-mmi`: 最终共享库

**流程**: 解压 → 配置 → Patch → 编译 → 共享库

详见 [03_Build_Integration.md](03_Build_Integration.md)

---

### 🚀 依赖关系

**直接依赖者**:
- `libinput:libinput-third-mmi`
- `multimodalinput/input:input-third-mmi`

**间接依赖者**: 270+ 个 MMI 子模块

详见 [04_Usage_in_OH.md](04_Usage_in_OH.md) 的"依赖关系图"

---

## 常见问题

### Q: 为什么 OH 修改了 mtdev 的行为？

**A**: 详见 [02_Patches.md](02_Patches.md) 的"修改目的分析"章节。主要原因包括：
- 确保触摸事件完整性
- 提高手势识别准确性
- 适配多种硬件驱动

---

### Q: 如何验证 Patch 是否生效？

**A**: 参考 [04_Usage_in_OH.md](04_Usage_in_OH.md) 的"验证 Patch 是否生效"章节：
1. 检查编译宏
2. 查看源码修改
3. 运行时日志检查

---

### Q: 升级上游版本需要注意什么？

**A**: 参考 [02_Patches.md](02_Patches.md) 的"回归风险"表格：
- 函数签名变化
- 代码重构
- 新增过滤逻辑
- 性能回归

---

### Q: 性能影响如何？

**A**: 详见 [04_Usage_in_OH.md](04_Usage_in_OH.md) 的"性能影响"章节：
- 事件量可能增加 2 倍
- CPU 占用增加约 10-20%
- 触摸延迟可能略微变化

---

## 学习资源

### 基础概念

| 概念 | 说明 | 文档 |
|-----|------|-----|
| MT 协议 | Linux 多点触控协议 | [01_Overview.md](01_Overview.md) |
| Type A/B | MT 协议的两种变体 | [01_Overview.md](01_Overview.md) |
| GN 构建系统 | OpenHarmony 构建工具 | [03_Build_Integration.md](03_Build_Integration.md) |
| MMI 服务 | 多模态输入服务 | [04_Usage_in_OH.md](04_Usage_in_OH.md) |

### 深入学习

| 主题 | 难度 | 文档 |
|-----|------|-----|
| Patch 机制 | 中等 | [02_Patches.md](02_Patches.md), [03_Build_Integration.md](03_Build_Integration.md) |
| 依赖管理 | 中等 | [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| 性能优化 | 高 | [02_Patches.md](02_Patches.md), [04_Usage_in_OH.md](04_Usage_in_OH.md) |

---

## 快速参考

### 常用命令

```bash
# 构建 mtdev
hb build mtdev

# 查看 Patch 是否生效
grep "DISABLE_FILTER" out/ohos-arm-release/gen/diff_libmtdev_mmi/src/core.c

# 运行 libinput 工具
./out/ohos-arm-release/bin/libinput-list-mmi --devices
```

### 关键文件

| 文件 | 说明 | 文档 |
|-----|------|-----|
| `BUILD.gn` | 主构建文件 | [03_Build_Integration.md](03_Build_Integration.md) |
| `patch/apply_patch.sh` | Patch 应用脚本 | [03_Build_Integration.md](03_Build_Integration.md) |
| `mtdev_0000.diff` | OH 特定 Patch | [02_Patches.md](02_Patches.md) |
| `bundle.json` | OH 组件定义 | [01_Overview.md](01_Overview.md) |

---

## 总结

| 角色 | 核心文档 | 预期时间 |
|-----|---------|---------|
| 开发者 | 01, 02, 04 | 30-45 分钟 |
| 构建工程师 | 01, 03, 02 | 45-60 分钟 |
| 架构师 | ASSESSMENT, 01, 02, 04 | 60-90 分钟 |
| 维护者 | 全部文档 | 2-3 小时 |

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
