# 编译产物说明

## 概述

本文档描述 ability_runtime 编译产物的类型、安装路径和运行时加载关系。

## 编译产物类型

| 产物类型 | 文件扩展名 | 说明 |
|---------|------------|------|
| 共享库 | `.so` / `.z.so` | 系统服务、N-API 模块 |
| 静态库 | `.a` | 静态链接库 |
| 可执行文件 | 无扩展名 | 工具程序 |
| 配置文件 | `.cfg` / `.json` | SA 配置、参数配置 |

## 系统服务产物

### AbilityManagerService

**文件路径**：
- `out/{product}/ability_runtime/lib/libability_manager_service.z.so`（32位）
- `out/{product}/ability_runtime/lib/libability_manager_service.so`（64位）

**安装路径**：
- `/system/lib64 Ability_manager_service.z.so`（64位设备）
- `/system/lib/ability_manager_service.z.so`（32位设备）

**SA ID**：`3701`

**依赖加载顺序**：
```
libz.so → libhilog.so → libipc.so → libsamgr.so → libability_manager_service.so
```

### AppManagerService

**文件路径**：
- `out/{product}/ability_runtime/lib/libapp_manager_service.z.so`
- `out/{product}/ability_runtime/lib/libapp_manager_service.so`

**安装路径**：
- `/system/lib64/libapp_manager_service.so`
- `/system/lib/libapp_manager_service.so`

**SA ID**：`1201`

### 其他系统服务

| 服务 | 文件名 | SA ID | 安装路径 |
|------|--------|-------|---------|
| QuickFixManager | `libquick_fix_manager_service.so` | 暂无 | `/system/lib64/` |
| UriPermissionManager | `liburi_permission_manager_service.so` | 暂无 | `/system/lib64/` |

## N-API 产物

### 主 N-API 模块

**文件路径**：
- `out/{product}/ability_runtime/lib/libability_napi.z.so`
- `out/{product}/ability_runtime/lib/libability_napi.so`

**安装路径**：
- `/system/lib64/module/libability_napi.z.so`
- `/system/lib/module/libability_napi.z.so`

**运行时加载**：
- 由 `libace_napi.z.so` 在应用启动时动态加载
- 通过 `NAPI_MODULE_REGISTER` 宏注册

### 子模块 N-API

各功能模块独立编译为 `.so`，最终打包到 `libability_napi.so`：

| 子模块 | 说明 |
|-------|------|
| `ability_manager` | Ability 管理 |
| `application` | 应用管理 |
| `ability_context` | Ability 上下文 |
| `callee` | 被调用者 |
| `caller` | 调用者 |
| `mission_manager` | 任务管理 |
| `wantagent` | Want 代理 |
| `uri_permission` | URI 权限 |

## ETS/ArkTS 产物

**文件路径**：
- `out/{product}/ability_runtime/lib/libability_ani.z.so`
- `out/{product}/ability_runtime/lib/libability_ani.so`

**安装路径**：
- `/system/lib64/module/libability_ani.z.so`
- `/system/lib/module/libability_ani.z.so`

**运行时加载**：
- 由 ArkTS 运行时（`libark_jsruntime.so`）加载

## C API 产物

**文件路径**：
- `out/{product}/ability_runtime/obj/ability_runtime/*.a`（静态库）

**使用方式**：
- 被其他 Native 模块静态链接
- 不直接安装到系统

## 配置文件

### SA 配置文件

**位置**：`services/sa_profile/`

| 文件 | 说明 |
|------|------|
| `abilitymgr_sa.cfg` | AbilityManagerService SA 配置 |
| `appmgr_sa.cfg` | AppManagerService SA 配置 |

**配置格式**：

```json
{
  "services": [
    {
      "name": "AbilityManagerService",
      "path": "/system/lib64/libability_manager_service.so",
      "run-on-create": true
    }
  ]
}
```

### 权限配置文件

**位置**：`services/common/etc/`

## 工具产物

### aa 命令

**源文件位置**：`tools/aa/`

**构建目标**：`tools_aa_source_set`

**产物路径**：
- `out/{product}/ability_runtime/tools/aa/aa`

**安装路径**：
- `/system/bin/aa`

**功能**：
- `aa start`：启动 Ability
- `aa dump`：打印 Ability 信息
- `aa force-stop`：强制停止应用

## 运行时加载关系

### 应用进程加载顺序

```
1. zygote 进程 fork
2. 加载系统共享库
   ├── libhilog.so
   ├── libipc.so
   ├── libsamgr.so
3. 加载应用 HAP
   ├── libability_napi.so (N-API 模块)
   ├── libability_ani.so (ANI 模块)
4. 实例化 Ability
```

### 系统服务启动顺序

```
1. samgr (System Ability Manager) 启动
2. 按需启动 AbilityManagerService
   ├── 加载 libability_manager_service.so
   ├── 注册 SA (ID: 3701)
3. 按需启动 AppManagerService
   ├── 加载 libapp_manager_service.so
   ├── 注册 SA (ID: 1201)
```

## 产物与模块映射

| 模块 | BUILD.gn 目标 | 输出产物 | 安装路径 |
|------|---------------|---------|---------|
| AbilityManagerService | `services/abilitymgr:BUILD.gn` | `libability_manager_service.so` | `/system/lib64/` |
| AppManagerService | `services/appmgr:BUILD.gn` | `libapp_manager_service.so` | `/system/lib64/` |
| N-API | `frameworks/js/napi:BUILD.gn` | `libability_napi.so` | `/system/lib64/module/` |
| ANI | `frameworks/ets/ani:BUILD.gn` | `libability_ani.so` | `/system/lib64/module/` |
| aa 工具 | `tools/aa:BUILD.gn` | `aa` | `/system/bin/` |
| SA 配置 | `services/sa_profile:BUILD.gn` | `*.cfg` | `/system/etc/sa/` |

## 相关文档

- [GN Targets](06_GN_Targets.md)
- [架构说明](03_Architecture.md)
