# 项目概览

## 项目定位

`base/global/timezone` 是 OpenHarmony 全球化子系统的**时区数据管理模块**，负责提供系统时区数据的**更新、编译、部署**能力。

### 核心能力

| 能力 | 描述 | 实现位置 |
|------|------|----------|
| 时区数据下载 | 从 IANA 官网自动下载最新时区数据 | `tool/update_tool/download_iana.py` |
| 时区数据编译 | 将 IANA 源数据编译为系统可用格式 | `tool/compile_tool/compile.sh` |
| 时区数据部署 | 通过 GN 构建将数据部署到系统 | `data/BUILD.gn` |

### 适用场景

- 定期更新系统时区数据以支持新时区规则
- 编译预定义的时区二进制数据
- 部署时区数据到 OpenHarmony 系统设备

## 运行环境

| 环境 | 要求 |
|------|------|
| Python | 3.x (用于下载脚本) |
| 操作系统 | Linux (推荐 Ubuntu/Debian) |
| 构建工具 | GN, Ninja, Make |
| 依赖项 | OpenHarmony 标准系统 |

## 依赖关系

### 上游依赖

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| IANA Time Zone Database | https://data.iana.org/time-zones/releases/ | 权威时区数据源 |
| ICU 69.1 | OpenHarmony 预编译 | ICU 格式时区数据 |

### 下游依赖

| 依赖模块 | 依赖内容 |
|----------|----------|
| global_i18n | 时区数据访问 |
| global_resmgr | 资源管理 |

## 项目结构

```
/base/global/timezone/
├── data/                      # 时区数据目录
│   ├── BUILD.gn             # GN 构建配置
│   ├── prebuild/            # 预编译数据
│   │   ├── icu/            # ICU 时区数据
│   │   │   ├── metaZones.res
│   │   │   ├── timezoneTypes.res
│   │   │   ├── windowsZones.res
│   │   │   └── zoneinfo64.res
│   │   ├── posix/          # POSIX 时区二进制
│   │   └── tool/linux/     # zic 工具
│   └── iana/               # IANA 源数据 (下载后)
├── tool/                    # 时区管理工具
│   ├── compile_tool/       # 编译工具
│   │   └── compile.sh      # 编译脚本
│   └── update_tool/        # 更新工具
│       └── download_iana.py # 下载脚本
├── README.md               # 英文说明
├── README_zh.md            # 中文说明
├── binary_file_build.md   # 二进制编译指导
├── bundle.json            # 组件配置
└── OAT.xml               # OSS Audit Tool 配置

预编译数据目录结构:
data/prebuild/posix/      # POSIX 时区数据 (安装到 /usr/share/zoneinfo)
data/prebuild/icu/        # ICU 格式时区数据 (安装到 /etc/icu_tzdata)
data/prebuild/tool/linux/zic  # zic 时区编译工具
```

## 关键概念

### IANA Time Zone Database

[IANA Time Zone Database](https://data.iana.org/time-zones/releases/) 是全球权威的时区数据源，包含：

- `tzdata`: 时区定义数据 (如 `asia`, `europe` 等区域文件)
- `tzcode`: 时区编译工具源代码

版本号格式: `YYYYx` (如 `2023c`, `2023d` 等)

### zic 工具

zic (Zone Information Compiler) 是 IANA 提供的时区数据编译器，用于将时区源数据编译为二进制格式。

### ICU 时区数据

ICU (International Components for Unicode) 格式的时区数据，包含：

| 文件 | 用途 |
|------|------|
| metaZones.res | 元时区信息 |
| timezoneTypes.res | 时区类型定义 |
| windowsZones.res | Windows 时区映射 |
| zoneinfo64.res | 64位时区信息 |

## 相关资源

- **代码仓库**: https://gitee.com/openharmony/global_timezone
- **IANA 官网**: https://data.iana.org/time-zones/releases/
- **ICU 项目**: https://icu.unicode.org/
- **OpenHarmony 文档**: https://gitee.com/openharmony/docs
