# 构建指南

## GN 构建配置

### 构建入口文件

| 文件 | 路径 | 说明 |
|------|------|------|
| 根构建文件 | `//base/global/timezone/data/BUILD.gn` | 定义时区数据构建目标 |
| 组件配置 | `bundle.json` | 声明组件元信息 |

### 主要构建目标

#### 1. icu_tzdata (Group Target)

**证据**: `data/BUILD.gn:44-54`

```gn
group("icu_tzdata") {
  if (is_standard_system) {
    deps = [
      ":metaZones",
      ":timezoneTypes",
      ":windowsZones",
      ":zoneinfo64",
    ]
  }
}
```

**说明**: 仅在标准系统 (`is_standard_system=true`) 时生效，聚合所有 ICU 时区数据目标。

#### 2. metaZones (Prebuilt ETC)

**证据**: `data/BUILD.gn:16-21`

```gn
ohos_prebuilt_etc("metaZones") {
  source = "//base/global/timezone/data/prebuild/icu/metaZones.res"
  module_install_dir = "etc/icu_tzdata"
  part_name = "timezone"
  subsystem_name = "global"
}
```

| 属性 | 值 |
|------|------|
| 类型 | prebuilt_etc |
| 源文件 | `data/prebuild/icu/metaZones.res` |
| 安装路径 | `/etc/icu_tzdata/` |
| Part 名称 | timezone |
| 子系统 | global |

#### 3. timezoneTypes (Prebuilt ETC)

**证据**: `data/BUILD.gn:23-28`

```gn
ohos_prebuilt_etc("timezoneTypes") {
  source = "//base/global/timezone/data/prebuild/icu/timezoneTypes.res"
  module_install_dir = "etc/icu_tzdata"
  part_name = "timezone"
  subsystem_name = "global"
}
```

#### 4. windowsZones (Prebuilt ETC)

**证据**: `data/BUILD.gn:30-35`

```gn
ohos_prebuilt_etc("windowsZones") {
  source = "//base/global/timezone/data/prebuild/icu/windowsZones.res"
  module_install_dir = "etc/icu_tzdata"
  part_name = "timezone"
  subsystem_name = "global"
}
```

#### 5. zoneinfo64 (Prebuilt ETC)

**证据**: `data/BUILD.gn:37-42`

```gn
ohos_prebuilt_etc("zoneinfo64") {
  source = "//base/global/timezone/data/prebuild/icu/zoneinfo64.res"
  module_install_dir = "etc/icu_tzdata"
  part_name = "timezone"
  subsystem_name = "global"
}
```

### 构建目标汇总表

| 目标名 | 类型 | 源文件 | 安装路径 | 依赖 |
|--------|------|--------|----------|------|
| icu_tzdata | group | - | - | metaZones, timezoneTypes, windowsZones, zoneinfo64 |
| metaZones | prebuilt_etc | prebuild/icu/metaZones.res | etc/icu_tzdata/ | 无 |
| timezoneTypes | prebuilt_etc | prebuild/icu/timezoneTypes.res | etc/icu_tzdata/ | 无 |
| windowsZones | prebuilt_etc | prebuild/icu/windowsZones.res | etc/icu_tzdata/ | 无 |
| zoneinfo64 | prebuilt_etc | prebuild/icu/zoneinfo64.res | etc/icu_tzdata/ | 无 |

## 编译产物

### 预编译数据文件

| 文件 | 路径 | 大小 | 格式 | 说明 |
|------|------|------|------|------|
| metaZones.res | `data/prebuild/icu/` | ~42KB | ICU binary | 元时区映射 |
| timezoneTypes.res | `data/prebuild/icu/` | ~21KB | ICU binary | 时区类型定义 |
| windowsZones.res | `data/prebuild/icu/` | ~22KB | ICU binary | Windows 时区映射 |
| zoneinfo64.res | `data/prebuild/icu/` | ~145KB | ICU binary | 64位时区信息 |

### 编译工具

| 文件 | 路径 | 说明 |
|------|------|------|
| zic | `data/prebuild/tool/linux/zic` | IANA 时区编译器 |

### 产物安装路径

| 目标路径 | 内容 | 访问权限 |
|----------|------|----------|
| `/etc/icu_tzdata/` | *.res 文件 | 系统只读 |
| `/usr/share/zoneinfo/` | POSIX 时区数据 | 系统只读 |

**证据**: `data/BUILD.gn:18, 25, 32, 39` 中的 `module_install_dir = "etc/icu_tzdata"`

### 组件声明

**证据**: `bundle.json:36-56`

```json
{
  "component": {
    "name": "timezone",
    "subsystem": "global",
    "features": [],
    "adapted_system_type": ["standard"],
    "build": {
      "sub_component": [
        "//base/global/timezone/data:icu_tzdata"
      ]
    }
  }
}
```

| 属性 | 值 |
|------|------|
| 组件名称 | timezone |
| 子系统 | global |
| 适配系统 | standard |
| 子组件 | //base/global/timezone/data:icu_tzdata |

## 构建命令

### 完整构建

```bash
# 使用 OpenHarmony 构建系统
hb build -p //base/global/timezone:timezone
```

### 单独构建

```bash
# 单独构建 icu_tzdata 目标
hb build //base/global/timezone/data:icu_tzdata
```

### 查看构建目标

```bash
# 列出所有时区相关目标
hb set | grep timezone
gn ls //base/global/timezone/
```

## 运行时加载关系

```
系统启动
    │
    ▼
┌─────────────────────────┐
│   加载 /etc/icu_tzdata/ │◄── ICU 库初始化时加载
│       *.res 文件        │
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│ global_i18n 模块访问     │
│ 时区数据 API             │
└─────────────────────────┘
```

## 产物与 Target 映射

| Target | 输出文件 | 运行时路径 | 使用方 |
|--------|----------|------------|--------|
| metaZones | metaZones.res | /etc/icu_tzdata/metaZones.res | ICU 库 |
| timezoneTypes | timezoneTypes.res | /etc/icu_tzdata/timezoneTypes.res | ICU 库 |
| windowsZones | windowsZones.res | /etc/icu_tzdata/windowsZones.res | ICU 库 |
| zoneinfo64 | zoneinfo64.res | /etc/icu_tzdata/zoneinfo64.res | ICU 库 |
| icu_tzdata | (无直接产出) | - | 构建系统 |

## 相关文档

- [架构说明](./01_Architecture.md)
- [使用指南](./04_Usage.md)
- [安全评审](./03_Security.md)
