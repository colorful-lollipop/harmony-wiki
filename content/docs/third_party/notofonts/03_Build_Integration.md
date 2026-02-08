# OpenHarmony 构建适配

本文档详细说明 notofonts 在 OpenHarmony 构建系统中的适配方式。

---

## 1. 构建系统概述

### 1.1 上游构建方式

notofonts 上游项目使用以下方式构建：
- **Python 构建工具**: notobuilder（基于 gftools）
- **GitHub Actions**: 自动化构建和发布
- **配置方式**: `sources/config-*.yaml` 文件

上游产出：
- Variable TTF（可变字体）
- Static TTF/OTF（静态字体）
- Hinted/Unhinted 版本

### 1.2 OH 构建方式

OpenHarmony 不编译字体，而是**直接预置**上游已编译的字体文件：

```
上游预编译字体 → OH 代码仓 → GN 预置规则 → 系统镜像
```

**OH 构建特点**:
- 不执行字体编译（无 fontmake/fonttools 依赖）
- 使用 `ohos_prebuilt_etc` 直接安装二进制
- 通过 GN 配置控制字体选择和设备适配

---

## 2. BUILD.gn 结构详解

### 2.1 文件位置

```
third_party/notofonts/
├── BUILD.gn           ← 主构建脚本
└── fonts_config.gni   ← 字体清单配置（被导入）
```

### 2.2 BUILD.gn 完整代码

```gn
# Copyright (c) 2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0

import("//build/ohos.gni")
import("fonts_config.gni")

dep_list = []

# 遍历所有字体配置，生成预置规则
foreach(font, notofonts_fonts_list) {
  isLoadFont = false
  # 检查当前设备类型是否支持该字体
  foreach(device, font.support_devices) {
    if (notofonts_font_feature_product == device) {
      isLoadFont = true
    }
  }

  if (isLoadFont) {
    font_name = font.font_name
    ohos_prebuilt_etc(font_name) {
      # 特殊处理：NotoSans 创建 Roboto 符号链接
      if (font_name == "NotoSans") {
        symlink_target_name = [ "Roboto-Regular.ttf" ]
      }
      source = font.font_path
      if (font.alias_name != "") {
        output = font.alias_name
      }
      module_install_dir = "fonts"
      subsystem_name = "thirdparty"
      part_name = "notofonts"
    }
    dep_list += [ font_name ]
  }
}

# 对外暴露的头文件组（字体库无头文件，仅为依赖管理）
ohos_shared_headers("fonts_notofonts") {
  include_dirs = []
  deps = []
  foreach(dep, dep_list) {
    deps += [ ":${dep}" ]
  }
  subsystem_name = "thirdparty"
  part_name = "notofonts"
}

# SDK 预览器字体复制（用于 IDE 预览）
ohos_copy("copy_preview_fonts_notofonts") {
  sources = []
  foreach(font, notofonts_fonts_list) {
    sources += [ font.font_path ]
  }
  outputs = [ target_out_dir + "/previewer/common/bin/fonts/{{source_file_part}}" ]
  module_source_dir = target_out_dir + "/previewer/common/bin/"
  module_install_name = ""
  subsystem_name = "thirdparty"
  part_name = "notofonts"
}

# SDK 预览器扩展资源复制
ohos_copy("copy_preview_fonts_notofonts_ext") {
  sources = []
  foreach(font, notofonts_fonts_list) {
    sources += [ font.font_path ]
  }
  outputs = [ target_out_dir + "/previewer/resources/fonts/{{source_file_part}}" ]
  module_source_dir = target_out_dir + "/previewer/resources"
  module_install_name = ""
  subsystem_name = "thirdparty"
  part_name = "notofonts"
}
```

### 2.3 关键构建规则解析

#### 2.3.1 ohos_prebuilt_etc

```gn
ohos_prebuilt_etc(font_name) {
  source = font.font_path          # 源字体文件路径
  output = font.alias_name         # 输出文件名（可选）
  module_install_dir = "fonts"     # 安装目录: /system/fonts/
  subsystem_name = "thirdparty"
  part_name = "notofonts"
}
```

