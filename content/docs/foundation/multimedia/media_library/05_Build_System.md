# MediaLibrary 构建系统

## GN 构建概述

MediaLibrary 使用 OpenHarmony 的 GN (Generate Ninja) 构建系统。

### 构建入口

| 文件 | 用途 | 说明 |
|-----|------|------|
| `BUILD.gn` | 根构建入口 | 导入 `//build/ohos.gni` |
| `media_library.gni` | 项目配置 | 定义路径、feature flags |

---

## 1. GN 根构建入口

### BUILD.gn

**文件**: `BUILD.gn`

```gn
# Copyright (C) 2021-2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/ohos.gni")
```

---

## 2. 项目配置 (media_library.gni)

### 路径变量定义

**文件**: `media_library.gni:14-52`

```gn
# 基础路径
MEDIALIB_ROOT_PATH = "//foundation/multimedia/media_library"

# 子模块路径
MEDIALIB_COMMON_PATH = "${MEDIALIB_ROOT_PATH}/common"
MEDIALIB_CLIENT_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/client"
MEDIALIB_INNERKITS_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/innerkitsimpl"
MEDIALIB_INNERIMPL_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/innerimpl"
MEDIALIB_INTERFACES_PATH = "${MEDIALIB_ROOT_PATH}/interfaces"
MEDIALIB_JS_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/js"
MEDIALIB_NATIVE_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/native"
MEDIALIB_UTILS_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/utils"

# 服务路径
MEDIALIB_ROOT_SERVICES_PATH = "${MEDIALIB_ROOT_PATH}/services"
MEDIALIB_SERVICES_PATH = "${MEDIALIB_ROOT_PATH}/frameworks/services"
MEDIALIB_NEW_SERVICES_PATH = "${MEDIALIB_ROOT_PATH}/services"

# 云同步路径
MEDIALIB_CLOUD_SYNC_PATH = "${MEDIALIB_SERVICES_PATH}/media_cloud_sync"
MEDIALIB_CLOUD_SYNC_SERVICE_PATH = "${MEDIALIB_ROOT_SERVICES_PATH}/media_cloud_sync_service"
```

### Feature Flags

**文件**: `media_library.gni:54-82`

```gn
declare_args() {
  # 基础配置
  media_library_link_opt = false
  
  # 功能开关
  media_library_feature_mtp = true           # MTP 功能
  media_library_feature_back_up = true       # 备份功能
  media_library_feature_cloud_enhancement = true  # 云增强
  media_library_feature_cloud_download = true   # 云下载
  media_library_cloud_sync_enable = true     # 云同步
  media_library_facard_enable = true        # FACard
  media_library_analysis_data_enable = true  # 分析数据
  media_library_feature_custom_restore = true # 自定义恢复
  
  # 可选功能
  media_library_feature_device_manager = false
  media_library_feature_hicollie_enable = false
}
```

---

## 3. bundle.json 配置

### 组件定义

**文件**: `bundle.json`

```json
{
  "name": "@ohos/media_library",
  "description": "provides a set of easy-to-use APIs for getting media file metadata information",
  "version": "4.0",
  "component": {
    "name": "media_library",
    "subsystem": "multimedia",
    "features": [
      "media_library_link_opt",
      "media_library_feature_mtp",
      "media_library_feature_back_up",
      "media_library_feature_cloud_enhancement",
      "media_library_feature_cloud_download",
      "media_library_cloud_sync_enable",
      "media_library_facard_enable",
      "media_library_analysis_data_enable",
      "media_library_feature_custom_restore"
    ],
    "adapted_system_type": [ "small", "standard" ],
    "rom": "10444KB",
    "ram": "35093KB"
  }
}
```

### 系统能力依赖

```json
"syscap": [
  "SystemCapability.FileManagement.UserFileManager.Core",
  "SystemCapability.FileManagement.UserFileManager.DistributedCore",
  "SystemCapability.FileManagement.PhotoAccessHelper.Core"
]
```

---

## 4. 主要 Targets

### interfaces/kits/js/BUILD.gn

**文件**: `interfaces/kits/js/BUILD.gn`

