# 关键编译宏与 Feature Flags

## 目的

说明 appverify 模块的关键编译宏和 Feature Flags。

## 适用范围

- 目标读者：构建系统开发者、模块开发者
- 涵盖内容：编译宏、Feature Flags、配置选项

## 编译宏清单

### 系统级宏

| 宏 | 默认值 | 设置位置 | 说明 | 影响 |
|----|--------|----------|------|------|
| os_level | - | 编译参数 | standard/small/mini | 选择系统版本 |
| is_standard_system | false | BUILD.gn 条件 | 标准系统 | 启用标准系统特性 |
| build_public_version | false | 编译参数 | 公开版本 | 使用公开配置文件 |
| is_emulator | false | 编译参数 | 模拟器 | 启用模拟器模式 |
| ohos_kernel_type | - | 编译参数 | linux/liteos_a/liteos_m | 选择内核类型 |
| ohos_sign_haps_by_server | false | 编译参数 | 服务器签名 | 启用服务器签名模式 |

### 功能宏

| 宏 | 默认值 | 设置位置 | 说明 | 影响 |
|----|--------|----------|------|------|
| HILOG_ENABLE | true | cflags_cc | 启用 Hilog 日志 | 日志输出 |
| OPENSSL_SUPPRESS_DEPRECATED | true | defines | 抑制 OpenSSL 警告 | 编译警告 |
| X86_EMULATOR_MODE | false | defines | x86 模拟器 | 模拟器特定逻辑 |
| SUPPORT_GET_DEVICE_TYPES | false | defines | 获取设备类型 | 非标准系统 |
| STANDARD_SYSTEM | 条件 | is_standard_system | 标准系统 | 特定代码路径 |
| PARSE_PEM_FORMAT_SIGNED_DATA | true | defines | PEM 解析 | Lite 系统 |

### 安全特性宏

| 宏 | 值 | 说明 |
|----|-----|------|
| boundary_sanitize | true | 边界检查 |
| cfi | true | 控制流完整性 |
| cfi_cross_dso | true | 跨 DSO CFI |
| integer_overflow | true | 整数溢出检查 |
| ubsan | true | 未定义行为检查 |
| branch_protector_ret | "pac_ret" | 返回地址保护 |

---

## Feature Flags 详解

### 1. HILOG_ENABLE

**位置**：interfaces/innerkits/appverify/BUILD.gn:62

**定义**：
```gn
cflags_cc = [
  "-DHILOG_ENABLE",
  "-fvisibility=hidden",
]
```

**使用**：
```cpp
// hap_verify_log.h
#ifdef HILOG_ENABLE
    HAPVERIFY_LOG_INFO("message");
#endif
```

**影响**：
- 启用：日志输出到 Hilog
- 禁用：静默编译（无日志）

---

### 2. STANDARD_SYSTEM

**位置**：interfaces/innerkits/appverify/BUILD.gn:76

**定义**：
```gn
if (is_standard_system) {
  defines += [ "STANDARD_SYSTEM" ]
  external_deps += [
    "hilog:libhilog",
    "init:libbegetutil",
  ]
} else {
  external_deps += [
    "ipc:ipc_core",
    "os_account:libaccountkits",
  ]
}
```

**使用**：
```cpp
// hap_verify.cpp:76
#ifdef STANDARD_SYSTEM
    // 标准系统特定逻辑
#else
    // 非标准系统特定逻辑
#endif
```

**影响**：
- 标准系统：使用 hilog、init（直接调用）
- 非标准系统：使用 IPC、os_account

---

### 3. X86_EMULATOR_MODE

**位置**：interfaces/innerkits/appverify/BUILD.gn:100

**定义**：
```gn
if (is_emulator) {
  defines += [ "X86_EMULATOR_MODE" ]
}
```

**使用**：
```cpp
// (推测用于模拟器特定逻辑）
#ifdef X86_EMULATOR_MODE
    // 模拟器特定代码路径
#endif
```

**影响**：
- 启用：模拟器特定优化或兼容逻辑

---

### 4. SUPPORT_GET_DEVICE_TYPES

**位置**：interfaces/innerkits/appverify/BUILD.gn:91

**定义**：
```gn
if (!build_public_version) {
  defines += [ "SUPPORT_GET_DEVICE_TYPES" ]
}
```

