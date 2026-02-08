# 架构设计

## 系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (Application)                       │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐                     │
│  │   JS/ArkTS      │    │   ETS           │                     │
│  │   (@ohos.config │    │   (@ohos.confi │                     │
│  │    gPolicy)     │    │    gPolicy)     │                     │
│  └────────┬────────┘    └────────┬────────┘                     │
├───────────┼──────────────────────┼───────────────────────────────┤
│           │                      │                               │
│           ▼                      ▼                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    N-API 层                              │   │
│  │  ┌──────────────────┐  ┌──────────────────┐               │   │
│  │  │ config_policy_   │  │ custom_config_   │               │   │
│  │  │    napi.cpp      │  │    napi.cpp      │               │   │
│  │  └────────┬─────────┘  └────────┬─────────┘               │   │
│  └───────────┼─────────────────────┼───────────────────────────┘ │
│              │                     │                            │
│              ▼                     ▼                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   C++ 绑定层                             │   │
│  │  ┌──────────────────┐  ┌──────────────────┐               │   │
│  │  │ ConfigPolicyNapi│  │ 自定义 C++ 接口  │               │   │
│  │  └────────┬─────────┘  └────────┬─────────┘               │   │
│  └───────────┼─────────────────────┼───────────────────────────┘ │
│              │                     │                            │
│              └──────────►           │                            │
│                           ▼        ▼                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │               核心逻辑层 (frameworks/)                   │   │
│  │  ┌──────────────────────────────────────────────────┐   │   │
│  │  │            config_policy_utils.c                 │   │   │
│  │  │  ┌─────────────┐  ┌─────────────────────────┐   │   │   │
│  │  │  │ 配置层级管理 │  │ FollowX 机制实现       │   │   │   │
│  │  │  │ GetCfgDir   │  │ GetOpkeyPath            │   │   │   │
│  │  │  │ List        │  │ GetFollowXRule          │   │   │   │
│  │  │  └─────────────┘  └─────────────────────────┘   │   │   │
│  │  └──────────────────────────────────────────────────┘   │   │
│  └───────────────┬─────────────────────┬───────────────────┘ │
│                  │                     │                       │
│                  ▼                     ▼                       │
│  ┌─────────────────────────────┐  ┌─────────────────────────┐ │
│  │    系统参数 (SystemParam)    │  │  文件系统 (access)       │ │
│  │    init 模块                 │  │  VFS                    │ │
│  └─────────────────────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 数据流

### 配置文件查询流程

```
1. JS/ArkTS 调用入口
   ↓
2. N-API 绑定层接收参数
   ↓
3. 参数校验 (relPath, followMode, extra)
   ↓
4. 调用 C++ 核心函数
   ├─ GetOneCfgFile()     // 获取单个最高优先级文件
   ├─ GetCfgFiles()       // 获取所有层级文件
   └─ GetCfgDirList()     // 获取配置目录列表
   ↓
5. CfgDirList() 读取系统参数
   ├─ 非 LiteOS: SystemGetParameter(CUST_KEY_POLICY_LAYER)
   ├─ LiteOS_M: SetMiniConfigPolicy()
   └─ LiteOS__: DEFAULT_LAYER
   ↓
6. 文件系统查询 (access())
   ↓
7. 返回结果
```

**代码证据**: `frameworks/config_policy/src/config_policy_utils.c:525-548`

---

## FollowX 机制

FollowX 是配置策略组件的核心特性，支持根据 SIM 卡、运营商等条件动态选择配置文件。

### FollowX 模式

| 模式 | 值 | 描述 | 代码证据 |
|------|-----|------|----------|
| `FOLLOWX_MODE_DEFAULT` | 0 | 使用默认 Follow 规则 | `config_policy_utils.h:44` |
| `FOLLOWX_MODE_NO_RULE_FOLLOWED` | 1 | 不使用任何 Follow 规则 | `config_policy_utils.h:46` |
| `FOLLOWX_MODE_SIM_DEFAULT` | 10 | 根据默认 SIM 卡选择 | `config_policy_utils.h:48` |
| `FOLLOWX_MODE_SIM_1` | 11 | 根据 SIM 1 选择 | `config_policy_utils.h:50` |
| `FOLLOWX_MODE_SIM_2` | 12 | 根据 SIM 2 选择 | `config_policy_utils.h:52` |
| `FOLLOWX_MODE_USER_DEFINED` | 100 | 用户自定义规则 | `config_policy_utils.h:55` |

### FollowX 规则配置

FollowX 规则存储在系统参数 `CUST_FOLLOW_X_RULES` 中，格式为：

```
:relPath,mode[,extra][:]...
```

**示例**:
```
:etc/xml/config.xml,10:etc/xml/config1.xml,100,etc/carrier/${key:-value}
```

**代码证据**: `frameworks/config_policy/src/config_policy_utils.c:189-235`

---

## 配置层级

配置层级通过 `CUST_KEY_POLICY_LAYER` 系统参数定义，默认值为 `DEFAULT_LAYER`。

层级路径按优先级从低到高排列，查找时从最高优先级开始。

**代码证据**: `frameworks/config_policy/src/config_policy_utils.c:385-406`

---

## 线程模型

### 同步与异步

| 接口类型 | 线程模型 | 说明 |
|----------|----------|------|
| `*Sync` 系列 | 同步调用 | 直接返回结果 |
| `*` 系列（无 Sync 后缀） | 异步调用 | 通过 Promise 或 Callback 返回 |
| `getCfgDirListSync` | 同步 | 直接调用 `GetCfgDirList()` |

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp:257-280`

### 异步执行流程

```
1. napi_create_promise() / napi_create_reference()
2. napi_create_async_work()
3. napi_queue_async_work_with_qos() - 使用 user_initiated QoS
4. execute callback (NativeGetOneCfgFile 等)
5. complete callback (NativeCallbackComplete)
6. resolve promise / call callback
```

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp:257-280`

---

## 内存管理

### 分配与释放

| 分配函数 | 释放函数 | 用途 |
|----------|----------|------|
| `malloc()` / `calloc()` | `free()` | 通用内存分配 |
| `strdup()` | `free()` | 字符串复制 |
| `GetCfgDirList()` | `FreeCfgDirList()` | 配置目录列表 |
| `GetCfgFiles()` | `FreeCfgFiles()` | 配置文件列表 |

**代码证据**: `frameworks/config_policy/src/config_policy_utils.c:408-432`

---

## 错误处理

### 错误码

| 错误码 | 含义 | 触发条件 |
|--------|------|----------|
| 401 | 参数错误 | 参数类型或数量不正确 |
| NULL | 未找到 | 配置文件或目录不存在 |

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp:46`

### 错误抛送

```cpp
napi_throw_error(env, std::to_string(errCode).c_str(), errMessage.c_str());
```

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp:522-526`

---

## 相关文档

- [N-API 接口文档](./04_N-API_Reference.md)
- [C++ 内部 API](./05_Cpp_Inner_API.md)