```gn
# JS N-API 库
ohos_shared_library("medialibrary_napi") {
  sources = [
    "src/native_module_ohos_medialibrary.cpp",
    "src/media_library_napi.cpp",
    "src/file_asset_napi.cpp",
    "src/album_napi.cpp",
    "src/fetch_file_result_napi.cpp",
    # ... 更多源文件
  ]
  
  include_dirs = [
    "interfaces/kits/js/include",
    "interfaces/kits/js/include/napi",
    "frameworks/js/napi",
    # ... 更多头文件路径
  ]
  
  deps = [
    "//foundation/ability/ability_runtime/interfaces/kits/ability/native:ability_native",
    "//foundation/multimedia/media_framework/interfaces/kits/napi:media_napi",
    "//foundation/multimedia/media_library/interfaces/inner_api/media_library_helper:media_library_helper",
  ]
  
  cflags = [
    "-DOHOS_MEDIA_LIBRARY",
  ]
  
  output_name = "libmedialibrary_napi.z.so"
}
```

### interfaces/kits/c/BUILD.gn

**文件**: `interfaces/kits/c/BUILD.gn`

```gn
# C API 库
ohos_shared_library("media_library_capi") {
  sources = [
    "src/media_asset_capi.cpp",
    "src/media_asset_manager_capi.cpp",
    "src/media_access_helper_capi.cpp",
  ]
  
  include_dirs = [
    "interfaces/kits/c",
  ]
  
  deps = [
    "//foundation/multimedia/media_library/interfaces/inner_api/media_library_helper:media_library_helper",
  ]
  
  output_name = "libmedia_library_capi.z.so"
}
```

---

## 5. 编译产物清单

### 产物类型

| 产物类型 | 文件名 | 路径 | 说明 |
|---------|-------|------|------|
| **N-API 库** | `libmedialibrary_napi.z.so` | `out/` | JS 接口实现 |
| **C API 库** | `libmedia_library_capi.z.so` | `out/` | C 语言接口 |
| **Native 库** | `libmedia_library.z.so` | `out/` | 内部 Native 实现 |
| **SA 服务** | `libmedia_library_service.z.so` | `out/` | System Ability |

### 产物安装路径

| 产物 | 安装路径 | 说明 |
|-----|---------|------|
| N-API 库 | `/system/lib64/module/multimedia/` | 系统库目录 |
| C API 库 | `/system/lib64/module/multimedia/` | 系统库目录 |
| 资源文件 | `/system/profile/` | SA 配置文件 |

---

## 6. 关键依赖

### 系统依赖

| 依赖组件 | 用途 | 配置来源 |
|---------|------|---------|
| `napi` | Native API 框架 | `//build/ohos.gni` |
| `ipc` | IPC 通信 | `media_library.gni` |
| `safwk` | System Ability Framework | `media_library.gni` |
| `relational_store` | 关系型数据库 | `media_library.gni` |
| `data_share` | 数据共享 | `media_library.gni` |
| `access_token` | 权限管理 | `media_library.gni` |

### 内部依赖

| 源 | 目标 | 类型 |
|-----|------|------|
| `kits/js/` | `inner_api/` | 接口依赖 |
| `innerkitsimpl/` | `services/` | IPC 依赖 |
| `services/` | `common/` | 工具依赖 |

---

## 7. 构建配置说明

### Feature 开关使用

```gn
# 条件编译示例
if (media_library_cloud_sync_enable) {
  deps += [ "//foundation/multimedia/media_library/services/media_cloud_sync_service" ]
}

if (media_library_feature_mtp) {
  sources += [ "src/mtp_support.cpp" ]
}
```

### 条件链接

```gn
# 可选链接
if (media_library_link_opt) {
  libs += [ "libmedia_library_opt.z.so" ]
} else {
  libs += [ "libmedia_library.z.so" ]
}
```

---

## 8. 构建命令

### 全量构建

```bash
# 编译整个 media_library 模块
hb build -p multimedia/media_library
```

### 增量构建

```bash
# 增量编译
hb build -p multimedia/media_library --fast
```

### 单模块构建

```bash
# 编译单个 target
gn gen out/default --check
ninja -C out/default medialibrary_napi
```

---

## 9. 构建产物验证

### 产物检查清单

| 检查项 | 验证方法 | 预期结果 |
|-------|---------|---------|
| N-API 库 | `file libmedialibrary_napi.z.so` | ELF 64-bit |
| 符号导出 | `nm -D libmedialibrary_napi.z.so | grep " T "` | N-API 函数 |
| 依赖检查 | `ldd libmedialibrary_napi.z.so` | 无未定义符号 |
| 权限检查 | `ls -la lib*.so` | 644 权限 |

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构 |
| [02_Architecture](02_Architecture.md) | 架构设计 |
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 详细配置项 |
