# 构建与编译产物

## 构建系统

| 属性 | 值 |
|------|------|
| 构建工具 | GN (Generate Ninja) |
| 构建配置 | `BUILD.gn`, `.gni` |
| NDK 配置 | `ndk_lib` |
| 组件类型 | `lite_component`, `lite_library`, `ohos_static_library` |

---

## GN 构建 Targets

### 根构建入口

**文件**：`services/BUILD.gn`

```gn
if (os_level != "mini") {
  lite_component("permission_lite") {
    deps = [ "pms_base:pms_base" ]
    features = [
      "ipc_auth:ipc_auth_target",
      "pms:pms_target",
      "pms_client:pms_client",
    ]
  }
} else {
  lite_component("permission_lite") {
    features = [ "pms:pms_target_static" ]
  }
}
```

**说明**：根据系统类型（mini/small）选择不同的构建配置

**代码证据**：`services/BUILD.gn:17-34`

---

### ipc_auth 模块

**文件**：`services/ipc_auth/BUILD.gn`

| Target | 类型 | 系统 |
|--------|------|------|
| `ipc_auth_target` | shared_library | small |

#### 源码文件

| 文件 | 描述 |
|------|------|
| `src/ipc_auth_impl.c` | IPC 认证实现 |
| `src/ipc_auth_lite.c` | Lite 版本实现 |

#### Include 目录

```
- interfaces/innerkits
- services/ipc_auth/include
- ohos_product_adapter_dir/security/permission_lite/ipc_auth/include
- services/pms_base/include
```

#### 依赖

```
- hilog_lite:frameworks/featured:hilog_shared
- samgr_lite/samgr:samgr
- services/pms_base:pms_base
- third_party/bounds_checking_function:libsec_shared
```

#### 条件依赖（liteos_a）

```
- aafwk_lite/interfaces/kits/want_lite
- appexecfwk_lite/interfaces/kits/bundle_lite
- appexecfwk_lite/frameworks/bundle_lite:bundle
- appexecfwk_lite/services/bundlemgr_lite:appexecfwk_services_lite
```

**宏定义**：
- `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER`
- `OHOS_APPFWK_ENABLE`

**代码证据**：`services/ipc_auth/BUILD.gn:17-56`

---

### pms 模块

**文件**：`services/pms/BUILD.gn`

| Target | 类型 | 系统 |
|--------|------|------|
| `pms_target` | shared_library | small |
| `pms_target_static` | static_library | mini |

#### 源码文件（small）

| 文件 | 描述 |
|------|------|
| `src/perm_operate.c` | 权限操作 |
| `src/pms_impl.c` | PMS 实现 |
| `src/pms_inner.c` | 内部实现 |
| `src/pms_server.c` | 服务端 |
| `src/pms_server_internal.c` | 服务端内部 |

#### 源码文件（mini）

| 文件 | 描述 |
|------|------|
| `src/perm_operate.c` | 权限操作 |
| `src/pms_impl.c` | PMS 实现 |

#### Include 目录

```
- interfaces/kits
- services/pms/include
- services/pms/include/hals
- services/pms_base/include
```

#### 依赖（small）

```
- ipc/interfaces/innerkits/c/ipc:ipc_single
- ohos_product_adapter_dir/security/permission_lite:hal_pms
- samgr_lite/samgr:samgr
- services/pms_base:pms_base
- build/lite/config/component/cJSON:cjson_shared
- third_party/bounds_checking_function:libsec_shared
```

#### 依赖（mini）

```
- permission_lite_pms_hal_path:hal_pms_static
- build/lite/config/component/cJSON:cjson_static
- hilog_lite/interfaces/native/kits/hilog_lite
```

**代码证据**：`services/pms/BUILD.gn:18-67`

---

### pms_client 模块

**文件**：`services/pms_client/BUILD.gn`

| Target | 类型 | 系统 |
|--------|------|------|
| `pms_client` | shared_library | small |

#### 源码文件

| 文件 | 描述 |
|------|------|
| `perm_client.c` | 客户端实现 |

#### Include 目录

```
- interfaces/innerkits
- interfaces/kits
- services/pms/include
- services/pms_base/include
```

#### 依赖

```
- hilog_lite/frameworks/featured:hilog_shared
- ipc/interfaces/innerkits/c/ipc:ipc_single
- samgr_lite/samgr:samgr
- build/lite/config/component/cJSON:cjson_shared
- third_party/bounds_checking_function:libsec_shared
```

