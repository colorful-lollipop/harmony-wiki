# GN Targets 与构建配置

**适用范围**: 本文档适用于所有需要了解构建配置和依赖关系的人员
**目的**: 说明 GN 目标、构建配置、依赖关系
**关键结论**: 本仓库定义两个 `ohos_prebuilt_etc` targets，用于安装服务启动配置文件

---

## GN 构建系统概述

### 构建入口

**文件路径**: `/bundle.json`
**构建入口**: [bundle.json:22-25](../bundle.json:22)

```json
{
  "build": {
    "sub_component": [
      "//base/sensors/start/etc/init:sensors.rc",
      "//base/sensors/start/etc/init:msdp.rc"
    ]
  }
}
```

**说明**: bundle.json 声明两个构建子组件，GN 编译时会构建这两个 targets

---

## GN Target 列表

### Target 1: sensors.rc

**Target 路径**: `//base/sensors/start/etc/init:sensors.rc`
**文件位置**: `/etc/init/BUILD.gn:18-27`

#### GN Target 定义

```gn
ohos_prebuilt_etc("sensors.rc") {
  if (use_musl) {
    source = "sensors_musl.cfg"
  } else {
    source = "sensors.cfg"
  }
  relative_install_dir = "init"
  subsystem_name = "sensors"
  part_name = "start"
}
```

**完整代码**: [etc/init/BUILD.gn:18-27](../etc/init/BUILD.gn:18)

#### Target 属性说明

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_prebuilt_etc` | 预构建 ETC 文件模板 |
| source | 条件选择 | 根据 `use_musl` 选择源文件 |
| relative_install_dir | "init" | 安装到 `/etc/init/` |
| subsystem_name | "sensors" | 所属子系统 |
| part_name | "start" | 所属组件 |

#### 条件编译

**GN 变量**: `use_musl`

| use_musl 值 | 源文件 | 说明 |
|-------------|--------|------|
| false (默认) | `sensors.cfg` | 使用非 musl 版本 |
| true | `sensors_musl.cfg` | 使用 musl libc 版本 |

**代码证据**: [etc/init/BUILD.gn:19-23](../etc/init/BUILD.gn:19)

#### 安装路径

```
源文件: etc/init/sensors.cfg 或 etc/init/sensors_musl.cfg
    ↓
GN 构建
    ↓
安装到: /etc/init/sensors.rc
```

**说明**: `relative_install_dir = "init"` → 安装到 `/etc/init/`，目标文件名为 target 名称

---

### Target 2: msdp.rc

**Target 路径**: `//base/sensors/start/etc/init:msdp.rc`
**文件位置**: `/etc/init/BUILD.gn:29-38`

#### GN Target 定义

```gn
ohos_prebuilt_etc("msdp.rc") {
  if (use_musl) {
    source = "msdp_musl.cfg"
  } else {
    source = "msdp.cfg"
  }
  relative_install_dir = "init"
  subsystem_name = "msdp"
  part_name = "start"
}
```

**完整代码**: [etc/init/BUILD.gn:29-38](../etc/init/BUILD.gn:29)

#### Target 属性说明

| 属性 | 值 | 说明 |
|------|-----|------|
| 类型 | `ohos_prebuilt_etc` | 预构建 ETC 文件模板 |
| source | 条件选择 | 根据 `use_musl` 选择源文件 |
| relative_install_dir | "init" | 安装到 `/etc/init/` |
| subsystem_name | "msdp" | 所属子系统 |
| part_name | "start" | 所属组件 |

**注意**: `subsystem_name` 为 "msdp"，与 sensors target 不同

#### 条件编译

**GN 变量**: `use_musl`

| use_musl 值 | 源文件 | 说明 |
|-------------|--------|------|
| false (默认) | `msdp.cfg` | 使用非 musl 版本 |
| true | `msdp_musl.cfg` | 使用 musl libc 版本 |

