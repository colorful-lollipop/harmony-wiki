# OpenHarmony Print Scan Framework - GN Targets 与编译产物

**目的**: 理解构建系统结构、关键 targets、依赖关系和编译产物

**适用范围**: OpenHarmony Print Scan Framework 3.1 - GN 构建系统

---

## 目录

- [构建概览](#构建概览)
- [核心构建目标](#核心构建目标)
- [编译特性开关](#编译特性开关)
- [编译产物清单](#编译产物清单)
- [安装路径说明](#安装路径说明)

---

## 构建概览

### 构建系统
- **构建工具**: GN (Generate Ninja) + Ninja
- **配置文件**: `print.gni`（根配置）、各 BUILD.gn
- **目标产物**: .so（共享库）、.a（静态库）、.hap（应用包）、可执行文件

### 子系统与组件信息
| 属性 | 值 |
|------|-----|
| **子系统名称** | `print` |
| **组件名称** | `print_fwk` |
| **系统能力** | `SystemCapability.Print.PrintFramework` |
| **ROM 占用** | 约 2MB |
| **RAM 占用** | 约 10MB |

---

## 核心构建目标

### 打印相关 Targets

| Target 名称 | 类型 | BUILD.gn 位置 | 产物 | 说明 |
|------------|------|--------------|------|------|
| **print_napi** | ohos_shared_library | `interfaces/kits/napi/print_napi/BUILD.gn` | `libprint_napi.so` | 打印 N-API 模块 |
| **print_client** | ohos_shared_library | `frameworks/innerkitsimpl/print_impl/BUILD.gn` | `libprint_client.so` | 打印客户端（IPC 客户端） |
| **print_helper** | ohos_shared_library | `frameworks/helper/print_helper/BUILD.gn` | `libprint_helper.so` | 打印辅助工具库 |
| **print_models** | ohos_shared_library | `frameworks/models/print_models/BUILD.gn` | `libprint_models.so` | 打印数据模型库 |
| **ohprint** | ohos_shared_library | `frameworks/ohprint/BUILD.gn` | `libohprint.so` | 打印 NDK 接口库 |
| **print_service** | ohos_shared_library | `services/print_service/BUILD.gn` | `libprint_service.so` | 打印系统服务（SA） |
| **print_extension_framework** | ohos_shared_library | `frameworks/kits/extension/BUILD.gn` | `libprint_extension_framework.so` | 打印扩展框架 |
| **printextensionability_napi** | ohos_shared_library | `interfaces/kits/jsnapi/print_extension/BUILD.gn` | `libprintextensionability_napi.so` | 打印扩展能力 N-API |
| **printextensioncontext_napi** | ohos_shared_library | `interfaces/kits/jsnapi/print_extensionctx/BUILD.gn` | `libprintextensioncontext_napi.so` | 打印扩展上下文 N-API |
| **print_service_test** | ohos_static_library | `services/print_service/BUILD.gn` | `libprint_service_test.a` | 打印服务测试库 |

### 扫描相关 Targets

| Target 名称 | 类型 | BUILD.gn 位置 | 产物 | 说明 |
|------------|------|--------------|------|------|
| **scan_napi** | ohos_shared_library | `interfaces/kits/napi/scan_napi/BUILD.gn` | `libscan_napi.so` | 扫描 N-API 模块 |
| **scan_client** | ohos_shared_library | `frameworks/innerkitsimpl/scan_impl/BUILD.gn` | `libscan_client.so` | 扫描客户端（IPC 客户端） |
| **scan_helper** | ohos_shared_library | `frameworks/helper/scan_helper/BUILD.gn` | `libscan_helper.so` | 扫描辅助工具库 |
| **scan_models** | ohos_shared_library | `frameworks/models/scan_models/BUILD.gn` | `libscan_models.so` | 扫描数据模型库 |
| **ohscan** | ohos_shared_library | `frameworks/ohscan/BUILD.gn` | `libohscan.so` | 扫描 NDK 接口库 |
| **scan_service** | ohos_shared_library | `services/scan_service/BUILD.gn` | `libscan_service.so` | 扫描系统服务（SA） |
| **scan_service_test** | ohos_static_library | `services/scan_service/BUILD.gn` | `libscan_service_test.a` | 扫描服务测试库 |

### SANE 相关 Targets

| Target 名称 | 类型 | BUILD.gn 位置 | 产物 | 说明 |
|------------|------|--------------|------|------|
| **sane_backends** | ohos_shared_library | `frameworks/ISaneBackends/BUILD.gn` | `libsane_backends.so` | SANE 后端接口库 |
| **sane_service** | ohos_shared_library | `services/sane_service/BUILD.gn` | `libsane_service.so` | SANE 后端服务（SA） |

### 配置文件 Targets

| Target 名称 | 类型 | BUILD.gn 位置 | 产物 | 说明 |
|------------|------|--------------|------|------|
| **print_sa_profiles** | - | `profile/BUILD.gn` | `print_sa_profiles.xml` | 打印服务 SA Profile |
| **scan_sa_profiles** | - | `profile/BUILD.gn` | `scan_sa_profiles.xml` | 扫描服务 SA Profile |
| **sane_sa_profiles** | - | `profile/BUILD.gn` | `sane_sa_profiles.xml` | SANE 服务 SA Profile |
| **printservice.rc** | - | `etc/init/BUILD.gn` | `printservice.rc` | 打印服务启动配置 |
| **scanservice.rc** | - | `etc/init/BUILD.gn` | `scanservice.rc` | 扫描服务启动配置 |
| **saneservice.rc** | - | `etc/init/BUILD.gn` | `saneservice.rc` | SANE 服务启动配置 |
| **cupsd.conf** | - | `etc/init/BUILD.gn` | `cupsd.conf` | CUPS 打印服务器配置 |
| **cups-files.conf** | - | `etc/init/BUILD.gn` | `cups-files.conf` | CUPS 文件配置 |
| **cups_service.cfg** | - | `etc/init/BUILD.gn` | `cups_service.cfg` | CUPS 服务配置 |
| **scanservice.cfg** | - | `etc/init/BUILD.gn` | `scanservice.cfg` | 扫描服务配置 |
| **saneservice.cfg** | - | `etc/init/BUILD.gn` | `saneservice.cfg` | SANE 服务配置 |
| **print.para** | - | `etc/param/BUILD.gn` | `print.para` | 打印参数配置 |
| **print.para.dac** | - | `etc/param/BUILD.gn` | `print.para.dac` | 打印参数 DAC 配置 |
| **enterprise_cfgs** | - | `etc/init/BUILD.gn` | 企业模式配置文件 |

### 动画相关 Targets

| Target 名称 | 类型 | BUILD.gn 位置 | 产物 | 说明 |
|------------|------|--------------|------|------|
| **anipackage** | ohos_shared_library | `interfaces/kits/ani/printani/BUILD.gn` | `libanipackage.so` | 动画包 |

---

## 编译特性开关

### Feature Flags（从 bundle.json）

| Feature Flag | 默认值 | 说明 |
|-------------|---------|------|
| **print_fwk_feature_enterprise** | 可选 | 企业模式，支持企业级打印策略 |
| **print_fwk_feature_edm_service** | 可选 | EDM (Enterprise Device Management) 服务支持 |
| **print_fwk_feature_virtual_printer** | 可选 | 虚拟打印机，支持 PDF 输出 |
| **print_fwk_feature_smb_printer** | 可选 | SMB 网络打印机支持 |

### 条件编译选项

| 选项 | BUILD.gn 位置 | 条件 | 说明 |
|------|--------------|-------|------|
| **cups_enable** | `services/print_service/BUILD.gn` | - | 启用 CUPS 支持 |
| **security_guard_enabled** | `services/print_service/BUILD.gn` | - | 启用 Security Guard 支持 |
| **debug_enable** | `services/scan_service/BUILD.gn` | `if (debug_enable)` | 启用调试功能 |
| **build_variant** | 多个 BUILD.gn | `if (build_variant == "user")` | 用户发布版本 |

### 定义宏

| 宏 | 说明 |
|-----|------|
| **CUPS_ENABLE** | `-DCUPS_ENABLE` - 启用 CUPS 支持 |
| **ENTERPRISE_ENABLE** | `-DENTERPRISE_ENABLE` - 启用企业模式 |
| **EDM_SERVICE_ENABLE** | `-DEDM_SERVICE_ENABLE` - 启用 EDM 服务 |
| **VIRTUAL_PRINTER_ENABLE** | `-DVIRTUAL_PRINTER_ENABLE` - 启用虚拟打印机 |
| **HAVE_SMB_PRINTER** | `-DHAVE_SMB_PRINTER` - 启用 SMB 打印机 |
| **SECURITY_GUARDE_ENABLE** | `-DSECURITY_GUARDE_ENABLE` - 启用安全守卫 |
| **DEBUG_ENABLE** | `-DDEBUG_ENABLE` - 启用调试模式 |
| **IS_RELEASE_VERSION** | `-DIS_RELEASE_VERSION` - 发布版本（移除调试代码） |

---

## 编译产物清单

### N-API 模块（.so）

| 产物名称 | 产物文件 | 安装目录 | 加载方 |
|----------|----------|----------|--------|
| **print_napi** | `libprint_napi.so` | `/system/lib/module/` | JS 应用动态加载 |
| **scan_napi** | `libscan_napi.so` | `/system/lib/module/` | JS 应用动态加载 |
| **printextensionability_napi** | `libprintextensionability_napi.so` | `/system/lib/module/app/ability/` | 打印扩展动态加载 |
| **printextensioncontext_napi** | `libprintextensioncontext_napi.so` | `/system/lib/module/` | 打印扩展上下文动态加载 |

### NDK 接口库（.so）

| 产物名称 | 产物文件 | 安装目录 | 加载方 |
|----------|----------|----------|--------|
| **ohprint** | `libohprint.so` | `/system/lib/ndk/` | Native 应用静态/动态链接 |
| **ohscan** | `libohscan.so` | `/system/lib/ndk/` | Native 应用静态/动态链接 |

### 框架内部库（.so）

| 产物名称 | 产物文件 | 安装目录 | 加载方 |
|----------|----------|----------|--------|
| **print_client** | `libprint_client.so` | `/system/lib/` | 打印框架内部链接 |
| **scan_client** | `libscan_client.so` | `/system/lib/` | 扫描框架内部链接 |
| **print_helper** | `libprint_helper.so` | `/system/lib/` | 打印辅助库链接 |
| **scan_helper** | `libscan_helper.so` | `/system/lib/` | 扫描辅助库链接 |
| **print_models** | `libprint_models.so` | `/system/lib/` | 打印模型库链接 |
| **scan_models** | `libscan_models.so` | `/system/lib/` | 扫描模型库链接 |
| **print_extension_framework** | `libprint_extension_framework.so` | `/system/lib/` | 扩展框架链接 |
| **anipackage** | `libanipackage.so` | `/system/lib/` | 动画包链接 |
| **sane_backends** | `libsane_backends.so` | `/system/lib/` | SANE 后端接口链接 |

### 系统服务库（.so）

| 产物名称 | 产物文件 | 安装目录 | SA 注册 |
|----------|----------|----------|--------|
| **print_service** | `libprint_service.so` | `/system/lib/` | PrintServiceAbility（PRINT_SERVICE_ID） |
| **scan_service** | `libscan_service.so` | `/system/lib/` | ScanServiceAbility（SCAN_SERVICE_ID） |
| **sane_service** | `libsane_service.so` | `/system/lib/` | SaneServerManager（SANE_SERVICE_ID） |

### 测试库（.a）

| 产物名称 | 产物文件 | 用途 |
|----------|----------|--------|
| **print_service_test** | `libprint_service_test.a` | 打印服务单元测试 |
| **scan_service_test** | `libscan_service_test.a` | 扫描服务单元测试 |

---

## 安装路径说明

### 系统库安装路径

```
/system/lib/
├── module/
│   ├── libprint_napi.so                 # 打印 N-API 模块
│   ├── libscan_napi.so                  # 扫描 N-API 模块
│   └── app/ability/
│       └── libprintextensionability_napi.so  # 打印扩展能力
├── ndk/
│   ├── libohprint.so                   # 打印 NDK 接口
│   └── libohscan.so                    # 扫描 NDK 接口
└── (其他内部库）
    ├── libprint_client.so                # 打印客户端
    ├── libscan_client.so                 # 扫描客户端
    ├── libprint_helper.so               # 打印辅助库
    ├── libscan_helper.so                # 扫描辅助库
    ├── libprint_models.so               # 打印模型库
    ├── libscan_models.so                # 扫描模型库
    ├── libprint_extension_framework.so    # 扩展框架
    ├── libanipackage.so                # 动画包
    └── libsane_backends.so              # SANE 后端接口
```

### 配置文件安装路径

```
/etc/
├── printservice.rc                    # 打印服务启动配置
├── scanservice.rc                     # 扫描服务启动配置
├── saneservice.rc                     # SANE 服务启动配置
└── cups/
    ├── cupsd.conf                       # CUPS 打印服务器配置
    ├── cups-files.conf                   # CUPS 文件配置
    └── (企业模式配置文件)
```

### 服务数据路径

```
/data/service/el2/public/print_service/
├── printers.json                     # 打印机列表
├── printers_enterprise/              # 企业打印机列表（企业模式）
├── sane/tmp/                          # SANE 临时目录
└── (打印任务和用户数据）
```

### SA Profile 安装路径

```
/system/profile/
├── print_sa_profiles.xml              # 打印服务 SA 配置
├── scan_sa_profiles.xml               # 扫描服务 SA 配置
└── sane_sa_profiles.xml               # SANE 服务 SA 配置
```

---

## 运行时加载关系

### 打印系统加载顺序

```
系统启动
  ↓
init 进程加载 printservice.rc
  ↓
PrintServiceAbility 服务启动
  ↓
PrintServiceAbility 注册为 SA (PRINT_SERVICE_ID)
  ↓
SystemAbilityManager 管理服务生命周期
  ↓
JS 应用调用 import('print')
  ↓
libprint_napi.so 动态加载
  ↓
N-API 绑定初始化
```

### 扫描系统加载顺序

```
系统启动
  ↓
init 进程加载 scanservice.rc
  ↓
ScanServiceAbility 服务启动
  ↓
ScanServiceAbility 注册为 SA (SCAN_SERVICE_ID)
  ↓
SaneManagerClient 检查/加载 SANE 服务
  ↓
SaneServerManager 注册为 SA (SANE_SERVICE_ID = 3709)
  ↓
JS 应用调用 import('scan')
  ↓
libscan_napi.so 动态加载
  ↓
N-API 绑定初始化
```

### 扩展系统加载顺序

```
打印扩展应用启动
  ↓
libprintextensionability_napi.so 动态加载
  ↓
PrintExtensionAbility 初始化
  ↓
PrintExtensionAbility 连接到 PrintServiceAbility
  ↓
通过扩展接口发现和管理打印机
```

---

## 依赖关系图

```
应用层依赖:
print_napi, scan_napi, printextensionability_napi
    ↓ depends on
print_client, scan_client, print_helper, scan_helper, print_models, scan_models
    ↓
print_client, scan_client
    ↓ IPC to
print_service, scan_service, sane_service
    ↓
print_service
    ↓ uses
CUPS, VendorDrivers (BSUNI, IPP Everywhere, WLAN, SMB)
    ↓
scan_service
    ↓ uses
SaneServerManager
    ↓
SANE Backends (third_party)
```

---

## 编译命令

### 基本编译

```bash
# 生成构建文件
gn gen out --args="target_os=ohos --default-target-arch=arm64"

# 编译所有目标
ninja -C out

# 编译特定目标
ninja -C out print_napi
ninja -C out scan_napi
ninja -C out print_service
ninja -C out scan_service
ninja -C out sane_service
```

### 清理命令

```bash
# 清理所有构建产物
ninja -C out -t clean

# 清理 GN 生成文件
rm -rf out/
```

---

**相关链接**:
- [项目概览](00_Overview.md)
- [对外 API](04_External_API.md)
- [内部架构](05_Internal_API.md)
- [安全风险评审](08_Security_Review.md)
