# 附录: 配置参数与编译开关

> 完整的功能开关、编译参数和运行时配置说明。

## 1. Feature Flags

### 1.1 全局 Feature

**文件**: `adapter/default_config/feature_config/standard/config.gni`

| Feature | 默认值 | 说明 |
|---------|--------|------|
| `ability_ability_runtime_enable` | true/false | 是否启用 Ability Runtime |
| `communication_netmanager_base_enable` | true/false | 是否启用网络管理器 |

**证据**: `adapter/default_config/feature_config/standard/config.gni`

---

### 1.2 服务层 Feature

**文件**: `services/engine/engine_sa.gni`

| Feature | 默认值 | 说明 |
|---------|--------|------|
| `ability_ability_base_enable` | true/false | 是否启用 Ability Base |
| `preference_native_preferences_enable` | true/false | 是否启用原生偏好设置 |
| `update_service_enable_run_on_demand_qos` | true/false | 是否启用 QOS 优先级调整 |

**证据**: `services/engine/engine_sa.gni`

---

### 1.3 数据库层 Feature

**文件**: `services/core/ability/sqlite/sqlite.gni`

| Feature | 默认值 | 说明 |
|---------|--------|------|
| `relational_store_native_rdb_enable` | true/false | 是否启用 RDB |

**证据**: `services/core/ability/sqlite/sqlite.gni`

---

## 2. 编译 Defines

### 2.1 核心 Defines

| Define | 条件 | 说明 |
|--------|------|------|
| `DUAL_ADAPTER` | 始终 | 双适配器支持 |
| `UPDATE_SERVICE` | 始终 | 升级服务标记 |

---

### 2.2 条件 Defines

| Define | 条件 | 说明 |
|--------|------|------|
| `ABILITY_BASE_ENABLE` | `ability_ability_base_enable=true` | Ability Base 支持 |
| `ABILITY_RUNTIME_ENABLE` | `ability_ability_runtime_enable=true` | Ability Runtime 支持 |
| `ABILITY_RUNTIME_INNER_ENABLE` | `ability_ability_runtime_enable=false` | Ability Runtime 内部支持 |
| `NETMANAGER_BASE_ENABLE` | `communication_netmanager_base_enable=true` | 网络管理器支持 |
| `NATIVE_PREFERENCES_ENABLE` | `preference_native_preferences_enable=true` | 原生偏好设置 |
| `UPDATE_SERVICE_ENABLE_RUN_ON_DEMAND_QOS` | `update_service_enable_run_on_demand_qos=true` | QOS 优先级 |
| `RELATIONAL_STORE_NATIVE_RDB_ENABLE` | `relational_store_native_rdb_enable=true` | RDB 支持 |

---

### 2.3 测试 Defines

| Define | 说明 |
|--------|------|
| `UPDATER_UT` | 单元测试模式 |
| `UPDATER_UT` | 禁用某些运行时检查 |

---

## 3. SA 配置参数

### 3.1 SA Profile

**文件**: `services/engine/sa_profile/3006.json`

```json
{
  "name": 3006,
  "libpath": "libupdateservice.z.so",
  "run-on-create": false,
  "distributed": false,
  "bootphase": "BootStartPhase",
  "dump-level": 1,
  "auto-restart": true,
  "start-on-demand": {
    "allow-update": true,
    "commonevent": ["usual.event.BOOT_COMPLETED"],
    "timedevent": {"name": "loopevent", "value": "14400"}
  }
}
```

| 参数 | 值 | 说明 |
|------|-------|------|
| `name` | 3006 | SA ID |
| `libpath` | libupdateservice.z.so | 库路径 |
| `run-on-create` | false | 是否启动时创建 |
| `bootphase` | BootStartPhase | 启动阶段 |
| `auto-restart` | true | 异常自动重启 |
| `timedevent.value` | 14400 | 空闲超时(秒) |

**证据**: `services/engine/sa_profile/3006.json`

---

### 3.2 SA 服务配置

**文件**: `services/engine/etc/updater_sa.cfg`

```json
{
  "uid": "update",
  "gid": ["update", "netsys_socket"],
  "permission": [
    "ohos.permission.UPDATE_SYSTEM",
    "ohos.permission.GET_NETWORK_INFO",
    "ohos.permission.STORAGE_MANAGER_CRYPT"
  ],
  "secon": "u:r:updater_sa:s0"
}
```

