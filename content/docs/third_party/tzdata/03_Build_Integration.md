# 03_Build_Integration.md - OH 构建适配

## 1. BUILD.gn 结构说明

### 1.1 文件位置

```
third_party/tzdata/
└── data/
    └── BUILD.gn    # 唯一的 BUILD.gn 文件
```

### 1.2 完整配置

```gn
# Copyright (c) 2021-2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")

ohos_prebuilt_etc("iana_tzdata") {
  source = "//third_party/tzdata/data/prebuild/posix/tzdata"
  module_install_dir = "etc/zoneinfo"
  part_name = "tzdata"
  subsystem_name = "thirdparty"
}

ohos_prebuilt_etc("timezone_list_cfg") {
  source = "//third_party/tzdata/data/prebuild/posix/timezone_list.cfg"
  module_install_dir = "etc/zoneinfo"
  part_name = "tzdata"
  subsystem_name = "thirdparty"
}

ohos_prebuilt_etc("watch_timezone_list_cfg") {
  source = "//third_party/tzdata/data/prebuild/wearable/timezone_list.cfg"
  module_install_dir = "etc/zoneinfo"
  part_name = "tzdata"
  subsystem_name = "thirdparty"
}

group("zoneinfo") {
  # just for standard system
  if (is_standard_system) {
    deps = [ ":iana_tzdata" ]
    if (target_platform == "watch") {
      deps += [ ":watch_timezone_list_cfg" ]
    } else {
      deps += [ ":timezone_list_cfg" ]
    }
  }
}
```

### 1.3 配置解析

#### 导入语句

```gn
import("//build/ohos.gni")
```

导入 OpenHarmony 构建系统的标准 GN 导入文件，提供 `ohos_prebuilt_etc`、`ohos_shared_library` 等标准模板。

#### 目标定义

| 目标名称 | 类型 | 作用 |
|---------|------|------|
| `iana_tzdata` | `ohos_prebuilt_etc` | 主时区数据文件 |
| `timezone_list_cfg` | `ohos_prebuilt_etc` | 标准设备时区列表 |
| `watch_timezone_list_cfg` | `ohos_prebuilt_etc` | 穿戴设备时区列表 |
| `zoneinfo` | `group` | 聚合目标，对外暴露的接口 |

---

## 2. 关键编译配置

### 2.1 ohos_prebuilt_etc 详解

`ohos_prebuilt_etc` 是 OH 构建系统提供的模板，用于将预构建的配置文件/数据文件安装到系统目录。

#### 通用配置

| 属性 | 说明 | 示例值 |
|-----|------|-------|
| `source` | 源文件路径 | `//third_party/tzdata/data/prebuild/posix/tzdata` |
| `module_install_dir` | 系统安装目录 | `etc/zoneinfo` |
| `part_name` | 所属部件名称 | `tzdata` |
| `subsystem_name` | 所属子系统名称 | `thirdparty` |

#### 安装路径

最终的系统安装路径为：
```
/system/etc/zoneinfo/
├── tzdata              # 来自 iana_tzdata
├── timezone_list.cfg   # 来自 timezone_list_cfg 或 watch_timezone_list_cfg
```

### 2.2 条件编译配置

```gn
group("zoneinfo") {
  if (is_standard_system) {
    deps = [ ":iana_tzdata" ]
    if (target_platform == "watch") {
      deps += [ ":watch_timezone_list_cfg" ]
    } else {
      deps += [ ":timezone_list_cfg" ]
    }
  }
}
```

#### 条件变量说明

| 变量 | 类型 | 说明 |
|-----|------|------|
| `is_standard_system` | bool | 是否为 standard 系统类型（由 bundle.json 中的 `adapted_system_type` 定义） |
| `target_platform` | string | 目标平台，如 `"watch"` 表示穿戴设备 |

#### 条件逻辑

```
is_standard_system?
├── false → 不包含任何时区数据（小型系统可能不需要）
└── true → 包含 iana_tzdata
    └── target_platform == "watch"?
        ├── true → 使用 watch_timezone_list_cfg（精简列表）
        └── false → 使用 timezone_list_cfg（完整列表）
```

### 2.3 与上游构建系统的对比

#### 上游 Makefile（传统方式）

