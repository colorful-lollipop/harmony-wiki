# 配置标志

## 目的

本文档列述 ASSET 服务的关键构建配置标志、特性开关和宏定义。

## 适用范围

- 涵盖内容：config.gni 变量、feature flags、编译宏
- 目标读者：构建工程师、集成工程师

---

## config.gni 变量

### 全局配置变量

**文件**：`config.gni:14-21`

| 变量名 | 默认值 | 类型 | 用途 | 证据 |
|---------|---------|------|------|
| `enable_local_test` | false | boolean | 启用本地测试（添加 `--cfg feature="AssetTest"`） | config.gni:15 |
| `asset_access_control_enabled` | false | boolean | 启用访问控制特性 | config.gni:16 |
| `asset_split_hap_list` | "{}" | string | HAP 拆分配置 JSON | config.gni:17 |
| `asset_extend_timeout` | false | boolean | 启用扩展超时特性 | config.gni:18 |
| `asset_ce_upgrade_list` | "" | string | CE 升级列表配置 | config.gni:19 |

### 变量详解

#### enable_local_test

**用途**：启用本地测试功能，不影响生产构建

**影响**：
- 添加 `--cfg feature="AssetTest"` 到 Rust 编译参数
- 启用测试用的额外代码路径

**使用方法**：
```bash
./build.sh --product-name rk3568 --build-target asset_bin_test \
    --gn-args "enable_local_test=true"
```

**证据**：`services/core_service/BUILD.gn:38-40`

#### asset_access_control_enabled

**用途**：启用访问控制特性，允许更精细的权限管理

**影响**：
- 启用跨用户访问控制
- 启用组访问验证
- 可能影响性能（额外的权限检查）

**默认值**：`false`（生产环境通常禁用）

**证据**：`bundle.json:22-27`（features 列表）

#### asset_split_hap_list

**用途**：配置需要拆分的 HAP（Harmony Ability Package）列表

**格式**：JSON 字符串

**示例**：
```json
{
  "com.example.app1": {"userId": "100"},
  "com.example.app2": {"userId": "101"}
}
```

**使用位置**：
- `services/os_dependency/BUILD.gn:47-49`（`ASSET_UPGRADE_HAP_CONFIG`）

**证据**：`services/os_dependency/BUILD.gn:47-48`

#### asset_extend_timeout

**用途**：启用扩展超时特性，允许更长的操作超时时间

**影响**：
- 定义 `ASSET_ENABLE_EXTEND_LOAD_TIMEOUT` 宏
- 可能增加操作的等待时间

**证据**：`interfaces/inner_kits/rs/BUILD.gn:37-38`

#### asset_ce_upgrade_list

**用途**：配置需要 CE（Credential Encrypted）升级的数据库列表

**格式**：字符串列表（逗号分隔）

**示例**：
```
"100,101,102"
```

**使用位置**：
- `services/os_dependency/BUILD.gn:49-50`（`ASSET_CE_UPGRADE_CONFIG`）

**证据**：`services/os_dependency/BUILD.gn:49-50`

---

## 条件编译宏

### support_jsapi

**定义**：`BUILD.gn:26-28`

```gn
group("asset_component") {
  deps = [
    # ... 其他依赖

    if (support_jsapi) {
      deps += [ "frameworks/js/napi:asset_napi" ]
    }
  }
}
```

**用途**：控制是否编译 N-API 模块（JS API 绑定）

**影响**：
- `true`：编译 `libasset_napi.so`，支持 JS/TS 应用
- `false`：不编译 N-API 模块，仅支持 C NDK

**证据**：`BUILD.gn:26-28`

---

## 服务定义

### SA 配置 (8100.json)

**文件**：`sa_profile/8100.json:2-36`

#### SA 基本配置

| 字段 | 值 | 说明 |
|------|-----|------|
| `process` | "asset_service" | 服务进程名 |
| `systemability` | 8100 | SA ID |
| `libpath` | "libasset_service.dylib.so" | 服务库路径 |
| `run-on-create` | false | 不自动启动，按需启动 |
| `distributed` | false | 不支持分布式 |
| `dump_level` | 1 | Dump 详细级别 |
| `recycle-strategy` | "low-memory" | 内存回收策略 |

