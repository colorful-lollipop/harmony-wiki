# Patch 分析

## 概述

**结论: notofonts 库无 Patch 文件**

本文档分析 notofonts 在 OpenHarmony 中的 Patch 情况。由于 notofonts 是**纯字体资源库**（仅包含 .ttf/.otf 字体文件），不涉及源代码，因此不存在传统意义上的源代码 Patch。

---

## 1. Patch 文件清单

### 1.1 搜索结果

| 搜索范围 | 搜索模式 | 结果 |
|----------|----------|------|
| 全库递归 | `*.patch` | 0 个文件 |
| 全库递归 | `patches/` 目录 | 0 个目录 |

**搜索命令**:
```bash
find . -name "*.patch" -o -name "patches" -type d
```

### 1.2 结论

- ✅ 无源代码 Patch
- ✅ 无构建脚本 Patch
- ✅ 无配置文件 Patch

---

## 2. 无 Patch 原因分析

### 2.1 库类型特性

notofonts 属于**纯二进制资源库**，具有以下特点：

| 特性 | 说明 |
|------|------|
| **无源代码** | 仅包含编译后的字体文件（.ttf/.otf） |
| **标准化格式** | 字体文件遵循 OpenType 规范 |
| **跨平台** | 字体文件本身就是跨平台的 |
| **只读使用** | 系统运行时只读取，不修改 |

### 2.2 不需要 Patch 的场景

```
┌─────────────────────────────────────────────────────────┐
│              为什么字体库不需要 Patch?                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 字体是最终产物                                       │
│     - 不需要编译，不需要源码级修改                         │
│     - 上游直接提供可用的 .ttf 文件                        │
│                                                         │
│  2. 定制在构建层完成                                      │
│     - 选择哪些字体预置 → fonts_config.gni                │
│     - 如何安装到系统 → BUILD.gn                          │
│     - 设备差异化 → support_devices 配置                  │
│                                                         │
│  3. 运行时通过文件系统访问                                 │
│     - 不涉及 API 调用                                    │
│     - 不需要接口适配                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 2.3 对比需要 Patch 的库

| 库类型 | 典型 Patch 需求 | notofonts |
|--------|-----------------|-----------|
| **源码库** (curl, openssl) | 平台适配、功能定制 | ❌ 无源码 |
| **构建脚本** (ffmpeg) | 编译选项、依赖处理 | ❌ 无需编译 |
| **配置文件** (json-c) | 配置选项调整 | ❌ 无复杂配置 |
| **字体库** (notofonts) | - | ✅ 纯资源，无需 Patch |

---

## 3. OH 的定制方式（替代 Patch）

虽然无 Patch，但 OH 通过以下方式实现定制：

### 3.1 字体选择定制（fonts_config.gni）

```gn
# 示例：从 198 个字体中选择 140+ 个预置
notofonts_fonts_list = [
  {
    font_name = "NotoSansThai"
    font_path = "fonts/NotoSansThai/googlefonts/variable-ttf/NotoSansThai[wdth,wght].ttf"
    support_devices = [ "default", "watch" ]  # ← 设备定制
    alias_name = ""
  },
  # ... 更多字体
]
```

**定制点**:
- 选择预置哪些字体
- 控制不同设备的字体集
- 设置字体别名

### 3.2 构建规则定制（BUILD.gn）

```gn
# 使用 ohos_prebuilt_etc 预置字体
ohos_prebuilt_etc(font_name) {
  source = font.font_path
  module_install_dir = "fonts"  # ← 安装到 /system/fonts/
  # ...
}
```

**定制点**:
- 安装路径 (`/system/fonts/`)
- 符号链接（Roboto-Regular.ttf → NotoSans）
- SDK 预览器复制规则

### 3.3 特性开关（declare_args）

```gn
declare_args() {
  notofonts_font_feature_product = "default"  # ← 可通过 gn args 覆盖
}
```

---

## 4. 字体更新与版本管理

### 4.1 更新流程

由于无 Patch，字体更新流程简单：

```
上游发布新版本
       ↓
下载新字体文件
       ↓
替换 fonts/ 目录下对应文件
       ↓
更新 fonts_config.gni（如有新增字体）
       ↓
验证渲染效果
       ↓
提交代码
```

### 4.2 版本追踪

| 项目 | 当前版本 | 追踪方式 |
|------|----------|----------|
| 上游 NotoSansMath | v2.539 | README.OpenSource |
| OH 组件版本 | 4.1 | bundle.json |

### 4.3 升级建议

1. **常规升级**: 直接替换字体文件，无兼容性风险
2. **新增字体**: 修改 fonts_config.gni 添加配置
3. **废弃字体**: 从 fonts_config.gni 移除，保留文件（兼容性）

---

## 5. 与其他字体库的对比

### 5.1 noto-cjk（CJK 字体）

| 项目 | notofonts | noto-cjk |
|------|-----------|----------|
| **Patch 数量** | 0 | 可能有* |
| **内容** | 非 CJK 字体 | 中日韩字体 |
| **定制方式** | 构建配置 | 构建配置 + 可能有 Patch |

*注：noto-cjk 可能有构建脚本 Patch，需单独分析。

### 5.2 系统字体（Roboto/HMOS）

| 项目 | notofonts | 系统字体 |
|------|-----------|----------|
| **来源** | 上游 Google | OH 自研或定制 |
| **Patch** | 0 | N/A |
| **关系** | 补充多语言 | 主界面字体 |

---

## 6. 总结

### 6.1 Patch 状态总览

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 源代码 Patch | ✅ 无 | 无源代码 |
| 构建脚本 Patch | ✅ 无 | 原生 GN 脚本 |
| 配置 Patch | ✅ 无 | 纯新增配置 |
| 许可证 Patch | ✅ 无 | OAT.xml 为新增合规配置 |

### 6.2 OH 定制总结

| 定制类型 | 实现方式 | 文件 |
|----------|----------|------|
| 字体选择 | fonts_config.gni 清单 | `fonts_config.gni` |
| 构建规则 | GN 模板 | `BUILD.gn` |
| 设备差异化 | support_devices 数组 | `fonts_config.gni` |
| 安装路径 | module_install_dir | `BUILD.gn` |

### 6.3 维护建议

1. **无需 Patch 维护**: 享受无 Patch 的轻松维护
2. **关注上游更新**: 定期同步上游字体新版本
3. **按需扩展**: 需要新语言支持时添加对应字体
4. **监控体积**: 字体文件较大，注意系统镜像体积

---

## 附录：字体文件清单（部分）

详见 `fonts_config.gni` 中的完整列表，主要包含以下类别：

- **基础字体**: NotoSans, NotoSerif, NotoSansMono
- **印度语系**: Devanagari, Bengali, Gujarati, Gurmukhi, Tamil, Telugu, Kannada, Malayalam
- **东南亚**: Thai, Khmer, Lao, Myanmar
- **中东**: Arabic, Hebrew, NaskhArabic, NastaliqUrdu
- **非洲**: Ethiopic, Adlam, Bamum
- **古代文字**: EgyptianHieroglyphs, Cuneiform, LinearB
- **符号**: Math, Symbols, Symbols2

完整列表参见：
- [fonts_config.gni](../../fonts_config.gni)
- [fontrepos.json](../../fontrepos.json)
