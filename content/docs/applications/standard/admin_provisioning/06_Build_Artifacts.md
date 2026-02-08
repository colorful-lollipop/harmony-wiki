# 06_编译产物

> 本文档基于代码证据编写，证据来源见各章节引用。

## 产物概述

AdminProvisioning 项目编译生成 **HAP (Harmony Ability Package)** 安装包，这是 OpenHarmony 应用的标准分发格式。

### 产物类型

| 产物类型 | 格式 | 说明 |
|---------|-----|-----|
| **HAP** | `.hap` | Harmony Ability Package，可安装的应用包 |

---

## 产物清单

### Debug 构建

| 产物 | 预期路径 | 实际路径 | 说明 |
|-----|---------|---------|-----|
| adminprovisioning.hap | `build/outputs/hap/debug/phone/` | `build/outputs/hap/debug/phone/adminprovisioning.hap` | 未签名调试版本 |

**证据** (`doc/Instructions.md:120`):
> "编译完成后，hap包会生成在工程目录下的 `\build\outputs\hap\debug\phone\` 路径下"

### Release 构建

| 产物 | 预期路径 | 实际路径 | 说明 |
|-----|---------|---------|-----|
| adminprovisioning.hap | `build/outputs/hap/release/phone/` | `build/outputs/hap/release/phone/adminprovisioning.hap` | 已签名发布版本 |

**证据** (`doc/Instructions.md:132`):
> "编译完成后，hap包会生成在工程目录下的 `\build\outputs\hap\release\phone\` 路径下（配置好签名后，生成的hap包会显示signed）"

---

## 产物结构

### HAP 包内部结构

```
adminprovisioning.hap
├── META-INF/
│   ├── CERT.SF
│   ├── CERT.RSA
│   └── MANIFEST.MF
├── libs/                           # 共享库（如果有）
├── entry/
│   ├── src/
│   │   └── main/
│   │       ├── ets/               # ArkTS 源码编译产物
│   │       │   ├── Application/
│   │       │   ├── MainAbility/
│   │       │   ├── pages/
│   │       │   └── common/
│   │       └── resources/         # 资源文件
│   └── module.json                # 模块配置
└── profile.cfg                    # 配置文件
```

### 关键文件说明

| 文件/目录 | 用途 |
|----------|-----|
| `META-INF/` | 签名信息 |
| `entry/` | 模块根目录 |
| `entry/src/main/ets/` | ArkTS 字节码 (.abc) |
| `entry/src/main/resources/` | 编译后的资源 |
| `entry/module.json` | 模块配置 |
| `profile.cfg` | 能力配置 |

---

## 安装路径

### 系统应用安装

**安装路径**: `/system/app/com.ohos.adminprovisioning/`

**证据** (`BUILD.gn:26`):
```gn
module_install_dir = "app/com.ohos.adminprovisioning"
```

**证据** (`doc/Instructions.md:168-186`):
```
8. 将签名好的 hap 包放入设备的 `/system/app` 目录下，并修改hap包的权限。
   hdc file send 本地路径 /system/app/hap包名称
   例如：hdc file send adminprovisioning.hap /system/app/adminprovisioning.hap
```

### 权限要求

安装到 `/system/app` 需要：
1. **系统权限**: 必须是系统应用
2. **设备 root**: 需要执行 `hdc target mount`
3. **重启生效**: 安装后需要重启系统

**证据** (`doc/Instructions.md:188`):
> "AdminProvisioning属于系统应用，在将签名的 hap 包放入 `/system/app` 目录后，重启系统，应用会自动拉起。"

---

## 运行时加载关系

### 应用启动流程

```
系统启动
    ↓
读取 /system/app/com.ohos.adminprovisioning/adminprovisioning.hap
    ↓
解析 META-INF/ 签名信息
    ↓
加载 module.json 配置
    ↓
实例化 AbilityStage
    ↓
根据 intent 启动对应 Ability
    ├── MainAbility (DA 模式)
    ├── AutoManagerAbility (SDA 模式)
    └── MDMUIExtensionAbility (UI 扩展)
    ↓
加载页面 UI
    ↓
应用就绪
```

### 模块加载顺序

```
1. 加载 META-INF 签名验证
2. 解析 module.json
3. 初始化 AbilityStage
4. 按需创建 Ability 实例
5. 加载资源文件
6. 渲染页面 UI
```

---

## 签名配置

### 签名文件

| 文件 | 路径 | 用途 |
|-----|-----|-----|
| adminprovisioning.p7b | `signature/adminprovisioning.p7b` | 发布签名证书 |

**证据** (`BUILD.gn:22`):
```gn
certificate_profile = "signature/adminprovisioning.p7b"
```

### 签名验证

HAP 包安装时会验证：
1. **签名完整性**: 确保包未被篡改
2. **证书有效期**: 检查签名证书是否有效
3. **应用权限**: 根据签名授予相应权限

---

## 产物版本信息

### 版本定义

**来源**: `AppScope/app.json:5-6`

```json
{
  "versionCode": 1000001,
  "versionName": "1.0.1"
}
```

| 字段 | 值 | 说明 |
|-----|-----|-----|
| versionCode | 1000001 | 内部版本号 (用于版本比较) |
| versionName | 1.0.1 | 用户可见版本号 |

### API 版本

**来源**: `AppScope/app.json:10-11`

```json
{
  "minAPIVersion": 9,
  "targetAPIVersion": 9
}
```

| 字段 | 值 | 说明 |
|-----|-----|-----|
| minAPIVersion | 9 | 最低支持的系统 API 版本 |
| targetAPIVersion | 9 | 目标系统 API 版本 |

---

## 构建产物验证

### 文件完整性检查

```bash
# 检查 HAP 文件是否存在
ls -la build/outputs/hap/debug/phone/adminprovisioning.hap
ls -la build/outputs/hap/release/phone/adminprovisioning.hap

# 检查签名
hdc shell
hdc app verify --path /system/app/adminprovisioning.hap
```

### 安装验证

```bash
# 推送 HAP 到设备
hdc file send adminprovisioning.hap /system/app/

# 设置权限
hdc shell chmod 666 /system/app/adminprovisioning.hap

# 重启验证
hdc shell reboot

# 检查应用状态
hdc shell bm dump
```

---

## 常见问题

### Q1: Debug 和 Release 构建有什么区别？

| 特性 | Debug | Release |
|-----|-------|---------|
| 签名 | 未签名 | 已签名 |
| 优化 | 无 | 代码优化 |
| 大小 | 较大 | 较小 |
| 用途 | 开发调试 | 生产发布 |

### Q2: HAP 安装后应用不启动？

可能原因：
1. 签名未正确配置
2. 权限未声明
3. 系统版本不兼容

**解决方案** (`doc/Instructions.md:194-198`):
```bash
# 清除缓存后重试
hdc shell rm -rf /data/misc_de/0/mdds/0/default/bundle_manager_service
hdc shell rm -rf /data/accounts
```

### Q3: 如何确认 HAP 已正确安装？

```bash
# 查看已安装应用列表
hdc shell bm list

# 查看应用信息
hdc shell bm dump -n com.ohos.adminprovisioning
```

---

*文档版本: 1.0*
