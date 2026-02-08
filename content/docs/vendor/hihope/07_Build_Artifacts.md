# 编译产物说明

## 文档信息

- **目的**：详细说明 vendor_hihope 仓库的编译产物类型、生成位置、安装路径和运行时加载关系
- **适用范围**：vendor_hihope 仓库的所有产品线
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 仓库主要生成配置文件和预构建文件
  - 实际的 .so/.a/.hap 等产物由 OpenHarmony 框架生成
  - 配置文件（.json/.xml/.hcs）安装到系统目录

## 编译产物概述

### 产物分类

| 产物类型 | 扩展名 | 生成位置 | 安装路径 | 说明 |
|---------|---------|----------|------|------|
| **配置文件** | .json, .xml, .hcs | {product}/ | /etc/, /data/, /vendor/ | 系统配置文件 |
| **共享库** | .so | out/{product}/libs/ | /usr/lib/ | 动态链接库 |
| **静态库** | .a | out/{product}/obj/ | - | 链接到可执行文件 |
| **HAP 包** | .hap | out/{product}/packages/phone/ | /data/app/ | 应用包 |
| **系统镜像** | .bin, .img, .img | out/{product}/packages/phone/ | / | 系统镜像 |
| **可执行文件** | 无（标准系统） | / | - | 通常为系统服务 |

## 配置文件产物

### 标准产品配置文件

#### Audio HAL 配置文件（rk3568 产物）

| 配置文件 | 产物 | 安装路径 | 说明 |
|---------|------|---------|------|
| **alsa_paths.json** | hdfconfig/alsa_paths.json | /etc/hdfconfig/ | ALSA 设备路径列表 |
| **alsa_adapter.json** | hdfconfig/alsa_adapter.json | /etc/hdfconfig/ | ALSA 适配器配置 |
| **audio_effect.json** | hdfconfig/audio_effect.json | /etc/hdfconfig/ | 音效配置 |
| **audio_path.json** | hdfconfig/audio_path.json | /etc/hdfconfig/ | 音频设备路径 |
| **audio_adapter.json** | hdfconfig/audio_adapter.json | /etc/hdfconfig/ | 音频适配器配置 |
| **audio_policy_config.xml** | audio_policy_config.xml | /etc/audio/ | ARM 音频策略 |
| **audio_policy_config_new.xml** | audio_policy_config_new.xml | /etc/audio/ | 新音频策略 |

**证据**：
- 构建定义：`rk3568/hals/audio/BUILD.gn:16-49`
- 安装配置：`relative_install_dir = "audio"` 和 `install_images = [chipset_base_dir]`

#### 预安装应用配置

| 配置文件 | 产物 | 安装路径 | 说明 |
|---------|------|---------|------|
| **install_list.json** | install_list.json | /etc/preinstall_config/ | 预安装应用列表 |
| **install_list_permissions.json** | install_list_permissions.json | /etc/preinstall_config/ | 预安装应用权限 |
| **install_list_capability.json** | install_list_capability.json | /etc/preinstall_config/ | 预安装应用能力 |

**证据**：
- 构建定义：`rk3568/preinstall-config/BUILD.gn`
- 安装配置：`relative_install_dir = "etc/preinstall_config"` 和 `install_images = [system_base_dir]`

#### 安全配置文件

| 配置文件 | 产物 | 安装路径 | 说明 |
|---------|------|---------|------|
| **critical_reboot_process_list.json** | critical_reboot_process_list.json | /etc/ | 关键重启进程列表 |
| **high_privilege_process_list.json** | high_privilege_process_list.json | /etc/ | 高权限进程列表 |

**证据**：
- 构建定义：`rk3568/security_config/BUILD.gn`

#### HDF 配置文件

**UHDF 和 KHDF 配置**：.hcs 文件通过 HDF 框架加载到内核或用户空间，不需要单独安装到文件系统。

## 系统镜像产物

### 镜像类型

| 镜像类型 | 典型文件名 | 说明 | 生成位置 |
|---------|------------|------|---------|
| **系统镜像** | system.img, vendor.img | OpenHarmony 完整系统镜像 | OpenHarmony 构建系统 |
| **Boot 镜像** | boot.img, ramdisk.img | 启动镜像 | OpenHarmony 构建系统 |
| **Updater 镜像** | updater.img | OTA 更新镜像 | OpenHarmony 构建系统 |
| **参数镜像** | params.bin | 系统参数镜像 | OpenHarmony 构建系统 |

**说明**：
- ✅ 这些镜像由 OpenHarmony 构建系统生成
- ⚠️ vendor_hihope 仓库**不生成**这些镜像
- ✅ vendor_hihope 仓库仅提供配置信息给构建系统

### 镜像安装路径

| 镜像类型 | 安装位置 | 说明 |
|---------|----------|------|
| **系统镜像** | `/` | 根分区（system, vendor, data, product） |
| **Boot 镜像** | `/` | 启动分区（boot） |
| **Updater 镜像** | `/` | 升级分区（update 或 updater） |

