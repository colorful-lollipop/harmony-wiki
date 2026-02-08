# 配置开关说明

## 1. GN 构建开关

### 1.1 系统级开关

| 开关名称 | 类型 | 默认值 | 影响范围 |
|---------|------|-------|---------|
| `ohos_lite` | boolean | false | 轻量系统构建 |
| `ohos_kernel_type` | enum | linux | 内核类型（liteos_m/linux） |
| `support_jsapi` | boolean | true | JS API 支持 |
| `is_lite_system` | boolean | false | 轻量系统标志 |
| `product_name` | string | - | 产品名称 |

**使用示例**：

```gn
# 轻量系统构建
ohos_lite = true
ohos_kernel_type = "liteos_m"

# 标准系统构建
ohos_lite = false
support_jsapi = true
```

### 1.2 功能开关

| 开关名称 | 类型 | 默认值 | 说明 |
|---------|------|-------|------|
| `support_jsapi` | boolean | true | 是否编译 JS API 相关代码 |
| `support_screenlock` | boolean | true | 锁屏交互支持 |
| `support_msdp` | boolean | false | MSDP 空间感知支持 |
| `device_manager_feature_product` | string | "default" | 功能产品配置 |
| `device_manager_common` | boolean | false | 通用功能模式 |

**使用示例**：

```gn
# 启用 MSDP
support_msdp = true

# 禁用锁屏
support_screenlock = false
```

### 1.3 编译选项

| 选项 | 类型 | 默认值 | 说明 |
|-----|------|-------|------|
| `branch_protector_ret` | string | "pac_ret" | PAC/BTI 返回保护 |
| `-Werror` | flag | true | 将警告视为错误 |
| `-fstack-protector-strong` | flag | true | 栈保护 |
| `-fPIC` | flag | true | 位置无关代码 |

**Sanitize 选项**：

```gn
sanitize = {
  boundary_sanitize = true   # 边界检查
  cfi = true                  # 控制流完整性
  cfi_cross_dso = true        # 跨 DSO CFI
  integer_overflow = true     # 整数溢出检查
  ubsan = true                # 未定义行为检查
}
```

**链接选项**：

```gn
ldflags = [
  "-Wl,-z,relro",   # 只读重定位
  "-Wl,-z,now",     # 立即绑定
  "-Wl,-z,noexecstack",  # 禁止执行栈
]
```

## 2. SA 配置开关

### 2.1 服务配置

**文件**：`sa_profile/device_manager.cfg`

| 配置项 | 类型 | 默认值 | 说明 |
|-------|------|-------|------|
| `apl` | string | "system_basic" | 权限等级 |
| `start-on-demand` | object | - | 按需启动配置 |
| `permission` | array | - | 权限列表 |

**示例**：

```json
{
  "name": "device_manager",
  "apl": "system_basic",
  "start-on-demand": {
    "commonevent": [
      {
        "name": "usual.event.BOOT_COMPLETED"
      }
    ]
  }
}
```

### 2.2 权限配置

| 权限 | 敏感等级 | 使用场景 |
|-----|---------|---------|
| `DISTRIBUTED_DATASYNC` | 中 | 分布式数据同步 |
| `DISTRIBUTED_SOFTBUS_CENTER` | 高 | 软总线中心控制 |
| `ENABLE_DISTRIBUTED_HARDWARE` | 高 | 使能分布式硬件 |
| `MANAGE_SECURE_SETTINGS` | 高 | 管理安全设置 |
| `ACCESS_BLUETOOTH` | 中 | 蓝牙访问 |

### 2.3 访问控制

```json
{
  "permission_acls": [
    "ohos.permission.MANAGE_SOFTBUS_NETWORK",
    "ohos.permission.ACCESS_DEVAUTH_CRED_PRIVILEGE",
    "ohos.permission.ACCESS_IDS"
  ]
}
```

## 3. 运行时开关

### 3.1 功能标志

| 标志 | 类型 | 默认值 | 作用 |
|-----|------|-------|------|
| `DEVICE_MANAGER_COMMON_FLAG` | define | false | 通用功能模式 |
| `SUPPORT_SCREENLOCK` | define | true | 锁屏支持 |
| `SUPPORT_MSDP` | define | false | MSDP 支持 |

**检测代码**：

```cpp
#ifdef DEVICE_MANAGER_COMMON_FLAG
// 通用功能模式代码
#endif

#ifdef SUPPORT_SCREENLOCK
// 锁屏支持代码
#endif
```

### 3.2 日志开关

| 开关 | 类型 | 默认值 | 级别 |
|-----|------|-------|------|
| `DH_LOG_ENABLE` | define | true | INFO+ |
| `DH_LOG_TAG` | string | - | 日志标签 |
| `LOG_DOMAIN` | number | 0xD004110 | 日志域 |

**日志级别**：

