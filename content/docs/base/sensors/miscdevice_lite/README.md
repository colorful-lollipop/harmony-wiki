# sensors_miscdevice_lite Wiki

## 文档概述

本文档为 OpenHarmony `sensors_miscdevice_lite` 组件的工程 Wiki，旨在帮助开发者理解该组件的定位、架构、API 及构建方式。

## ⚠️ 重要声明

**当前状态**: 本仓库为**元数据/文档仓库**，**不包含源代码实现**。

### 代码证据状态

| 类别 | 状态 | 说明 |
|------|------|------|
| 源代码 (.c/.cpp/.h) | ❌ 不存在 | Git 历史中无源码文件 |
| N-API 实现 | ❌ 不存在 | 无 JS/TS 绑定代码 |
| BUILD.gn 构建 | ❌ 不存在 | 无构建配置 |
| 文档 | ✅ 存在 | README、 LICENSE、 bundle.json 元数据 |

**推断**: 基于 `bundle.json` 元数据和 OpenHarmony 标准架构，该组件应有以下结构，但代码实现位于其他仓库或待实现。

---

## Wiki 覆盖范围

### ✅ 已覆盖（基于文档证据）
- 项目定位与目标系统
- 组件元数据（bundle.json）
- 所属子系统关系
- 系统能力声明

### ❌ 未覆盖（无代码证据）
- N-API 接口清单与调用链
- Native Service 实现
- HAL 抽象层
- 内部模块依赖关系
- 编译产物清单
- 详细安全风险分析

---

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航
├── 01_Overview.md              # 项目概览
├── 02_Architecture.md          # 架构说明（推断）
├── 03_API.md                   # API 文档（N/A）
├── 04_Build.md                 # 构建配置（N/A）
├── 05_Security.md              # 安全评审
├── 06_Troubleshooting.md       # 常见问题
└── appendix/
    └── Callgraphs.md           # 调用链（待补充）
```

---

## 如何贡献

### 方式一：完善本文档
1. Fork 本仓库
2. 编辑 `wiki/` 目录下的 Markdown 文件
3. 提交 PR

### 方式二：补充代码实现
如您知道该组件的实际代码位置或有实现计划，请：

1. 在 [OpenHarmony Gitee](https://gitee.com/openharmony) 创建 Issue 讨论
2. 或提交 PR 添加源代码到本仓库

---

## 相关链接

### 官方资源
- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Pan-sensor 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/泛Sensor子系统.md)
- [sensors_sensor_lite 仓库](https://gitee.com/openharmony/sensors_sensor_lite)

### 系统能力
- `SystemCapability.Sensors.MiscDevice_Lite` - 小器件系统能力

---

## 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2025-02-06 | 1.0 | 初始版本，标记代码缺失状态 |

---

## 联系方式

如需协助，请：

1. 查看 [Gitee Issues](https://gitee.com/openharmony/sensors_miscdevice_lite/issues)
2. 或在 OpenHarmony 社区提问
