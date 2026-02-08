# 项目概览

## 项目定位

包管理子系统 (Bundle Framework) 是 OpenHarmony 系统的核心子系统之一，负责应用安装包的全生命周期管理。

**证据来源**: `README_zh.md:1-5`

### 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 安装管理 | HAP 文件安装、更新、卸载 | `services/bundlemgr/src/bundle_installer.cpp` |
| 包信息查询 | 查询应用/组件/权限信息 | `services/bundlemgr/src/bundle_mgr_host_impl.cpp` |
| 安全管理 | 签名校验、权限授予与校验 | `services/bundlemgr/src/bundle_permission_mgr.cpp` |
| 包存储 | 包信息持久化存储 (RDB) | `services/bundlemgr/src/rdb/` |
| 设备管理 | 监听设备上下线状态 | `services/bundlemgr/src/` |
| 特权操作 | 文件系统操作 (Installd) | `services/bundlemgr/src/installd/` |

---

## 项目边界

### 输入边界

| 输入类型 | 来源 | 处理方式 |
|----------|------|----------|
| HAP 文件 | 安装命令、应用市场 | 解析、签名校验、安装 |
| 安装命令 | 开发者命令、应用市场 | 参数校验、权限检查 |
| 查询请求 | 应用程序、系统服务 | 权限校验、数据返回 |
| 配置参数 | 系统参数、配置文件 | 加载、缓存、应用 |

### 输出边界

| 输出类型 | 目标 | 说明 |
|----------|------|------|
| 安装结果 | 调用方 | 成功/失败及错误码 |
| 包信息 | 应用程序、系统服务 | BundleInfo/AbilityInfo 等 |
| 事件通知 | HiSysEvent | 安装、签名验证等事件 |
| 文件操作 | 文件系统 | 创建、删除、修改文件 |

### 外部依赖

| 依赖子系统 | 依赖类型 | 说明 |
|------------|----------|------|
| ability_runtime | 强依赖 | Ability 框架集成 |
| samgr | 强依赖 | SA 注册与管理 |
| ipc | 强依赖 | IPC 通信框架 |
| storage_service | 强依赖 | 文件存储服务 |
| access_token | 强依赖 | 权限管理 |
| resource_manager | 强依赖 | 资源管理 |
| appverify | 强依赖 | 应用签名验证 |
| power_manager | 可选 | 电源管理 (free_install) |
| distributed_schedule | 可选 | 分布式能力 |
| window_manager | 可选 | 窗口管理 |

---

## 关键概念

### Bundle 与 HAP

| 概念 | 说明 |
|------|------|
| **Bundle** | 应用安装包，包含代码、资源、配置 |
| **HAP** | Harmony Ability Package，OpenHarmony 应用包格式 |
| **Module** | HAP 文件中包含的一个或多个 Ability |
| **Ability** | 应用组件，代表功能的基本单元 |

### InnerBundleInfo 与 ApplicationInfo

| 概念 | 说明 |
|------|------|
| **InnerBundleInfo** | Bundle 内部表示，包含丰富元数据 |
| **ApplicationInfo** | 应用级信息（包名、版本、权限等） |
| **AbilityInfo** | 组件级信息（类型、启动模式、权限等） |

### 核心数据结构

```cpp
// 关键数据结构 (位于 interfaces/inner_api/)
struct InnerBundleInfo {
    std::string bundleName;          // 包名
    ApplicationInfo appInfo;         // 应用信息
    std::map<std::string, AbilityInfo> abilityInfos;  // Ability 列表
    // ...
};
```

---

## 运行环境

### 进程模型

| 进程 | SA ID | 库 | 权限级别 |
|------|-------|-----|----------|
| foundation | 401 | libbms.z.so | 普通 |
| installs | 511 | libinstalls.z.so | **特权** |

**证据来源**: `sa_profile/401.json:5-10`, `sa_profile/511.json:5-10`

### 系统要求

| 要求 | 说明 |
|------|------|
| OpenHarmony SDK | 构建必需 |
| HB 工具 | 包管理工具 |
| Linux 构建环境 | Ubuntu 18.04+ 推荐 |

### 编译命令

```bash
# 完整构建
./build.sh --product-name <product> --build-target bundle_framework

# 仅构建服务层
./build.sh --product-name <product> --build-target services/bundlemgr:bms_target
```

---

## 功能模块概览

### 接口层 (interfaces/)

| 目录 | 类型 | 说明 |
|------|------|------|
| `kits/js/` | N-API | JS 接口（14 个模块） |
| `kits/native/` | NDK | Native 接口 |
| `kits/ani/` | ANI | ArkUI Native Interface |
| `inner_api/` | Inner | 子系统内部 API |

### 服务层 (services/)

| 模块 | 职责 |
|------|------|
| bundle_mgr_service | 主服务入口 |
| bundle_data_mgr | 数据管理 |
| bundle_installer | 安装逻辑 |
| bundle_permission_mgr | 权限管理 |
| verify/ | 签名校验 |
| ipc/ | IPC 通信 |
| installd/ | Installd 客户端 |
| free_install/ | 自由安装 |
| overlay/ | 叠加包 |
| quick_fix/ | 快速修复 |
| sandbox_app/ | 沙箱应用 |
| clone/ | 应用克隆 |
| bundle_backup/ | 备份恢复 |

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [bundlemanager_bundle_framework](https://gitee.com/openharmony/bundlemanager_bundle_framework) | 本仓库 |
| [bundlemanager_bundle_tool](https://gitee.com/openharmony/bundlemanager_bundle_tool) | 包管理工具 |
| [bundlemanager_distributed_bundle_framework](https://gitee.com/openharmony/bundlemanager_distributed_bundle_framework) | 分布式包管理 |
| [developtools_packing_tool](https://gitee.com/openharmony/developtools_packing_tool) | 打包工具 |

---

## 延伸阅读

- [目录结构](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [N-API 参考](03_N-API_Reference.md)
