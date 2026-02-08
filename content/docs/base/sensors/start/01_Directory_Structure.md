# 目录结构与模块职责

**适用范围**: 本文档适用于所有需要了解仓库组织结构的人员
**目的**: 说明目录结构、文件组织、模块职责
**关键结论**: 本仓库是一个纯配置型组件，文件结构简单清晰

---

## 顶层目录结构

```
/base/sensors/start
├── LICENSE                      # Apache License 2.0
├── OAT.xml                      # OpenHarmony 测试准入文件
├── README.md                    # 项目说明（英文）
├── README_zh.md                 # 项目说明（中文）
├── bundle.json                  # 组件元数据定义
└── etc/init/
    ├── BUILD.gn                 # GN 构建脚本
    ├── sensors.cfg              # sensors 服务启动配置（非 musl）
    ├── sensors_musl.cfg         # sensors 服务启动配置（musl libc）
    ├── msdp.cfg                 # msdp 服务启动配置（非 musl）
    └── msdp_musl.cfg            # msdp 服务启动配置（musl libc）
```

**统计**: 总共 10 个文件（不含 wiki 目录）

---

## 根目录文件

### LICENSE

**路径**: `/LICENSE`
**类型**: 文本文件
**说明**: Apache License 2.0 开源协议

**相关**: [bundle.json:5](../bundle.json:5) 声明许可证为 Apache License 2.0

### OAT.xml

**路径**: `/OAT.xml`
**类型**: XML 文件
**说明**: OpenHarmony 测试准入文件

**用途**: 定义组件的测试准入标准

### README.md

**路径**: `/README.md`
**类型**: Markdown 文件
**说明**: 项目英文说明

**主要内容**:
- 项目介绍
- 目录结构
- 使用说明
- 依赖仓库列表