```makefile
# 上游 tzdata 的 Makefile 主要目标：
# - 编译 zic, zdump 等工具
# - 使用时区规则文本文件生成二进制数据
# - 安装到 $(TOPDIR)/usr/share/zoneinfo/

# 典型用法：
make
make install
```

#### OpenHarmony BUILD.gn（OH 方式）

```gn
# OH 构建方式：
# - 不编译任何工具
# - 直接使用预构建的二进制数据
# - 通过 ohos_prebuilt_etc 安装到 /system/etc/zoneinfo/

# 典型用法：
# 在 bundle.json 中引用：//third_party/tzdata/data:zoneinfo
```

| 对比项 | 上游 Makefile | OH BUILD.gn |
|-------|--------------|-------------|
| 构建工具 | 编译 zic, zdump | 不编译工具 |
| 生成数据 | 从文本规则生成 | 使用预构建数据 |
| 安装路径 | /usr/share/zoneinfo/ | /system/etc/zoneinfo/ |
| 平台适配 | 通过 Makefile 变量 | 通过 GN 条件语句 |

---

## 3. 特殊处理

### 3.1 平台差异化处理

#### 标准设备 vs 穿戴设备

```
data/prebuild/
├── posix/
│   ├── tzdata
│   └── timezone_list.cfg      # 标准设备：完整时区列表（TODO: 确认具体数量）
└── wearable/
    └── timezone_list.cfg      # 穿戴设备：精简时区列表（TODO: 确认具体数量）
```

#### 实现方式

```gn
if (target_platform == "watch") {
  deps += [ ":watch_timezone_list_cfg" ]
} else {
  deps += [ ":timezone_list_cfg" ]
}
```

**优点**：
- 避免在可穿戴设备上浪费存储空间
- 无需修改源码即可实现裁剪
- 运行时根据平台自动选择正确的配置

### 3.2 子系统类型限制

```gn
if (is_standard_system) {
  deps = [ ":iana_tzdata" ]
  # ...
}
```

这与 `bundle.json` 中的配置一致：
```json
"adapted_system_type": [ "standard" ]
```

说明 tzdata 组件仅适配 standard 系统类型，小型/嵌入式系统可能：
- 不需要时区数据（使用 UTC 或固定时区）
- 通过其他方式提供时区功能

---

## 4. bundle.json 配置

### 4.1 完整配置

```json
{
    "name": "@ohos/tzdata",
    "description": "The Time Zone Database (often called tz or zoneinfo) contains code and data that represent the history of local time for many representative locations around the globe. It is updated periodically to reflect changes made by political bodies to time zone boundaries, UTC offsets, and daylight-saving rules.",
    "version": "4.0",
    "license": "BSD 3-Clause",
    "publishAs": "code-segment",
    "segment": {
        "destPath": "third_party/tzdata"
    },
    "dirs": {},
    "scripts": {},
    "licensePath": "LICENSE",
    "component": {
        "name": "tzdata",
        "subsystem": "thirdparty",
        "syscap": [],
        "features": [],
        "adapted_system_type": [ "standard" ],
        "rom": "0",
        "ram": "0",
        "deps": {
            "components": [],
            "third_party": []
        },
        "build": {
            "sub_component": [
                "//third_party/tzdata/data:zoneinfo"
            ],
            "inner_kits": [],
            "test": []
        }
    }
}
```

### 4.2 关键配置说明

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `name` | `@ohos/tzdata` | NPM 风格的包名 |
| `version` | `4.0` | OH 组件版本（非上游版本） |
| `subsystem` | `thirdparty` | 所属子系统 |
| `adapted_system_type` | `["standard"]` | 适配的系统类型 |
| `build.sub_component` | `//third_party/tzdata/data:zoneinfo` | 构建入口点 |

### 4.3 版本号说明

```json
"version": "4.0"
```

这是 OH 组件的版本号，与上游 tzdata 版本（2025b）是**不同的版本体系**：
- OH 版本 `4.0`：OH 组件配置格式的版本
- 上游版本 `2025b`：IANA 时区数据的版本

---

## 5. 预构建数据来源

### 5.1 预构建数据文件

```
data/prebuild/
├── posix/
│   ├── tzdata              # IANA 时区数据库二进制文件
│   └── timezone_list.cfg   # 时区名称列表
└── wearable/
    └── timezone_list.cfg   # 精简版时区名称列表
```