## 运行时加载关系

### HDF 驱动加载

```
┌─────────────────────────────────────────┐
│  系统启动                    │
│  (Kernel/Init)              │
└────────────┬────────────────────┘
             │
        ┌────────────▼──────────────┐
        │   HDF 加载器             │  [内核/HDF 框架]
        │  （hdf_devmgr）           │
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │  解析 .hcs 配置           │  [HDF 配置解析器]
        │  （解析 device_info.hcs）     │
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │  加载内核驱动模块         │  [内核驱动]
        │  （按 priority 加载）       │
        │  - Platform GPIO           │
        │  - Platform UART           │
        │  - Sensor Drivers          │
        │  - ...                     │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │  注册 HDF 服务             │  [HDF 服务管理器]
        │  （注册到 HDF 框架）       │
        │  - UHDF Services          │  [用户态服务]
        │  - KHDF Services          │  [内核态服务]
        └───────────────────────────────┘
             │
        ┌────────────▼──────────────┐
        │  应用访问 HDF 服务         │  [应用/框架]
        │  （通过 HDI 接口）         │
        │  - Audio HAL               │
        │  - Camera HAL               │
        │  - Sensor HAL               │
        │  - ...                     │
        └───────────────────────────────┘
```

### 加载顺序

**priority 值**（证据：HDF 配置）：
- 较小的值 = 早期加载（如 10-50）
- 较大的值 = 后期加载（如 100-300）

**加载流程**：
1. 系统启动
2. HDF DevMgr 初始化
3. 解析 .hcs 配置文件
4. 按 priority 顺序加载驱动
5. 驱动初始化
6. 服务注册到 System Ability Manager
7. 等待应用访问

### 运行时设备节点

**HDF 设备节点**：
- 位置：`/dev/{service_name}或{module_name}`
- 权限：由 HDF 管理（通常是 root 或 system）
- 访问：通过 HDI 接口或/dev 节点

**证据**：
- 配置：`rk3568/hdf_config/khdf/device_info.hcs`（定义 moduleName 和 serviceName）
- 设备节点：`/dev/HDF_PLATFORM_UART_0` 等

## 产物验证

### 验证配置文件

```bash
# 检查配置文件是否安装
ls -l /etc/hdfconfig/alsa_paths.json
ls -l /etc/audio/audio_policy_config.xml
ls -l /etc/preinstall_config/install_list.json
```

### 验证服务状态

```bash
# 查看 HDF 服务状态
hdc shell
hdf -h

# 查看特定服务
hdc shell hilog -T HDF | grep service_name
```

### 验证模块加载

```bash
# 检查内核模块加载
hdc shell
lsmod | grep driver_name

# 检查 dmesg 日志
hdc shell
dmesg | grep -i hdf
```

## 产物管理

### 备份建议

| 产物类型 | 备份建议 | 备份位置 |
|---------|----------|----------|
| **配置文件** | 定期备份 Git | 仓库 Git 历史 |
| **系统镜像** | 导出镜像文件 | 安全存储位置 |
| **HAP 包** | 保留发布版本 | 版本管理 |

### 清理策略

| 产物类型 | 清理方法 | 清理命令 |
|---------|----------|----------|
| **临时文件** | 删除缓存目录 | `hb clean` 或手动删除 |
| **旧镜像** | 删除旧的系统镜像 | 手动清理或 `rm` |
| **调试符号** | 剥离可执行文件 | `hb build --strip` |

## 产品特定产物差异

### Standard 系统产物

| 产品 | 配置文件数 | 镜像类型 | 说明 |
|------|----------|----------|------|
| **rk3568** | 最完整 | system.img, boot.img, ramdisk.img | 最全面的系统镜像 |
| **dayu210** | 完整 | 同 rk3568 | DAYU210 特定配置 |
| **tablet_core_system** | 完整 | 同 rk3568 | 平板特定配置 |
| **tv** | 完整 | 同 rk3568 | 电视特定配置 |
| **wearable** | 完整 | 同 rk3568 | 可穿戴特定配置 |
| **default_core_system** | 完整 | 同 rk3568 | 基础配置 |
| **ipcamera_core_system** | 完整 | 同 rk3568 | IPC 摄像头特定配置 |

### Mini 系统产物

| 产品 | 配置文件数 | 镜像类型 | 说明 |
|------|----------|----------|------|
| **neptune_iotlink_demo** | 精简 | LiteOS-M 系统镜像 | 小型系统镜像 |
| **nearlink_dk_3863** | 精简 | LiteOS-M 系统镜像 | NearLink 系统镜像 |

**差异**：
- ✅ Mini 系统使用更小的镜像
- ✅ 不包含完整的图形组件
- ✅ 系统服务精简

## 相关跳转

- [GN Targets 说明](06_GN_Targets.md)
- [目录结构详解](02_Directory_Structure.md)
- [返回 Wiki 首页](SUMMARY.md)
