# 编译产物

> 本文档描述用户证书管理部件的编译产物、安装路径和运行时加载关系

---

## 目的

本文档帮助运维和测试人员了解编译产物的位置、内容和安装方式。

## 适用范围

- OpenHarmony 运维工程师
- 测试人员
- 需要部署应用的技术人员

## 关键结论

1. **单一 HAP 产物**: 最终产物为 `CertificateManager.hap`
2. **安装路径**: `app/com.ohos.certificatemanager`
3. **无 Native 代码**: 全部为 ArkTS，无 .so/.a 产物
4. **资源嵌入**: 资源文件打包在 HAP 内
5. **签名必需**: 需要签名证书才能安装

## 相关跳转

- [06_GN_Targets.md](wiki/06_GN_Targets.md) - GN Targets
- [09_FAQ.md](wiki/09_FAQ.md) - 常见问题

---

## 1. 编译产物清单

### 1.1 最终产物

| 产物名称 | 文件类型 | 大小估计 | 说明 |
|---------|----------|----------|------|
| `CertificateManager.hap` | HAP (HarmonyOS Ability Package) | ~2-5MB | 应用安装包 |

**证据位置**: `BUILD.gn:22`, `README.md:103`

### 1.2 中间产物

| 产物类型 | 文件 | 说明 |
|----------|------|------|
| `*.abc` | ArkTS 编译产物 | 编译后的字节码 |
| 资源包 | `resources.index` | 资源索引 |
| 配置文件 | `module.json`, `app.json` | 模块和应用配置 |

---

## 2. 产物位置

### 2.1 编译输出目录

```
out/rk3568/obj/applications/standard/user_certificate_manager/user_cert_manager/
└── CertificateManager.hap
```

**说明**: 产物路径随产品名称变化

**证据位置**: `README.md:103`

### 2.2 安装路径

```
/system/app/com.ohos.certificatemanager/
```

**说明**: 应用安装后的系统路径

**证据位置**: `BUILD.gn:26`

---

## 3. HAP 内容结构

### 3.1 HAP 内部结构

```
CertificateManager.hap
├── app.json                    # 应用配置
│   ├── bundleName: com.ohos.certmanager
│   ├── versionCode: 1000000
│   └── versionName: 1.0.0
├── module.json                 # 模块配置
│   ├── name: CertManager
│   ├── type: feature
│   ├── abilities: [MainAbility]
│   ├── extensionAbilities: [MainExtensionAbility, CertPickerUIExtAbility]
│   └── requestPermissions: [...]
├── assets/
│   ├── *.abc                 # 编译后的 ArkTS 代码
│   └── resources/             # 资源文件
│       ├── base/
│       ├── en_US/
│       └── zh_CN/
└── signature/
    └── privacyCenter.p7b        # 签名证书
```

**证据位置**: `BUILD.gn:21`, `certmanager/src/main/module.json`, `AppScope/app.json`

---

## 4. 运行时加载

### 4.1 加载流程

```
1. 包管理器读取 app.json
   └── 识别 Bundle 名称和版本

2. 包管理器读取 module.json
   └── 识别模块、Ability、权限

3. 包管理器安装 HAP
   └── 解压到 /system/app/com.ohos.certificatemanager/

4. Ability 框架加载 MainAbility
   └── 启动应用主入口

5. Extension 框架加载 UIExtensionAbility
   └── 支持隐私中心入口和文件选择器
```

### 4.2 依赖加载

| 依赖模块 | 加载时机 | 说明 |
|---------|----------|------|
| `@ohos.security.certManager` | 运行时 | 证书管理服务 |
| `@ohos.userIAM.userAuth` | 运行时 | 用户认证服务 |
| `@ohos.bundle.bundleManager` | 运行时 | 包管理服务 |
| `@ohos.file.fs` | 运行时 | 文件系统 |
| `@ohos.window` | 运行时 | 窗口管理 |

**证据位置**: `certmanager/src/main/ets/model/*.ets` (import 语句)

---

## 5. 安装与部署

### 5.1 编译步骤

```bash
# 1. 编译
./build.sh --product-name rk3568 --ccache --build-target user_certificate_manager

# 2. 查找产物
out/rk3568/obj/applications/standard/user_certificate_manager/user_cert_manager/CertificateManager.hap
```

**证据位置**: `README.md:89-100`

### 5.2 安装步骤

```bash
# 1. 连接设备
hdc list targets

# 2. 安装 HAP
hdc install CertificateManager.hap

# 3. 验证安装
hdc shell pm list installed -u 0 | grep com.ohos.certmanager
```

**证据位置**: `README.md:106-109`

### 5.3 卸载步骤

