# OpenHarmony Updater 问题排查指南

## 目的

本文档提供 Updater 子系统常见问题的定位方法和调试技巧。

## 适用范围

- 现场工程师
- 调试工程师
- 子系统开发者

## 常见问题

### 1. 无法进入 Updater 模式

#### 症状
执行 `reboot updater` 后设备进入正常系统而非 Updater 模式。

#### 检查清单

1. **检查 Misc 分区写入**

```bash
# 读取 Misc 分区查看命令
dd if=/dev/block/misc of=/tmp/misc bs=1k count=2
hexdump -C /tmp/misc | head
```

期望看到: `boot_updater` 字符串

2. **检查 Bootloader 支持**

```bash
# 查看 Bootloader 日志
dmesg | grep -i misc
dmesg | grep -i updater
```

3. **检查分区表**

```bash
# 确认存在 misc 和 updater 分区
cat /proc/partitions | grep -E "misc|updater"
```

#### 代码定位

```cpp
// services/main.cpp:47-60
int main(int argc, char **argv)
{
    // 读取 Misc 分区决定进入模式
    Updater::UpdateMessage msg;
    Updater::ReadUpdaterMiscMsg(msg);
    
    if (IsFlashd(msg)) {
        // 进入 Flashd 模式
    } else if (IsUpdater(msg)) {
        // 进入 Updater 模式
    }
}
```

### 2. 升级包验证失败

#### 症状
升级过程中提示 "verify fail" 或 `UPDATE_CORRUPT`。

#### 检查清单

1. **检查升级包完整性**

```bash
# 计算本地哈希
sha256sum /data/update/ota.zip

# 与预期哈希对比
```

2. **检查签名证书**

```bash
# 确认证书存在
ls -la /system/etc/*.pem

# 检查证书权限
ls -laZ /system/etc/*.pem
```

3. **查看详细日志**

```bash
# Updater 模式日志
hilog | grep -i verify

# 内核日志
dmesg | grep -i signature
```

#### 代码定位

```cpp
// services/updater_main.cpp:339-342
int32_t verifyret = OtaUpdatePreCheck(manager, STREAM_ZIP_PATH);
if (verifyret != UPDATE_SUCCESS) {
    UpdaterVerifyFailEntry((verifyret == PKG_INVALID_DIGEST) && 
                           (upParams.updateMode == HOTA_UPDATE));
}

// services/package/pkg_verify/pkg_verify_util.cpp:90-93
int32_t ret = Pkcs7verify(signData, hash);
if (ret != PKG_SUCCESS) {
    PKG_LOGE("pkcs7 verify fail!");
    UPDATER_LAST_WORD(ret, "pkcs7 verify fail!");
}
```

#### 错误码说明

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `PKG_INVALID_SIGNATURE` | - | 签名无效 |
| `PKG_INVALID_DIGEST` | - | 摘要验证失败 |
| `PKG_INVALID_PARAM` | - | 参数错误 |

### 3. 分区挂载失败

#### 症状
升级过程中提示 "mount fail"。

#### 检查清单

1. **检查 fstab 配置**

```bash
# 查看 fstab 配置
cat /etc/fstab.updater

# 检查分区存在
ls -la /dev/block/by-name/
```

2. **检查文件系统**

```bash
# 检查文件系统状态
fsck.ext4 -n /dev/block/system_a
```

3. **查看挂载日志**

```bash
hilog | grep -i mount
dmesg | grep -i mount
```

#### 代码定位

```cpp
// services/fs_manager/mount.cpp
int MountPartition(const std::string &partition, const std::string &mountPoint, 
                   const std::string &fsType, uint64_t mountFlags) {
    int ret = mount(partition.c_str(), mountPoint.c_str(), fsType.c_str(), 
                    mountFlags, nullptr);
    if (ret != 0) {
        LOG(ERROR) << "Mount " << partition << " to " << mountPoint << " failed";
    }
    return ret;
}
```

### 4. 空间不足

#### 症状
升级过程中提示 `UPDATE_SPACE_NOTENOUGH`。

#### 检查清单

```bash
# 检查各分区空间
df -h

# 检查特定目录
ls -la /data/update/
du -sh /data/update/

# 检查缓存
ls -la /cache/
```