**相关文档**:
- [README_zh.md](#readme_zhmd) - 中文版本

**证据**: [README.md:1-60](../README.md:1)

### README_zh.md

**路径**: `/README_zh.md`
**类型**: Markdown 文件
**说明**: 项目中文说明

**用途**: README.md 的中文版本

### bundle.json

**路径**: `/bundle.json`
**类型**: JSON 文件
**说明**: 组件元数据定义

**主要内容**:
- 组件名称和描述
- 版本信息
- 所属子系统
- 系统能力 (syscap)
- 依赖组件
- 构建配置

**关键配置**:
```json
{
  "name": "@ohos/start",
  "version": "3.1",
  "subsystem": "sensors",
  "build": {
    "sub_component": [
      "//base/sensors/start/etc/init:sensors.rc",
      "//base/sensors/start/etc/init:msdp.rc"
    ]
  }
}
```

**证据**: [bundle.json:1-28](../bundle.json:1)

---

## etc/init 目录

**路径**: `/etc/init/`
**职责**: 存放 INIT 系统启动配置文件和 GN 构建脚本

### BUILD.gn

**路径**: `/etc/init/BUILD.gn`
**类型**: GN 构建脚本
**职责**: 定义构建目标，将配置文件安装到系统

**Targets**:
1. `sensors.rc` - sensors 服务启动配置
2. `msdp.rc` - msdp 服务启动配置

**详细说明**: [05_GN_Targets.md](./05_GN_Targets.md)

**证据**: [etc/init/BUILD.gn:1-39](../etc/init/BUILD.gn:1)

### sensors.cfg

**路径**: `/etc/init/sensors.cfg`
**类型**: JSON 配置文件
**职责**: sensors 服务启动配置（非 musl 版本）

**主要配置**:
- 服务名称: `sensors`
- 启动路径: `/system/bin/sa_main /system/profile/sensors.json`
- UID/GID: `sensor` / `[sensor, shell]`
- 权限: 2 个权限
- Boot Job: 创建 `/data/service/el1/public/sensor` 目录

**详细说明**: [appendix/Config_Files.md](./appendix/Config_Files.md#sensorscfg-详解)

**证据**: [etc/init/sensors.cfg:1-24](../etc/init/sensors.cfg:1)

### sensors_musl.cfg

**路径**: `/etc/init/sensors_musl.cfg`
**类型**: JSON 配置文件
**职责**: sensors 服务启动配置（musl libc 版本）

**用途**: 当 GN 变量 `use_musl = true` 时使用

**差异**: TODO: 需要与 `sensors.cfg` 详细对比差异

**证据**: [etc/init/BUILD.gn:19-23](../etc/init/BUILD.gn:19)

### msdp.cfg

**路径**: `/etc/init/msdp.cfg`
**类型**: JSON 配置文件
**职责**: msdp 服务启动配置（非 musl 版本）

**主要配置**:
- 服务名称: `msdp`
- 启动路径: `/system/bin/sa_main /system/profile/msdp.json`
- UID/GID: `msdp` / `[msdp, shell, input, access_token]`
- 权限: 26 个权限，包括 8 个敏感 ACL 权限
- Boot Job: 创建 `/data/service/el1/public/msdp` 目录并启动服务

**详细说明**: [appendix/Config_Files.md](./appendix/Config_Files.md#msdpcfg-详解)

**证据**: [etc/init/msdp.cfg:1-57](../etc/init/msdp.cfg:1)

### msdp_musl.cfg

**路径**: `/etc/init/msdp_musl.cfg`
**类型**: JSON 配置文件
**职责**: msdp 服务启动配置（musl libc 版本）

**用途**: 当 GN 变量 `use_musl = true` 时使用

**差异**: TODO: 需要与 `msdp.cfg` 详细对比差异

**证据**: [etc/init/BUILD.gn:30-34](../etc/init/BUILD.gn:30)

---

## 模块职责划分

### 根目录模块

**职责**:
- 组件元数据定义
- 项目文档
- 许可证声明

**文件**:
- `bundle.json` - 组件定义
- `README.md` / `README_zh.md` - 项目说明
- `LICENSE` - 许可证
- `OAT.xml` - 测试准入

### etc/init 模块

**职责**:
- 提供服务启动配置
- 定义构建目标

**文件**:
- `BUILD.gn` - GN 构建脚本
- `sensors.cfg` / `sensors_musl.cfg` - sensors 服务配置
- `msdp.cfg` / `msdp_musl.cfg` - msdp 服务配置

**输出**:
- `/etc/init/sensors.rc` - 安装后的 sensors 服务配置
- `/etc/init/msdp.rc` - 安装后的 msdp 服务配置

---

## 文件类型分布

| 文件类型 | 数量 | 说明 |
|----------|------|------|
| 文档文件 (.md) | 2 | README.md, README_zh.md |
| 配置文件 (.json) | 1 | bundle.json |
| 许可证文件 | 1 | LICENSE |
| XML 文件 | 1 | OAT.xml |
| GN 构建脚本 | 1 | etc/init/BUILD.gn |
| INIT 配置文件 (.cfg) | 4 | sensors/musl, msdp/musl |
| **总计** | **10** | 不含 wiki 目录 |

---

## 无源代码说明

⚠️ **重要提示**: 本仓库不包含任何源代码文件

### 无源代码证据

通过仓库文件列表可以确认:
- ❌ 无 `.cpp` / `.c` 源代码文件
- ❌ 无 `.h` / `.hpp` 头文件
- ❌ 无 `.ts` / `.js` TypeScript/JavaScript 文件
- ❌ 无 `.java` / `.kt` Java/Kotlin 文件

### 无源代码原因

本组件是一个**纯配置型组件**，其职责仅为:
- 提供服务启动配置文件
- 定义构建目标

实际的业务逻辑实现位于:
- `sensors_sensor` 仓库 - 传感器服务实现
- `sensors_miscdevice` 仓库 - 杂项设备服务实现
- msdp 相关仓库 - msdp 服务实现

**证据**:
- [README.md:24](../README.md:24) - "Service code for sensor and small device services... is sensors_sensor and sensors_miscdevice"
- [README.md:55-57](../README.md:55) - "Repositories Involved: sensors_sensor, sensors_miscdevice"

---

## 依赖关系（文件级）

### BUILD.gn 依赖

```
etc/init/BUILD.gn
    ├── 导入: //build/ohos.gni
    ├── 源文件: sensors.cfg (非 musl) / sensors_musl.cfg (musl)
    └── 源文件: msdp.cfg (非 musl) / msdp_musl.cfg (musl)
```

### bundle.json 依赖

```
bundle.json
    └── 构建子组件:
        ├── //base/sensors/start/etc/init:sensors.rc
        └── //base/sensors/start/etc/init:msdp.rc
```

**证据**: [bundle.json:22-25](../bundle.json:22)

### 运行时依赖（配置文件引用）

```
sensors.cfg
    └── 引用: /system/bin/sa_main
    └── 引用: /system/profile/sensors.json

msdp.cfg
    └── 引用: /system/bin/sa_main
    └── 引用: /system/profile/msdp.json
```

**说明**: `sensors.json` 和 `msdp.json` 不在本仓库，由其他组件生成

---

## 安装目录

### 编译后安装路径

| GN Target | 源文件 | 安装路径 |
|-----------|--------|----------|
| sensors.rc | sensors.cfg 或 sensors_musl.cfg | `/etc/init/sensors.rc` |
| msdp.rc | msdp.cfg 或 msdp_musl.cfg | `/etc/init/msdp.rc` |

**配置逻辑**:
- 当 `use_musl = false`: 使用 `*.cfg`
- 当 `use_musl = true`: 使用 `*_musl.cfg`

**证据**: [etc/init/BUILD.gn:24](../etc/init/BUILD.gn:24) - `relative_install_dir = "init"`

### 运行时使用

**INIT 进程** (`/system/bin/init`) 在启动时:
1. 读取 `/etc/init/sensors.rc` - 启动 sensors 服务
2. 读取 `/etc/init/msdp.rc` - 启动 msdp 服务

**详细流程**: [02_Architecture.md](./02_Architecture.md)

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位和核心能力
- [架构说明](./02_Architecture.md) - 服务启动流程
- [GN Targets](./05_GN_Targets.md) - 构建脚本详解
- [编译产物](./06_Build_Artifacts.md) - 安装路径和加载关系
- [配置文件详解](./appendix/Config_Files.md) - 配置参数完整说明

---

**最后更新**: 2026-02-06
