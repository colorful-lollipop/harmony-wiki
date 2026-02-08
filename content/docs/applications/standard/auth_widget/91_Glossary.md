# 91_Glossary - 术语表

## A

### API (Application Programming Interface)
- **中文**: 应用程序编程接口
- **描述**: 定义软件组件之间交互的接口规范
- **证据**: `Index.ets` 行 17, 174

### AppStorage
- **中文**: 应用存储
- **描述**: ArkTS 框架提供的全局状态管理机制，用于组件间数据共享
- **证据**: `UserAuthAbility.ts` 行 32, 50-51

### ArkTS
- **中文**: Ark TypeScript
- **描述**: OpenHarmony 应用的编程语言，基于 TypeScript 扩展
- **证据**: `BUILD.gn` 行 27

### ArkUI
- **中文**: Ark 用户界面
- **描述**: OpenHarmony 的声明式 UI 开发框架
- **使用位置**: `entry/src/main/ets/` 下的所有 UI 组件

## B

### bundle.json
- **中文**: 模块清单文件
- **描述**: 定义 OpenHarmony 模块元信息的配置文件
- **证据**: `bundle.json` 行 1-33

## C

### CmdData
- **中文**: 指令数据
- **描述**: 认证指令的负载数据，包含认证类型、剩余尝试次数、锁定时长等信息
- **证据**: `Constants.ts` 行 149-157

### CmdType
- **中文**: 指令类型
- **描述**: 认证指令类型，包含事件和负载
- **证据**: `Constants.ts` 行 159-162

## D

### DialogType
- **中文**: 对话框类型
- **描述**: 认证对话框的类型枚举（ALL, PIN_ONLY, FINGERPRINT_ONLY, FACE_ONLY）
- **证据**: `DialogType.ts`

## E

### Extension Ability
- **中文**: 扩展能力
- **描述**: OpenHarmony 中用于实现系统级功能的组件基类
- **证据**: `UserAuthAbility.ts` 行 29

## F

### FACE (人脸认证)
- **中文**: 人脸认证
- **描述**: 基于人脸生物特征的认证方式
- **证据**: `Constants.ts` 行 43, `FaceAuth.ets`

## G

### GN (Generate Ninja)
- **中文**: GN 构建系统
- **描述**: OpenHarmony 使用的构建系统，生成 Ninja 构建文件
- **证据**: `BUILD.gn` 行 14

## H

### HAP (Harmony Ability Package)
- **中文**: Harmony 能力包
- **描述**: OpenHarmony 应用的安装包格式
- **证据**: `BUILD.gn` 行 36

## I

### IPC (Inter-Process Communication)
- **中文**: 进程间通信
- **描述**: 不同进程之间交换数据的机制
- **使用位置**: `user_auth_framework` 与 auth_widget 之间的通信

## M

### module.json
- **中文**: 模块配置文件
- **描述**: 定义模块能力、组件、权限等信息的配置文件
- **证据**: `entry/src/main/module.json`

## N

### NoticeType
- **中文**: 通知类型
- **描述**: 认证 Widget 通知类型枚举
- **证据**: `AuthUtils.ts` 行 48

## P

### PIN (Personal Identification Number)
- **中文**: 个人识别码
- **描述**: 数字密码认证方式
- **证据**: `Constants.ts` 行 21-23, 42

### PINAuth
- **中文**: PIN 认证器
- **描述**: OpenHarmony 提供的 PIN 认证能力
- **证据**: `PasswordAuth.ets` 行 16, 34, 102

## U

### UIExtension
- **中文**: UI 扩展
- **描述**: OpenHarmony 中实现系统级 UI 的扩展机制
- **证据**: `UserAuthAbility.ts` 行 19, 48-52

### UserAuthExtensionAbility
- **中文**: 用户认证扩展能力
- **描述**: auth_widget 继承的基类，提供认证 UI 扩展生命周期
- **证据**: `UserAuthAbility.ts` 行 17, 29

### UserAuthWidgetMgr
- **中文**: 用户认证 Widget 管理器
- **描述**: 用于与认证框架通信的管理器实例
- **证据**: `Index.ets` 行 34, 174

## W

### Want
- **中文**: 意图
- **描述**: OpenHarmony 中用于组件间通信的意图对象
- **证据**: `UserAuthAbility.ts` 行 21, 48

### WantParams
- **中文**: 意图参数
- **描述**: 认证请求参数，封装在 Want 中传递
- **证据**: `Constants.ts` 行 172-181

### WidgetCommand
- **中文**: Widget 指令
- **描述**: 认证框架下发给 Widget 的指令
- **证据**: `Constants.ts` 行 183-187

## 其他术语

| 英文 | 中文 | 描述 |
|------|------|------|
| FINGERPRINT | 指纹 | 生物特征认证 |
| @Component | 组件装饰器 | ArkUI 组件定义 |
| @Entry | 入口装饰器 | ArkUI 页面入口 |
| @State | 状态装饰器 | ArkUI 响应式状态 |
| @Link | 链接装饰器 | ArkUI 双向绑定 |

## 相关文档

- [概览](00_Overview.md) - 项目背景与运行环境
- [架构说明](01_Architecture.md) - 组件关系图
- [认证框架 API](02_UserAuth_API.md) - API 详细说明
- [常见问题](90_FAQ.md) - 常见问题解答