**证据**：`sa_profile/8100.json:3-11`

#### On-Demand 启动触发器

| 事件 | 类型 | 说明 |
|------|------|------|
| `usual.event.PACKAGE_REMOVED` | 常用事件 | 应用卸载时启动 |
| `usual.event.SANDBOX_PACKAGE_REMOVED` | 常用事件 | 沙盒应用卸载 |
| `usual.event.USER_REMOVED` | 常用事件 | 用户删除时启动 |
| `usual.event.CHARGING` | 常用事件 | 充电时启动 |
| `usual.event.USER_UNLOCKED` | 常用事件 | 用户解锁时启动 |
| `usual.event.RESTORE_START` | 常用事件 | 恢复开始时启动 |
| `USER_PIN_CREATED_EVENT` | 常用事件 | PIN 创建时启动 |
| `usual.event.BOOT_COMPLETED` | 常用事件 | 系统启动完成时启动 |
| `usual.event.CONNECTIVITY_CHANGE` | 常用事件 | 网络变化时启动 |
| `usual.event.DATA_SHARE_READY` | 常用事件 | 数据共享就绪时启动 |
| `loopevent` | 定时事件 | 每 129600 秒（36 小时）启动 |

**证据**：`sa_profile/8100.json:13-32`

#### 扩展支持

| 扩展 | 说明 |
|------|------|
| `backup` | 数据备份扩展 |
| `restore` | 数据恢复扩展 |
| `RssSaExtension` | 资源调度系统扩展 |

**证据**：`sa_profile/8100.json:33`

---

## 服务启动配置

### asset_service.cfg

**文件**：`etc/init/asset_service.cfg`

#### 进程配置

| 字段 | 值 | 说明 |
|------|-----|------|
| `name` | "asset_service" | 服务名称 |
| `uid` | 3057 | 服务 UID |
| `gid` | 3057 | 服务 GID |
| `caps` | [] | 能力列表 |
| `seclabel` | "u:r_asset_service:s0" | SELinux 上下文 |
| `ondemand` | true | 按需启动 |

**证据**：`etc/init/asset_service.cfg:1-13`

#### 权限列表

```json
{
  "permissions": [
    "ohos.permission.INTERACT_ACROSS_LOCAL_ACCOUNTS"
  ]
}
```

**权限说明**：
- `INTERACT_ACROSS_LOCAL_ACCOUNTS`：允许服务进行跨用户操作

**证据**：`etc/init/asset_service.cfg:14-17`

---

## HUKS 配置

### 密钥参数

**位置**：`services/crypto_manager/src/huks_wrapper.c:1-61`

#### 密钥结构

```c
struct KeyId {
    int32_t userId;              // 用户 ID（多用户支持）
    struct HksBlob alias;        // 密钥别名/标识符
    enum Accessibility accessibility;  // 可访问性级别
};
```

#### 可访问性级别映射

| Accessibility | HKS 存储级别 | 说明 |
|------------|-------------|------|
| `DEVICE_POWERED_ON` (0) | `HKS_AUTH_STORAGE_LEVEL_DE` | 设备开机即可访问 |
| `DEVICE_FIRST_UNLOCKED` (1) | `HKS_AUTH_STORAGE_LEVEL_CE` | 首次解锁后可访问 |
| `DEVICE_UNLOCKED` (2) | `HKS_AUTH_STORAGE_LEVEL_ECE` | 设备解锁时才可访问 |

**证据**：`services/crypto_manager/src/huks_wrapper.c:11-21`

---

## 数据库配置

### SQLCipher 配置

**位置**：`services/db_operator/src/sqlite3_wrapper.c:5-13`

#### 加密参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `DEFAULT_CIPHER` | "aes-256-gcm" | 加密算法 |
| `DEFAULT_H_MAC_ALGO` | "SHA1" | HMAC 算法 |
| `DEFAULT_KDF_ALGO` | "KDF_SHA1" | 密钥派生函数 |
| `DEFAULT_ITER` | 10000 | KDF 迭代次数 |
| `DEFAULT_PAGE_SIZE` | 1024 | 数据库页面大小 |

**证据**：`services/db_operator/src/sqlite3_wrapper.c:5-13`

#### 数据库路径