**代码证据**: [etc/init/BUILD.gn:30-34](../etc/init/BUILD.gn:30)

#### 安装路径

```
源文件: etc/init/msdp.cfg 或 etc/init/msdp_musl.cfg
    ↓
GN 构建
    ↓
安装到: /etc/init/msdp.rc
```

---

## GN 构建配置

### BUILD.gn 完整分析

**文件路径**: `/etc/init/BUILD.gn`

#### 导入语句

```gn
import("//build/ohos.gni")
```

**说明**: 导入 OpenHarmony 构建系统的公共定义

**代码位置**: [etc/init/BUILD.gn:14](../etc/init/BUILD.gn:14)

#### 构建配置汇总表

| Target | 源文件 (非 musl) | 源文件 (musl) | 安装路径 | subsystem | part |
|--------|------------------|---------------|----------|-----------|------|
| sensors.rc | sensors.cfg | sensors_musl.cfg | /etc/init/sensors.rc | sensors | start |
| msdp.rc | msdp.cfg | msdp_musl.cfg | /etc/init/msdp.rc | msdp | start |

---

## 依赖关系

### Bundle.json 依赖声明

**文件路径**: `/bundle.json`

#### 组件依赖

```json
"deps": {
  "components": [],
  "third_party": []
}
```

**说明**: bundle.json 声明无组件和第三方库依赖

**代码位置**: [bundle.json:17-20](../bundle.json:17)

**实际依赖**: 虽然声明无依赖，但实际运行时依赖:
- `sensors_sensor` 组件
- `sensors_miscdevice` 组件
- `msdp` 相关组件

这些依赖通过 SA 配置文件 (`sensors.json`, `msdp.json`) 间接引用，不在本仓库

### GN Target 依赖图

```
bundle.json (构建入口)
    ├─→ //base/sensors/start/etc/init:sensors.rc
    │       ├─→ ohos.gni (公共 GN 模板)
    │       ├─→ sensors.cfg (use_musl=false)
    │       └─→ sensors_musl.cfg (use_musl=true)
    │
    └─→ //base/sensors/start/etc/init:msdp.rc
            ├─→ ohos.gni (公共 GN 模板)
            ├─→ msdp.cfg (use_musl=false)
            └─→ msdp_musl.cfg (use_musl=true)
```

**说明**:
- GN targets 之间无依赖关系
- 两个 target 可并行构建
- 仅依赖公共 GN 模板 `ohos.gni`

---

## 配置文件映射

### 源文件 → 安装文件映射

| GN 变量 | sensors.rc | msdp.rc |
|---------|-----------|---------|
| use_musl=false | sensors.cfg → /etc/init/sensors.rc | msdp.cfg → /etc/init/msdp.rc |
| use_musl=true | sensors_musl.cfg → /etc/init/sensors.rc | msdp_musl.cfg → /etc/init/msdp.rc |

### 编译流程

```
1. GN 解析 bundle.json
    ↓
2. 构建子组件:
   ├─ //base/sensors/start/etc/init:sensors.rc
   └─ //base/sensors/start/etc/init:msdp.rc
    ↓
3. 读取 etc/init/BUILD.gn
    ↓
4. 评估 use_musl GN 变量
    ├─ false: 使用 *.cfg
    └─ true: 使用 *_musl.cfg
    ↓
5. 应用 ohos_prebuilt_etc 模板
    ├─ 复制源文件
    ├─ 设置安装路径
    └─ 设置元数据
    ↓
6. Ninja 执行构建
    ↓
7. 安装到系统镜像
    ├─ /etc/init/sensors.rc
    └─ /etc/init/msdp.rc
```

---

## 构建变量

### use_musl

**类型**: Boolean GN 变量
**默认值**: false
**作用**: 选择使用 musl libc 或默认 libc

**影响范围**:
- sensors.rc target 源文件选择
- msdp.rc target 源文件选择

**使用方式**:
```bash
# 默认构建（非 musl）
./build.sh --product-name <product>

# 使用 musl 构建
./build.sh --product-name <product> --ccache --gn-args use_musl=true
```

