# OpenHarmony Updater 项目概览

## 目的

本文档提供 OpenHarmony Updater 子系统的整体介绍，包括项目定位、核心能力、运行环境和关键概念。

## 适用范围

- 应用开发者（调用 Updater API）
- 子系统开发者（修改/扩展 Updater）
- 安全工程师（评估安全风险）
- 构建工程师（定制编译配置）

## 项目定位

Updater 是 OpenHarmony **升级子系统**的核心组件，负责系统 OTA（Over-The-Air）升级包的安装。它运行在独立的 **Updater 分区**，通过读取 Misc 分区信息触发升级流程。

### 核心职责

1. **升级包校验**：验证升级包签名和完整性（`services/package/pkg_verify/`）
2. **升级包解析**：解析 ZIP/LZ4/GZIP/升级包格式（`services/package/`）
3. **差分恢复**：应用差分补丁（`services/diffpatch/`）
4. **分区操作**：擦写、挂载、格式化分区（`services/fs_manager/`）
5. **脚本执行**：执行升级脚本（`services/script/`）
6. **AB 分区切换**：支持无缝升级（`interfaces/kits/slot_info/`）

## 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|---------|
| **标准 OTA 升级** | 下载完整升级包后重启安装 | `services/updater_main.cpp` |
| **SD 卡升级** | 从 SD 卡读取升级包安装 | `services/sdcard_update/` |
| **差分升级** | 应用差分补丁减少流量 | `services/diffpatch/` |
| **AB 流式升级** | 后台流式写入，无需存储完整包 | `services/stream_update/` |
| **Flashd 刷机** | 工厂模式刷机、格式化、擦除 | `services/flashd/` |
| **恢复出厂** | 清除用户数据 | `services/factory_reset/` |
| **UI 界面** | 显示升级进度和状态 | `services/ui/` |

## 运行环境

### 分区要求

| 分区 | 大小 | 文件系统 | 说明 |
|------|------|---------|------|
| **Updater 分区** | ≥32MB | ext4/ramdisk | 独立升级分区 |
| **Misc 分区** | ~1MB | Raw | 存储升级命令和状态 |
| **System 分区** | - | ext4/erofs | 系统分区（被升级） |
| **Data 分区** | - | ext4/f2fs | 存储升级包（可选） |

### 启动流程

```
正常模式 → write_updater命令 → misc分区写入 → reboot updater
    ↓
Bootloader 读取 misc → 加载 Updater 分区 → 启动 init
    ↓
init 启动 updater 进程 → 读取升级包 → 执行升级
    ↓
升级完成 → 写入 misc → reboot
```

### 依赖组件

```json
// bundle.json:52-71
"deps": {
    "components": [
        "init",                    // 初始化系统
        "hdc",                     // 调试通道
        "drivers_interface_input", // 输入驱动
        "drivers_peripheral_partitionslot", // 分区槽驱动
        "c_utils",                 // C 工具库
        "hilog",                   // 日志系统
        "selinux_adapter",         // SELinux 适配
        "ui_lite",                 // 轻量 UI
        "graphic_utils_lite",      // 图形工具
        "bounds_checking_function", // 边界检查
        "bzip2", "cJSON", "libdrm", "libpng",
        "libuv", "lz4", "openssl", "selinux", "zlib"
    ]
}
```

## 关键概念

### Misc 分区结构

```cpp
// interfaces/kits/include/misc_info/misc_info.h:49-56
struct UpdateMessage {
    char command[MAX_COMMAND_SIZE];      // 命令: boot_updater/boot_flash
    char status[MAX_STATUS_SIZE];        // 状态
    char update[MAX_UPDATE_SIZE];        // 升级包路径
    char stage[MAX_STAGE_SIZE];          // 阶段
    char faultinfo[MAX_FAULTINFO_SIZE];  // 故障信息
    char reserved[MAX_RESERVED_SIZE];    // 保留
};
```

### 升级模式

```cpp
// services/include/updater/updater.h:39-44
enum PackageUpdateMode {
    HOTA_UPDATE = 0,       // 标准 OTA
    SDCARD_UPDATE,         // SD 卡升级
    SUBPKG_UPDATE,         // 子包升级
    UNKNOWN_UPDATE,
};
```

### 升级状态

```cpp
// services/include/updater/updater.h:26-35
enum UpdaterStatus {
    UPDATE_ERROR = -1,           // 通用错误
    UPDATE_SUCCESS,              // 成功
    UPDATE_CORRUPT,              // 包损坏或校验失败
    UPDATE_SKIP,                 // 跳过（如电量低）
    UPDATE_RETRY,                // 可重试
    UPDATE_RETRY_FAIL,           // 重试失败
    UPDATE_SPACE_NOTENOUGH,      // 空间不足
    UPDATE_UNKNOWN
};
```

### AB 分区模型

| 概念 | 说明 |
|------|------|
| **Slot A/B** | 两套系统分区，一套活跃，一套非活跃 |
| **Active Slot** | 当前运行的分区槽 |
| **Inactive Slot** | 待升级的目标分区槽 |
| **Seamless Update** | 后台升级，下次重启切换 |

```cpp
// interfaces/kits/include/slot_info/slot_info.h
namespace Updater {
    void GetPartitionSuffix(std::string &suffix);        // 获取非活跃槽后缀
    void GetActivePartitionSuffix(std::string &suffix);  // 获取活跃槽后缀
    void SetActiveSlot();                                 // 切换活跃槽
}
```

## Feature 开关

| Feature | 默认 | 说明 | 配置位置 |
|---------|------|------|---------|
| `updater_ui_support` | true | UI 支持 | `updater_default_cfg.gni` |
| `init_feature_ab_partition` | true | AB 分区支持 | 全局配置 |
| `updater_feature_use_ptable` | true | 分区表支持 | `updater_default_cfg.gni` |
| `updater_feature_sign_on_server` | true | 服务器签名 | `updater_default_cfg.gni` |
| `updater_hdc_depend` | true | HDC 调试支持 | `updater_default_cfg.gni` |

## 关键结论

1. **独立分区运行**：Updater 运行在独立的 Updater 分区，与正常系统隔离，确保升级失败时不影响正常启动。

2. **多种升级方式**：支持标准 OTA、SD 卡升级、差分升级、AB 流式升级，适应不同场景需求。

3. **Misc 分区通信**：通过 Misc 分区在 Bootloader、正常系统、Updater 之间传递命令和状态。

4. **无 N-API**：当前对外接口为纯 C/C++ 库，无 JavaScript 绑定，应用通过 Native 代码调用。

5. **安全优先**：升级包签名验证、哈希校验、完整性检查贯穿整个升级流程。

## 相关跳转

- [架构说明](./01_Architecture.md)
- [目录结构](./02_Directory_Structure.md)
- [对外 API](./03_Public_API.md)
- [Misc 信息操作](./03_Public_API.md#misc_info)
- [分区槽管理](./03_Public_API.md#slot_info)
