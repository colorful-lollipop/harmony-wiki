# 目录结构与代码地图 (CodeMap)

> Sandbox Manager 项目目录结构、关键文件定位与代码导航指南

---

## 3.1 顶层目录结构

```
sandbox_manager/
├── interfaces/                    # 接口层（对外暴露的 API）
│   └── inner_api/
│       └── sandbox_manager/
│           ├── include/            # SDK 头文件
│           │   ├── sandbox_manager_kit.h      # 主 Kit 头文件
│           │   ├── sandbox_manager_err_code.h # 错误码定义
│           │   └── policy_info.h              # 策略数据结构
│           └── src/                 # SDK 实现
├── frameworks/                     # 框架层（IPC 通信、业务封装）
│   ├── common/                     # 公共框架组件
│   ├── sandbox_manager/             # 框架核心
│   │   ├── ISandboxManager.idl      # IPC 接口定义
│   │   ├── include/                 # 框架头文件
│   │   └── src/                     # 框架实现
│   ├── inner_api/
│   │   └── sandbox_manager/
│   │       ├── include/             # 内部 API 头文件
│   │       │   ├── sandbox_manager_client.h  # 客户端代理
│   │       │   └── ...
│   │       └── src/                 # 内部 API 实现
│   └── sandbox_test_common/         # 测试公共组件
├── services/                        # 服务层（核心业务逻辑）
│   ├── common/                      # 服务公共组件
│   │   ├── database/                # 数据库操作
│   │   └── utils/                   # 工具类
│   └── sandbox_manager/
│       ├── main/
│       │   ├── cpp/
│       │   │   ├── include/         # 服务头文件
│       │   │   │   ├── database/     # 数据库头文件
│       │   │   │   ├── mac/         # MAC 适配器头文件
│       │   │   │   ├── service/     # 服务核心头文件
│       │   │   │   └── sensitive/   # 敏感操作头文件
│       │   │   └── src/             # 服务实现
│       │   │       ├── database/     # 数据库实现
│       │   │       ├── mac/         # MAC 适配器实现
│       │   │       ├── service/     # 服务核心实现
│       │   │       ├── share/       # 共享功能
│       │   │       └── media/       # 媒体路径支持
│       │   └── sa_profile/          # SA 配置文件
│       └── test/                    # 服务层测试
├── config/                          # 配置文件
├── test/                            # 测试目录
│   ├── fuzztest/                    # Fuzz 测试
│   └── ...
├── sandbox_manager.gni              # GN 配置
└── bundle.json                      # 组件配置
```

---

## 3.2 核心文件定位

### 接口层 (Interface Layer)

| 文件 | 路径 | 职责 |
|-----|------|-----|
| `sandbox_manager_kit.h` | `interfaces/inner_api/sandbox_manager/include/` | 主 SDK 头文件，定义 `SandboxManagerKit` 类 |
| `sandbox_manager_err_code.h` | `interfaces/inner_api/sandbox_manager/include/` | 错误码枚举定义 |
| `policy_info.h` | `interfaces/inner_api/sandbox_manager/include/` | `PolicyInfo`、`PolicyType`、`OperateMode` 定义 |

### 框架层 (Framework Layer)

| 文件 | 路径 | 职责 |
|-----|------|-----|
| `ISandboxManager.idl` | `frameworks/sandbox_manager/` | IPC 接口定义语言文件 |
| `sandbox_manager_client.h` | `frameworks/inner_api/sandbox_manager/include/` | 客户端代理头文件 |
| `sandbox_manager_client.cpp` | `frameworks/inner_api/sandbox_manager/src/` | 客户端代理实现，包含 `CallProxyWithRetry()` |
| `sandbox_manager_kit.cpp` | `frameworks/inner_api/sandbox_manager/src/` | SDK 实现，调用客户端代理 |
| `sandbox_manager_stub.h` | `frameworks/sandbox_manager/include/` | IPC 存根基类（IDL 生成） |

### 服务层 (Framework Layer)

| 文件 | 路径 | 职责 |
|-----|------|-----|
| `sandbox_manager_service.h` | `services/sandbox_manager/main/cpp/include/service/` | 服务主类定义 |
| `sandbox_manager_service.cpp` | `services/sandbox_manager/main/cpp/src/service/` | 服务实现，包含所有 IPC 处理 |
| `sandbox_manager_const.h` | `services/sandbox_manager/main/cpp/include/service/` | 常量定义（权限、UID、路径限制） |
| `policy_info_manager.h` | `services/sandbox_manager/main/cpp/include/service/` | 策略管理器定义 |
| `policy_info_manager.cpp` | `services/sandbox_manager/main/cpp/src/service/` | 策略管理器实现，包含所有验证逻辑 |
| `policy_trie.h` | `services/sandbox_manager/main/cpp/include/service/` | 路径 Trie 树定义 |
| `policy_trie.cpp` | `services/sandbox_manager/main/cpp/src/service/` | 路径 Trie 树实现 |

### 数据库层 (Database Layer)

| 文件 | 路径 | 职责 |
|-----|------|-----|
| `sandbox_manager_rdb.h` | `services/sandbox_manager/main/cpp/include/database/` | RDB 操作头文件 |
| `sandbox_manager_rdb.cpp` | `services/sandbox_manager/main/cpp/src/database/` | RDB 操作实现 |
| `sandbox_manager_rdb_utils.h` | `services/sandbox_manager/main/cpp/include/database/` | RDB 工具类 |
| `sandbox_manager_rdb_open_callback.h` | `services/sandbox_manager/main/cpp/include/database/` | RDB 打开回调 |
| `policy_field_const.h` | `services/sandbox_manager/main/cpp/include/database/` | 数据库字段常量 |
| `policy_field_const.cpp` | `services/sandbox_manager/main/cpp/src/database/` | 数据库字段常量实现 |