| 用户类型 | 路径模板 |
|---------|---------|
| DE（Device Encrypted） | `/data/service/el2/user_{user_id}/` |
| CE（Credential Encrypted） | `/data/service/el2/user_{user_id}/` |

**证据**：`frameworks/os_dependency/file/src/de_operator.rs:15-35`

---

## 编译器/链接器标志

### 安全标志

**应用目标**：`asset_ndk`, `asset_sdk`, `asset_napi`, `asset_openssl_wrapper`

| 标志 | 说明 | 证据 |
|------|------|------|
| `branch_protector_ret = "pac_ret"` | 返回地址保护（PAC） |
| `sanitize = { "integer_overflow": true }` | 整数溢出检查 |
| `sanitize = { "cfi": true }` | 控制流完整性 |
| `sanitize = { "cfi_cross_dso": true }` | 跨 DSO CFI |
| `sanitize = { "boundary_sanitize": true }` | 边界检查 |
| `sanitize = { "ubsan": true }` | 未定义行为检查 |
| `sanitize = { "debug": false }` | 禁用调试符号 |

**证据**：`interfaces/kits/c/BUILD.gn:43-50`

### C/C++ 编译标志

**应用目标**：所有 C/C++ 模块

| 标志 | 说明 |
|------|------|
| `-Wall` | 启用所有警告 |
| `-Werror` | 将警告视为错误 |
| `-fPIC` | 生成位置无关代码 |
| `-std=c++17` | C++17 标准（如适用） |

**证据**：`frameworks/js/napi/BUILD.gn:51-55`

### Rust 编译标志

**应用目标**：所有 Rust 模块

| 条件 | 标志 | 说明 |
|------|------|------|
| `enable_local_test = true` | `--cfg feature="AssetTest"` | 启用测试代码 |
| `asset_extend_timeout = true` | `--cfg feature="extend_timeout"` | 启用扩展超时 |
| 默认 | `--edition 2021` | Rust 2021 版本 |
| 默认 | `--crate-type dylib` | 动态库类型 |

**证据**：`services/core_service/BUILD.gn:37-41`

---

## 日志配置

### HiSysEvent 配置

**文件**：`hisysevent.yaml:14-31`

#### 事件定义

| 事件 | 类型 | 级别 | 描述 |
|------|------|------|
| `SECRET_STORE_INFO_COLLECTION` | STATISTIC | MINOR | 统计事件（正常操作） |
| `SECRET_STORE_OPERATION_FAILED` | FAULT | CRITICAL | 故障事件（失败操作） |

#### 事件参数

**SECRET_STORE_INFO_COLLECTION**：
- `FUNCTION`：操作名称
- `USER_ID`：用户 ID
- `CALLER`：调用者信息
- `RUN_TIME`：运行时间（毫秒）
- `EXTRA`：额外数据

**SECRET_STORE_OPERATION_FAILED**：
- `FUNCTION`：操作名称
- `USER_ID`：用户 ID
- `CALLER`：调用者信息
- `ERROR_CODE`：错误码
- `EXTRA`：额外数据

**证据**：`hisysevent.yaml:16-30`

### Hilog 配置

**日志级别使用**：
- `LOGI`：常规信息
- `LOGE`：错误信息
- `LOGF`：致命错误
- `LOGD`：调试信息

**日志宏使用**：
```c
#define LOGI(fmt, ...) HILOG_ERROR(LOG_CORE, fmt, ##__VA_ARGS__)
#define LOGE(fmt, ...) HILOG_ERROR(LOG_CORE, fmt, ##__VA_ARGS__)
```

**证据**：`frameworks/os_dependency/log/src/lib.rs`

---

## 运行时配置

### 内存配置

**资源限制**：
- **ROM**：5120KB（静态存储）
- **RAM**：4828KB（运行时内存）

**证据**：`bundle.json:34-35`

### 并发配置

**线程模型**：
- **SA 主线程**：生命周期管理
- **异步运行时**：`ylong_runtime` 异步任务
- **SQLite**：事务线程

**证据**：`services/core_service/src/lib.rs:16-100`

---

## 相关跳转

- [GN Targets](05_GN_Targets.md) - 了解构建配置
- [编译产物](06_Build_Artifacts.md) - 了解编译产物
- [项目概述](00_Overview.md) - 了解项目特性