#### 代码定位

```cpp
// services/include/updater/updater.h:33
enum UpdaterStatus {
    // ...
    UPDATE_SPACE_NOTENOUGH,  // 空间不足错误码
    // ...
};

// services/updater_main.cpp
UpdaterStatus IsSpaceCapacitySufficient(UpdaterParams &upParams) {
    // 计算所需空间
    // 检查可用空间
}
```

### 5. UI 不显示

#### 症状
进入 Updater 模式后屏幕黑屏或无进度显示。

#### 检查清单

1. **检查 UI 是否编译**

```bash
# 检查 UI 库存在
ls -la /updater/libui*

# 检查资源文件
ls -la /updater/resources/
```

2. **检查驱动加载**

```bash
# 查看 DRM/Framebuffer 设备
ls -la /dev/dri/
ls -la /dev/fb*

# 查看输入设备
ls -la /dev/input/
```

3. **检查日志**

```bash
hilog | grep -i ui
hilog | grep -i graphic
```

#### 代码定位

```cpp
// services/ui/updater_ui_env.cpp
bool InitUI() {
    // 初始化显示驱动
    // 加载资源
    // 创建 UI 组件
}

// services/ui/driver/drm_driver.cpp
class DrmDriver {
    // DRM 显示实现
};
```

### 6. Flashd 模式问题

#### 症状
无法进入 Flashd 模式或刷机失败。

#### 检查清单

```bash
# 写入 Flashd 命令
write_updater boot_flash
reboot updater

# 检查 HDC 连接
hdc list targets

# 查看 Flashd 日志
hilog | grep -i flashd
```

#### 代码定位

```cpp
// services/main.cpp
if (IsFlashd(msg)) {
    return StartFlashd(argc, argv);
}

// services/flashd/daemon/daemon_updater.cpp
int StartFlashd(int argc, char **argv) {
    // 启动 Flashd 服务
}
```

## 调试技巧

### 1. 日志收集

```bash
# 收集所有日志
hilog > /data/updater_logs.txt
dmesg > /data/kernel_logs.txt

# 收集崩溃日志
cat /data/faultlog/*

# 收集 Misc 信息
dd if=/dev/block/misc of=/data/misc_dump bs=1k count=2
```

### 2. 手动触发升级

```cpp
// 在代码中手动触发（调试用）
#include "updaterkits/updaterkits.h"

void TriggerUpdate() {
    std::vector<std::string> packages = {"/data/update/test.zip"};
    int ret = RebootAndInstallUpgradePackage("/dev/block/misc", 
                                              packages, 
                                              UPGRADE_TYPE_OTA);
    LOGI("Trigger update result: %d", ret);
}
```

### 3. 脚本调试

```bash
# 查看升级脚本内容
unzip -p /data/update/ota.zip updater-script

# 手动执行脚本（调试用）
# 注意: 仅在测试环境使用
```

### 4. 使用 write_updater 工具

```bash
# 查看可用命令
write_updater --help

# 写入升级命令
write_updater bin /data/update/ota.zip

# 写入恢复出厂命令
write_updater factory_reset

# 写入 SD 卡升级命令
write_updater sdcard_update
```

代码位置: `utils/write_updater.cpp:141-229`

## 定位路径速查表

| 问题类型 | 关键文件 | 日志关键词 |
|---------|---------|-----------|
| 升级包验证 | `services/package/pkg_verify/` | "verify", "signature", "digest" |
| 分区操作 | `services/fs_manager/` | "mount", "format", "partition" |
| 脚本执行 | `services/script/` | "script", "instruction" |
| 补丁应用 | `services/applypatch/` | "patch", "apply", "transfer" |
| UI 显示 | `services/ui/` | "ui", "graphic", "display" |
| 包解析 | `services/package/pkg_manager/` | "package", "parse", "load" |
| Misc 操作 | `interfaces/kits/misc_info/` | "misc", "message" |
| 启动流程 | `services/main.cpp`, `services/updater_main.cpp` | "boot", "mode", "start" |

## 相关跳转

- [项目概览](./00_Overview.md)
- [架构说明](./01_Architecture.md)
- [内部接口](./04_Inner_API.md)
