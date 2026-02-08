# 编译产物

## 模块类型说明

Camera 应用采用多模块设计，各模块类型如下：

| 模块 | 类型 | 说明 | 可独立运行 |
|------|------|------|-----------|
| phone | entry | 手机形态应用入口 | 是 |
| tablet | entry | 平板形态应用入口 | 是 |
| common | har | 静态共享包 | 否 |
| photo | har | 拍照功能模块 | 否 |
| video | har | 录像功能模块 | 否 |
| multi | har | 多机位协同模块 | 否 |

**证据**: `build-profile.json5:41-82`

```json
{
  "modules": [
    { "name": "phone", "srcPath": "./product/phone" },
    { "name": "tablet", "srcPath": "./product/tablet" },
    { "name": "common", "srcPath": "./common" },
    { "name": "multi", "srcPath": "./features/multi" },
    { "name": "photo", "srcPath": "./features/photo" },
    { "name": "video", "srcPath": "./features/video" }
  ]
}
```

## 构建配置

### build-profile.json5

**路径**: `./build-profile.json5`

**关键配置**:

```json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "signingConfig": "default",          // 签名配置
        "compileSdkVersion": 23,             // 编译 SDK 版本
        "compatibleSdkVersion": 23           // 兼容 SDK 版本
      }
    ],
    "signingConfigs": [
      {
        "name": "default",
        "material": {
          "storePassword": "...",            // 密钥库密码
          "certpath": "signature/camera.cer", // 证书路径
          "keyAlias": "Camera",               // 密钥别名
          "keyPassword": "...",               // 密钥密码
          "profile": "signature/camera.p7b",  // 调试/发布 profile
          "signAlg": "SHA256withECDSA",       // 签名算法
          "storeFile": "signature/camera.p12" // 密钥库文件
        }
      }
    ]
  }
}
```

### 模块级配置

#### Entry 模块 (Phone/Tablet)

**路径**: `product/phone/src/main/module.json5`

```json5
{
  "module": {
    "name": "phone",
    "type": "entry",
    "srcEntry": "./ets/Application/AbilityStage.ts",
    "mainElement": "com.ohos.camera.MainAbility",
    "deviceTypes": ["default"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "requestPermissions": [/* 13项权限 */],
    "abilities": [/* Ability 配置 */],
    "extensionAbilities": [/* 扩展 Ability 配置 */]
  }
}
```

#### HAR 模块 (Common/Features)

**路径**: `common/src/main/module.json5`

```json5
{
  "module": {
    "name": "common",
    "type": "har",
    "deviceTypes": ["default", "tablet"]
  }
}
```

**路径**: `features/photo/src/main/module.json5`

```json5
{
  "module": {
    "name": "photo",
    "type": "har",
    "deviceTypes": ["default", "tablet"]
  }
}
```

### 包配置

#### oh-package.json5

**路径**: `./oh-package.json5`

```json5
{
  "modelVersion": "5.0.2",
  "license": "ISC",
  "devDependencies": {
    "@ohos/hypium": "1.0.6"    // 测试框架
  },
  "name": "camera",
  "description": "example description",
  "repository": {},
  "version": "1.0.0",
  "dependencies": {}
}
```

#### bundle.json

**路径**: `./bundle.json`

OpenHarmony 组件发布配置：

```json
{
  "name": "@ohos/camera",
  "description": "Camera app for standard system.",
  "version": "3.0",
  "license": "Apache License 2.0",
  "publishAs": "code-segment",
  "segment": {
    "destPath": "applications/standard/camera"
  },
  "component": {
    "name": "camera",
    "subsystem": "applications",
    "syscap": [],
    "features": [],
    "adapted_system_type": ["standard"],
    "rom": "0KB",
    "ram": "0KB"
  }
}
```

## HAP 构建流程

### 构建工具链

1. **hvigor**: 构建任务调度器
2. **ArkCompiler**: ArkTS 编译器
3. **打包工具**: HAP 包生成

**证据**: `hvigorfile.js`

```javascript
// 构建脚本入口
module.exports = require('@ohos/hvigor-ohos-plugin').hapTasks
```

### 构建步骤

```
1. 编译 ArkTS 源码
   ├─ 类型检查
   ├─ AST 转换
   └─ 字节码生成

2. 处理资源文件
   ├─ 合并 resources
   ├─ 编译 json/xml
   └─ 处理图片资源

3. 打包 HAR 模块
   ├─ common.har
   ├─ photo.har
   ├─ video.har
   └─ multi.har

4. 打包 Entry 模块
   ├─ phone-entry.hap
   └─ tablet-entry.hap

5. 签名
   ├─ 使用 camera.p12 密钥库
   ├─ SHA256withECDSA 签名
   └─ 附加 camera.p7b profile
```

### 构建命令

```bash
# 安装依赖
ohpm install

# 构建所有模块
hvigor build

# 构建特定模块
hvigor build -p product=phone
hvigor build -p product=tablet

# 清理构建产物
hvigor clean
```

## 产物清单

### 输出目录结构