### MAC 层 (MAC Layer)

| 文件 | 路径 | 职责 |
|-----|------|-----|
| `mac_adapter.h` | `services/sandbox_manager/main/cpp/include/mac/` | MAC 适配器头文件 |
| `mac_adapter.cpp` | `services/sandbox_manager/main/cpp/src/mac/` | MAC 适配器实现，所有 ioctl 调用 |

---

## 3.3 代码导航图

### 按功能定位文件

#### 策略设置流程

```
入口: sandbox_manager_kit.cpp
    ↓
sandbox_manager_client.cpp (IPC 调用)
    ↓
sandbox_manager_service.cpp (IPC 处理)
    ↓
policy_info_manager.cpp (业务逻辑)
    ↓
    ├── mac_adapter.cpp (MAC 层)
    └── sandbox_manager_rdb.cpp (数据库)
```

#### 权限检查流程

```
权限检查: sandbox_manager_service.cpp::CheckPermission()
    ↓
access_token::AccessTokenKit::VerifyAccessToken()
    ↓
返回: PERMISSION_GRANTED / PERMISSION_DENIED
```

#### 路径验证流程

```
路径验证: policy_info_manager.cpp::CheckPathIsBlocked()
    ↓
    ├── AdjustPath()          // 路径规范化
    ├── GetDepth()            // 计算深度
    ├── CheckPathWithinRule() // 规则检查
    └── CheckPathWithinBundleName() // Bundle 验证
```

---

## 3.4 关键符号速查

### 类符号

| 类名 | 文件 | 行号 | 职责 |
|-----|------|------|-----|
| `SandboxManagerKit` | `sandbox_manager_kit.h` | 28 | SDK 主类 |
| `SandboxManagerService` | `sandbox_manager_service.h` | 34 | 系统服务主类 |
| `SandboxManagerClient` | `sandbox_manager_client.h` | 72 | IPC 客户端代理 |
| `PolicyInfoManager` | `policy_info_manager.h` | 42 | 策略管理器 |
| `MacAdapter` | `mac_adapter.h` | 32 | MAC 适配器 |
| `SandboxManagerRdb` | `sandbox_manager_rdb.h` | - | RDB 操作类 |

### 关键函数

| 函数名 | 文件 | 行号 | 职责 |
|-------|------|------|-----|
| `CheckPermission()` | `sandbox_manager_service.cpp` | 829 | 权限验证 |
| `CheckPolicyValidity()` | `policy_info_manager.cpp` | 1138 | 策略验证 |
| `CheckPathIsBlocked()` | `policy_info_manager.cpp` | 1341 | 路径检查 |
| `FilterValidPolicyInBatch()` | `policy_info_manager.cpp` | 337 | 批量验证 |
| `CallProxyWithRetry()` | `sandbox_manager_client.cpp` | 359 | IPC 重试 |

### 宏与常量

| 符号 | 文件 | 行号 | 值 | 职责 |
|-----|------|------|-----|------|
| `POLICY_PATH_LIMIT` | `sandbox_manager_const.h` | 25 | 4095 | 路径最大长度 |
| `SET_POLICY_PERMISSION_NAME` | `sandbox_manager_const.h` | 30 | "ohos.permission.SET_SANDBOX_POLICY" | 设置策略权限 |
| `FOUNDATION_UID` | `sandbox_manager_const.h` | 36 | 5523 | Foundation UID |
| `SPACE_MGR_SERVICE_UID` | `sandbox_manager_const.h` | 35 | 7013 | Space Manager UID |

---

## 3.5 常用代码模式

### 模式 1：添加新 IPC 方法

```
1. 在 ISandboxManager.idl 添加方法定义
2. 重新编译生成 stub/proxy 代码
3. 在 SandboxManagerStub 中实现 OnRemoteRequest 分发
4. 在 SandboxManagerService 中实现方法
5. 在 SandboxManagerClient 中调用代理方法
6. 在 SandboxManagerKit 中添加静态方法
```

### 模式 2：添加新验证规则

```
1. 在 policy_info_manager.h 声明验证函数
2. 在 policy_info_manager.cpp 实现验证逻辑
3. 在 CheckPathIsBlocked() 中调用新验证函数
4. 返回适当的错误码 (SandboxRetType)
```

### 模式 3：添加新数据库操作

```
1. 在 sandbox_manager_rdb.h 声明方法
2. 在 sandbox_manager_rdb.cpp 实现 SQL 操作
3. 在 policy_info_manager.cpp 调用新方法
4. 处理数据库错误返回码
```

---

## 3.6 测试文件位置

| 测试类型 | 目录 |
|---------|------|
| **单元测试** | `frameworks/inner_api/sandbox_manager/test/` |
| **服务测试** | `services/sandbox_manager/test/` |
| **Fuzz 测试** | `test/fuzztest/` |
| **框架测试** | `frameworks/test/` |

---

## 3.7 构建产物

| 产物类型 | 位置 | 说明 |
|---------|------|-----|
| **动态库** | `out/.../libsandbox_manager_sdk.z.so` | SDK 动态库 |
| **静态库** | `out/.../libsandbox_manager_sdk.a` | SDK 静态库 |
| **可执行文件** | `out/.../sandbox_manager_service` | 服务可执行文件 |
| **配置文件** | `system/etc/sandbox_manager/` | SA 配置文件 |

**证据来源**：`bundle.json:52-70`

---

*文档更新时间: 2025-02-07*