| 参数 | 值 | 说明 |
|------|-------|------|
| `uid` | update | 运行用户 |
| `gid` | update, netsys_socket | 运行组 |
| `secon` | u:r:updater_sa:s0 | SELinux 上下文 |

**证据**: `services/engine/etc/updater_sa.cfg`

---

### 3.3 RC 脚本配置

**文件**: `services/engine/etc/updater_sa.rc`

```bash
service updater_sa /system/bin/sa_main /system/profile/updater_sa.json
    class z_core
    user system
    group system shell
    seclabel u:r:updater_sa:s0
```

**证据**: `services/engine/etc/updater_sa.rc`

---

## 4. 运行时配置

### 4.1 数据目录配置

**文件**: `services/engine/etc/dupdate_config.json`

```json
{
  "update_engine": {
    "data_root_path": "/data/service/el1/public/update",
    "package_path": "/data/update/ota_package"
  }
}
```

| 路径 | 用途 |
|------|------|
| `/data/service/el1/public/update` | 升级数据根目录 |
| `/data/service/el1/public/update/dupdate_engine/databases` | 数据库目录 |
| `/data/service/el1/public/update/dupdate_engine/preferences` | 偏好设置 |
| `/data/service/el1/public/update/dupdate_engine/files` | 文件存储 |
| `/data/update/ota_package` | OTA 升级包目录 |

**证据**: `services/engine/etc/dupdate_config.json`

---

### 4.2 文件权限配置

**文件**: `services/core/ability/common/include/constant.h`

```cpp
static const std::vector<DirInfo> BASE_DIR_INFOS {
    {Constant::UPDATE_ENCRYPTED_ROOT_PATH, 0751, false},
    {Constant::DUPDATE_ENGINE_ENCRYPTED_ROOT_PATH, 0700, false},
    {Constant::DATABASES_ROOT_PATH, 0700, false},
    {Constant::PREFERENCES_ROOT_PATH, 0700, false},
    {Constant::FILES_ROOT_PATH, 0700, true},
    {Constant::UPDATE_PACKAGE_ROOT_PATH, 0770, false},
    {Constant::DUPDATE_ENGINE_PACKAGE_ROOT_PATH, 0770, true}
};
```

| 目录 | 权限 | 说明 |
|------|------|------|
| `/data/service/el1/public/update/` | 0751 | 根目录 |
| `/data/service/el1/public/update/dupdate_engine/` | 0700 | 引擎数据 |
| `/data/update/ota_package/` | 0770 | 升级包 |

**证据**: `constant.h:63-71`

---

## 5. N-API 参数

### 5.1 模块配置

| 参数 | 值 | 说明 |
|------|-------|------|
| `NAPI_VERSION` | 8 | N-API 版本 |
| `nm_modname` | "update" | 模块名 |

**证据**: `frameworks/js/napi/update/BUILD.gn:72`

---

### 5.2 会话类型常量

**文件**: `frameworks/js/napi/update/include/session_type.h`

| 常量 | 值 | 说明 |
|------|-------|------|
| `SESSION_CHECK_VERSION` | 0 | 检查版本 |
| `SESSION_DOWNLOAD` | 1 | 下载 |
| `SESSION_PAUSE_DOWNLOAD` | 2 | 暂停下载 |
| `SESSION_RESUME_DOWNLOAD` | 3 | 恢复下载 |
| `SESSION_UPGRADE` | 4 | 升级 |
| `SESSION_SET_POLICY` | 5 | 设置策略 |
| `SESSION_GET_POLICY` | 6 | 获取策略 |
| `SESSION_FACTORY_RESET` | 15 | 恢复出厂 |
| `SESSION_VERIFY_PACKAGE` | 16 | 验证升级包 |

---

## 6. 错误码配置

### 6.1 业务错误码

**文件**: `foundations/model/include/call_result.h`

| 错误码 | 常量 | 描述 |
|--------|------|------|
| 0 | SUCCESS | 成功 |
| 100 | FAIL | 通用失败 |
| 103 | FORBIDDEN | 禁止操作 |
| 201 | APP_NOT_GRANTED | 无权限 |
| 202 | NOT_SYSTEM_APP | 非系统应用 |
| 401 | PARAM_ERR | 参数错误 |
| 801 | UN_SUPPORT | 不支持 |

---

### 6.2 内部错误码

