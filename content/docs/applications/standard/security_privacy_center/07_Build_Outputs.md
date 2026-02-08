# 编译产物

## 产物概述

本项目编译生成的最终产物为 **HarmonyOS Ability Package（.hap）** 文件。

| 属性 | 值 |
|-----|---|
| 产物类型 | .hap（HarmonyOS Ability Package） |
| 模块类型 | entry（入口模块） |
| 签名方式 | OpenHarmony 默认签名 |

## 产物清单

### Debug 模式产物

```
entry/build/default/outputs/default/
└── entry-default-signed.hap
```

### Release 模式产物

```
entry/build/release/outputs/release/
└── entry-release-signed.hap
```

### 产物结构（解压后）

```
entry-default-signed.hap/
├── META-INF/
│   └── MANIFEST.MF          # 签名清单
├── libs/                     # 依赖库（如有）
│   └── lib*.so              # Native 库（本项目无）
├── resources.index           # 资源索引
├── Module4陕西梆子ect.png       # 模块图标
├── oh-package.json5         # 包描述
├── config.json              # 模块配置
└── ets/                      # ArkTS/ETS 字节码
    ├── index.abc            # Index 页面
    ├── locationServices.abc # 位置服务页
    └── ...
```

## 产物说明

### 1. MANIFEST.MF

包含签名信息，用于验证包的完整性和来源。

### 2. .abc 文件

ArkTS/ETS 编译生成的字节码文件，由方舟运行时解释执行。

| 字节码文件 | 对应页面/模块 |
|-----------|--------------|
| index.abc | Index.ets（首页） |
| locationServices.abc | locationServices.ets（位置服务页） |
| UiExtensionPage.abc | UiExtensionPage.ets |
| 公共模块.abc | common/、model/、utils/ 等 |

### 3. config.json

模块配置信息，同步自 `module.json5`。

## 安装路径

### 开发调试安装

| 安装方式 | 目标路径 |
|---------|---------|
| hdc install | /data/app/ |
| DevEco Studio 部署 | /data/app/ |

### 系统安装

集成到系统镜像后：
- 路径：`/system/app/PrivacyCenter/` 或类似路径
- 权限：系统应用权限

## 运行时加载关系

### 加载流程

```
用户点击"设置" → "隐私"菜单
        │
        ▼
系统加载安全隐私中心模块
        │
        ▼
加载 entry-default-signed.hap
        │
        ├──▶ 解析 MANIFEST.MF（验证签名）
        ├──▶ 加载 ets/ 字节码
        │       │
        │       ├──▶ Index.abc（首页）
        │       ├──▶ locationServices.abc（位置页）
        │       └──▶ UiExtensionPage.abc（UIExtension）
        │
        └──▶ 初始化 EntryAbility
                │
                ├──▶ onCreate()
                ├──▶ onWindowStageCreate()
                │       │
                │       └──▶ loadContent('pages/Index')
                │               │
                │               └──▶ 渲染 Index 页面
                │
                └──▶ 初始化位置服务监听
```

### 资源加载

```
resources.index
    │
    ├──▶ 加载默认语言资源（base/）
    │       │
    │       ├──▶ string.json（字符串）
    │       ├──▶ color.json（颜色）
    │       └──▶ float.json（尺寸）
    │
    └──▶ 根据设备语言加载对应翻译
            │
            ├──▶ zh_CN/element/string.json（中文）
            └──▶ en_US/element/string.json（英文）
```

## 产物签名

### 签名配置

| 配置项 | 值 |
|-------|---|
| 签名算法 | SHA256withECDSA |
| 密钥库 | signature/OpenHarmony.p12 |
| 证书 | signature/OpenHarmonyApplication.cer |
| 签名证书 Profile | signature/privacyCenter.p7b |

**证据来源**：`build-profile.json5:18-30`

### 签名验证

产物安装时，系统会验证：
1. 证书链完整性
2. 签名有效性
3. Profile 匹配

## 产物大小

| 模式 | 估计大小 | 说明 |
|-----|---------|------|
| debug | ~500KB | 包含调试信息 |
| release | ~300KB | 经过优化压缩 |

**注意**：实际大小取决于编译时的资源内容。

## 产物验证

### 本地验证

```bash
# 使用 hdc 查看已安装应用
hdc shell bm dump -n com.ohos.security.privacycenter

# 查看应用信息
hdc shell cat /data/app/xxx/xxx.hap
```

### 签名验证

```bash
# 验证 HAP 签名
java -jar hapverify.jar -path entry-default-signed.hap
```

## 产物与源码映射

| 产物组件 | 对应源码目录/文件 |
|---------|------------------|
| ets/*.abc | entry/src/main/ets/ |
| resources/* | entry/src/main/resources/ |
| config.json | entry/src/main/module.json5 |
| libs/*.so | 本项目无 Native 库 |

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [06_Build_Config.md](./06_Build_Config.md) → 构建配置
- [08_Security_Review.md](./08_Security_Review.md) → 安全评审