**代码证据**:
- sensors.rc: [etc/init/BUILD.gn:19-23](../etc/init/BUILD.gn:19)
- msdp.rc: [etc/init/BUILD.gn:30-34](../etc/init/BUILD.gn:30)

---

## GN 模板: ohos_prebuilt_etc

### 模板说明

**模板定义**: `//build/ohos.gni` (外部模板，不在本仓库)

**用途**: 将预构建文件安装到系统的指定目录

**常用参数**:
| 参数 | 说明 |
|------|------|
| source | 源文件路径 |
| relative_install_dir | 安装目录（相对于根目录） |
| subsystem_name | 所属子系统 |
| part_name | 所属组件 |

**输出**:
- 目标文件名 = target 名称
- 安装路径 = `/` + `relative_install_dir` + `/` + 目标文件名

**示例**:
```gn
ohos_prebuilt_etc("sensors.rc") {
  source = "sensors.cfg"
  relative_install_dir = "init"
  # 输出: /etc/init/sensors.rc
}
```

---

## 构建产物

### 编译输出文件

| Target | 输出文件 | 安装位置 |
|--------|----------|----------|
| sensors.rc | sensors.rc | /etc/init/sensors.rc |
| msdp.rc | msdp.rc | /etc/init/msdp.rc |

### 构建产物验证

**构建完成后检查**:
```bash
# 检查安装目录
ls -l /etc/init/sensors.rc
ls -l /etc/init/msdp.rc

# 检查配置文件内容
cat /etc/init/sensors.rc
cat /etc/init/msdp.rc
```

---

## 构建命令

### 完整构建命令

```bash
# 进入 OpenHarmony 根目录
cd <openharmony_root>

# 编译 sensors_start 组件（包含在 sensors 子系统）
./build.sh --product-name <product> --build-target sensors_start

# 或编译整个传感器子系统
./build.sh --product-name <product> --build-target sensors
```

### 指定 use_musl 构建

```bash
# 使用 musl libc 构建
./build.sh --product-name <product> --gn-args use_musl=true
```

---

## 常见问题

### Q: 为什么两个 targets 都安装到 /etc/init/？

A: 因为两个 targets 都配置了 `relative_install_dir = "init"`，目标文件名分别为 `sensors.rc` 和 `msdp.rc`，不会冲突。

**代码证据**: [etc/init/BUILD.gn:24](../etc/init/BUILD.gn:24)

### Q: sensors.rc 和 msdp.rc 的 subsystem_name 为什么不同？

A:
- sensors.rc 的 subsystem_name 是 "sensors" [等价: etc/init/BUILD.gn:25](../etc/init/BUILD.gn:25)
- msdp.rc 的 subsystem_name 是 "msdp" [等价: etc/init/BUILD.gn:36](../etc/init/BUILD.gn:36)

这是因为两个服务属于不同的子系统，虽然都在 `start` 组件中。

### Q: use_musl 的具体差异是什么？

A: TODO: 需要对比 `sensors.cfg` vs `sensors_musl.cfg`, `msdp.cfg` vs `msdp_musl.cfg` 的内容差异。

### Q: 为什么使用 ohos_prebuilt_etc 而不是 ohos_etc？

A: `ohos_prebuilt_etc` 用于预构建文件（直接复制），而 `ohos_etc` 用于需要编译的文件（如需要变量替换的模板文件）。

本仓库的 `.cfg` 文件是静态配置文件，不需要编译，因此使用 `ohos_prebuilt_etc`。

---

## 相关跳转

- [目录结构](./01_Directory_Structure.md) - 文件组织
- [架构说明](./02_Architecture.md) - 构建流程
- [编译产物](./06_Build_Artifacts.md) - 详细产物说明
- [配置文件详解](./appendix/Config_Files.md) - 配置文件内容

---

**最后更新**: 2026-02-06