**作用**: 将字体文件作为预置资源安装到系统镜像的 `/system/fonts/` 目录。

#### 2.3.2 符号链接处理

```gn
if (font_name == "NotoSans") {
  symlink_target_name = [ "Roboto-Regular.ttf" ]
}
```

**说明**: NotoSans 同时作为 Roboto-Regular.ttf 的别名，保持与 Android 生态的兼容性。

#### 2.3.3 ohos_shared_headers

```gn
ohos_shared_headers("fonts_notofonts") {
  deps = [ ":NotoSans", ":NotoSerif", ... ]  # 所有字体依赖
}
```

**作用**: 虽然字体库无头文件，但该 target 用于：
- 统一管理所有字体资源的依赖
- 作为 bundle.json 中声明的 `inner_kits`

#### 2.3.4 ohos_copy（预览器）

```gn
ohos_copy("copy_preview_fonts_notofonts") {
  outputs = [ target_out_dir + "/previewer/common/bin/fonts/..." ]
}
```

**作用**: 为 DevEco Studio 的预览功能复制字体资源。

---

## 3. fonts_config.gni 配置详解

### 3.1 文件作用

`fonts_config.gni` 是 GN 配置导入文件，定义：
- 参数声明 (`declare_args`)
- 字体清单 (`notofonts_fonts_list`)

### 3.2 参数声明

```gn
declare_args() {
  notofonts_font_feature_product = "default"
}
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `notofonts_font_feature_product` | `"default"` | 字体特性产品类型，用于设备过滤 |

**使用方式**: 可在 `gn args` 中覆盖，例如：
```bash
gn args out/default --args='notofonts_font_feature_product="watch"'
```

### 3.3 字体清单结构

```gn
notofonts_fonts_list = [
  {
    font_name = "NotoSansEthiopic"
    font_path = "fonts/NotoSansEthiopic/googlefonts/variable-ttf/NotoSansEthiopic[wdth,wght].ttf"
    support_devices = [ "default" ]
    alias_name = ""
  },
  # ... 更多字体
]
```

#### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `font_name` | string | 是 | 字体的 GN target 名称 |
| `font_path` | string | 是 | 字体文件相对路径 |
| `support_devices` | list | 是 | 支持的设备类型数组 |
| `alias_name` | string | 否 | 输出别名（重命名） |

#### support_devices 取值

| 值 | 说明 |
|----|------|
| `"default"` | 默认设备（手机、平板、标准系统） |
| `"watch"` | 手表设备（资源受限，可能使用精简字体） |

### 3.4 设备差异化示例

**手机/平板 (default)**:
```gn
{
  font_name = "NotoSansDevanagari"
  font_path = "fonts/NotoSansDevanagari/googlefonts/variable-ttf/NotoSansDevanagari[wdth,wght].ttf"
  support_devices = [ "default" ]
}
```

**手表 (watch)** - 使用 Condensed 版本:
```gn
{
  font_name = "NotoSansDevanagari-Condensed"
  font_path = "fonts/NotoSansDevanagari/hinted/ttf/NotoSansDevanagari-Condensed.ttf"
  support_devices = [ "watch" ]
}
```

---

## 4. 与上游构建的对比

### 4.1 构建流程对比

| 阶段 | 上游 (notofonts) | OpenHarmony |
|------|------------------|-------------|
| **源码** | sources/*.ufo 或 *.glyphs | 无（直接使用二进制） |
| **编译** | fontmake → TTF | 无 |
| **配置** | config-*.yaml | fonts_config.gni |
| **构建工具** | Python (notobuilder) | GN + Ninja |
| **输出** | fonts/*/*.ttf | 系统镜像 /system/fonts/ |
| **QA** | fontbakery 检查 | 集成测试 |

### 4.2 配置方式对比

**上游 (YAML)**:
```yaml
# sources/config-sans.yaml
familyName: "Noto Sans"
buildVariable: true
includeSubsets:
  - name: "GF Glyph Sets/GF-latin-core"
    from: "Noto Sans"