```cpp
#ifdef DH_LOG_ENABLE
#define LOGD(...) // 调试
#define LOGI(...) // 信息
#define LOGW(...) // 警告
#define LOGE(...) // 错误
#endif
```

## 4. 设备发现参数

### 4.1 发现模式

| 模式值 | 说明 | 使用场景 |
|-------|------|---------|
| `0xAA` | 主动发现 | 发现周边设备 |
| `0x55` | 被动发现 | 响应其他设备 |

### 4.2 媒介类型

| 媒介值 | 说明 |
|-------|------|
| `0` | 自动 |
| `1` | 蓝牙 |
| `2` | Wi-Fi |
| `3` | CoAP |

### 4.3 发现频率

| 频率值 | 说明 |
|-------|------|
| `2` | 中等频率 |
| `3` | 高频率 |
| `4` | 低功耗频率 |

## 5. 认证参数

### 5.1 认证类型

| 类型值 | 说明 |
|-------|------|
| `1` | PIN 码认证 |
| `2` | 免密认证 |
| `3` | 生物识别认证 |

### 5.2 授权类型

| 类型值 | 说明 |
|-------|------|
| `0` | 不需要授权 |
| `1` | 需要用户授权 |
| `2` | 设备间授权 |

### 5.3 设备显示拥有者

| 值 | 说明 |
|---|------|
| `0` | 默认 UI 显示 |
| `1` | 自定义 UI 显示 |

## 6. 开发调试开关

### 6.1 调试宏

| 宏 | 作用 | 风险 |
|---|------|-----|
| `ENABLE_DEBUG_LOG` | 启用详细日志 | 信息泄露 |
| `ENABLE_PERF_TRACE` | 性能追踪 | 性能影响 |
| `DUMP_IPC_MESSAGE` | 打印 IPC 消息 | 敏感信息泄露 |

### 6.2 测试开关

| 开关 | 作用 |
|-----|------|
| `ENABLE_MOCK_DSOFTBUS` | 模拟 DSoftBus |
| `ENABLE_MOCK_HICHAIN` | 模拟 HiChain |
| `ENABLE_UNIT_TEST` | 单元测试 |

## 7. 产品定制开关

### 7.1 默认产品

```gn
device_manager_feature_product = "default"
```

**支持的定制**：
- `default`：默认功能集
- `lite`：轻量功能集
- `premium`：完整功能集

### 7.2 特性组合

```gn
# 标准配置
device_manager_feature_product = "default"
support_screenlock = true
support_msdp = false

# 轻量配置
device_manager_feature_product = "lite"
support_screenlock = false
support_msdp = false

# 完整配置
device_manager_feature_product = "premium"
support_screenlock = true
support_msdp = true
```

## 8. 兼容性开关

### 8.1 API 版本

| 版本 | 开关 | 说明 |
|-----|------|------|
| 3.2+ | `OHOS_API_3_2` | OpenHarmony 3.2 |
| 4.0+ | `OHOS_API_4_0` | OpenHarmony 4.0 |
| 4.1+ | `OHOS_API_4_1` | OpenHarmony 4.1 |

### 8.2 N-API 版本

```cpp
#if NAPI_VERSION >= 8
// NAPI 8 特性
#endif
```

## 9. 安全开关

### 9.1 安全编译

| 开关 | 作用 | 推荐 |
|-----|------|-----|
| `-fstack-protector-strong` | 栈保护 | ✅ 启用 |
| `-fPIE` | 位置无关可执行 | ✅ 启用 |
| `-Wl,-z,relro` | 只读重定位 | ✅ 启用 |
| `-Wl,-z,now` | 立即绑定 | ✅ 启用 |
| `-fPIC` | 位置无关代码 | ✅ 启用 |

### 9.2 安全特性

| 特性 | 开关 | 作用 |
|-----|------|------|
| ASan | `use_asan` | 内存错误检测 |
| UBsan | `use_ubsan` | 未定义行为检测 |
| CFI | `cfi` | 控制流完整性 |

## 10. 开关配置建议

### 10.1 开发环境

```gn
# 开发配置
sanitize = {
  boundary_sanitize = true
  cfi = false  # 调试时禁用
  integer_overflow = true
  ubsan = true
}

# 启用调试日志
DH_LOG_ENABLE = true
```

### 10.2 测试环境

```gn
# 测试配置
sanitize = {
  boundary_sanitize = true
  cfi = true
  integer_overflow = true
  ubsan = true
}

# 启用完整日志
DH_LOG_ENABLE = true
ENABLE_DEBUG_LOG = true
```

### 10.3 生产环境

```gn
# 生产配置
sanitize = {
  boundary_sanitize = false
  cfi = true
  integer_overflow = true
  ubsan = false
}

# 最小化日志
DH_LOG_ENABLE = true
ENABLE_DEBUG_LOG = false
```
