# alsa-utils Wiki 文档

> OpenHarmony 第三方库 - ALSA 工具集适配文档

---

## 📚 文档导航

本文档集记录 `alsa-utils` 在 OpenHarmony 中的集成与适配情况,重点说明 OH 的定制化内容。

### 核心文档

| 文档 | 说明 | 推荐阅读 |
|------|------|----------|
| [SUMMARY.md](SUMMARY.md) | 📖 阅读路线建议 | ⭐ **首先阅读** |
| [01_Overview.md](01_Overview.md) | 原始库简介 & OH 定位 | ⭐ **必读** |
| [02_Patches.md](02_Patches.md) | Patch 详细分析 | 参考 |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建系统适配 | ⭐ **必读** |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系 & 使用场景 | ⭐ **必读** |
| [05_API_Differences.md](05_API_Differences.md) | API/接口差异 | 参考 |
| [06_Security.md](06_Security.md) | 安全风险分析 | ⭐ **必读** |

### 工作文档

| 文档 | 说明 |
|------|------|
| [`_work/ASSESSMENT.md`](_work/ASSESSMENT.md) | 📋 项目评估报告 |
| [`_work/NOTES.md`](_work/NOTES.md) | 分析过程记录 |
| [`_work/PLAN.md`](_work/PLAN.md) | 任务进度 |

---

## 🎯 快速了解

### alsa-utils 是什么?

**alsa-utils** 是 Advanced Linux Sound Architecture (ALSA) 项目的命令行工具集,提供 Linux 系统的音频和 MIDI 功能。

### 在 OpenHarmony 中的定位

> **作为一种 HDI 之外的定位工具,便于问题定界、方便对 HDI 不熟悉的驱动移植生态伙伴使用。**

### 关键特点

| 特点 | 说明 |
|------|------|
| **适配方式** | 纯构建系统适配,无源代码修改 |
| **Patch 数量** | 0 个 |
| **功能裁剪** | 从上游 15+ 工具精简到 5 个核心工具 |
| **依赖关系** | 仅依赖 alsa-lib,OH 中无直接依赖者 |
| **安全风险** | 低 (已修复所有 CVE) |

### 编译的工具

```
✅ alsactl       - 声卡配置管理
✅ amixer        - 命令行混音器
✅ aplay         - 音频播放 (含 arecord 链接)
✅ speaker-test  - 扬声器测试
✅ aconnect      - MIDI 连接管理

❌ alsamixer     - 需要 ncurses,未集成
❌ amidi         - MIDI 工具,未集成
❌ alsaloop      - PCM 回环,未集成
❌ alsaucm       - 用例管理器,未集成
```

---

## 🚀 快速开始

### 编译

```bash
# 方法 1: 直接编译
./build.sh --product-name [PRODUCT_NAME] --ccache \
    --build-target third_party/alsa-utils:alsa-utils

# 方法 2: 在 bundle.json 中添加
"build": {
  "sub_component": ["//third_party/alsa-utils:alsa-utils"]
}
```

### 使用

```bash
# 音频播放
aplay /data/test.wav

# 音频录制
arecord -d 30 -f cd -r 44100 -c 2 -t wav test.wav

# 查看混音器控件
amixer contents

# 设置混音器
amixer cset numid=1 6

# 扬声器测试
speaker-test -t wav -c 2
```

---

## 📊 项目状态

| 指标 | 值 |
|------|-----|
| **OH 版本** | 1.2.11 |
| **上游版本** | 1.2.15.2 (2025-01-08) |
| **版本差距** | 约 1 个大版本 |
| **适配复杂度** | 🟢 低 |
| **维护成本** | 🟢 低 |
| **安全风险** | 🟢 低 |
| **维护活跃度** | ⭐⭐⭐⭐⭐ |

---

## 🔗 相关资源

- [ALSA 官方网站](http://www.alsa-project.org)
- [alsa-utils GitHub](https://github.com/alsa-project/alsa-utils)
- [上游 README](../README.md)
- [OH 适配说明](../README_zh.md)

---

## 📝 维护说明

本文档由 OpenHarmony Wiki Agent 自动生成,记录 alsa-utils 在 OH 中的适配情况。

**最后更新**: 2026-02-08