**代码证据**：`services/pms_client/BUILD.gn:17-36`

---

### pms_base 模块

**文件**：`services/pms_base/BUILD.gn`

| Target | 类型 |
|--------|------|
| `pms_base` | shared_library |

#### 源码文件

| 文件 | 描述 |
|------|------|
| `src/permission_service.c` | 服务注册 |

#### Include 目录

| 目录 |
|------|
| `services/pms_base/include` |

#### 依赖

```
- hilog_lite/frameworks/featured:hilog_shared
- samgr_lite/samgr:samgr
```

**代码证据**：`services/pms_base/BUILD.gn:17-26`

---

### interfaces/kits 模块（NDK）

**文件**：`interfaces/kits/BUILD.gn`

| Target | 类型 | 产物 |
|--------|------|------|
| `permission_notes` | ndk_lib | libpermission_notes.so |

#### 依赖

```
- services/pms_client:pms_client
```

#### 头文件

```
- interfaces/kits (作为 header_base)
```

**说明**：生成 NDK 库，供外部调用 PMS API

**代码证据**：`interfaces/kits/BUILD.gn:17-21`

---

## 编译产物清单

### Small System 产物

| 产物 | 类型 | 来源 Target | 路径 |
|------|------|-------------|------|
| `libipc_auth.z.so` | 动态库 | ipc_auth_target | out/... |
| `libpms.z.so` | 动态库 | pms_target | out/... |
| `libpms_client.z.so` | 动态库 | pms_client | out/... |
| `libpms_base.z.so` | 动态库 | pms_base | out/... |
| `libpermission_notes.so` | NDK 库 | permission_notes | out/... |

### Mini System 产物

| 产物 | 类型 | 来源 Target | 路径 |
|------|------|-------------|------|
| `libpms.a` | 静态库 | pms_target_static | out/... |

---

## 运行时加载关系

```
┌─────────────────────────────────────────────────────────────┐
│                        运行时加载链                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  App/Service                                                 │
│      │                                                       │
│      ▼ 动态加载                                              │
│  ┌───────────────────────────────────────────────────┐      │
│  │            libpermission_notes.so (NDK)           │      │
│  │                                                   │      │
│  │         └─▶ libpms_client.z.so                    │      │
│  │                       │                            │      │
│  │                       ▼ 动态加载                    │      │
│  │         ┌────────────────────────────────────┐   │      │
│  │         │        libpms.z.so (PMS Server)    │   │      │
│  │         │              │                       │   │      │
│  │         │              ▼ IPC 调用              │   │      │
│  │         │  ┌────────────────────────────┐     │   │      │
│  │         │  │    libipc_auth.z.so        │     │   │      │
│  │         │  │  (IPC 认证服务)             │     │   │      │
│  │         │  └────────────────────────────┘     │   │      │
│  │         └────────────────────────────────────┘   │      │
│  └───────────────────────────────────────────────────┘      │
│                                                              │
│  libpms_base.z.so ───────────────────────────── SAMGR 依赖   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 构建命令

### Full Build

```bash
# 在 OpenHarmony 根目录执行
./build.sh --product-name <product> --build-mode release
```

### Incremental Build

```bash
# 单独构建 permission_lite
hb set permission_lite
hb build -f
```

### GN Check

```bash
# 检查 GN 语法
gn check out/...
```

---

## 依赖组件版本

| 组件 | 版本要求 | 来源 |
|------|----------|------|
| hilog_lite | - | OpenHarmony |
| samgr_lite | - | OpenHarmony |
| ipc | - | OpenHarmony |
| cJSON | - | third_party |
| bounds_checking_function | - | third_party |

---

## 构建配置参数

### 系统类型判断

```gn
if (os_level != "mini") {
  # Small System 构建
} else {
  # Mini System 构建
}
```

### 内核类型判断

```gn
if (ohos_kernel_type == "liteos_a") {
  # 特定于 liteos_a 的配置
}
```

### 构建类型判断

```gn
if (ohos_build_type == "debug") {
  # Debug 构建包含单元测试
}
```

---

## 相关文档

- 项目概览 → `01_Overview.md`
- 架构说明 → `02_Architecture.md`
- API 接口 → `03_APIs.md`
- 安全评审 → `05_Security.md`
- 配置参数 → `appendix/08_Config_Flags.md`
