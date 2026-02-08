# 常见问题与排查

## 目的

本文档汇总 bundle_framework_lite 的常见构建、运行和调试问题，帮助开发者快速定位和解决问题。

## 构建问题

### 问题 1: 编译失败，找不到头文件

**现象**:
```
fatal error: 'bundle_manager.h' file not found
#include "bundle_manager.h"
         ^~~~~~~~~~~~~~~~~~
```

**原因**: 包含路径配置不正确

**解决**:
1. 检查 `BUILD.gn` 中的 `include_dirs` 是否包含正确路径
2. 确保依赖的组件已编译

```gn
# 在 BUILD.gn 中添加
include_dirs = [
  "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
  "${appexecfwk_lite_path}/interfaces/inner_api/bundlemgr_lite",
]
```

### 问题 2: 链接失败，未定义引用

**现象**:
```
undefined reference to `OHOS::BundleManager::Install(char const*, InstallParam const*, void (*)(unsigned char, void const*))'
```

**原因**: 未链接 libbundle.so 或链接顺序错误

**解决**:
```gn
deps = [
  "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
]
```

### 问题 3: 编译时宏未定义

**现象**: 某些 API 不可用（如 `HasSystemCapability`）

**原因**: 未定义 `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` 宏

**解决**:
```gn
defines = [ "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER" ]
```

## 运行问题

### 问题 4: 安装应用失败，返回错误码

**排查步骤**:

1. **查看错误码含义**

**证据**: `interfaces/kits/bundle_lite/appexecfwk_errors.h`

| 错误码 | 值 | 含义 | 排查方向 |
|--------|-----|------|----------|
| 0x41 | 65 | 参数错误 | 检查 hapPath 和 installParam |
| 0x42 | 66 | 文件路径无效 | 检查路径是否存在、是否绝对路径 |
| 0x44 | 68 | 文件不存在 | 检查 HAP 文件是否存在 |
| 0x45 | 69 | 文件损坏 | 检查 HAP 包完整性 |
| 0x46 | 70 | 签名验证失败 | 检查签名是否正确 |
| 0x47 | 71 | 版本降级 | 新版本号必须大于旧版本 |
| 0x48 | 72 | 签名不兼容 | 更新时签名必须一致 |

2. **检查日志**

```bash
# 查看 BMS 日志
hilog | grep BundleMS

# 查看 Daemon 日志
hilog | grep BundleDaemon
```

### 问题 5: BMS 服务未启动

**现象**: 调用 API 返回失败，提示服务不可用

**排查**:

1. 检查服务是否注册
```bash
# 查看 SAMGR 注册的服务
ls /dev/samgr/
```

2. 检查 foundation 进程
```bash
ps | grep foundation
```

3. 查看启动日志
```bash
hilog | grep "BundleMS"
```

**常见原因**:
- 系统启动时 BMS 初始化失败
- Bundle Daemon 未启动
- 内存不足

### 问题 6: 安装后应用无法找到

**排查步骤**:

1. 检查 Bundle 信息是否写入
```bash
# 查看 Bundle 配置文件
ls /data/accounts/account_0/applications/
cat /data/accounts/account_0/applications/{bundleName}.json
```

2. 检查安装目录
```bash
ls /data/app/{bundleName}/
```

3. 检查权限
```bash
ls -l /data/app/{bundleName}/
# 应该显示应用 UID/GID
```

### 问题 7: 卸载失败，提示应用不可卸载

**现象**: 返回 `ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE`

**原因**: 系统应用（`isSystemApp=true`）不允许卸载

**证据**: `services/bundlemgr_lite/src/bundle_installer.cpp:436-438`

```cpp
if (bundleInfo->isSystemApp) {
    return ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE;
}
```

**解决**: 对于系统应用，需要通过系统升级或刷机方式更新

## 调试方法

### 方法 1: 使用 hilog 查看日志

**日志标签**:
```cpp
// BMS 日志
HILOG_MODULE_APP  // 应用模块
HILOG_MODULE_AAFWK // AAFWK 模块
```

**查看日志**:
```bash
# 查看所有 BMS 日志
hilog | grep -E "(BundleMS|BundleInstaller|BundleDaemon)"

# 查看错误日志
hilog | grep ERROR

# 实时查看
hilog -v
```

### 方法 2: 使用 bm 工具

**安装应用**:
```bash
bm install -p /path/to/app.hap
```

**卸载应用**:
```bash
bm uninstall -n com.example.app
```

**查询应用信息**:
```bash
# 查询特定应用
bm dump -n com.example.app

