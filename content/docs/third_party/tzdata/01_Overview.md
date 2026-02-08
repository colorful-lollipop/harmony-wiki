# 01_Overview.md - 原始库简介与 OpenHarmony 定位

## 1. 原始库简介

### 1.1 库基本信息

| 属性 | 值 |
|-----|-----|
| **全称** | IANA Time Zone Database |
| **简称** | tzdata / tzdb / zoneinfo |
| **版本** | 2025b（声明）/ 2024a（version 文件） |
| **许可证** | Public Domain / BSD 3-Clause |
| **上游地址** | https://github.com/eggert/tz |
| **官方主页** | https://www.iana.org/time-zones |

### 1.2 功能描述

IANA Time Zone Database（通常称为 tz、tzdb 或 zoneinfo）包含代表全球许多代表性位置本地时间历史的代码和数据。它定期更新以反映政治机构对时区边界、UTC 偏移和夏令时规则的更改。

**核心组成**：

1. **时区规则数据文件**（纯文本）
   - `africa`, `asia`, `australasia`, `europe`, `northamerica`, `southamerica` - 各大洲时区规则
   - `backward` - 向后兼容的旧时区名称
   - `backzone` - 历史时区数据
   - `etcetera` - 特殊时区（如 UTC、GMT）
   - `factory` - 默认时区规则

2. **编译工具**（C 语言）
   - `zic` (Zone Information Compiler) - 将文本规则编译为二进制时区文件
   - `zdump` - 时区数据转储工具
   - `tzselect` - 时区选择脚本

3. **运行时库代码**（C 语言）
   - `localtime.c` - 本地时间转换核心逻辑
   - `asctime.c` - 时间格式化
   - `strftime.c` - 字符串格式化时间
   - `difftime.c` - 时间差计算

### 1.3 数据结构

```
原始库结构：
├── 时区规则文件（文本）
│   ├── africa          # 非洲时区规则
│   ├── asia            # 亚洲时区规则
│   ├── australasia     # 澳洲时区规则
│   ├── europe          # 欧洲时区规则
│   ├── northamerica    # 北美时区规则
│   ├── southamerica    # 南美时区规则
│   └── ...
├── 编译工具
│   ├── zic.c           # 时区编译器
│   ├── zdump.c         # 时区数据查看器
│   └── tzselect.ksh    # 时区选择脚本
├── 运行时库
│   ├── localtime.c     # 本地时间处理
│   ├── asctime.c       # ASCII 时间转换
│   └── strftime.c      # 时间格式化
└── 数据格式定义
    ├── tzfile.h        # 时区文件格式定义
    └── tzfile.5        # 时区文件格式文档
```

### 1.4 许可证说明

根据 `LICENSE` 文件和 `README`：

- 本项目文件处于 **Public Domain**（公有领域），2009-05-17 由 Arthur David Olson 明确声明
- 部分文件采用 **BSD 许可证**，具体详见 `LICENSE` 文件
- 允许自由使用、修改和分发，无需授权

---

## 2. OpenHarmony 中的定位和作用

### 2.1 OH 中的独特地位

tzdata 在 OpenHarmony 中具有**特殊地位**：

| 特点 | 说明 |
|-----|------|
| **纯数据组件** | 不参与代码编译链接，仅提供时区数据资源 |
| **无 Patch** | 本库是 OH 第三方库中极少数无 Patch 的库之一 |
| **预构建交付** | 使用预编译的二进制数据，而非源码编译 |
| **系统级基础设施** | 为整个系统提供时区信息支持 |

### 2.2 OH 中的使用方式

在 OpenHarmony 中，tzdata 的使用方式与常规 Linux 系统类似但不完全相同：

**传统 Linux 系统**：
```
源码 → zic 编译 → /usr/share/zoneinfo/ → 应用通过 libc 读取
```

**OpenHarmony 系统**：
```
预构建数据 → 打包 → /system/etc/zoneinfo/ → ICU/应用读取
                            ↓
                    /system/etc/icu_tzdata/ (ICU 格式)
```

### 2.3 OH 中的目录结构