### 5.2 tzdata 文件格式

`tzdata` 是 IANA 标准的二进制时区数据库格式，由 `zic` 工具生成：

```
# 上游生成方式（供参考）：
zic -d output_dir africa asia europe northamerica southamerica ...
# 生成 output_dir/tzdata
```

### 5.3 timezone_list.cfg 格式

纯文本文件，每行一个时区名称：

```
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
...
Asia/Shanghai
...
```

**TODO**: 需要确认：
- [ ] 标准版和穿戴版具体包含多少个时区
- [ ] 筛选标准是什么
- [ ] 如何生成此列表

---

## 6. 依赖关系

### 6.1 本库依赖

```json
"deps": {
    "components": [],
    "third_party": []
}
```

tzdata **没有**声明任何依赖，因为它只是简单的数据文件复制。

### 6.2 被依赖关系

```
其他组件 → //third_party/tzdata/data:zoneinfo
```

tzdata 作为基础组件，被系统框架层依赖（详见 04_Usage_in_OH.md）。

---

## 7. 构建流程

### 7.1 标准构建流程

```
1. GN 解析阶段
   └── 读取 data/BUILD.gn
       └── 定义 iana_tzdata, timezone_list_cfg, watch_timezone_list_cfg, zoneinfo 目标

2. 构建阶段
   └── 根据 is_standard_system 和 target_platform 条件
       └── 复制相应的预构建文件到输出目录

3. 打包阶段
   └── 将文件打包到 system.img 的 /system/etc/zoneinfo/ 目录
```

### 7.2 调试构建

查看构建日志：
```bash
# 查看构建目标
gn desc out/default //third_party/tzdata/data:zoneinfo

# 查看实际构建的文件
ls -la out/default/gen/third_party/tzdata/data/
```

---

## 8. 维护和更新指南

### 8.1 更新时区数据

当 IANA 发布新的时区数据版本时：

1. **获取上游更新**
   ```bash
   # 从 https://github.com/eggert/tz 获取新版本
   ```

2. **重新编译时区数据**
   ```bash
   # 使用新版本的 zic 工具编译
   make zic
   ./zic -d output africa asia australasia europe northamerica southamerica ...
   ```

3. **替换预构建文件**
   ```bash
   cp output/tzdata third_party/tzdata/data/prebuild/posix/
   ```

4. **更新版本号**
   ```bash
   echo "2025b" > third_party/tzdata/version
   # 更新 README.OpenSource 中的版本声明
   ```

5. **更新时区列表（如需要）**
   ```bash
   # 检查是否有新增或删除的时区
   # 更新 timezone_list.cfg
   ```

6. **验证构建**
   ```bash
   # 重新构建并验证
   ```

### 8.2 添加新平台支持

如需支持新的平台类型（如 IoT 设备）：

1. 创建新的时区列表文件：
   ```
   data/prebuild/iot/timezone_list.cfg
   ```

2. 修改 BUILD.gn：
   ```gn
   ohos_prebuilt_etc("iot_timezone_list_cfg") {
     source = "//third_party/tzdata/data/prebuild/iot/timezone_list.cfg"
     module_install_dir = "etc/zoneinfo"
     part_name = "tzdata"
     subsystem_name = "thirdparty"
   }
   
   group("zoneinfo") {
     if (is_standard_system) {
       deps = [ ":iana_tzdata" ]
       if (target_platform == "watch") {
         deps += [ ":watch_timezone_list_cfg" ]
       } else if (target_platform == "iot") {
         deps += [ ":iot_timezone_list_cfg" ]
       } else {
         deps += [ ":timezone_list_cfg" ]
       }
     }
   }
   ```

---

## 9. 总结

| 方面 | 说明 |
|-----|------|
| **构建方式** | 预构建数据文件 + ohos_prebuilt_etc |
| **关键配置** | BUILD.gn 中的条件编译（standard/watch） |
| **安装路径** | /system/etc/zoneinfo/ |
| **版本管理** | 需要同步更新 version 文件和 README.OpenSource |
| **平台适配** | 通过差异化数据文件实现 |
| **维护难度** | 低（无代码编译，纯数据替换） |