# 查询所有应用
bm dump -l
```

### 方法 3: 检查配置文件

**Bundle 配置**:
```bash
cat /data/accounts/account_0/applications/{bundleName}.json
```

**UID 映射**:
```bash
cat /data/accounts/account_0/applications/uid_gid_map.json
```

### 方法 4: 使用 GDB 调试

**调试 BMS**:
```bash
# 附加到 foundation 进程
gdb-pid $(pidof foundation)

# 设置断点
break BundleInstaller::Install
break ManagerService::ServiceMsgProcess
```

**调试 Daemon**:
```bash
# 启动 Daemon 时调试
gdb bundle_daemon

# 或附加到运行中的 Daemon
gdb-pid $(pidof bundle_daemon)
```

## 性能问题

### 问题 8: 安装大应用时卡顿

**原因**: HAP 解压和文件复制耗时

**优化建议**:
1. 减小 HAP 包大小
2. 优化存储设备性能
3. 使用异步安装接口

### 问题 9: 查询 Bundle 信息慢

**原因**: Bundle 信息存储在内存 map 中，量大时遍历慢

**优化建议**:
1. 使用索引加速查询
2. 缓存常用查询结果
3. 减少不必要的查询

## 兼容性问题

### 问题 10: LiteOS-M 和 LiteOS-A 代码差异

**差异点**:

| 特性 | LiteOS-M | LiteOS-A/Linux |
|------|----------|----------------|
| 库类型 | 静态库 (.a) | 共享库 (.so) |
| IPC 机制 | 简化 IPC | 完整 LiteIPC |
| 文件系统 | 简化 FS | 完整 FS |
| 源码文件 | slite/ 目录 | 根目录 |

**条件编译**:
```cpp
#ifdef _MINI_BMS_
// LiteOS-M 特定代码
#else
// LiteOS-A/Linux 代码
#endif
```

### 问题 11: 版本升级兼容性

**升级限制**:
1. 版本号必须递增（`versionCode`）
2. 签名必须一致
3. 包名不能改变

**证据**: `services/bundlemgr_lite/src/bundle_installer.cpp:701-727`

```cpp
if (oldBundleInfo->versionCode > bundleInfo->versionCode) {
    return ERR_APPEXECFWK_INSTALL_FAILED_VERSION_DOWNGRADE;
}
if (strcmp(oldBundleInfo->appId, bundleInfo->appId) != 0) {
    return ERR_APPEXECFWK_INSTALL_FAILED_INCOMPATIBLE_SIGNATURE;
}
```

## 调试开关

### 启用调试日志

**证据**: `services/bundlemgr_lite/include/bundle_manager_service.h:60-63`

```cpp
#ifdef OHOS_DEBUG
uint8_t SetSignMode(bool enable);
bool IsSignMode() const;
#endif
```

**启用方法**:
```cpp
// 设置调试模式
ManagerService::GetInstance().SetDebugMode(true);

// 设置签名模式（仅 OHOS_DEBUG）
ManagerService::GetInstance().SetSignMode(false);  // 关闭签名验证
```

⚠️ **警告**: 仅用于调试，生产环境必须禁用

## 常见问题速查表

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 安装返回 0x42 | 路径无效 | 使用绝对路径 |
| 安装返回 0x44 | 文件不存在 | 检查文件路径 |
| 安装返回 0x46 | 签名错误 | 检查签名配置 |
| 安装返回 0x47 | 版本降级 | 增加 versionCode |
| 卸载返回 0x63 | 系统应用 | 系统应用不可卸载 |
| 查询返回 0x81 | 参数错误 | 检查参数是否为 NULL |
| 服务无响应 | 服务未启动 | 检查 foundation 进程 |
| 权限不足 | UID 不匹配 | 检查应用权限 |

## 获取帮助

### 日志收集

```bash
# 收集完整日志
hilog > /data/bms_log.txt

# 收集特定时间日志
hilog -t 100  # 最近 100 条
```

### 问题报告模板

```
问题描述:
[清晰描述问题]

复现步骤:
1. [步骤1]
2. [步骤2]
3. [步骤3]

期望结果:
[期望发生什么]

实际结果:
[实际发生什么]

环境信息:
- 设备: [型号]
- 系统版本: [版本]
- 代码版本: [commit]

日志:
[相关日志片段]
```

---

**相关链接**:
- [项目概览](00_Overview.md)
- [对外 API](03_Public_API.md)
- [安全风险分析](07_Security_Analysis.md)