```
third_party/tzdata/
├── data/
│   ├── BUILD.gn                    # OH 构建配置
│   └── prebuild/
│       ├── posix/
│       │   ├── tzdata              # 预构建的 IANA 时区数据库
│       │   └── timezone_list.cfg   # 时区列表（标准设备）
│       └── wearable/
│           └── timezone_list.cfg   # 时区列表（穿戴设备精简版）
├── bundle.json                     # OH 组件配置
├── README.OpenSource               # 开源说明
├── version                         # 版本号（2024a）
└── [原始上游源码和数据文件]...
```

### 2.4 在 OH 架构中的位置

```
┌────────────────────────────────────────────────────────────────────┐
│                         应用层                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                │
│  │  系统应用    │  │  三方应用    │  │  系统服务    │                │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌────────────────────────────────────────────────────────────────────┐
│                      框架层 (Framework)                             │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    ICU (International Components for Unicode) │  │
│  │  - 时区转换 API                                              │  │
│  │  - 读取 /system/etc/icu_tzdata/zoneinfo64.res               │  │
│  │  - 回退 /system/etc/zoneinfo/                                │  │
│  └─────────────────────────────────────────────────────────────┘  │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    Time Service (global/timezone)            │  │
│  └─────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────────────────┐
│                      系统资源层                                      │
│  ┌──────────────────────┐  ┌──────────────────────┐               │
│  │   tzdata (本库)       │  │   icu_tzdata         │               │
│  │   /system/etc/       │  │   /system/etc/       │               │
│  │       zoneinfo/      │  │       icu_tzdata/    │               │
│  │       - tzdata       │  │       - zoneinfo64.res│               │
│  │       - timezone_    │  │                      │               │
│  │         list.cfg     │  │                      │               │
│  └──────────────────────┘  └──────────────────────┘               │
└────────────────────────────────────────────────────────────────────┘
```

### 2.5 与其他时区相关组件的关系

| 组件 | 路径 | 与本库关系 | 用途 |
|-----|------|-----------|------|
| **tzdata** | `third_party/tzdata` | 本文档主体 | 提供 IANA 原生格式时区数据 |
| **icu_tzdata** | `base/global/timezone` | 互补 | 提供 ICU 二进制格式时区数据 |
| **icu** | `third_party/icu` | 使用者 | 国际化库，读取时区数据 |

**说明**：
- ICU 优先使用 `/system/etc/icu_tzdata/` 下的数据
- ICU 配置了 `DISTRO_TZDATA_DIR` 指向 `/system/etc/tzdata_distro`，可能作为回退
- tzdata 提供的是标准的 IANA 格式，可被标准 C 库或直接使用 zic 编译的应用使用

---

## 3. 关键差异总结

| 方面 | 原始上游库 | OpenHarmony 适配 |
|-----|-----------|-----------------|
| **编译方式** | 使用 Makefile 从源码编译 | 使用预构建二进制数据 |
| **Patch** | 无（上游标准） | 无 Patch 文件 |
| **数据格式** | 文本规则 + 二进制输出 | 仅二进制数据 |
| **安装路径** | /usr/share/zoneinfo/ | /system/etc/zoneinfo/ |
| **使用方式** | 系统标准组件 | 通过 ICU 间接使用 |
| **平台适配** | 通用 | 支持 wearable 差异化 |

---

## 4. 维护注意事项

### 4.1 版本更新

tzdata 需要**定期更新**以反映全球时区规则的变化：

1. **更新触发条件**：
   - 国家/地区更改时区规则
   - 夏令时政策变更
   - IANA 发布新版本

2. **OH 更新流程**（推测）：
   - 从上游获取新版本时区规则
   - 使用 zic 编译为二进制格式
   - 更新 `data/prebuild/posix/tzdata`
   - 更新 `version` 文件

### 4.2 已知问题

- **版本号不一致**：`version` 文件显示 2024a，但 `README.OpenSource` 声明 2025b
  - 可能原因：预构建数据未同步更新
  - 建议：确认预构建数据的实际版本

---

## 5. 参考资料

- [IANA Time Zone Database](https://www.iana.org/time-zones)
- [上游 GitHub 仓库](https://github.com/eggert/tz)
- [时区数据库理论说明](../theory.html)
- [ICU 时区处理文档](https://unicode-org.github.io/icu/userguide/datetime/timezone/)

