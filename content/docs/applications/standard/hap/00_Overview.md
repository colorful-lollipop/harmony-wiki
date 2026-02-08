# 项目概览

## 项目定位

`applications/standard/hap` 是 OpenHarmony 标准系统的重要组成部分，职责如下：

### 核心定位

- **系统应用归档仓**: 以预构建 HAP 包的形式归档系统应用
- **流水线构建记录**: 记录通过 OpenHarmony 流水线的永久构建产物地址
- **资源分发**: 提供系统应用运行所需的预制资源

### 关键证据

> "本仓以预构建 HAP 包的形式归档这些应用"
> — `README.md:19`

## 系统应用分类

### 核心系统应用

| 应用 | Bundle Name | 说明 |
|------|-------------|------|
| Launcher | com.ohos.launcher | 桌面，应用入口 |
| SystemUI | com.ohos.systemui | 导航栏、状态栏、锁屏等 |
| Settings | com.ohos.settings | 系统设置 |
| Settings_FaceAuth | com.ohos.settings.faceauth | 人脸认证设置 |
| SecurityPrivacyCenter | com.ohos.security.privacycenter | 隐私安全中心 |

### 媒体与数据应用

| 应用 | Bundle Name | 说明 |
|------|-------------|------|
| Camera | com.ohos.camera | 相机应用 |
| Photos | com.ohos.photos | 图库应用 |
| MediaLibrary | com.ohos.medialibrary.MediaLibraryDataA | 媒体库数据 |
| MediaScanner | com.ohos.medialibrary.MediaScannerAbilityA | 媒体扫描器 |
| Music_Demo | com.ohos.distributedmusicplayer | 音乐播放器示例 |

### 通信应用

| 应用 | Bundle Name | 说明 |
|------|-------------|------|
| Contacts | com.ohos.contacts | 联系人 |
| Mms | com.ohos.mms | 短信 |
| CallUI | com.ohos.callui | 通话界面 |
| MobileDataSettings | com.ohos.callui | 移动数据设置 |

### 工具与系统工具

| 应用 | Bundle Name | 说明 |
|------|-------------|------|
| Note | com.ohos.note | 备忘录 |
| CalendarData | com.ohos.calendardata | 日历数据 |
| FilePicker | com.ohos.filepicker | 文件选择器 |
| AudioPicker | com.ohos.filepicker | 音频选择器 |
| PrintSpooler | com.ohos.spooler | 打印服务 |
| ScreenShot | com.ohos.screenshot | 截屏工具 |
| UpdateApp | com.ohos.updateapp | 系统更新 |

### 示例应用

| 应用 | Bundle Name | 说明 |
|------|-------------|------|
| Calc_Demo | com.example.distributedcalc | 计算器示例 |
| Clock_Demo | ohos.samples.clock | 时钟示例 |
| kikaInput | com.example.kikakeyboard | 输入法示例 |

## 运行环境要求

### 系统要求

- **OpenHarmony 版本**: 3.1+ (部分应用支持 3.1.7.7+)
- **SDK 版本**: 10, 11, 12, 14, 18, 19, 20
- **系统能力**: 根据应用类型，需要不同的 system capability

### 硬件要求

- **默认配置**: 标准 OpenHarmony 设备
- **手表配置**: watchos 产品变体（部分应用不适用）
- **开发板**: rk3568, ohos-arm64

### 权限要求

各应用根据功能需要不同的系统权限，详情参见对应源码仓库的 `module.json5` 配置。

## 关键概念

### HAP (Harmony Ability Package)

HAP 是 OpenHarmony 的应用安装包格式，包含：
- 编译后的 ArkTS/TS/JS 代码
- 资源文件
- 配置文件 (module.json5)
- 签名信息

### 预构建 (Prebuilt)

本仓库采用预构建模式：
- HAP 文件已编译签名
- 通过 GN `ohos_prebuilt_etc` 模板集成到系统构建
- 安装路径在 `BUILD.gn` 中指定

### hisysevent (系统事件)

系统事件配置用于：
- 故障监控 (FAULT)
- 行为追踪 (BEHAVIOR)
- 性能统计 (STATISTIC)

## 相关源码仓库

| 仓库 | 说明 |
|------|------|
| [applications_standard_settings](https://gitee.com/openharmony/applications_settings) | Settings 应用源码 |
| [applications_standard_launcher](https://gitee.com/openharmony/applications_launcher) | Launcher 桌面源码 |
| [applications_standard_systemui](https://gitee.com/openharmony/applications_systemui) | SystemUI 组件源码 |

## 下一步阅读

1. [01_Directory_Structure.md](./01_Directory_Structure.md) - 深入了解目录结构
2. [05_Build_System.md](./05_Build_System.md) - 理解构建配置
3. [06_Artifacts.md](./06_Artifacts.md) - 查看产物清单