**使用**：
```cpp
// device_type_manager.cpp (推测）
#ifdef SUPPORT_GET_DEVICE_TYPES
    // 获取设备类型逻辑
#endif
```

**影响**：
- 启用：支持获取设备类型
- 禁用：不支持设备类型查询

---

### 5. PARSE_PEM_FORMAT_SIGNED_DATA

**位置**：interfaces/innerkits/appverify_lite/BUILD.gn

**定义**：
```gn
defines = [
  "PARSE_PEM_FORMAT_SIGNED_DATA",
]
```

**使用**：
```cpp
// appverify_lite (推测）
#ifdef PARSE_PEM_FORMAT_SIGNED_DATA
    // 解析 PEM 格式签名数据
#endif
```

**影响**：
- 启用：支持 PEM 格式签名
- 禁用：仅支持 DER 格式

---

### 6. OHOS_SIGN_HAPS_BY_SERVER

**位置**：interfaces/innerkits/appverify_lite/BUILD.gn

**定义**：
```gn
if (ohos_sign_haps_by_server) {
  defines += [ "OHOS_SIGN_HAPS_BY_SERVER" ]
}
```

**使用**：
```cpp
// appverify_lite (推测）
#ifdef OHOS_SIGN_HAPS_BY_SERVER
    // 服务器签名模式逻辑
#endif
```

**影响**：
- 启用：应用由服务器签名
- 禁用：应用由客户端签名

---

## 安全编译选项

### Sanitize 选项

**位置**：interfaces/innerkits/appverify/BUILD.gn:25-32

**定义**：
```gn
sanitize = {
  boundary_sanitize = true,    # 边界检查
  cfi = true,               # 控制流完整性
  cfi_cross_dso = true,     # 跨 DSO CFI
  debug = false,
  integer_overflow = true,    # 整数溢出检查
  ubsan = true,             # 未定义行为检查
}
```

**说明**：
- **boundary_sanitize**：检测数组越界、缓冲区溢出
- **cfi**：控制流完整性，防止 ROP/JOP 攻击
- **integer_overflow**：检测整数溢出
- **ubsan**：检测未定义行为（空指针解引用、未初始化内存）

### 分支保护

**位置**：interfaces/innerkits/appverify/BUILD.gn:23

**定义**：
```gn
branch_protector_ret = "pac_ret"
```

**说明**：
- PAC (Pointer Authentication)：保护返回地址
- 防止返回地址篡改攻击

---

## 配置示例

### 标准系统构建

```bash
# hb set
hb set --ccache --product-name rk3568

# 构建
hb build -f --build-target appverify_components
```

**编译宏**：
- `STANDARD_SYSTEM` = true
- `HILOG_ENABLE` = true
- `OPENSSL_SUPPRESS_DEPRECATED` = true

### Lite 系统构建

```bash
# hb set
hb set --ccache --product-name ipcamera_hispark_taurus

# 构建
hb build -f --build-target verify
```

**编译宏**：
- `PARSE_PEM_FORMAT_SIGNED_DATA` = true
- `OHOS_SIGN_HAPS_BY_SERVER` = false

### 模拟器构建

```bash
# 启用模拟器模式
hb build -f --build-target libhapverify --ccache --target-cpu x86_64

# 自动设置 is_emulator = true
```

**编译宏**：
- `X86_EMULATOR_MODE` = true

---

## 条件编译树

### libhapverify 条件编译

```
libhapverify
  ├─ os_level == standard
  │   ├─ is_standard_system == true
  │   │   ├─ STANDARD_SYSTEM
  │   │   ├─ HILOG_ENABLE
  │   │   ├─ hilog:libhilog
  │   │   └─ init:libbegetutil
  │   └─ is_standard_system == false
  │       ├─ SUPPORT_GET_DEVICE_TYPES [!build_public_version]
  │       ├─ ipc:ipc_core
  │       └─ os_account:libaccountkits
  └─ is_emulator == true
      └─ X86_EMULATOR_MODE

  通用：
    ├─ OPENSSL_SUPPRESS_DEPRECATED
    ├─ -fvisibility=hidden
    ├─ boundary_sanitize
    ├─ cfi
    ├─ integer_overflow
    └─ ubsan
```

---

## 相关跳转

- [GN 构建目标](06_GN_Targets.md) - Targets 配置
- [编译产物](07_Build_Artifacts.md) - 产物说明
- [对外 API](04_Public_API.md) - API 使用
