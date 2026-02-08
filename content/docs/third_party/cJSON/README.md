# cJSON - OpenHarmony 第三方库文档

## 库概览

| 属性 | 值 |
|-----|-----|
| **库名称** | cJSON |
| **版本** | OH: 3.1 / Upstream: 1.7.19 |
| **许可证** | MIT License |
| **上游地址** | https://github.com/DaveGamble/cJSON |
| **功能** | 超轻量级 ANSI C JSON 解析器 |

## OpenHarmony 适配概述

cJSON 是 OpenHarmony 系统中用于 JSON 数据处理的轻量级基础库。该库在 OH 中的集成具有以下特点：

### 主要适配内容

1. **构建系统适配**：使用 GN 构建系统替代上游的 CMake
2. **嵌套深度限制**：调整为 128（上游默认 1000），适合嵌入式场景
3. **栈保护**：标准 OH 模式启用 PAC 栈保护
4. **安装配置**：动态库安装到 system 和 updater 分区

### Patch 状态

**本库未应用任何 Patch**。这表明 cJSON 本身具有良好的跨平台兼容性，无需针对 OH 进行代码修改。

## 文档导航

### 必读文档

| 文档 | 说明 |
|-----|------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议和文档概览 |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配详解 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用场景 |

### 详细信息

| 文档 | 说明 |
|-----|------|
| [01_Overview.md](01_Overview.md) | 原始库功能和 OH 定位 |
| [02_Patches.md](02_Patches.md) | Patch 分析（本库无 Patch） |
| [05_API_Differences.md](05_API_Differences.md) | API 差异分析 |
| [06_Security.md](06_Security.md) | 安全风险分析 |

## 在 OpenHarmony 中的使用

### 主要使用场景

- **Bundle 管理**：应用配置文件的解析
- **资源工具**：资源配置工具中的 JSON 处理
- **IDE 预览器**：开发工具中的数据交换

### 依赖方式

```c
// 头文件引用
#include <cjson/cJSON.h>
```

### 链接方式

- **静态链接**：`//third_party/cJSON:cjson_static`
- **动态链接**：`//third_party/cJSON:cjson`

## 版本信息

| 版本 | 日期 | 说明 |
|-----|-----|------|
| 3.1 | - | OpenHarmony 版本 |
| 1.7.19 | - | 上游最新版本 |

## 相关资源

- [上游项目地址](https://github.com/DaveGamble/cJSON)
- [OH bundle.json](bundle.json)
- [BUILD.gn 构建配置](BUILD.gn)
