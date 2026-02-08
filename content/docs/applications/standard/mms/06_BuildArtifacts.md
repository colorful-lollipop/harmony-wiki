# 06. 构建产物

## 目的与适用范围

本文档描述 MMS 应用的构建配置、输出产物和部署信息。

**适用读者**: 构建工程师、部署工程师  
**阅读时间**: 约 15 分钟

---

## 构建系统

### Hvigor 构建工具

MMS 使用 OpenHarmony 推荐的 Hvigor 构建系统。

```
┌─────────────────────────────────────────┐
│           Hvigor 构建流程               │
│                                         │
│  1. 编译 ArkTS/TS 源码                  │
│     ↓                                   │
│  2. 处理资源文件                        │
│     ↓                                   │
│  3. 打包 HAP                            │
│     ↓                                   │
│  4. 签名                                │
│     ↓                                   │
│  5. 输出                                │
└─────────────────────────────────────────┘
```

### 构建配置

#### 项目级配置 (build-profile.json5)

```json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "debug",
        "material": {
          "storePassword": "***",
          "certpath": "signs/IDE.cer",
          "keyAlias": "OpenHarmony Application CA",
          "keyPassword": "***",
          "profile": "signs/com.ohos.mms.p7b",
          "signAlg": "SHA256withECDSA",
          "storeFile": "signs/OpenHarmony.p12"
        }
      }
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "debug",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry"
    }
  ]
}
```

**关键配置项**:

| 配置项 | 值 | 说明 |
|--------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |
| signingConfig | debug | 使用 debug 签名 |
| signAlg | SHA256withECDSA | 签名算法 |

#### 模块配置 (entry/src/main/module.json5)

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "srcEntry": "./ets/Application/MyAbilityStage.ts",
    "description": "$string:entry_description",
    "mainElement": "com.ohos.mms.MainAbility",
    "deviceTypes": ["default"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    // ...
  }
}
```

**模块属性**:

| 属性 | 值 | 说明 |
|------|-----|------|
| type | entry | 入口模块 |
| deliveryWithInstall | true | 随应用安装 |
| installationFree | false | 需要安装 |
| deviceTypes | ["default"] | 设备类型 |

---

## 构建产物

### 产物清单

| 产物 | 类型 | 路径 | 说明 |
|------|------|------|------|
| entry-default-signed.hap | HAP | build/outputs/default/ | 签名的 HAP 包 |
| entry-default-unsigned.hap | HAP | build/outputs/default/ | 未签名 HAP 包 |
| ets/ | 目录 | build/default/intermediates/ | 编译后的 JS |
| resources/ | 目录 | build/default/intermediates/ | 处理后的资源 |

### HAP 包结构

```
entry-default-signed.hap
├── ets/                          # 编译后的代码
│   ├── default/                  # 业务代码
│   │   ├── Application/
│   │   ├── MainAbility/
│   │   ├── StaticSubscriber/
│   │   ├── data/
│   │   ├── model/
│   │   ├── pages/
│   │   ├── service/
│   │   ├── utils/
│   │   ├── views/
│   │   └── workers/
│   └── default.pages.json        # 页面配置
├── resources/                    # 资源文件
│   ├── base/
│   ├── en_US/
│   └── zh_CN/
├── config.json                   # 模块配置
├── module.json                   # 模块信息
└── signature/                    # 签名信息
    └── ...
```

### 产物类型说明

| 类型 | 后缀 | 用途 |
|------|------|------|
| HAP | .hap | HarmonyOS Ability Package，可安装包 |
| ets | .js | 编译后的 JavaScript |
| 资源 | 多类型 | 图片、字符串、颜色等 |

---

## 签名配置

### 签名文件

```
signs/
├── IDE.cer                      # 证书文件
├── OpenHarmony.p12              # 密钥库
└── com.ohos.mms.p7b             # Profile 文件
```

### 签名信息

| 属性 | 值 |
|------|-----|
| Bundle Name | com.ohos.mms |
| 签名算法 | SHA256withECDSA |
| 证书类型 | OpenHarmony Application CA |
| Profile | 调试/发布配置 |

### 系统应用签名

作为预置系统应用，MMS 需要系统签名才能获取敏感权限：

```json5
// module.json5 中的权限声明
"requestPermissions": [
    {"name": "ohos.permission.SEND_MESSAGES"},
    {"name": "ohos.permission.RECEIVE_SMS"},
    {"name": "ohos.permission.READ_MESSAGES"},
    {"name": "ohos.permission.NOTIFICATION_CONTROLLER"},
    // ...
]
```

---

## 安装路径

### 预置安装

作为系统应用，MMS 预置在系统镜像中：

```
/system/app/com.ohos.mms/
├── com.ohos.mms.apk/hap         # 应用包
└── ...
```

### 数据目录

```
/data/app/el1/bundle/public/com.ohos.mms/     # 应用安装目录
/data/app/el2/100/base/com.ohos.mms/          # 应用数据目录
├── cache/                                    # 缓存
├── files/                                    # 文件
└── preferences/                              # 偏好设置
```

### 数据库存储

| 数据库 | URI | 路径 |
|--------|-----|------|
| 短信数据库 | datashare:///com.ohos.smsmmsability | /data/misc_de/0/mms/ |
| 联系人数据库 | datashare:///com.ohos.contactsdataability | /data/misc_de/0/contacts/ |

---

## 运行时加载关系

### 启动流程

```
1. 系统启动
   ↓
2. 加载 com.ohos.mms HAP
   ↓
3. 创建 Application (MyAbilityStage)
   ↓
4. 启动 MainAbility
   ↓
5. 加载首页 (pages/index)
   ↓
6. 初始化 Service 和 Model
   ↓
7. 注册 StaticSubscriber
```

### 依赖服务

| 服务 | 包名 | 用途 |
|------|------|------|
| 短信服务 | com.ohos.smsmmsability | 短信数据存储 |
| 联系人服务 | com.ohos.contactsdataability | 联系人查询 |
| 电话服务 | 系统服务 | 拨打电话 |
| 通知服务 | 系统服务 | 发送通知 |

---

## 构建命令

### 常用命令

```bash
# 调试构建
hvigorw assembleDebug

# 发布构建
hvigorw assembleRelease

# 清理
hvigorw clean

# 安装到设备
hdc install entry/build/outputs/default/entry-default-signed.hap

# 卸载
hdc uninstall com.ohos.mms
```

### 构建变体

| 变体 | 用途 | 签名 |
|------|------|------|
| debug | 开发调试 | 调试签名 |
| release | 正式发布 | 发布签名 |

---

## 相关链接

- [目录结构](02_DirectoryStructure.md) - 源代码组织
- [项目概览](00_Overview.md) - 项目信息

---

*构建配置基于: build-profile.json5, module.json5*
