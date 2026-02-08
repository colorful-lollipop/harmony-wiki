# User File Service - 编译产物

## 概述

本文档列出 user_file_service 的所有编译产物，包括库文件、HAP 包和配置文件。

---

## 产物清单

### 共享库 (.so)

| 产物路径 | 目标 Target | 大小估算 | 说明 |
|----------|------------|----------|------|
| `/system/lib64/libfile_access_service.z.so` | `services:file_access_service` | ~2MB | 文件访问系统服务 |
| `/system/lib/module/file/libfileaccess.z.so` | `frameworks:fileaccess` | ~500KB | 文件访问 N-API |
| `/system/lib/module/file/libpicker.z.so` | `picker:picker` | ~1MB | 文件选择器 |
| `/system/lib/module/file/libcj_picker_ffi.z.so` | `picker:cj_picker_ffi` | ~200KB | CJ FFI |
| `/system/lib64/libcloud_disk_service.z.so` | `services:cloud_disk_service` | ~500KB | 云盘服务 |
| `/system/lib64/libnotify_work_service.z.so` | `services:notify_event` | ~100KB | 通知服务 |

### HAP 包

| 产物路径 | 目标 Target | 大小估算 | 说明 |
|----------|------------|----------|------|
| `/app/com.ohos.UserFile.ExternalFileManager/external_file_manager.hap` | `external_file_manager_hap` | ~5MB | 外部文件管理器 |

### 配置文件

| 产物路径 | 类型 | 说明 |
|----------|------|------|
| `/system/etc/init/file_access_service.cfg` | 配置 | SA 启动配置 |
| `/system/etc/param/file_access_service.para` | 参数 | SA 参数 |
| `/system/etc/param/file_access_service.para.dac` | DAC | DAC 参数 |
| `/system/profile/5010.json` | Profile | SA Profile |

---

## 运行时加载关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用进程                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐│
│  │ @ohos.file.      │  │ @ohos.file.     │  │ 其他应用         ││
│  │ fileAccess       │  │ picker          │  │                 ││
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘│
│           │                     │                       │          │
│           ▼                     ▼                       ▼          │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │ libfileaccess.   │  │ libpicker.       │                     │
│  │ z.so             │  │ z.so             │                     │
│  │ (dlopen)         │  │ (dlopen)         │                     │
│  └────────┬─────────┘  └────────┬─────────┘                     │
│           │                     │                                │
│           └──────────┬──────────┘                                │
│                      │                                           │
│                      ▼                                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                   system能力框架 (samgr)                      ││
│  │                      │                                       ││
│                      ▼                                         ││
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              FileAccessService (独立进程)                    ││
│  │  ┌────────────────────┐  ┌────────────────────┐              ││
│  │  │ libfile_access_    │  │ libcloud_disk_     │              ││
│  │  │ service.z.so       │  │ service.z.so       │              ││
│  │  │ (加载时机: SA启动)  │  │ (条件加载)          │              ││
│  │  └────────────────────┘  └────────────────────┘              ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 安装路径详情

### 系统库路径

| 路径 | 权限 | 说明 |
|------|------|------|
| `/system/lib64/` | ro | 系统 native 库 |
| `/system/lib/module/file/` | ro | N-API 模块库 |

### HAP 安装路径

| 路径 | 权限 | 说明 |
|------|------|------|
| `/app/com.ohos.UserFile.ExternalFileManager/` | rw | 外部文件管理器目录 |

### 配置路径

| 路径 | 说明 |
|------|------|
| `/system/etc/init/` | init 配置 |
| `/system/etc/param/` | 参数配置 |
| `/system/profile/` | SA Profile |

---

## SA 注册信息

**SA ID**: 5010

**SA Profile** (`services/5010.json`):

```json
{
    "name": "FileAccessService",
    "lib-path": "libfile_access_service.z.so",
    "run-on-create": false,
    "allow-stop": true,
    "disallowed-shared-groups": []
}
```

**配置项** (`services/file_access_service.cfg`):

```json
{
    "services": [
        {
            "name": "FileAccessService",
            "path": "/system/etc/init/file_access_service.cfg"
        }
    ]
}
```

---

## 版本兼容性

| 产物版本 | 系统版本 | 兼容性 |
|----------|----------|--------|
| 3.1 | OpenHarmony 3.1+ | ✅ |
| 3.2 | OpenHarmony 3.2+ | ✅ |
| 4.0 | OpenHarmony 4.0+ | ✅ |

---

## 产物验证

### 检查产物是否存在

```bash
# 检查 N-API 库
ls -la /system/lib/module/file/libfileaccess.z.so

# 检查 SA 库
ls -la /system/lib64/libfile_access_service.z.so

# 检查 HAP
ls -la /app/com.ohos.UserFile.ExternalFileManager/external_file_manager.hap
```

### 检查依赖

```bash
# 检查库依赖
ldd /system/lib/module/file/libfileaccess.z.so

# 预期依赖：
# - libnapi.z.so (N-API 框架)
# - libipc_core.z.so (IPC)
# - libhilog.z.so (日志)
```

---

## 调试符号

### 符号表位置

| 产物 | 符号文件 | 位置 |
|------|----------|------|
| libfile_access_service.z.so | libfile_access_service.z.so.dbg | /system/lib64/ |
| libfileaccess.z.so | libfileaccess.z.so.dbg | /system/lib/module/file/ |

### 使用符号调试

```bash
# 使用 addr2line
addr2line -e /system/lib64/libfile_access_service.z.so.dbg 0x12345

# 使用 ndk-stack
ndk-stack -sym /system/lib64/libfile_access_service.z.so.dbg -dump log.txt
```