```

**OH (GNI)**:
```gn
# fonts_config.gni
{
  font_name = "NotoSans"
  font_path = "fonts/NotoSans/googlefonts/variable-ttf/NotoSans[wdth,wght].ttf"
  support_devices = [ "default" ]
}
```

### 4.3 差异总结

| 维度 | 上游 | OH |
|------|------|-----|
| **关注点** | 字体设计、编译 | 系统集成、设备适配 |
| **定制方式** | 字形修改、子集包含 | 字体选择、设备过滤 |
| **复杂度** | 高（需要字体工程知识） | 低（纯配置） |
| **维护成本** | 高（需要跟随上游重新编译） | 低（替换文件即可） |

---

## 5. OAT.xml 许可证配置

### 5.1 文件位置

`third_party/notofonts/OAT.xml`

### 5.2 配置内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <oatconfig>
        <policylist>
            <policy name="projectPolicy" desc="">
                <policyitem type="compatibility" name="OFL-1.1-no-RFN" 
                           path="fonts/LICENSE" 
                           desc="SIL Open Font 1.1 License"/>
            </policy>
        </policylist>
        <filefilterlist>
            <filefilter name="binaryFileTypePolicyFilter" desc="Filters for binary file policies">
                <filteritem type="filename" name="*.otf|*.ttf" desc="官方字体文件"/>
                <filteritem type="filename" name="*.png" desc="官方文件，不使用"/>
            </filefilter>
        </filefilterlist>
    </oatconfig>
</configuration>
```

### 5.3 配置说明

| 配置项 | 说明 |
|--------|------|
| **OFL-1.1-no-RFN** | 声明字体使用 SIL Open Font License 1.1（无保留字体名） |
| **binaryFileTypePolicyFilter** | 标记字体文件为二进制资源，豁免某些源码检查 |
| ***.otf\|*.ttf** | 字体文件通配符 |

---

## 6. 构建输出

### 6.1 系统镜像中的位置

```
/system/fonts/
├── NotoSans[wdth,wght].ttf
├── NotoSerif[wdth,wght].ttf
├── NotoSansDevanagari[wdth,wght].ttf
├── NotoSansThai[wdth,wght].ttf
├── NotoSansHebrew[wdth,wght].ttf
└── Roboto-Regular.ttf -> NotoSans[wdth,wght].ttf  (符号链接)
```

### 6.2 SDK 预览器中的位置

```
previewer/common/bin/fonts/
├── NotoSans[wdth,wght].ttf
└── ...

previewer/resources/fonts/
├── NotoSans[wdth,wght].ttf
└── ...
```

### 6.3 依赖关系

```
//third_party/notofonts:fonts_notofonts
├── :NotoSans
├── :NotoSerif
├── :NotoSansDevanagari
├── :NotoSansThai
└── ... (140+ 个字体 target)
```

---

## 7. 常见问题与调试

### 7.1 字体未预置到系统

**检查项**:
1. 确认 `fonts_config.gni` 中包含该字体
2. 确认 `support_devices` 包含当前设备类型
3. 确认 `gn args` 中的 `notofonts_font_feature_product` 设置
4. 检查 `bundle.json` 中是否正确声明 sub_component

### 7.2 添加新字体流程

```
1. 将字体文件放入 fonts/<FamilyName>/
2. 在 fonts_config.gni 中添加配置项
3. 确保 font_path 指向正确的 .ttf 文件
4. 设置合适的 support_devices
5. 编译验证
```

### 7.3 减小系统镜像体积

**方法**:
- 精简 `support_devices` 列表，只为目标设备预必要字体
- 使用 `alias_name` 避免重复安装相同字体
- 优先使用 Variable TTF（单文件支持多字重）

---

## 8. 参考文档

- [GN 参考文档](https://gn.googlesource.com/gn/+/main/docs/reference.md)
- [OpenHarmony 构建系统](https://gitee.com/openharmony/build)
- [ohos_prebuilt_etc 模板](https://gitee.com/openharmony/build/blob/master/ohos.gni)
