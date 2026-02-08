# NotoFonts 概述

## 1. 原始库简介

### 1.1 基本信息

| 属性 | 值 |
|------|-----|
| **项目名称** | Noto Fonts |
| **上游地址** | https://github.com/notofonts/notofonts.github.io |
| **官方项目** | https://github.com/notofonts |
| **许可证** | SIL Open Font License 1.1 (字体) / Apache 2.0 (代码和文档) |
| **上游版本** | NotoSansMath-v2.539 |
| **维护者** | Google Fonts / Noto Project |

### 1.2 原始功能

Noto Fonts（"No Tofu" 的缩写）是 Google 发起的开源字体项目，目标是消除数字文本显示中的"豆腐块"（无法显示的字符方框）。项目提供：

- **全球语言覆盖**: 支持世界上几乎所有书写系统（除 CJK 和 emoji 外）
- **统一设计风格**: 所有字体遵循一致的视觉设计语言
- **多格式支持**: 提供可变字体（Variable Fonts）和静态字体
- **开源免费**: 可在任何项目中自由使用

### 1.3 设计理念

```
No more 'Tofu' - 消除不可见字符
```

Noto 字体家族的设计目标是：当用户在设备上看到任何语言的文本时，都能正确渲染，不会出现空白方块或乱码。

---

## 2. 在 OpenHarmony 中的作用

### 2.1 引入背景

在 OpenHarmony 系统中，为构建**全球化语言字库能力**，覆盖全球国家和地区，需引入该三方库来丰富语言字库合集。

**关键需求**:
- 支持全球 150+ 种语言的文字显示
- 为非 CJK 语言提供高质量字体资源
- 确保国际化应用的正确显示

### 2.2 在 OH 中的定位

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 字体体系                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   系统字体    │    │   中文字体    │    │   全球字体    │  │
│  │ (Roboto/HMOS)│    │  (Noto CJK)  │    │  (NotoFonts) │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                   │                   │          │
│         └───────────────────┼───────────────────┘          │
│                             │                              │
│                    ┌────────▼────────┐                     │
│                    │  字体管理系统    │                     │
│                    │  (Font Manager) │                     │
│                    └─────────────────┘                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 覆盖语言范围

NotoFonts 在 OH 中主要覆盖以下语系：

| 语系 | 代表性语言/文字 | 典型字体 |
|------|----------------|----------|
| **印度-雅利安** | 印地语、孟加拉语、旁遮普语、古吉拉特语 | NotoSansDevanagari, NotoSansBengali, ... |
| **达罗毗荼** | 泰米尔语、泰卢固语、卡纳达语、马拉雅拉姆语 | NotoSansTamil, NotoSansTelugu, ... |
| **东南亚** | 泰语、高棉语、老挝语、缅甸语 | NotoSansThai, NotoSansKhmer, ... |
| **中东** | 阿拉伯语、希伯来语、波斯语、乌尔都语 | NotoNaskhArabic, NotoSansHebrew, ... |
| **非洲** | 阿姆哈拉语（埃塞俄比亚）、阿德拉姆文 | NotoSansEthiopic, NotoSansAdlam |
| **欧洲古典** | 希腊语、西里尔语、亚美尼亚语、格鲁吉亚语 | NotoSans, NotoSerif |
| **古代文字** | 埃及象形文字、楔形文字、线形文字 B | NotoSansEgyptianHieroglyphs, ... |
| **符号** | 数学符号、通用符号 | NotoSansMath, NotoSansSymbols |

---

## 3. 字体资源统计

### 3.1 规模概览

| 统计项 | 数值 |
|--------|------|
| **字体家族数** | 198 个 |
| **字体文件数** | ~11,675 个 |
| **OH 预置字体** | ~140+ 个（fonts_config.gni 配置） |
| **书写系统** | 150+ 种 |

### 3.2 字体格式

| 格式 | 说明 | 用途 |
|------|------|------|
| **Variable TTF** | 可变字体 (.ttf) | 主要格式，支持动态调节字重/宽度 |
| **Static TTF** | 静态字体 (.ttf) | UI 专用、特定字重版本 |
| **OTF** | OpenType 字体 (.otf) | 部分字体提供 |
| **Hinted** | 精调版本 | 小字号显示优化 |

### 3.3 字体族结构示例

以 Devanagari（天城文）为例：

```
fonts/NotoSansDevanagari/
├── googlefonts/
│   ├── variable-ttf/
│   │   └── NotoSansDevanagari[wdth,wght].ttf  ← 可变字体
│   └── ttf/
│       ├── NotoSansDevanagUI-Bold.ttf         ← UI 专用字重
│       ├── NotoSansDevanagUI-Medium.ttf
│       ├── NotoSansDevanagUI-Regular.ttf
│       └── NotoSansDevanagUI-SemiBold.ttf
└── hinted/ttf/
    └── NotoSansDevanagari-Condensed.ttf       ← 压缩版（手表）
```

---

## 4. OH 与上游的差异

### 4.1 无源代码差异

notofonts 作为纯字体资源库：
- ✅ 无 Patch 文件
- ✅ 无源代码修改
- ✅ 无 `#ifdef OHOS` 适配

### 4.2 构建层定制

OH 的定制全部体现在构建配置：
- `BUILD.gn`: GN 构建规则
- `fonts_config.gni`: 字体清单和过滤配置
- `OAT.xml`: 许可证合规配置

### 4.3 设备差异化

上游 Noto 提供统一字体包，OH 实现了：
- **设备类型过滤**: `support_devices` 控制不同设备预置字体
- **手表优化**: 部分字体提供 Condensed/Hinted 版本用于手表
- **按需加载**: 通过 feature flag 控制字体集

---

## 5. 与其他组件的关系

### 5.1 配套字体库

| 库 | 路径 | 说明 |
|----|------|------|
| **noto-cjk** | `//third_party/noto-cjk` | CJK 字体（中文、日文、韩文） |
| **notofonts** | `//third_party/notofonts` | 非 CJK 全球字体（本文档） |

### 5.2 相关子系统

| 子系统 | 使用方式 |
|--------|----------|
| **ArkUI** | 通过字体管理系统加载 |
| **图形子系统** | 文本渲染时引用 |
| **Web 引擎** | WebView 文本渲染 |
| **SDK 预览器** | 直接复制字体文件 |

---

## 6. 许可证说明

### 6.1 双许可证

| 范围 | 许可证 | 文件 |
|------|--------|------|
| **根目录代码/文档** | Apache License 2.0 | `LICENSE` |
| **字体文件** | SIL Open Font License 1.1 | `fonts/LICENSE` |

### 6.2 OFL 1.1 要点

- ✅ 可自由使用、修改、分发
- ✅ 可嵌入商业产品
- ✅ 可与其他软件捆绑
- ❌ 不能单独销售字体文件
- ❌ 衍生作品必须使用 OFL 许可证

---

## 7. 参考链接

- **Noto 项目主页**: https://fonts.google.com/noto
- **GitHub 组织**: https://github.com/notofonts
- **字体报告工具**: http://notofonts.github.io/reporter.html
- **字体仓库索引**: https://notofonts.github.io/
- **OpenHarmony 相关仓**:
  - [global_system_resources](https://gitee.com/openharmony/global_system_resources)
  - [third_party_noto-cjk](https://gitee.com/openharmony/third_party_noto-cjk)
