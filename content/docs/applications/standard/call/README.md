# applications_call Wiki

## 项目概述

**applications_call** 是 OpenHarmony 系统原生通话管理应用，提供完整的通话功能。

### 核心能力

| 功能模块 | 描述 |
|---------|------|
| 语音通话 | 支持语音电话拨打、接听、挂断 |
| 视频通话 | 支持视频通话功能 |
| 通话设置 | 通话相关参数配置 |
| 移动网络设置 | APN、网络模式配置 |
| SIM卡管理 | SIM 卡状态和信息管理 |
| 紧急拨号 | 紧急呼叫功能 |

## 快速开始

### 环境要求

- OpenHarmony SDK 9+
- hvigor 构建工具
- Node.js 环境

### 构建命令

```bash
# 安装依赖
npm install

# 构建 HAP 包
hb build -f

# 或使用 hvigor
hvigor --mode module -p product=default assembleHap --strict-mode=true
```

### 运行

1. 部署到设备：`hdc install`
2. 或通过 IDE 安装运行

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                    # 全站导航
├── 01_Overview.md              # 项目概述
├── 02_Architecture.md           # 架构说明
├── 03_API_Reference.md         # API 参考
├── 04_Build_Guide.md           # 构建指南
├── 05_Security_Review.md       # 安全评审
├── 06_Module_Details.md        # 模块详解
└── appendix/
    ├── Callgraphs.md           # 调用图谱
    └── Config_Flags.md         # 配置项说明
```

## 阅读路线

### 新人入门
1. [项目概述](01_Overview.md)
2. [架构说明](02_Architecture.md)
3. [快速开始](#快速开始)

### 开发者参考
1. [API 参考](03_API_Reference.md)
2. [模块详解](06_Module_Details.md)
3. [构建指南](04_Build_Guide.md)

### 安全相关
1. [安全评审](05_Security_Review.md)

## 版本信息

- **当前版本**: 1.0.4.032
- **API Level**: 9
- **目标 API Level**: 9
- **许可协议**: Apache-2.0

## 相关资源

- 源码仓库: [applications_call](https://gitee.com/openharmony/applications_call)
- OpenHarmony 官网: https://www.openharmony.cn/
- API 文档: https://developer.harmonyos.com