```bash
# 1. 卸载应用
hdc shell pm uninstall com.ohos.certmanager

# 2. 清理残留数据 (可选)
hdc shell rm -rf /data/app/el2/100/database/com.ohos.certmanager
```

---

## 6. 运行时资源

### 6.1 资源文件

| 资源类型 | 目录 | 用途 |
|----------|------|------|
| 字符串资源 | `resources/*/element/string.json` | 多语言文本 |
| 颜色资源 | `resources/base/element/color.json` | 颜色定义 |
| 浮点资源 | `resources/base/element/float.json` | 尺寸、间距 |
| 媒体资源 | `resources/base/media/` | 图片、图标 |
| 原始文件 | `resources/rawfile/` | 隐私中心配置 |

**证据位置**: `certmanager/src/main/resources/` 目录结构

### 6.2 支持的语言

| 语言 | 目录 | 状态 |
|------|------|------|
| 默认 | `base/` | ✓ |
| 英语 | `en_US/` | ✓ |
| 简体中文 | `zh_CN/` | ✓ |

**证据位置**: `certmanager/src/main/resources/` 目录

---

## 7. 签名与权限

### 7.1 签名配置

| 配置项 | 值 | 证据 |
|--------|-----|------|
| 签名证书 | `signature/privacyCenter.p7b` | `BUILD.gn:21` |
| publicity_file | `publicity.xml` | `BUILD.gn:20` |

**证据位置**: `BUILD.gn:20-21`

### 7.2 申请的权限

| 权限名称 | 用途 | 证据 |
|----------|------|------|
| `ohos.permission.GET_BUNDLE_INFO` | 获取应用信息 | `module.json:86` |
| `ohos.permission.ACCESS_CERT_MANAGER_INTERNAL` | 访问证书管理服务内部接口 | `module.json:89` |
| `ohos.permission.ACCESS_CERT_MANAGER` | 访问证书管理服务 | `module.json:92` |
| `ohos.permission.GET_BUNDLE_RESOURCES` | 获取应用资源 | `module.json:95` |
| `ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER` | 访问安全隐私中心 | `module.json:98` |
| `ohos.permission.ACCESS_BIOMETRIC` | 生物识别认证 | `module.json:101` |
| `ohos.permission.PRIVACY_WINDOW` | 防截屏窗口 | `module.json:104` |
| `ohos.permission.ACCESS_SYSTEM_APP_CERT` | 访问系统应用证书 | `module.json:107` |
| `ohos.permission.ACCESS_USER_TRUSTED_CERT` | 访问用户信任证书 | `module.json:110` |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 获取应用信息 (特权) | `module.json:113` |

**证据位置**: `certmanager/src/main/module.json:84-115`

---

## 8. 调试与日志

### 8.1 日志输出

| 模块 | TAG | 位置 |
|------|-----|------|
| CertMangerModel | `'CertMangerModel'` | `CertMangerModel.ets:79` |
| CheckUserAuthModel | `'CheckUserAuthModel'` | `CheckUserAuthModel.ets:23` |
| BundleModel | `'certManager BUNDLE:'` | `BundleModel.ets:22` |
| FileIoModel | `'CertManager FA'` | `FileIoModel.ets:22` |
| PreventScreenshotsModel | `'PreventScreenshotsModel'` | `PreventScreenshotsModel.ets:22` |
| CmFaPresenter | `'CMFaPresenter: '` | `CmFaPresenter.ets:27` |

**日志 API**: `@ohos.hilog`

**DOMAIN**: `0x0000`

**证据位置**: `certmanager/src/main/ets/model/*.ets`, `certmanager/src/main/ets/presenter/*.ets`

### 8.2 查看日志

```bash
# 查看应用日志
hdc shell hilog -x | grep CertManager

# 查看所有日志
hdc shell hilog -x

# 清除日志缓冲
hdc shell hilog -r
```

---

## 9. 常见编译问题

### 9.1 编译失败

| 问题 | 可能原因 | 解决方法 |
|------|---------|---------|
| 找不到 SDK | SDK 路径配置错误 | 检查 `sdk_home` 配置 |
| 签名失败 | 签名证书过期或不匹配 | 检查 `certificate_profile` |
| 资源编译失败 | 资源文件格式错误 | 检查 JSON 格式 |

### 9.2 安装失败

| 问题 | 可能原因 | 解决方法 |
|------|---------|---------|
| 签名不匹配 | 签名不一致 | 重新签名 |
| 权限不足 | 系统版本不支持 | 升级系统版本 |
| 存储空间不足 | 磁盘空间不足 | 清理存储空间 |

---

**END OF 07_Build_Artifacts.md**