```
build/
├── default/
│   ├── intermediates/
│   │   ├── common/              # Common 层中间产物
│   │   ├── photo/               # Photo 层中间产物
│   │   ├── video/               # Video 层中间产物
│   │   ├── multi/               # Multi 层中间产物
│   │   ├── phone/               # Phone 层中间产物
│   │   └── tablet/              # Tablet 层中间产物
│   └── outputs/
│       ├── default/
│       │   ├── phone-
entry-default-unsigned.hap    # Phone 未签名 HAP
│       │   ├── phone-entry-default-signed.hap      # Phone 签名 HAP
│       │   ├── tablet-entry-default-unsigned.hap   # Tablet 未签名 HAP
│       │   ├── tablet-entry-default-signed.hap     # Tablet 签名 HAP
│       │   ├── common.har                          # Common HAR
│       │   ├── photo.har                           # Photo HAR
│       │   ├── video.har                           # Video HAR
│       │   └── multi.har                           # Multi HAR
```

### 产物类型说明

| 产物类型 | 扩展名 | 说明 | 安装位置 |
|----------|--------|------|----------|
| HAP | .hap | HarmonyOS Ability Package | `/data/app/el1/bundle/public/com.ohos.camera/` |
| HAR | .har | HarmonyOS Archive | 内嵌到 HAP 中 |

### Phone 产物内容

**phone-entry-signed.hap** 包含:

```
phone-entry-signed.hap
├── ets/
│   ├── modules.abc          # ArkTS 字节码
│   ├── sourceMaps.map       # Source Map
│   └── /phone/              # Phone 模块源码
├── resources/
│   ├── base/                # 基础资源
│   ├── rawfile/             # 原始文件
│   ├── resfile/             # 资源文件
│   └── module.json          # 模块配置
├── libs/
│   └── arm64-v8a/           # 原生库 (如有)
├── assets/
│   ├── common.har           # 内嵌 HAR
│   ├── photo.har
│   ├── video.har
│   └── multi.har
├── config.json              # 应用配置
└── signature.bin            # 签名信息
```

## 安装路径

### 系统安装路径

**HAP 安装目录**:
```
/data/app/el1/bundle/public/com.ohos.camera/
├── com.ohos.camera/
│   ├── phone_entry_signed.hap
│   ├── ets/
│   ├── resources/
│   └── ...
```

**应用数据目录**:
```
/data/app/el2/100/base/com.ohos.camera/
├── files/                    # 应用文件
├── preferences/              # SharedPreferences
├── database/                 # 数据库
└── cache/                    # 缓存
```

### 媒体存储路径

拍摄的照片和视频保存到系统相册:

```
/storage/media/100/local/files/
├── DCIM/Camera/              # 照片
│   ├── IMG_20240205_120000.jpg
│   └── ...
└── DCIM/Camera/              # 视频
    ├── VID_20240205_120000.mp4
    └── ...
```

**代码证据**: `common/src/main/ets/default/camera/SaveCameraAsset.ts`

## 运行时加载关系

### 模块加载流程

```
系统启动应用
    │
    ▼
加载 phone-entry.hap
    │
    ├─ 解压并加载 ets/modules.abc
    ├─ 加载内嵌 HAR (common.har, photo.har, video.har, multi.har)
    └─ 初始化 AbilityStage
    │
    ▼
创建 MainAbility
    │
    ├─ 执行 onCreate()
    ├─ 初始化 CameraBasicFunction
    ├─ 初始化 FeatureManager
    └─ 加载页面资源
    │
    ▼
页面渲染
    │
    ├─ 加载 index.ets 页面
    ├─ 创建 XComponent (预览)
    ├─ 初始化 CameraService
    └─ 绑定 UI 事件
```

### 依赖加载顺序

```
1. @ohos.app.ability.UIAbility (系统框架)
2. @ohos.multimedia.camera (系统相机服务)
3. @ohos/common (应用 HAR)
4. @ohos/photo/video/multi (功能 HAR)
5. Product 页面组件
```

## 签名信息

### 签名配置

**密钥库**: `signature/camera.p12`
**证书**: `signature/camera.cer`
**Profile**: `signature/camera.p7b`

**算法**: SHA256withECDSA

### 系统签名

Camera 作为系统应用，使用系统签名:

```json
{
  "signingConfigs": {
    "material": {
      "storeFile": "signature/camera.p12",
      "certpath": "signature/camera.cer",
      "profile": "signature/camera.p7b",
      "signAlg": "SHA256withECDSA"
    }
  }
}
```

**证据**: `build-profile.json5:27-38`

## 调试构建

### Debug 模式

默认构建配置即为调试模式，包含:
- Source Map 生成
- 未压缩代码
- 调试符号保留

### Release 模式

生产构建需要:
1. 切换到发布证书
2. 启用代码压缩
3. 移除调试信息

## 常见问题

### 1. 构建失败 - 签名错误

**症状**: `Error: Failed to sign hap`

**解决**:
- 检查 signature/ 目录下证书文件是否存在
- 验证签名密码是否正确

### 2. 运行时 - 模块未找到

**症状**: `Error: Cannot find module '@ohos/xxx'`

**解决**:
- 确认 oh-package.json5 中依赖声明
- 执行 `ohpm install` 安装依赖

### 3. HAP 安装失败

**症状**: `Install failed: signature verification failed`

**解决**:
- 确认使用正确的调试证书
- 检查设备是否允许安装调试应用
