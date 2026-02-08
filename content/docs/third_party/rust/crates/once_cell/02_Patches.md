# Patch 详细分析

## 概述

once_cell 在 OpenHarmony 中的集成是**最简单和最干净的适配模式**：

- ✅ **无传统 .patch 文件**: 所有修改通过原生 Git 提交跟踪
- ✅ **零源代码修改**: 所有修改均为构建系统和合规性配置
- ✅ **极低回归风险**: 升级上游版本只需更新构建配置

**修改类型分布**:
- 构建系统适配: 50% (BUILD.gn, bundle.json)
- 合规性配置: 25% (OAT.xml, README.OpenSource)
- 格式修复: 25% (README.OpenSource)

---

## Patch 清单

| 序号 | 提交哈希 | 提交时间 | 作者 | 修改文件 | 类型 | 目的 |
|------|----------|----------|------|----------|------|
| 1 | 2ca1eb957ea0 | 2023-04-12 | lubinglun@huawei.com | BUILD.gn (新增) | GN 构建系统集成 |
| 2 | 68596469a0ca | 2023-04-14 | fangting12@huawei.com | OAT.xml, README.OpenSource (新增) | OSS 合规性审计 |
| 3 | 4839b7a2788d | 2024-02-19 | liangxinyan2@huawei.com | README.OpenSource (修改) | 许可证格式修复 |
| 4 | c1909402d5ff | 2025-11-25 | longjianyin@h-partners.com | bundle.json, BUILD.gn (修改) | 组件 bundle 配置 |

---

## Patch #1: GN 构建系统集成

### 基本信息

| 属性 | 值 |
|------|-----|
| **提交哈希** | 2ca1eb957ea0d66529bc0efb5f5189cb4c52201b |
| **提交者** | lubinglun@huawei.com |
| **提交时间** | 2023-04-12 |
| **关联 Issue** | https://gitee.com/openharmony/build/issues/I6UFTP |
| **修改文件** | BUILD.gn (新增，36 行) |
| **修改类型** | 构建系统适配 |

### 修改目的

**原始问题**: OpenHarmony 使用 GN (Generate Ninja) 构建系统，而 once_cell 是纯 Rust crate，使用 Cargo 构建。需要将 once_cell 集成到 OH 的 GN/Ninja 构建流程中。

**OH 需求**: 为 Rust 第三方库添加统一的 GN 构建脚本模板，使其能够在 OH 的统一构建系统中编译。

### 修改内容

**新增文件**: `BUILD.gn`

```gn
# Copyright (c) 2023 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0...

import("//build/ohos.gni")

ohos_cargo_crate("lib") {
    crate_name = "once_cell"
    crate_type = "rlib"
    crate_root = "src/lib.rs"

    sources = ["src/lib.rs"]
    edition = "2021"
    cargo_pkg_version = "1.17.0"
    cargo_pkg_authors = "Aleksey Kladov <aleksey.kladov@gmail.com>"
    cargo_pkg_name = "once_cell"
    cargo_pkg_description = "Single assignment cells and lazy values."

    features = [
        "alloc",  # 启用 alloc 模块
        "race",   # 启用 race 模块 (first one wins 初始化)
        "std",    # 启用标准库支持
    ]

    module_output_extension = ".rlib"
    part_name = "rust_once_cell"
    subsystem_name = "thirdparty"
}
```

### 关键代码变更分析

#### 1. 使用 `ohos_cargo_crate` 模板
```gn
ohos_cargo_crate("lib") {
    ...
}
```
- 这是 OH 提供的 GN 模板，用于编译 Rust crates
- 内部调用 `rustc` 并处理依赖解析
- 与 OH 的其他 Rust crate 保持一致的构建方式

#### 2. 特性 (features) 选择
```gn
features = [
    "alloc",  # ✅ 启用: 提供堆内存分配支持
    "race",   # ✅ 启用: 提供 first one wins 初始化模式
    "std",    # ✅ 启用: 提供标准库集成
]
```

**未启用的特性**:
- `parking_lot`: ❌ 未启用 - 使用 parking_lot 优化内存（可选性能优化）
- `critical-section`: ❌ 未启用 - no_std 平台的 critical-section 实现

**选择策略**: OH 使用标准功能集，避免依赖额外的性能优化库。

#### 3. 组件元数据
```gn
part_name = "rust_once_cell"
subsystem_name = "thirdparty"
```
- 将 once_cell 归类到 `thirdparty` 子系统
- 部件名称为 `rust_once_cell`

