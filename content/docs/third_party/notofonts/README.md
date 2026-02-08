# NotoFonts OpenHarmony Wiki

> 本文档记录 notofonts 在 OpenHarmony 中的集成、适配和使用方式。

---

## 快速概览

| 属性 | 值 |
|------|-----|
| **库名称** | Noto Fonts |
| **上游版本** | NotoSansMath-v2.539 |
| **OH 组件** | @ohos/notofonts v4.1 |
| **子系统** | thirdparty |
| **许可证** | OFL 1.1 (字体) / Apache 2.0 (代码) |
| **Patch 数量** | **0** (无 Patch) |
| **字体家族** | 198 个 (OH 预置 140+) |
| **字体文件** | ~11,675 个 |

---

## 核心特点

### 🔤 全球化字体覆盖

- 支持 **150+ 种书写系统**
- 覆盖印度语、东南亚语、中东语、非洲语等
- 与 noto-cjk 配合实现全球语言覆盖

### 🔧 零 Patch 维护

- 纯字体资源库，无源代码修改
- 定制全部通过构建配置 (BUILD.gn, fonts_config.gni) 实现
- 升级简单，直接替换字体文件

### 📱 设备差异化支持

- 通过 `support_devices` 配置不同设备字体集
- 手表设备使用 Condensed/Hinted 优化版本
- Feature flag 控制字体集大小

---

## 文档导航

### 入门必读

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、在 OH 中的定位和作用 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（无 Patch 的说明） |

### 技术细节

| 文档 | 内容 |
|------|------|
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 结构、fonts_config.gni 配置详解 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、字体回退机制 |
| [05_API_Differences.md](./05_API_Differences.md) | 字体规格说明、与上游的差异 |

### 运维管理

| 文档 | 内容 |
|------|------|
| [06_Security.md](./06_Security.md) | 安全风险分析、CVE 状态、安全建议 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | Phase 0 项目评估报告 |

---

## 典型使用场景

```
印度用户打开应用
    ↓
应用显示印地语文本
    ↓
系统字体管理器识别天城文字符
    ↓
自动回退到 NotoSansDevanagari
    ↓
正确渲染显示
```

---

## 目录结构

```
third_party/notofonts/
├── wiki/                      # 本文档目录
│   ├── README.md              # 本文件
│   ├── SUMMARY.md             # 阅读路线建议
│   ├── 01_Overview.md         # 库概览
│   ├── 02_Patches.md          # Patch 分析
│   ├── 03_Build_Integration.md # 构建适配
│   ├── 04_Usage_in_OH.md      # 依赖与使用
│   ├── 05_API_Differences.md  # API/规格差异
│   ├── 06_Security.md         # 安全分析
│   └── _work/                 # 工作文档
│       └── ASSESSMENT.md      # 项目评估
│
├── fonts/                     # 字体文件目录
│   ├── NotoSans/              # 基础字体
│   ├── NotoSansDevanagari/    # 印度语字体
│   ├── NotoSansThai/          # 泰语字体
│   └── ... (198 个家族)
│
├── BUILD.gn                   # OH 构建脚本
├── fonts_config.gni           # 字体清单配置
├── bundle.json                # OH 组件配置
├── README.OpenSource          # 开源信息
└── OAT.xml                    # 许可证合规配置
```

---

## 关键配置速查

### 添加新字体

编辑 `fonts_config.gni`:
```gn
{
  font_name = "NotoSansNewFont"
  font_path = "fonts/NotoSansNewFont/googlefonts/variable-ttf/NotoSansNewFont[wght].ttf"
  support_devices = [ "default", "watch" ]
  alias_name = ""
}
```

### 设备类型过滤

```gn
support_devices = [ 
  "default",    # 手机/平板
  "watch"       # 手表
]
```

### 构建目标

```bash
# 编译字体模块
ninja -C out/default fonts_notofonts

# 查看字体预置列表
cat third_party/notofonts/fonts_config.gni
```

---

## 相关资源

### 上游项目

- [Noto Fonts GitHub](https://github.com/notofonts)
- [Google Fonts - Noto](https://fonts.google.com/noto)
- [Noto 字体报告工具](http://notofonts.github.io/reporter.html)

### OpenHarmony 相关

- [noto-cjk](https://gitee.com/openharmony/third_party_noto-cjk) - CJK 字体配套库
- [global_system_resources](https://gitee.com/openharmony/global_system_resources) - 系统资源仓

### 许可证

- 字体: [SIL Open Font License 1.1](../fonts/LICENSE)
- 代码/文档: [Apache License 2.0](../LICENSE)

---

## 维护者

- **OH 组件负责人**: jiali17@huawei.com
- **上游项目**: Google Fonts / Noto Project

---

## 更新记录

| 日期 | 版本 | 说明 |
|------|------|------|
| 2025-02 | 4.1 | 初始 Wiki 文档创建 |

---

> 📖 **建议阅读路线**: [SUMMARY.md](./SUMMARY.md)
