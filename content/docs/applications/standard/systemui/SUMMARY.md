# SystemUI Wiki 导航

> 全站文档索引与阅读路线

## 文档索引

### 快速入门

| 文档 | 说明 | 必读 |
|------|------|------|
| [README](README.md) | Wiki 说明与快速开始 | ✅ |
| [SUMMARY](SUMMARY.md) | 本文档，全站导航 | ✅ |
| [01_Overview](01_Overview.md) | 项目定位与核心能力 | ✅ |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块划分 | ✅ |

### 核心架构

| 文档 | 说明 | 必读 |
|------|------|------|
| [03_Architecture](03_Architecture.md) | 系统架构、组件图、数据流 | ✅ |
| [04_API_Inner](04_API_Inner.md) | 内部 API 与模块接口 | 🔶 |
| [05_Build](05_Build.md) | 构建配置与编译产物 | 🔶 |
| [06_Security](06_Security.md) | 安全风险与最佳实践 | 🔶 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 关键配置项 |

## 新人阅读顺序

```
第 1 天: 概览与结构
├── README.md (5 分钟)
├── 01_Overview.md (10 分钟)
└── 02_Directory_Structure.md (10 分钟)

第 1-2 天: 架构理解
├── 03_Architecture.md (20 分钟)
├── 05_Build.md (15 分钟)
└── 04_API_Inner.md (15 分钟)

第 3 天: 安全与实践
└── 06_Security.md (15 分钟)
```

## 模块快速索引

### features (功能组件)

| 组件 | 职责 | 类型 |
|------|------|------|
| [airplanecomponent](02_Directory_Structure.md#31-features) | 飞行模式 | HAR |
| [batterycomponent](02_Directory_Structure.md#31-features) | 电池状态 | HAR |
| [brightnesscomponent](02_Directory_Structure.md#31-features) | 屏幕亮度 | HAR |
| [capsulecomponent](02_Directory_Structure.md#31-features) | 通知胶囊 | HAR |
| [clockcomponent](02_Directory_Structure.md#31-features) | 系统时钟 | HAR |
| [controlcentercomponent](02_Directory_Structure.md#31-features) | 控制中心 | HAR |
| [noticeitem](02_Directory_Structure.md#31-features) | 通知项 | HAR |
| [notificationservice](02_Directory_Structure.md#31-features) | 通知服务 | HAR |
| [statusbarcomponent](02_Directory_Structure.md#31-features) | 状态栏 | HAR |
| [volumecomponent](02_Directory_Structure.md#31-features) | 音量控制 | HAR |
| [wificomponent](02_Directory_Structure.md#31-features) | WiFi 状态 | HAR |

### product (产品模块)

| 模块 | 设备 | 职责 |
|------|------|------|
| [navigationBar](02_Directory_Structure.md#32-product) | default | 导航栏 |
| [statusbar](02_Directory_Structure.md#32-product) | phone/pc | 状态栏 |
| [notificationmanagement](02_Directory_Structure.md#32-product) | default | 通知管理 |
| [controlpanel](02_Directory_Structure.md#32-product) | pc | 控制面板 |
| [dropdownpanel](02_Directory_Structure.md#32-product) phone | 下拉面板 |
| [notificationpanel](02_Directory_Structure.md#32-product) | pc | 通知面板 |
| [volumepanel](02_Directory_Structure.md#32-product) | default | 音量面板 |
| [dialog](02_Directory_Structure.md#32-product) | default | 对话框 |

### common (通用模块)

| 模块 | 职责 |
|------|------|
| [Log](04_API_Inner.md#41-log) | 日志输出 |
| [AbilityManager](04_API_Inner.md#42-abilitymanager) | 能力管理 |
| [EventManager](04_API_Inner.md#43-eventmanager) | 事件管理 |
| [WindowManager](04_API_Inner.md#44-windowmanager) | 窗口管理 |
| [NotificationManager](04_API_Inner.md#45-notificationmanager) | 通知管理 |

## 关键 API 快速链接

### 系统 API (ArkTS)

- [@ohos.notificationManager](04_API_Inner.md#51-通知系统) - 通知管理
- [@ohos.hilog](04_API_Inner.md#52-日志系统) - 日志输出
- [@ohos.app.ability.common](04_API_Inner.md#53-能力上下文) - 能力上下文

### 内部 API

- [EventBus.emit()](04_API_Inner.md#432-eventbus) - 发布事件
- [EventManager.publish()](04_API_Inner.md#431-eventmanager) - 发布事件
- [AbilityManager.startAbility()](04_API_Inner.md#421-abilitymanager) - 启动 Ability

## 常见问题

| 问题 | 答案 |
|------|------|
| 如何添加新功能？ | 参考 03_Architecture.md 的模块划分 |
| 如何调试？ | 使用 Log.showInfo/Log.showError |
| 如何构建？ | 参考 05_Build.md |
| 安全注意事项？ | 参考 06_Security.md |

## 文档更新日志

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2026-02-06 | 初始版本 |

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [ArkTS 语言参考](https://docs.openharmony.cn/pages/zh-cn/application-dev/reference/apis-arkts/)
- [ArkUI 组件参考](https://docs.openharmony.cn/pages/zh-cn/application-dev/reference/apis-arkui/)