### OH 价值

1. **统一构建系统**: 将 once_cell 纳入 OH 的 GN/Ninja 构建流程
2. **依赖管理**: 通过 GN 的 `deps` 机制管理与其他 OH 模块的依赖关系
3. **编译优化**: 与 OH 的整体编译优化策略保持一致

### 回归风险

| 风险 | 等级 | 说明 |
|------|------|------|
| **上游 API 变更** | 低 | BUILD.gn 只指定版本号，API 变更通过依赖传递 |
| **GN 模板变更** | 低 | `ohos_cargo_crate` 由 OH 构建团队维护，向后兼容 |
| **特性支持** | 低 | 只使用标准特性，上游不会移除 |

### 升级建议

**推向上游**: ❌ 不推荐
- OH 特定的 GN 构建配置与上游无关
- 上游不使用 GN 构建系统

**OH 特有**: ✅ 是
- 每次升级 upstream 版本时需要重新创建或更新 BUILD.gn
- 更新步骤:
  1. 修改 `cargo_pkg_version` 为新版本
  2. 检查上游是否移除或新增特性，相应调整 `features` 列表
  3. 重新编译验证

---

## Patch #2: OSS 审计工具 (OAT) 配置

### 基本信息

| 属性 | 值 |
|------|-----|
| **提交哈希** | 68596469a0ca524ffa033b9459ebf7c2c6962706 |
| **提交者** | fangting12@huawei.com |
| **提交时间** | 2023-04-14 |
| **修改文件** | OAT.xml (新增，66 行), README.OpenSource (新增，11 行) |
| **修改类型** | 合规性配置 |

### 修改目的

**原始问题**: OpenHarmony 要求对所有第三方开源库进行许可证合规性审计，确保符合 OH 的开源政策。

**OH 需求**: 添加 OSS Audit Tool (OAT) 配置文件，定义许可证扫描规则和豁免项。

### 修改内容

#### 文件 1: OAT.xml

**新增文件**: `OAT.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<OAT>
    <!-- OAT 配置说明 -->
    <!-- OSS 审计工具配置文件 -->
    <!-- 用途: 定义许可证扫描规则、排除项和输出格式 -->

    <FileFilter>
        <!-- 排除二进制文件不进行许可证扫描 -->
        <Exclude Path="design/logo.png" />
        <Exclude Path="design/icon.png" />
    </FileFilter>

    <License>
        <!-- 指定双重许可证 -->
        <LicenseFile Name="LICENSE-APACHE|LICENSE-MIT" />
    </License>
</OAT>
```

**关键配置项**:

1. **许可证文件配置**
   ```xml
   <LicenseFile Name="LICENSE-APACHE|LICENSE-MIT" />
   ```
   - 指定 once_cell 使用双重许可证：Apache 2.0 或 MIT
   - 允许用户选择使用任一许可证

2. **文件排除规则**
   ```xml
   <Exclude Path="design/logo.png" />
   <Exclude Path="design/icon.png" />
   ```
   - 排除二进制图片文件
   - 原因：二进制文件不包含可扫描的许可证信息

3. **配置指南注释**
   - 文件中包含详细的中文注释，说明各配置项的用途

#### 文件 2: README.OpenSource

**新增文件**: `README.OpenSource`

```json
[{
  "Name": "once_cell",
  "License": "Apache License 2.0, MIT",
  "License File": "LICENSE-APACHE|LICENSE-MIT",
  "Version Number": "1.17.0",
  "Owner": "fangting12@huawei.com",
  "Upstream URL": "https://github.com/matklad/once_cell",
  "Description": "A Rust library that provides a cell that can only be written to once."
}]
```

**元数据字段**:
- `Name`: 库名称
- `License`: 许可证类型（双许可证）
- `License File`: 许可证文件路径
- `Version Number`: 上游版本
- `Owner`: OH 负责人邮箱
- `Upstream URL`: 原始仓库地址
- `Description`: 库功能描述

### OH 价值

1. **许可证合规**: 确保 once_cell 的 Apache 2.0/MIT 许可证符合 OH 的开源政策
2. **自动化审计**: OAT 可以自动扫描并生成合规性报告
3. **追溯性**: 保存上游版本和地址信息，便于问题追踪

### 回归风险

| 风险 | 等级 | 说明 |
|------|------|------|
| **许可证变更** | 极低 | 一旦合并，上游不会随意变更许可证 |
| **OAT 工具变更** | 低 | OAT 工具的配置格式相对稳定 |
| **文件排除** | 无 | 只是豁免二进制文件，不影响合规性 |

