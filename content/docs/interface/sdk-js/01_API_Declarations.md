# API 声明文件

## 概述

本章节描述 `api/` 目录下的 API 声明文件组织方式、文件结构与规范。

## 目录结构

```
api/
├── @ohos.*.d.ts              # 公共 API 声明 (683个文件)
├── @system.*.d.ts            # 已废弃 API
├── @internal/                 # 内部 API (构建时过滤)
│   ├── component/ets/       # 组件内部 API (139个文件)
│   └── ets/                  # ETS 内部 API
├── config/                    # 类 Web 开发范式配置
│   ├── css/
│   ├── hml/
│   └── ...
└── form/                      # JS 服务卡片
    ├── action/
    ├── css/
    └── hml/
```

## API 类型分类

### 1. 公共 API (@ohos.*)

**定位**: OpenHarmony 公共接口，供所有应用使用

**命名规范**: `@ohos.{module}.d.ts`

**示例**:

| 模块 | 文件 | 功能 |
|------|------|------|
| ability | `@ohos.ability.ability.d.ts` | 能力基础接口 |
| account | `@ohos.account.appAccount.d.ts` | 应用账号 |
| accessibility | `@ohos.accessibility.d.ts` | 无障碍服务 |
| UiTest | `@ohos.UiTest.d.ts` | UI 测试 |

**证据**: `api/` 目录下 683 个 `.d.ts` 文件

### 2. Kit API (@kit.*)

**定位**: 功能模块化声明，按 Kit 组织

**命名规范**: `@kit.{KitName}.d.ts`

**Kit 列表**:

| Kit 名称 | 用途 | 关键接口 |
|----------|------|---------|
| AbilityKit | 能力框架 | Ability, Context |
| ArkUI | UI 框架 | Component, Attribute |
| MediaKit | 媒体能力 | Player, Recorder |
| AudioKit | 音频能力 | AudioRenderer, AudioCapturer |
| NetworkKit | 网络 | HttpRequest |
| NotificationKit | 通知 | NotificationRequest |
| SecurityGuardKit | 安全 | SecurityComponent |

**证据**: `kits/` 目录下 51 个 `@kit.*.d.ts` 文件

### 3. ArkTS 内置类型 (@arkts.*)

**定位**: ArkTS 语言内置类型声明

**文件列表**:

| 文件 | 用途 | 大小 |
|------|------|------|
| `@arkts.collections.d.ets` | 集合类型 | 624KB |
| `@arkts.collections.static.d.ets` | 静态集合 | 11KB |
| `@arkts.lang.d.ets` | 语言基础 | 1.4KB |
| `@arkts.math.Decimal.d.ets` | Decimal 类型 | 134KB |
| `@arkts.utils.d.ets` | 工具类型 | 45KB |

**证据**: `arkts/` 目录下 5 个声明文件

### 4. 内部 API (@internal)

**定位**: 内部使用接口，不对外公开

**构建时处理**:
- 通过 `ohos_copy_internal` 模板复制
- 根据 `remove_list.json` 规则过滤
- 公开构建时部分文件被移除

**证据**: `BUILD.gn:82-129`

### 5. 已废弃 API (@system.*)

**定位**: 已停止维护的接口

**示例**:
- `@system.app.d.ts`
- `@system.router.d.ts`
- `@system.prompt.d.ts`

**证据**: `interface_config.gni:15-22`

## d.ts 文件结构规范

### 标准结构

```typescript
/**
 * Copyright (c) 2022-2023 Huawei Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0
 */

/**
 * @file
 * @kit AbilityKit
 */

/*** if arkts dynamic */
import { Xxx as _Xxx } from './module/xxx';
/*** endif */

/*** if arkts static */
import { Xxx as _Xxx } from './module/xxx';
/*** endif */

/**
 * 模块说明
 * @namespace moduleName
 * @syscap SystemCapability.Xxx.Xxx
 * @since 9
 */
declare namespace moduleName {
  /**
   * 接口/类型说明
   * @typedef { _Xxx }
   * @syscap SystemCapability.Xxx.Xxx
   * @since 9
   */
  export type Xxx = _Xxx;

  // ... 更多声明
}
```

