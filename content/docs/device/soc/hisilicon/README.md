# OpenHarmony 海思芯片适配层 Wiki

**文档版本**: 2.0  
**更新日期**: 2026-02-07  
**评估状态**: Phase 0 项目评估已完成

## 项目概述

本 Wiki 文档覆盖 **device_soc_hisilicon** 仓库，这是 OpenHarmony 系统中针对上海海思（HiSilicon）芯片的设备适配层仓库。

**文档特色**: 本 Wiki 采用**证据优先原则**，所有技术结论均基于代码证据支撑，确保文档的可追溯性和准确性。

---

## 覆盖范围

### 包含的芯片型号

| SoC 型号 | 系统类型 | 领域 | 开发板 |
|---------|---------|------|--------|
| Hi3516DV300 | 小型系统/标准系统 | 智慧视觉 | HiSpark Taurus |
| Hi3518EV300 | 小型系统 | 智慧视觉 | HiSpark Aries |
| Hi3751V350 | 标准系统 | 智慧媒体 | HiSpark Phoenix |
| Hi3861V100 | 轻量系统 | 智慧 IoT | HiSpark Pegasus |
| WS63V100 | 轻量系统 | 智慧 IoT | NearLink_DK_WS63 |

### 主要功能模块

- **HAL (Hardware Abstraction Layer)**: 显示、媒体、USB、AI 等硬件抽象
- **Platform Drivers**: GPIO、I2C、SPI、UART、ADC、PWM、RTC 等外设驱动
- **SDK**: 各芯片的 SDK 实现和适配层
- **Boot**: 启动加载器、升级系统

---

## 不包含内容

- N-API 接口（本仓库为底层 SDK，不直接暴露 JS API）
- 完整的应用层代码
- 测试相关代码 (`test/`, `tests/`, `*_test.*`)

---

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── 01_Overview.md         # 项目概览
├── 02_Chips.md           # 芯片型号说明
├── 03_Directory_Structure.md  # 目录结构
├── 04_HAL_Modules.md     # HAL 模块说明
├── 05_Platform_Drivers.md # 平台驱动说明
├── 06_SDK_Architecture.md # SDK 架构说明
├── 07_Build_System.md    # 构建系统 (GN)
├── 08_Products.md        # 编译产物说明
├── 09_Security_Review.md # 安全风险评审（已增强，含 50+ 代码证据）
├── _work/
│   ├── ASSESSMENT.md     # Phase 0 项目评估报告
│   ├── PLAN.md           # 任务进度追踪
│   └── NOTES.md          # 代码证据库（50+ HAL API、60+ 驱动 API）
└── appendix/
    ├── Callgraphs.md     # 关键调用链
    └── Config_Flags.md   # 配置选项
```

---

## 阅读建议

### 新人入门顺序
```
README.md → 01_Overview.md → 02_Chips.md → 03_Directory_Structure.md
```

### HAL 开发
```
01_Overview.md → 04_HAL_Modules.md → 05_Platform_Drivers.md
```

### 构建与编译
```
07_Build_System.md → 08_Products.md
```

### 安全研究（推荐）
```
01_Overview.md → 09_Security_Review.md → _work/NOTES.md
```
**亮点**: 09_Security_Review.md 包含 50+ 条安全关键代码路径，所有风险点均有代码证据支撑。

---

## 文档增强说明（版本 2.0）

### Phase 0 项目评估
- ✅ 完成了项目类型判定（设备适配层 + SDK 仓库）
- ✅ 完成了受众需求分析（HAL 开发者优先）
- ✅ 制定了文档策略（精简版 + 证据增强）

### Phase 1 代码侦查（并行 4 个探索任务）
- ✅ HAL API 证据收集（50+ API 函数，代码路径定位）
- ✅ Platform Driver 入口收集（60+ 驱动 API，初始化函数定位）
- ✅ 安全关键代码收集（50+ 安全路径，10 类风险证据）
- ✅ 构建配置证据收集（80+ BUILD.gn，完整 targets 清单）

### Phase 2 Wiki 内容增强
- ✅ 09_Security_Review.md 大幅增强（新增 50+ 代码证据）
- ✅ _work/NOTES.md 创建完成（完整的证据库）
- ✅ 攻击面分析（输入源、信任边界、关键攻击面清单）
- ✅ 风险点分析（10 类风险，每条均有代码证据）

---

## 更新方式

当仓库代码发生重大变更时，应同步更新本 Wiki：
1. 修改对应模块文档
2. 更新 `SUMMARY.md` 中的链接
3. 更新本文档的"最后更新"时间戳
4. 如有重大安全相关变更，更新 `_work/NOTES.md` 证据库

---

## 相关链接

- **上游仓库**: [OpenHarmony Device Board Hisilicon](https://gitee.com/openharmony/device_board_hisilicon)
- **海思 SDK**: [device_board_hisilicon](https://gitee.com/openharmony/device_board_hisilicon)
- **vendor_hisilicon**: [vendor_hisilicon](https://gitee.com/openharmony/vendor_hihope)

---

*最后更新: 2026-02-07*