### 升级建议

**推向上游**: ❌ 不推荐
- OAT 是 OH 特定的工具和配置
- 上游不需要这些文件

**OH 特有**: ✅ 是
- 升级 upstream 版本时：
  - 如果上游更新了许可证（极不可能），同步更新 README.OpenSource
  - 如果新增了新的二进制文件，添加到 OAT.xml 的 `Exclude` 列表
  - 其他情况下无需修改

---

## Patch #3: 许可证文件格式修复

### 基本信息

| 属性 | 值 |
|------|-----|
| **提交哈希** | 4839b7a2788db103ee2a45adaca8dbd67894921c |
| **提交者** | liangxinyan2@huawei.com |
| **提交时间** | 2024-02-19 |
| **关联 Issue** | https://gitee.com/openharmony/third_party_rust_bindgen/issues/I8ZMCP |
| **修改文件** | README.OpenSource (修改 1 行) |
| **修改类型** | 格式修复 |

### 修改目的

**原始问题**: OH 的 OSS 审计工具在扫描 bindgen、cxx、nix 等依赖库时，发现许可证文件分隔符格式不统一。

**OH 需求**: 统一所有第三方库的 `README.OpenSource` 文件中的许可证文件分隔符，从管道符 (`|`) 改为逗号 (`,`)。

### 修改内容

**修改文件**: `README.OpenSource`

```diff
 {
   "Name": "once_cell",
   "License": "Apache License 2.0, MIT",
-  "License File": "LICENSE-APACHE|LICENSE-MIT",
+  "License File": "LICENSE-APACHE, LICENSE-MIT",
   "Version Number": "1.17.0",
   ...
 }
```

**变更说明**:
- 修改位置: `"License File"` 字段
- 变更内容: `|` → `,`
- 影响范围: 仅字符串格式，不涉及实际文件

### OH 价值

1. **格式统一**: 与 OH 其他第三方库保持一致的许可证文件格式
2. **工具兼容**: 确保 OAT 扫描工具能正确解析多许可证文件
3. **规范化**: 推进 OH 第三方库元数据的规范化

### 回归风险

| 风险 | 等级 | 说明 |
|------|------|------|
| **工具不兼容** | 极低 | OAT 工具已经支持逗号分隔符 |
| **上游变更** | 无 | 这是 OH 特定的元数据格式，与上游无关 |

### 升级建议

**推向上游**: ❌ 不推荐
- 这是 OH 特定的元数据格式要求
- 上游不需要了解 OH 的 OAT 工具

**OH 特有**: ✅ 是
- 升级 upstream 版本时：
  - 确保新版本的 README.OpenSource 使用逗号分隔符
  - 从 Patch #2 的模板复制即可

---

## Patch #4: 组件 bundle 配置

### 基本信息

| 属性 | 值 |
|------|-----|
| **提交哈希** | c1909402d5ff79b3447edde4df1313fb24162268 |
| **提交者** | longjianyin@h-partners.com |
| **提交时间** | 2025-11-25 |
| **修改文件** | bundle.json (新增，34 行), BUILD.gn (修改，添加 3 行) |
| **修改类型** | 组件化配置 |

### 修改目的

**原始问题**: OpenHarmony 推行"部件化" (componentization/modularization) 管理，所有第三方库需要通过 `bundle.json` 定义组件元数据，实现模块化管理和独立发布。

**OH 需求**: 为 once_cell 添加 `bundle.json`，使其成为一个标准的 OH 组件，支持独立构建、测试和分发。

### 修改内容

#### 文件 1: bundle.json

**新增文件**: `bundle.json`

```json
{
  "name": "@ohos/rust_once_cell",
  "description": "A Rust library that provides a cell that can only be written to once",
  "version": "6.1",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "third_party/rust/crates/once_cell"
  },
  "component": {
    "name": "rust_once_cell",
    "subsystem": "thirdparty",
    "adapted_system_type": [
      "standard"
    ],
    "deps": {
      "components": []
    },
    "build": {
      "sub_component": [],
      "inner_kits": [
        {
          "name": "//third_party/rust/crates/once_cell:lib"
        }
      ],
      "test": []
    }
  }
}
```