### 关键 JSDoc 标签

| 标签 | 用途 | 示例 |
|------|------|------|
| `@file` | 文件标注 | `@file` |
| `@kit` | Kit 归属 | `@kit AbilityKit` |
| `@namespace` | 命名空间 | `@namespace ability` |
| `@syscap` | 系统能力 | `@syscap SystemCapability.Ability.AbilityRuntime` |
| `@since` | 版本信息 | `@since 9` |
| `@FAModelOnly` | 仅 FA 模型 | `@FAModelOnly` |
| `@deprecated` | 废弃标记 | `@deprecated` |

### 条件编译

**动态/静态 SDK 区分**:

```typescript
/*** if arkts dynamic */
// 动态 SDK 专用导入
import { X as _X } from './dynamic/path';
/*** endif */

/*** if arkts static */
// 静态 SDK 专用导入
import { X as _X } from './static/path';
/*** endif */
```

**证据**: `api/@ohos.ability.ability.d.ts:1-70`

## API 声明示例

### 示例 1: 简单类型

```typescript
/**
 * 错误码定义
 * @enum {number}
 * @syscap SystemCapability.Ability.AbilityRuntime.AbilityCore
 * @since 9
 */
export enum ErrorCode {
  SUCCESS = 0,
  PERMISSION_DENIED = 1,
  INVALID_PARAMETER = 2,
}
```

### 示例 2: 接口定义

```typescript
/**
 *  Ability 生命周期回调接口
 * @interface IAblityLifecycleCallback
 * @syscap SystemCapability.Ability.AbilityRuntime.AbilityCore
 * @since 9
 */
export interface IAbilityLifecycleCallback {
  /**
   * Ability 创建回调
   * @param { string } bundleName - 包名
   * @param { Ability } ability - Ability 实例
   */
  onAbilityCreate(bundleName: string, ability: Ability): void;
}
```

## API 模块索引

### 按子系统索引

| 子系统 | 模块列表 | 文件位置 |
|--------|---------|---------|
| Ability | ability, featureAbility, particleAbility | `api/@ohos.ability.*.d.ts` |
| Account | appAccount, osAccount, distributedAccount | `api/@ohos.account.*.d.ts` |
| Accessibility | accessibility, config | `api/@ohos.accessibility.*.d.ts` |
| Window | window, display | `api/@ohos.window.*.d.ts` |
| Media | audio, video, image | `api/@ohos.media.*.d.ts` |

### 按 Kit 索引

| Kit | 声明文件 | 用途 |
|-----|---------|------|
| AbilityKit | `@kit.AbilityKit.d.ts` | 能力框架 |
| ArkUI | `@kit.ArkUI.d.ts` | UI 框架 |
| MediaKit | `@kit.MediaKit.d.ts` | 媒体 |
| AudioKit | `@kit.AudioKit.d.ts` | 音频 |
| NetworkKit | `@kit.NetworkKit.d.ts` | 网络 |

## 构建时处理

### 过滤规则

**remove_list.json 定义**:

```json
{
  "ets_internal_api": {
    "base": [
      "api/common/full/canvaspattern.d.ts",
      "api/common/full/featureability.d.ts"
    ]
  },
  "ets_component": {
    "sdk_build_public_remove": [
      "ability_component.d.ts",
      "animator.d.ts"
    ]
  }
}
```

**证据**: `remove_list.json:1-31`

### 模板处理

| 模板 | 用途 |
|------|------|
| `ohos_declaration_template` | 公共 API 处理 |
| `ohos_copy_internal` | 内部 API 处理 |
| `ohos_handle_declaration_template` | ArkUI 标签处理 |

**证据**: `BUILD.gn:82-207`

## 相关文档

- [构建工具](02_Build_Tools.md)
- [GN 配置](03_Build_Configuration.md)