| 错误码 | 常量 | 描述 |
|--------|------|------|
| 0 | INT_CALL_SUCCESS | 调用成功 |
| 100 | INT_CALL_FAIL | 调用失败 |
| 201 | INT_APP_NOT_GRANTED | 无权限 |
| 202 | INT_NOT_SYSTEM_APP | 非系统应用 |
| 401 | INT_PARAM_ERR | 参数错误 |
| 801 | INT_UN_SUPPORT | 不支持 |

**证据**: `call_result.h:44-58`

---

## 7. 权限配置

### 7.1 系统权限

| 权限名 | 描述 | 受保护操作 |
|--------|------|------------|
| `ohos.permission.UPDATE_SYSTEM` | 系统升级 | 核心升级操作 |
| `ohos.permission.FACTORY_RESET` | 恢复出厂 | factoryReset |
| `ohos.permission.FORCE_FACTORY_RESET` | 强制恢复 | forceFactoryReset |

---

### 7.2 UID/GID

| ID | 值 | 说明 |
|----|-------|------|
| ROOT_UID | 0 | root 用户 |
| EDM_UID | 3057 | Enterprise Device Management |
| UPDATE_UID | (运行时) | update 用户 |

**证据**: `update_service.cpp:54-55`

---

## 8. 日志配置

### 8.1 日志标签

| 标签 | 说明 | 域 ID |
|------|------|--------|
| `UPDATE_SERVICE_KITS` | Inner API 日志 | 0xD002E00 |
| `UPDATE_SERVICE` | 服务日志 | - |

**证据**: `services/engine/BUILD.gn:29-30`

---

### 8.2 日志级别

| 级别 | 说明 |
|------|------|
| `LOGI` | Info |
| `LOGW` | Warning |
| `LOGE` | Error |

---

## 9. 安全配置

### 9.1 编译时安全选项

| 选项 | 说明 | 适用范围 |
|------|------|----------|
| `-fPIC` | 位置无关代码 | 所有库 |
| `-Os` | 尺寸优化 | 发布版 |
| `-fstack-protector-strong` | 栈保护 | 所有库 |
| `boundary_sanitize` | 边界检查 | SA |
| `cfi` | 控制流完整性 | SA |
| `cfi_cross_dso` | 跨 DSO CFI | SA |
| `pac_ret` | PAC 返回地址保护 | Inner API |
| `integer_overflow` | 整数溢出检测 | Inner API |
| `ubsan` | 未定义行为检测 | Inner API |

**证据**: `services/engine/BUILD.gn:52-58`, `frameworks/js/napi/update/BUILD.gn:68-74`

---

### 9.2 CFI Blocklist

**文件**: `services/engine/cfi_blocklist.txt`

用于排除特定函数进行 CFI 检查。

---

## 10. 事件配置

### 10.1 HiSysEvent 配置

**文件**: `hisysevent.yaml`

| 事件 | 说明 |
|------|------|
| `permission_verify_failed` | 权限验证失败 |
| `pkg_verify_failed` | 包验证失败 |
| `system_reset` | 系统重置 |

**证据**: `hisysevent.yaml`, `update_system_event.h`

---

## 11. 第三方依赖配置

### 11.1 外部依赖

| 依赖 | 用途 |
|------|------|
| `curl` | HTTP 下载 |
| `openssl` | 加密/签名验证 |
| `cJSON` | JSON 解析 |
| `libxml2` | XML 解析 |

**证据**: `bundle.json:33-34`, `services/engine/BUILD.gn:external_deps`

---

## 12. 端口与地址

### 12.1 服务地址

| 地址 | 用途 |
|------|------|
| `UPDATE_DISTRIBUTED_SERVICE_ID` | SA ID (3006) |

---

## 13. 使用示例

### 13.1 查看当前配置

```bash
# 查看 GN 配置
gn args out/default --list | grep update

# 查看 Feature 状态
gn gen out/default
ninja -C out/default -t targets | grep update

# 查看安装产物
ls -la out/default/libs/ | grep update
```

---

## 14. 修改建议

### 14.1 启用/禁用 Feature

```bash
# 修改 config.gni
echo 'ability_ability_runtime_enable = true' >> args.gn

# 重新构建
hb build -f
```

### 14.2 调整 SA 配置

```bash
# 修改 3006.json
# 重启设备生效
reboot
```
