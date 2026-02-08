# 项目概览

## 1. 项目定位与边界

### 1.1 组件定位

`resource_management_lite` 是 OpenHarmony **Globalization 子系统**的核心组件，为应用提供**加载多语言 GUI 资源**的能力。

**核心职责**：
- 解析 HAP 包中的 `resources.index` 二进制资源索引文件
- 根据设备配置（语言、区域、设备类型、屏幕密度等）匹配最佳资源
- 提供 C 和 C++ 接口供应用获取字符串、颜色、布局等各类资源

**组件边界**：
- ✅ 本组件职责：资源加载、解析、匹配
- ❌ 不负责：UI 渲染、权限管理、文件系统管理

**证据来源**：
- `README.md`: "The resource management module provides the function of loading multi-language GUI resources"
- `bundle.json`: `"subsystem": "global"`, `"component": "resource_management_lite"`

### 1.2 核心能力

| 能力 | 说明 | 支持情况 |
|------|------|----------|
| 多语言资源加载 | 根据语言/区域匹配资源 | ✅ 支持 |
| 资源 ID 查询 | 通过整数 ID 获取资源 | ✅ 支持 |
| 资源名称查询 | 通过字符串名称获取资源 | ✅ 支持 |
| 限定符匹配 | 支持语言/区域/密度等多维匹配 | ✅ 支持 |
| RTL 布局支持 | 从右到左布局语言检测 | ✅ 支持 |
| 复数字符串 | 根据数量选择正确复数形式 | ✅ 支持 |
| 主题/图案资源 | 获取主题和图案定义 | ✅ 支持 |
| 媒体资源路径 | 获取媒体文件路径 | ✅ 支持 |

### 1.3 运行环境

**适配系统类型**：
- **mini** - 轻量级设备
- **small** - 小型设备

**支持的 Kernel**：
- **liteos_a** - 支持完整功能（C++ 源码）
- **liteos_m** - 支持基础功能（C 源码）
- **Windows (mingw)** - 模拟器支持

**性能指标**（来自 `bundle.json`）：
- **ROM**: 200KB
- **RAM**: 900KB

**证据来源**：
- `bundle.json`: `"adapted_system_type": ["mini", "small"]`
- `BUILD.gn`: `if (ohos_kernel_type == "liteos_a")` / `liteos_m`

### 1.4 关键概念

#### 资源类型 (ResType)

```
基础类型: VALUES, ANIMATION, DRAWABLE, LAYOUT, MENU, MIPMAP, RAW, XML
数值类型: INTEGER, STRING, STRINGARRAY, INTARRAY, BOOLEAN, DIMEN, COLOR, ID
主题类型: THEME, PLURALS, PATTERN
其他: FLOAT, MEDIA, PROF, SVG
```

#### 限定符 (Qualifier)

限定符用于区分不同配置的同类资源：

| KeyType | 说明 | 示例 |
|---------|------|------|
| LANGUAGES | 语言 | zh, en |
| REGION | 地区 | CN, US |
| SCRIPT | 脚本 | Hans, Hant, Latn |
| SCREEN_DENSITY | 屏幕密度 | 160 (mdpi), 320 (xldpi) |
| DIRECTION | 方向 | vertical, horizontal |
| DEVICETYPE | 设备类型 | phone, tablet, tv |
| NIGHTMODE | 夜间模式 | dark, light |

#### 资源 ID 格式

OpenHarmony 资源 ID 采用 32 位整数编码：
```
[31:24] - 模块 ID
[23:16] - 资源类型
[15:0]  - 资源索引
```

**证据来源**：
- `global_utils.h`: `ResType` 枚举定义 (lines 40-65)
- `res_common.h`: `KeyType` 枚举定义 (lines 43-55)

---

## 2. 依赖关系

### 2.1 子系统依赖

```
┌─────────────────────────────────────────────────────┐
│                resource_management_lite              │
├─────────────────────────────────────────────────────┤
│                    依赖关系                          │
│                                                  │
│  ┌─────────────────────────────────────────────┐  │
│  │             global_i18n_lite                │  │
│  │  - LocaleInfo 语言信息                      │  │
│  │  - 复数规则 (PluralFormat)                  │  │
│  │  - 区域匹配算法                             │  │
│  └─────────────────────────────────────────────┘  │
│                                                  │
│  ┌─────────────────────────────────────────────┐  │
│  │           utils_lite (commonlibrary)        │  │
│  │  - 内存管理                                 │  │
│  │  - 字符串操作                               │  │
│  └─────────────────────────────────────────────┘  │
│                                                  │
│  ┌─────────────────────────────────────────────┐  │
│  │      bounds_checking_function (sec_utils)  │  │
│  │  - 安全字符串函数 (strcpy_s 等)            │  │
│  └─────────────────────────────────────────────┘  │
│                                                  │
│  ┌─────────────────────────────────────────────┐  │
│  │                hilog_lite                   │  │
│  │  - 日志输出 (hilog_wrapper.h)               │  │
│  └─────────────────────────────────────────────┘  │
│                                                  │
│  ┌─────────────────────────────────────────────┐  │
│  │                  zlib                       │  │
│  │  - ZIP 文件解压 (minizip)                  │  │
│  └─────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### 2.2 依赖组件清单

| 组件 | 用途 | 必需性 |
|------|------|--------|
| `utils_lite` | 通用工具函数 | 必须 |
| `bounds_checking_function` | 安全字符串函数 | 必须 |
| `global_i18n_lite` | 国际化支持 | 必须 (liteos_a) |
| `hilog_lite` | 日志输出 | 可选 (liteos_a) |
| `zlib` | ZIP 解压 | 可选 (liteos_a) |

**证据来源**：
- `bundle.json`: `"deps": {"components": ["utils_lite", "bounds_checking_function"]}`
- `BUILD.gn`: `public_deps` 包含 `//base/global/i18n_lite/frameworks/i18n:global_i18n`

### 2.3 被依赖关系

本组件被以下子系统使用：
- **全局应用框架** - 获取应用资源
- **UI 框架** - 获取布局、图片、字符串
- **系统服务** - 获取配置信息

---

## 3. 代码统计

| 指标 | 数量 |
|------|------|
| 头文件数 | 17 |
| 源文件数 | 12 |
| 测试文件数 | 26+ |
| 对外 C API | 6 |
| 对外 C++ API | 40+ |
| 资源类型 | 23 |

**证据来源**：
- `frameworks/resmgr_lite/include/` - 头文件统计
- `frameworks/resmgr_lite/src/` - 源文件统计
- `global.h` - C API 统计
- `resource_manager.h` - C++ API 统计

---

## 4. 相关文档

| 文档 | 说明 |
|------|------|
| [README.md](./README.md) | 文档说明 |
| [SUMMARY.md](./SUMMARY.md) | 文档导航 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计 |
| [03_C_API.md](./03_C_API.md) | C API 接口 |
| [04_Cpp_API.md](./04_Cpp_API.md) | C++ API 接口 |
| [05_Build_System.md](./05_Build_System.md) | 构建系统 |
| [06_Security_Analysis.md](./06_Security_Analysis.md) | 安全分析 |
| [07_Troubleshooting.md](./07_Troubleshooting.md) | 常见问题 |
