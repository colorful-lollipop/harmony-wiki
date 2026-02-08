# 配置开关

## 1. GN 编译开关

### 1.1 全局开关 (user_auth_framework.gni)

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `user_auth_framework_enable_dynamic_load` | bool | false | 启用动态加载模式 |
| `user_auth_framework_path` | string | - | 模块路径常量 |
| `screenlock_client_enable` | bool | true | 启用锁屏客户端 |
| `user_auth_framework_has_ext` | bool | false | 是否有扩展部分 |

### 1.2 条件编译 (services/load_mode/BUILD.gn)

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `ENABLE_DYNAMIC_LOAD` | false | 定义后启用动态加载 |

---

## 2. Feature Flags (bundle.json)

### 2.1 功能开关

| Feature | 默认值 | 用途 |
|---------|--------|------|
| `user_auth_framework_enabled` | - | 启用框架主功能 |
| `user_auth_framework_enable_dynamic_load` | - | 启用动态加载 |

### 2.2 启用方式

```json
{
  "features": [
    {
      "name": "user_auth_framework_enabled",
      "value": true
    },
    {
      "name": "user_auth_framework_enable_dynamic_load",
      "value": false
    }
  ]
}
```

---

## 3. 系统能力 (Syscap)

| Syscap | 说明 |
|--------|------|
| `SystemCapability.UserIAM.UserAuth.Core` | 核心认证能力 |
| `SystemCapability.UserIAM.UserAuth.View` | UI 相关能力 |

---

## 4. 构建配置

### 4.1 编译类型

| 类型 | 配置 | 产物 |
|------|------|------|
| Release | 默认 | 优化产物 |
| Debug | `build_variant=userdebug` | 带调试符号 |

### 4.2 构建分组 (bundle.json)

```json
{
  "build": {
    "group_type": {
      "base_group": [],
      "fwk_group": [
        "//base/useriam/user_auth_framework/frameworks/js/napi/user_auth:userauth",
        "//base/useriam/user_auth_framework/frameworks/js/napi/user_access_ctrl:useraccessctrl"
      ],
      "service_group": [
        "//base/useriam/user_auth_framework/services:userauthservice"
      ]
    }
  }
}
```

---

## 5. SA 配置

### 5.1 默认配置

| SA ID | 配置文件 | 启动方式 |
|-------|----------|----------|
| 901 | sa_profile/default/901.json | 需时启动 |
| 921 | sa_profile/default/921.json | 需时启动 |
| 931 | sa_profile/default/931.json | 需时启动 |

### 5.2 动态加载配置

| SA ID | 配置文件 | 启动方式 |
|-------|----------|----------|
| 901 | sa_profile/dynamic_load/901.json | 需时启动 |
| 921 | sa_profile/dynamic_load/921.json | 需时启动 |
| 931 | sa_profile/dynamic_load/931.json | 需时启动 |

---

## 6. 参数配置

### 6.1 系统参数

| 参数 | 路径 | 用途 |
|------|------|------|
| `useriam.para` | param/useriam.para | 系统参数定义 |
| `useriam.para.dac` | param/useriam.para.dac | DAC 权限配置 |

---

## 7. 安全配置

### 7.1 安全等级

| 等级 | 枚举 | 用途 |
|------|------|------|
| `AuthTrustLevel` | ATL1-ATL4 | 认证信任等级 |
| `ExecutorSecureLevel` | ESL0-ESL3 | 执行器安全等级 |

### 7.2 权限检查

```cpp
// 文件: services/core/src/ipc_common.cpp
AccessTokenKit::CheckPermission(accessToken, permissionName);
```

---

## 8. 调试配置

### 8.1 日志级别

| 级别 | 宏 | 用途 |
|------|-----|------|
| INFO | `IAM_LOGI` | 普通信息 |
| WARN | `IAM_LOGW` | 警告 |
| ERROR | `IAM_LOGE` | 错误 |
| DEBUG | `IAM_LOGD` | 调试 |

### 8.2 HiTrace 配置

| 配置 | 说明 |
|------|------|
| `iam_hitrace_helper.h` | 性能追踪 |
| `hisysevent_adapter.h` | 事件上报 |

---

## 9. 相关文档

- [构建配置](04_Build.md)
- [架构说明](01_Architecture.md)
- [安全评审](05_Security.md)
