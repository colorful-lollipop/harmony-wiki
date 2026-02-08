# JSFramework (OpenHarmony 适配版)

> 基于 Apache Weex 的移动端跨平台 UI 框架在 OpenHarmony 中的适配实现

## 库信息

| 属性            | 值                      |
| --------------- | ----------------------- |
| **OH 组件名称** | jsframework             |
| **上游原始库**  | Apache Weex             |
| **上游版本**    | 0.30.0                  |
| **OH 适配版本** | 3.1                     |
| **许可证**      | Apache License V2.0     |
| **上游地址**    | https://weex.apache.org |
| **OH 子系统**   | thirdparty              |
| **bundle.json** | @ohos/jsframework       |

## 概述

JSFramework 是 OpenHarmony 中 JavaScript UI 框架的核心运行时，基于 Apache Weex 0.30.0 版本，针对 OpenHarmony 系统进行了深度适配。

### 核心功能

- **JS Bundle 解析**: 解析和执行 Weex 格式的 JavaScript 页面代码
- **虚拟 DOM**: 高性能虚拟 DOM 实现，支持细粒度更新
- **响应式系统**: 观察者模式的响应式数据绑定
- **DOM 操作**: 完整的 DOM API 实现，支持组件树操作
- **事件管理**: 事件冒泡、捕获和拦截机制

### OpenHarmony 特有功能

与上游 Weex 相比，本适配版本包含以下 OH 特有功能：

1. **50+ OH 原生组件**: 包括 XComponent、Camera、Web、Canvas 等系统级组件
2. **13 个系统模块**: system.\* 系列模块 + ohos.animator、digitalCrown
3. **Ark 字节码生成**: 支持 Ark 引擎的 `.abc` 字节码格式
4. **DPI/国际化适配**: 完整的设备适配层
5. **媒体查询**: 支持设备类型和屏幕状态的动态查询

## 文档导航

### 快速开始

- [README.md](./README.md) - 快速了解本 wiki 的结构和内容

### 核心文档

| 文档                                                 | 说明                  | 优先级 |
| ---------------------------------------------------- | --------------------- | ------ |
| [01_Overview.md](./01_Overview.md)                   | 库概览和 OH 适配概述  | ⭐⭐⭐ |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建配置详解 | ⭐⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md)             | 依赖关系和使用场景    | ⭐⭐⭐ |
| [05_API_Differences.md](./05_API_Differences.md)     | OH 新增 API 详细说明  | ⭐⭐   |

### 辅助文档

| 文档                                          | 说明             |
| --------------------------------------------- | ---------------- |
| [\_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目初始评估报告 |
| [\_work/NOTES.md](./_work/NOTES.md)           | 分析过程记录     |
| [\_work/PLAN.md](./_work/PLAN.md)             | 任务进度跟踪     |

## 快速查找

### 按功能查找

| 功能领域 | 相关文档                                             |
| -------- | ---------------------------------------------------- |
| 构建系统 | [03_Build_Integration.md](./03_Build_Integration.md) |
| 依赖关系 | [04_Usage_in_OH.md](./04_Usage_in_OH.md)             |
| API 变更 | [05_API_Differences.md](./05_API_Differences.md)     |
| 安全评估 | [06_Security.md](./06_Security.md)                   |

### 按场景查找

| 场景         | 相关章节                                                        |
| ------------ | --------------------------------------------------------------- |
| 理解框架结构 | [01_Overview.md](./01_Overview.md) - 架构设计                   |
| 构建产物说明 | [03_Build_Integration.md](./03_Build_Integration.md) - 构建产物 |
| 集成到新模块 | [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 集成指南             |
| 升级注意事项 | [05_API_Differences.md](./05_API_Differences.md) - 升级建议     |

## 关键路径

```
third_party/jsframework/
├── BUILD.gn              # OH 构建配置入口
├── runtime/              # 框架核心源码
│   ├── main/            # 主框架逻辑
│   │   ├── app/         # 应用管理
│   │   ├── extend/      # OH 扩展模块
│   │   ├── manage/      # 事件和实例管理
│   │   ├── model/       # 编译器和 DOM
│   │   ├── page/        # 页面相关
│   │   └── reactivity/  # 响应式系统
│   ├── preparation/     # 框架初始化
│   └── vdom/            # 虚拟 DOM 实现
├── package.json          # NPM 依赖配置
└── wiki/                 # 本文档目录
```

## 维护信息

### 最后更新

- **日期**: 2025-02-08
- **版本**: 1.0
- **状态**: 进行中

### 贡献指南

如需更新本文档，请：

1. 修改对应 Markdown 文件
2. 在 `_work/NOTES.md` 中记录变更
3. 更新版本号和日期

### 相关资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [Weex 官方文档](https://weex.apache.org)
- [Ace Engine 源码](https://gitee.com/openharmony/ace_engine)
