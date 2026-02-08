# 项目概览 - Bundle Framework Lite

## 目录

- [项目定位](#项目定位)
- [能力边界](#能力边界)
- [运行环境与依赖](#运行环境与依赖)
- [快速开始](#快速开始)

---

## 项目定位

### 一句话定义

**Bundle Framework Lite** 是 OpenHarmony 轻量级系统的包管理框架，负责应用包（HAP）的安装、卸载、管理和信息查询。

### 解决的问题

OpenHarmony 系统需要一个统一的应用包管理机制来：

1. **应用生命周期管理**：控制应用从安装到卸载的完整生命周期
2. **信息集中化**：统一存储和查询应用元数据
3. **安全管控**：通过签名验证和权限管理保护系统安全
4. **资源隔离**：通过独立的 Bundle Daemon 进程执行高权限操作

### 适用场景

- ✅ IoT 设备（资源受限）
- ✅ 智能家居设备
- ✅ 穿戴设备
- ❌ 标准 OpenHarmony 系统（使用 bundle_framework 完整版）

**证据**: `README.md:41-42`
```markdown
- The Bundle Manager Service is running in the foundation process.
- The Bundle Manager Service is registered with sa_manager.
```

---

## 能力边界

### 能做什么

| 功能 | 说明 |
|------|------|
| ✅ 安装 HAP 包 | 支持从本地文件系统安装应用包 |
| ✅ 卸载应用 | 支持卸载非系统应用 |
| ✅ 更新应用 | 支持覆盖安装更新已存在应用 |
| ✅ 查询应用信息 | 查询 BundleInfo、AbilityInfo、ModuleInfo |
| ✅ 监听状态变化 | 注册回调监听安装/卸载事件 |
| ✅ 签名验证 | 验证 HAP 包签名的有效性 |
| ✅ 权限管理 | 存储和管理应用权限 |
| ✅ 系统能力查询 | 查询系统支持的能力（syscap） |

### 不能做什么

| 限制 | 说明 |
|------|------|
| ❌ 应用运行时管理 | 不涉及应用启动、生命周期管理（由 Ability Manager 负责）|
| ❌ 动态权限授权 | 不包含运行时权限弹窗（由 Ability Framework 负责）|
| ❌ 应用沙箱隔离 | 不提供进程级别的沙箱（由系统内核负责）|
| ❌ 多用户管理 | Lite 版不支持多用户场景 |
| ❌ 应用商店功能 | 不包含应用分发、搜索等商店功能 |

---

## 运行环境与依赖

### 系统要求

**适配系统类型**：mini, small（轻量级系统）
**证据**: `bundle.json:22-25`
```json
"adapted_system_type": [
    "mini",
    "small"
]
```

### 资源占用

- **ROM**: 300KB
- **RAM**: >2MB
**证据**: `bundle.json:26-27`

### 内核支持

- **LiteOS-A**: 标准轻量系统
- **LiteOS-M**: 极轻量系统（MCU）

### 依赖的系统服务

| 组件 | 用途 |
|------|------|
| **SAMGR** | 服务框架，用于 BMS 注册和 IPC 通信 |
| **Permission Service** | 权限管理服务，存储和管理应用权限 |
| **Ability Service** | Ability 管理服务，BMS 启动应用时与其交互 |
| **HiLog** | 日志服务，用于调试和问题排查 |
| **Resource Manager** | 资源管理服务，管理应用资源 |

**证据**: `bundle.json:28-36`
```json
"deps": {
    "components": [
    "ability_lite",
    "utils_lite",
    "hilog_lite",
    "permission_lite",
    "samgr_lite",
    "resource_management_lite",
    "appverify"
    ]
}
```

### 依赖的第三方库

| 库 | 用途 |
|----|------|
| **zlib** | HAP 包压缩/解压缩 |
| **cJSON** | JSON 配置文件解析 |
| **jerryscript** | JavaScript 引擎（用于 JSI 绑定）|
| **bounds_checking_function** | 边界检查，防止缓冲区溢出 |

**证据**: `bundle.json:38-42`

---

## 快速开始

### 安装应用（C API 示例）

```c
#include "bundle_manager.h"

// 定义安装参数
InstallParam installParam = {
    .installLocation = INSTALL_LOCATION_INTERNAL_ONLY,
    .keepData = false
};

// 定义安装回调
void InstallCallback(uint8_t resultCode, const void *resultMessage) {
    if (resultCode == ERR_OK) {
        printf("安装成功！\n");
    } else {
        printf("安装失败: %s\n", (const char *)resultMessage);
    }
}

// 执行安装
const char *hapPath = "/sdcard/app.hap";
bool success = Install(hapPath, &installParam, InstallCallback);
```

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:72-125`

### 查询应用信息

```c
#include "bundle_manager.h"

// 查询指定应用的 BundleInfo
BundleInfo bundleInfo;
memset(&bundleInfo, 0, sizeof(BundleInfo));

uint8_t result = GetBundleInfo("com.example.app", 0, &bundleInfo);
if (result == ERR_OK) {
    printf("应用名称: %s\n", bundleInfo.bundleName);
    printf("版本: %s\n", bundleInfo.versionName);
    printf("是否系统应用: %d\n", bundleInfo.isSystemApp);
}
```

### 使用 bm 工具安装

```bash
# 安装 HAP 包
./bin/bm install -p /nfs/xxxx.hap

# 卸载应用
./bin/bm uninstall -n com.example.app

# 查询已安装应用
./bin/bm dump -l
```

**证据**: `README.md:44-48`

### 使用 JS API 查询系统能力

```javascript
import capability from '@system.capability';

// 查询系统是否支持特定能力
const hasCapability = capability.has('SystemCapability.BundleManager.BundleFramework');
console.log(hasCapability); // true or false
```

**证据**: `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp`

---

## 关键架构组件

### BundleKit（客户端 API 层）

**职责**：为应用提供 C/C++ 和 JavaScript API
**位置**: `frameworks/bundle_lite/`
**关键接口**：`Install()`, `Uninstall()`, `GetBundleInfo()`, `QueryAbilityInfo()`

### Bundle Manager Service (BMS)

**职责**：核心服务，管理应用生命周期和信息
**运行进程**：foundation
**服务名**：`bundlems`
**证据**: `interfaces/inner_api/bundlemgr_lite/bundle_service_interface.h:37`

### Bundle Daemon

**职责**：独立高权限进程，执行文件操作
**安全机制**：仅接受来自 BMS（UID=7）的请求
**证据**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon.cpp:108-111`

---

## 相关文档

- [架构与数据流](03_Architecture.md) - 深入理解组件协作
- [对外接口文档](04_Interface.md) - 完整 API 参考
- [攻击面分析](05_AttackSurface.md) - 安全入口点

---

**最后更新**: 2026-02-07
