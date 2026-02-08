# 项目概览

## 项目定位

`interface/sdk-js` 是 OpenHarmony 的 **JavaScript/TypeScript API 声明文件公共仓**，用于存储：

1. **API 声明文件** (`.d.ts`) - 为 OpenHarmony JS/TS 应用提供类型定义
2. **构建工具链** - 处理、解析、检查 API 声明
3. **Kit 工具包声明** - 功能模块的 API 声明

> **核心职责**: 提供声明式 API 定义，不包含运行时实现逻辑

## 关键属性

| 属性 | 值 |
|------|-----|
| 仓库路径 | `interface/sdk-js` |
| 子系统 | sdk |
| SDK 类型 | ets (ArkTS) |
| 许可证 | Apache License 2.0 |
| 适配系统 | mini, small, standard |

## 目录结构

```
sdk-js/
├── api/                          # API 声明文件主目录
│   ├── @ohos.*.d.ts             # 公共 API (683个文件)
│   ├── @system.*.d.ts           # 已废弃 API
│   ├── @internal/               # 内部 API (不对外公开)
│   │   ├── component/ets/       # 组件内部 API
│   │   └── ets/                 # ETS 内部 API
│   ├── config/                  # 类 Web 开发范式
│   └── form/                     # JS 服务卡片
├── kits/                         # Kit 工具包声明 (51个模块)
│   ├── @kit.AbilityKit.d.ts
│   ├── @kit.ArkUI.d.ts
│   └── ...
├── arkts/                        # ArkTS 内置类型声明
│   ├── @arkts.collections.d.ets
│   ├── @arkts.lang.d.ets
│   └── ...
├── build-tools/                  # 构建工具链 (30+工具)
│   ├── api_check_plugin/        # API 规范检查
│   ├── dts_parser/              # d.ts 解析工具
│   ├── collect_application_api/ # 应用 API 收集
│   ├── jsdoc_format_plugin/     # JSDoc 格式检查
│   └── ...
├── BUILD.gn                      # GN 构建入口
├── bundle.json                   # 组件配置
├── interface_config.gni          # 接口配置
└── OAT.xml                      # OSS 审计配置
```

## 核心能力

### 1. API 声明管理

| 能力 | 描述 |
|------|------|
| 公共 API | `@ohos.*.d.ts` - OpenHarmony 公共接口 |
| Kit API | `@kit.*.d.ts` - 功能模块化声明 |
| ArkTS API | `@arkts.*.d.ts` - ArkTS 语言内置 |
| 内部 API | `@internal/` - 内部使用接口 |
| 废弃 API | `@system.*.d.ts` - 已废弃接口 |

### 2. 构建工具支持

| 工具类别 | 功能 |
|---------|------|
| 解析工具 | d.ts 文件解析、AST 处理 |
| 检查工具 | API 规范检查、JSDoc 校验 |
| 收集工具 | 应用 API 使用分析 |
| 比较工具 | SDK 版本差异对比 |
| 转换工具 | ArkUI 标签处理、互操作 |

### 3. SDK 构建

| SDK 类型 | 路径 | 用途 |
|---------|------|------|
| Dynamic SDK | `ohos_dynamic/` | 动态运行时 SDK |
| Static SDK | `ohos_static/` | 静态编译 SDK |

## API 模块分类

### 按子系统分类

| 子系统 | 示例模块 | 文件数量 |
|--------|---------|---------|
| Ability | ability, featureAbility, particleAbility | ~20 |
| Accessibility | accessibility | ~5 |
| Account | account.appAccount, account.osAccount | ~10 |
| Media | media, audio, image | ~15 |
| Network | network, request | ~5 |
| Security | security, userAuth | ~10 |
| ... | ... | ... |

### 按 Kit 分类

| Kit 类别 | 示例 | 用途 |
|----------|------|------|
| AbilityKit | @kit.AbilityKit.d.ts | 能力框架 |
| ArkUI | @kit.ArkUI.d.ts | UI 框架 |
| MediaKit | @kit.MediaKit.d.ts | 媒体能力 |
| NetworkKit | @kit.NetworkKit.d.ts | 网络通信 |
| Security | @kit.SecurityGuardKit.d.ts | 安全能力 |

## 构建流程概述

```
Source Files                    Build Process                    Output
─────────────────────────────────────────────────────────────────────────
api/@ohos.*.d.ts  ──────────►  GN Build (BUILD.gn)  ─────────►  Dynamic SDK
kits/@kit.*.d.ts  ──────────►  Template Processing  ─────────►  Static SDK
arkts/@arkts.*.d.ets ───────►  Filtering & Copying  ─────────►  Intermediate
                                Interop Handling
```

## 相关仓库

| 仓库 | 关系 |
|------|------|
| [interface-sdk_js](https://gitee.com/openharmony/interface_sdk-js) | 主仓库 |
| [ets_frontend](https://gitee.com/openharmony/ets_frontend) | ETS 前端依赖 |
| [runtime_core](https://gitee.com/openharmony/runtime_core) | 运行时核心依赖 |

## 快速开始

### 新增 API 声明

1. 在 `api/` 或 `kits/` 目录创建 `.d.ts` 文件
2. 遵循 JSDoc 规范（`@syscap`, `@since` 等）
3. 提交 PR 进行审查

### 使用构建工具

```bash
# 进入构建工具目录
cd build-tools/

# 安装依赖
npm install

# 运行工具
node ./dts_parser/src/main.ts -N collect -C ./api --output ./
```

## 证据来源

- **项目定位**: `README.md`, `bundle.json`
- **目录结构**: 仓库根目录扫描
- **构建配置**: `BUILD.gn`, `interface_config.gni`
- **Kit 列表**: `kits/` 目录扫描
