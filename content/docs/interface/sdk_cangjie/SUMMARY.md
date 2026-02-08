# 文档导航

## 新人学习路线

```
1️⃣ 快速入门
   ├── [README](README.md)                    ← 必读，先了解项目定位
   └── [01_Overview](01_Overview.md)          ← 项目定位、核心能力

2️⃣ 项目结构
   ├── [02_Directory_Structure](02_Directory_Structure.md)  ← 目录组织、模块划分
   └── [03_Architecture](03_Architecture.md)                ← 系统架构、组件关系

3️⃣ API 参考
   ├── [04_Kit_API](04_Kit_API.md)                          ← Kit API 清单
   └── [05_Inner_API](05_Inner_API.md)                      ← 内部 API

4️⃣ 构建系统
   ├── [06_GN_Build](06_GN_Build.md)                        ← GN Targets
   └── [07_Build_Artifacts](07_Build_Artifacts.md)          ← 编译产物

5️⃣ 问题排查
   └── [09_FAQ](09_FAQ.md)                                  ← 常见问题
```

## 安全研究路线

```
🔐 安全分析
   ├── [05_AttackSurface](05_AttackSurface.md)              ← 构建脚本攻击面分析
   │                                          ← 外部输入入口、敏感操作清单
   │
   └── [08_Security_Review](08_Security_Review.md)          ← API声明安全风险评审
                                              ← 权限模型、信任边界、风险点
```

## 快速索引

### 按功能分类

| 功能 | 文档 |
|------|------|
| 应用能力（Ability） | [04_Kit_API → AbilityKit](04_Kit_API.md#abilitykit) |
| UI 框架 | [04_Kit_API → ArkUI](04_Kit_API.md#arkui) |
| 网络通信 | [04_Kit_API → NetworkKit](04_Kit_API.md#networkkit) |
| 文件系统 | [04_Kit_API → CoreFileKit](04_Kit_API.md#corefilekit) |
| 媒体能力 | [04_Kit_API → MediaKit](04_Kit_API.md#mediakit) |
| 安全与权限 | [04_Kit_API → UniversalKeystoreKit](04_Kit_API.md#universalkeystoretoolkit) |
| 进程间通信 | [04_Kit_API → IPCKit](04_Kit_API.md#ipckit) |
| 构建配置 | [06_GN_Build](06_GN_Build.md) |
| 安全机制 | [08_Security_Review](08_Security_Review.md) |

### 按 Kit 分类

| Kit | 接口声明位置 | Kit 文档 |
|-----|-------------|---------|
| AbilityKit | `api/AbilityKit/` + `kit.AbilityKit.cj.d` | [04_Kit_API](04_Kit_API.md#abilitykit) |
| ArkData | `api/ArkData/` + `kit.ArkData.cj.d` | [04_Kit_API](04_Kit_API.md#arkdata) |
| ArkGraphics2D | `api/ArkGraphics2D/` + `kit.ArkGraphics2D.cj.d` | [04_Kit_API](04_Kit_API.md#arkgraphics2d) |
| ArkUI | `api/ArkUI/` + `kit.ArkUI.cj.d` | [04_Kit_API](04_Kit_API.md#arkui) |
| ArkWeb | `api/ArkWeb/` + `kit.ArkWeb.cj.d` | [04_Kit_API](04_Kit_API.md#arkweb) |
| BasicServicesKit | `api/BasicServicesKit/` + `kit.BasicServicesKit.cj.d` | [04_Kit_API](04_Kit_API.md#basicserviceskit) |
| CameraKit | `api/CameraKit/` + `kit.CameraKit.cj.d` | [04_Kit_API](04_Kit_API.md#camerakit) |
| CangjieKit | `api/Cangjie/` + `kit.CangjieKit.cj.d` | [04_Kit_API](04_Kit_API.md#cangjiekit) |
| ConnectivityKit | `api/ConnectivityKit/` + `kit.ConnectivityKit.cj.d` | [04_Kit_API](04_Kit_API.md#connectivitykit) |
| CoreFileKit | `api/CoreFileKit/` + `kit.CoreFileKit.cj.d` | [04_Kit_API](04_Kit_API.md#corefilekit) |
| CryptoArchitectureKit | `api/CryptoArchitectureKit/` + `kit.CryptoArchitectureKit.cj.d` | [04_Kit_API](04_Kit_API.md#cryptoarchitecturekit) |
| IPCKit | `api/IPCKit/` + `kit.IPCKit.cj.d` | [04_Kit_API](04_Kit_API.md#ipckit) |
| ImageKit | `api/ImageKit/` + `kit.ImageKit.cj.d` | [04_Kit_API](04_Kit_API.md#imagekit) |
| LocalizationKit | `api/LocalizationKit/` + `kit.LocalizationKit.cj.d` | [04_Kit_API](04_Kit_API.md#localizationkit) |
| LocationKit | `api/LocationKit/` + `kit.LocationKit.cj.d` | [04_Kit_API](04_Kit_API.md#locationkit) |
| MediaKit | `api/MediaKit/` + `kit.MediaKit.cj.d` | [04_Kit_API](04_Kit_API.md#mediakit) |
| MediaLibraryKit | `api/MediaLibraryKit/` + `kit.MediaLibraryKit.cj.d` | [04_Kit_API](04_Kit_API.md#medialibrarykit) |
| NetworkKit | `api/NetworkKit/` + `kit.NetworkKit.cj.d` | [04_Kit_API](04_Kit_API.md#networkkit) |
| PerformanceAnalysisKit | `api/PerformanceAnalysisKit/` + `kit.PerformanceAnalysisKit.cj.d` | [04_Kit_API](04_Kit_API.md#performanceanalysiskit) |
| SensorServiceKit | `api/SensorServiceKit/` + `kit.SensorServiceKit.cj.d` | [04_Kit_API](04_Kit_API.md#sensorservicekit) |
| TelephonyKit | `api/TelephonyKit/` + `kit.TelephonyKit.cj.d` | [04_Kit_API](04_Kit_API.md#telephonykit) |
| TestKit | `api/TestKit/` + `kit.TestKit.cj.d` | [04_Kit_API](04_Kit_API.md#testkit) |
| UniversalKeystoreKit | `api/UniversalKeystoreKit/` + `kit.UniversalKeystoreKit.cj.d` | [04_Kit_API](04_Kit_API.md#universalkeystoretoolkit) |

## 外部链接

- [项目 README](../README.md)
- [中文 README](../README_zh.md)
- [构建指南](../docs/cangjie_sdk_build_guide.md)
- [cjo 序列化指南](../docs/cangjie_cjo_serialization_and_deserialization_guide.md)