**关键配置项解析**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `name` | @ohos/rust_once_cell | OH 组件完整名称（带命名空间） |
| `version` | 6.1 | OH 内部组件版本号（独立于上游版本） |
| `publishAs` | code-segment | 发布方式：代码段（非独立包） |
| `component.name` | rust_once_cell | 组件短名称 |
| `subsystem` | thirdparty | 所属子系统 |
| `adapted_system_type` | standard | 适配的系统类型：标准系统 |
| `inner_kits` | //third_party/rust/crates/once_cell:lib | 对外暴露的构建目标 |

#### 文件 2: BUILD.gn (修改)

**修改内容**: 在原有 BUILD.gn 基础上添加 3 行

```gn
ohos_cargo_crate("lib") {
    ...
    features = ["alloc", "race", "std"]
+   module_output_extension = ".rlib"
+   part_name = "rust_once_cell"
+   subsystem_name = "thirdparty"
}
```

**新增字段说明**:
- `module_output_extension = ".rlib"`: 明确指定输出文件扩展名
- `part_name = "rust_once_cell"`: 指定 OH 部件名称
- `subsystem_name = "thirdparty"`: 指定所属子系统

### OH 价值

1. **模块化管理**: once_cell 成为一个独立的 OH 组件，可以独立构建和测试
2. **依赖解析**: OH 的包管理工具可以解析 `bundle.json`，处理组件间依赖
3. **标准化**: 与 OH 其他第三方库使用统一的组件配置格式
4. **版本独立**: OH 组件版本 (6.1) 与上游版本 (1.17.0) 解耦，可以独立更新

### 回归风险

| 风险 | 等级 | 说明 |
|------|------|------|
| **bundle.json 格式变更** | 低 | OH 的 bundle.json 格式已经稳定 |
| **版本冲突** | 低 | OH 版本与上游版本独立管理 |
| **依赖解析失败** | 低 | once_cell 无外部依赖，解析简单 |

### 升级建议

**推向上游**: ❌ 不推荐
- `bundle.json` 是 OH 特定的组件元数据格式
- 上游不使用 OH 的组件管理系统

**OH 特有**: ✅ 是
- 升级 upstream 版本时：
  - 更新 `bundle.json` 中的 `description`（如果上游描述变更）
  - 保持 `version` 字段不变（OH 版本独立于上游）
  - BUILD.gn 中的 `cargo_pkg_version` 更新为新版本号
  - 重新编译验证

---

## Patch 维护建议

### Patch 分类总结

| 类型 | 数量 | 示例 | 维护难度 |
|------|------|------|---------|
| **构建系统适配** | 2 (BUILD.gn, bundle.json) | Patch #1, #4 | 低 |
| **合规性配置** | 2 (OAT.xml, README.OpenSource) | Patch #2, #3 | 极低 |

### 维护策略

#### 日常维护
1. **无源代码监控**: once_cell 源代码无需 OH 维护，完全与上游同步
2. **构建配置监控**: 关注 `ohos_cargo_crate` GN 模板的变更
3. **版本同步**: 定期检查上游新版本，评估升级价值

#### 升级流程
```
1. 检查上游新版本
   ↓
2. 评估变更：是否有破坏性 API 变更？
   ↓
3. 更新 BUILD.gn：
   - cargo_pkg_version = "新版本号"
   - 检查 features 列表是否需要调整
   ↓
4. 更新 README.OpenSource：
   - Version Number = "新版本号"
   ↓
5. 编译验证
   ↓
6. 运行测试
```

#### 回归测试
- 编译 once_cell crate
- 编译依赖者 (clap, rustix, which-rs)
- 运行基础功能测试

### 长期规划

⚠️ **重要提示**: once_cell 的 API 正在被提议纳入 Rust 标准库 (RFC 2788)

**未来影响**:
- 一旦标准库 API 稳定，OH 可能需要迁移到 `std::sync::OnceCell` 和 `std::sync::Lazy`
- 迁移后可以移除 once_cell 依赖，减少第三方库数量
- 但需要等待上游 RFC 合并和 API 稳定

**建议**:
- 暂时保持 once_cell 作为依赖
- 关注 RFC 2788 的进展
- 准备迁移计划（预计 1-2 年内可能发生）

---

## 结论

once_cell 在 OpenHarmony 中的适配是**教科书式的集成案例**：

✅ **最小侵入**: 仅添加 4 个文件，零源代码修改
✅ **低风险**: 升级容易，回归风险极低
✅ **标准化**: 完全遵循 OH 的构建和组件化规范
✅ **易维护**: 无需深度参与上游开发

这是一个值得其他 Rust crate 集成参考的最佳实践。
