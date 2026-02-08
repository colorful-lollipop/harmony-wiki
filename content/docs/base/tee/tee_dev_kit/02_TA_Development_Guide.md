# TA 开发指南

> **阅读时间**: 30 分钟 | **目标**: 开发第一个 Trusted Application

---

## 概述

本指南介绍如何使用 tee_dev_kit 开发可信应用 (Trusted Application, TA)。TA 是运行在 TEE (可信执行环境) 中的安全应用，用于处理敏感数据和加密操作。

**前置条件**:
- ✅ 已配置 LLVM 工具链（参考 [00_Overview.md](00_Overview.md)）
- ✅ 已导入第三方头文件 (musl, bounds_checking_function)
- ✅ 已安装 Python 依赖 (pycryptodome, defusedxml)

**参考资源**:
- 架构说明: [01_Architecture.md](01_Architecture.md)
- 示例代码: [05_Examples.md](05_Examples.md)
- 构建系统: [03_Build_System.md](03_Build_System.md)

---

## GP TEE 标准入口函数

TA 必须实现 GlobalPlatform TEE 标准定义的五大入口函数。这些函数是 TEE Framework 与 TA 之间的契约接口。

**代码证据**: `sdk/src/TA/helloworld_demo/ta_demo.c:50-158`

### 入口函数概述

| 入口函数 | 调用时机 | 职责 | 返回类型 |
|---------|---------|------|---------|
| `TA_CreateEntryPoint` | TA 实例创建时 | 构造函数，初始化全局资源 | `TEE_Result` |
| `TA_OpenSessionEntryPoint` | CA 请求打开会话 | 验证客户端，初始化会话状态 | `TEE_Result` |
| `TA_InvokeCommandEntryPoint` | CA 发送命令时 | 核心业务逻辑处理 | `TEE_Result` |
| `TA_CloseSessionEntryPoint` | CA 请求关闭会话 | 清理会话资源 | `void` |
| `TA_DestroyEntryPoint` | TA 实例销毁时 | 析构函数，释放全局资源 | `void` |

### 完整实现示例

**证据**: `sdk/src/TA/helloworld_demo/ta_demo.c:50-158`

#### 2.1 TA_CreateEntryPoint

**功能**: TA 实例构造函数，仅在 TA 首次加载时调用一次。

**代码位置**: `sdk/src/TA/helloworld_demo/ta_demo.c:50-68`

```c
/**
 * TA_CreateEntryPoint - TA 实例构造函数
 * @return: TEE_SUCCESS 成功，其他值表示错误
 *
 * 调用时机: TEE Framework 首次创建 TA 实例时
 * 用途: 初始化 TA 全局资源、添加 CA 白名单、打开安全存储等
 */
TEE_Result TA_CreateEntryPoint(void)
{
    /* 1. 添加 CA 白名单（可选安全措施） */
    const uint8_t hash[] = {
        0xca, 0x9f, 0x5e, 0xd7, /* ... 更多哈希值 ... */
    };
    AddCaller_CA(hash, sizeof(hash));

    /* 2. 初始化安全存储 */
    /* TEE_OpenPersistentObject(...) */

    /* 3. 初始化加密算法上下文 */
    /* crypto_context = TEE_AllocateOperation(...) */

    return TEE_SUCCESS;
}
```

**注意事项**:
- ✅ 只调用一次（在 TA 生命周期内）
- ✅ 用于初始化全局资源
- ⚠️ 不要在此处分配会话级资源（使用 `TA_OpenSessionEntryPoint`）

#### 2.2 TA_OpenSessionEntryPoint

**功能**: 会话打开入口，每次 CA 打开新会话时调用。

**代码位置**: `sdk/src/TA/helloworld_demo/ta_demo.c:70-88`

```c
/**
 * TA_OpenSessionEntryPoint - 会话打开入口
 * @param_types: 参数类型掩码
 * @param: 参数数组 (4 个)
 * @session_context: 会话上下文指针（用于传递会话数据）
 * @return: TEE_SUCCESS 成功，其他值表示错误
 *
 * 调用时机: CA 调用 TEEC_OpenSession() 时
 * 用途: 验证客户端身份、初始化会话状态
 */
TEE_Result TA_OpenSessionEntryPoint(uint32_t param_types,
    TEE_Param params[TEE_PARAM_COUNT], void** session_context)
{
    (void)param_types;
    (void)params;

    /* 1. 验证客户端身份（如果配置了白名单） */
    /* CheckCallerIdentity() */

    /* 2. 分配会话上下文 */
    my_session_t* session = TEE_Malloc(sizeof(my_session_t), 0);
    if (session == NULL) {
        return TEE_ERROR_OUT_OF_MEMORY;
    }

    /* 3. 初始化会话状态 */
    session->state = SESSION_STATE_OPEN;
    session->counter = 0;

    /* 4. 返回会话上下文 */
    *session_context = session;

    return TEE_SUCCESS;
}
```

**参数说明**:

| 参数 | 类型 | 说明 |
|------|------|------|
| `param_types` | `uint32_t` | 参数类型掩码，指示 params 数组中各参数的类型 |
| `params` | `TEE_Param[4]` | 来自 CA 的输入/输出参数 |
| `session_context` | `void**` | TA 向调用方返回的会话上下文指针 |

**参数类型常量**:

| 常量 | 说明 |
|------|------|
| `TEE_PARAM_TYPE_NONE` | 无参数 |
| `TEE_PARAM_TYPE_VALUE_INPUT` | 输入值 |
| `TEE_PARAM_TYPE_VALUE_OUTPUT` | 输出值 |
| `TEE_PARAM_TYPE_MEMREF_INPUT` | 输入内存引用 |
| `TEE_PARAM_TYPE_MEMREF_OUTPUT` | 输出内存引用 |
| `TEE_PARAM_TYPE_MEMREF_INOUT` | 输入输出内存引用 |

#### 2.3 TA_InvokeCommandEntryPoint

**功能**: 命令调用入口，TA 的核心业务逻辑处理函数。

**代码位置**: `sdk/src/TA/helloworld_demo/ta_demo.c:90-140`

```c
/**
 * TA_InvokeCommandEntryPoint - 命令调用入口
 * @session_context: 会话上下文（来自 OpenSession）
 * @cmd: 命令 ID（由 CA 和 TA 约定）
 * @param_types: 参数类型掩码
 * @param: 参数数组 (4 个)
 * @return: TEE_SUCCESS 成功，其他值表示错误
 *
 * 调用时机: CA 调用 TEEC_InvokeCommand() 时
 * 用途: 处理具体业务逻辑
 */
TEE_Result TA_InvokeCommandEntryPoint(void* session_context,
    uint32_t cmd, uint32_t param_types, TEE_Param params[TEE_PARAM_COUNT])
{
    my_session_t* session = (my_session_t*)session_context;

    /* 1. 验证会话状态 */
    if (session == NULL || session->state != SESSION_STATE_OPEN) {
        return TEE_ERROR_BAD_STATE;
    }

    /* 2. 参数校验 */
    if (!check_param_type(param_types,
        TEE_PARAM_TYPE_VALUE_INPUT,
        TEE_PARAM_TYPE_MEMREF_OUTPUT,
        TEE_PARAM_TYPE_NONE,
        TEE_PARAM_TYPE_NONE)) {
        return TEE_ERROR_BAD_PARAMETERS;
    }

    /* 3. 命令分发 */
    switch (cmd) {
        case CMD_GET_VERSION:
            return handle_get_version(params);
        case CMD_INIT:
            return handle_init(session, params);
        case CMD_PROCESS:
            return handle_process(session, params);
        default:
            return TEE_ERROR_BAD_PARAMETERS;
    }
}
```

**命令处理模式**:

```c
/* 常见命令 ID 定义 */
#define CMD_GET_VERSION    0x00000001
#define CMD_INIT           0x00000002
#define CMD_PROCESS        0x00000003
#define CMD_FINAL          0x00000004

/* 处理函数签名 */
static TEE_Result handle_get_version(TEE_Param params[])
{
    /* 验证输出参数 */
    if (params[1].memref.buffer == NULL ||
        params[1].memref.size < sizeof(uint32_t)) {
        return TEE_ERROR_BAD_PARAMETERS;
    }

    /* 返回版本号 */
    uint32_t* version = params[1].memref.buffer;
    *version = TA_VERSION;
    params[1].memref.size = sizeof(uint32_t);

    return TEE_SUCCESS;
}
```

#### 2.4 TA_CloseSessionEntryPoint

**功能**: 会话关闭入口，清理会话级资源。

**代码位置**: `sdk/src/TA/helloworld_demo/ta_demo.c:142-150`

```c
/**
 * TA_CloseSessionEntryPoint - 会话关闭入口
 * @session_context: 会话上下文（来自 OpenSession）
 *
 * 调用时机: CA 调用 TEEC_CloseSession() 时
 * 用途: 清理会话资源
 */
void TA_CloseSessionEntryPoint(void* session_context)
{
    my_session_t* session = (my_session_t*)session_context;

    if (session != NULL) {
        /* 1. 清理会话资源 */
        cleanup_session_resources(session);

        /* 2. 释放会话上下文 */
        TEE_Free(session);
    }
}
```

#### 2.5 TA_DestroyEntryPoint

**功能**: TA 实例析构函数，清理全局资源。

**代码位置**: `sdk/src/TA/helloworld_demo/ta_demo.c:152-158`

```c
/**
 * TA_DestroyEntryPoint - TA 实例析构函数
 *
 * 调用时机: TEE Framework 销毁 TA 实例时
 * 用途: 释放全局资源、关闭安全存储等
 */
void TA_DestroyEntryPoint(void)
{
    /* 1. 释放加密操作上下文 */
    if (crypto_context != NULL) {
        TEE_FreeOperation(crypto_context);
    }

    /* 2. 关闭安全存储 */
    /* TEE_CloseObject(...) */
}
```

---

## TA 配置详解

每个 TA 必须包含 `configs.xml` 文件，定义 TA 的基本信息和运行时属性。

**代码证据**: `sdk/build/TA_demo/configs.xml:1-25`

### 配置文件结构

```xml
<?xml version="1.0" encoding="utf-8"?>
<ConfigInfo>
  <TA_Basic_Info>
    <service_name>myta</service_name>
    <uuid>12345678-1234-1234-1234-123456789abc</uuid>
  </TA_Basic_Info>
  <TA_Manifest_Info>
    <instance_keep_alive>false</instance_keep_alive>
    <stack_size>8192</stack_size>
    <heap_size>81920</heap_size>
    <multi_session>false</multi_session>
    <single_instance>true</single_instance>
  </TA_Manifest_Info>
</ConfigInfo>
```

### 配置项详细说明

| 配置项 | 类型 | 必需 | 默认值 | 取值范围 | 说明 |
|-------|------|-----|--------|---------|------|
| `service_name` | String | ✅ | - | ≤64 字符， `[a-zA-Z0-9_-]+` | TA 名称，用于标识 |
| `uuid` | UUID | ✅ | - | 标准的 UUID 格式 | TA 唯一标识符 |
| `instance_keep_alive` | Bool | ❌ | false | true/false | 会话关闭后是否保留实例 |
| `stack_size` | Integer | ❌ | 8192 | 1024~1048576 | 每会话栈大小（字节） |
| `heap_size` | Integer | ❌ | 0 | 0~10485760 | TA 实例堆大小（字节） |
| `multi_session` | Bool | ❌ | false | false | 是否支持多会话（当前仅支持 false） |
| `single_instance` | Bool | ❌ | true | true | 是否单实例（当前仅支持 true） |

### UUID 生成

TA 的 UUID 必须全局唯一，推荐使用以下方式生成：

```bash
# 使用 Python 生成标准 UUID
python3 -c "import uuid; print(uuid.uuid4())"
```

或在线工具生成后转换为小横杠格式：

```
e3d37f4a-f24c-48d0-8884-3bdd6c44e988
```

### 配置解析

**证据**: `sdk/build/script/manifest.py:50-150`

配置解析由 `manifest.py` 脚本完成，关键处理逻辑：

| 处理阶段 | 函数 | 说明 |
|---------|------|------|
| XML 解析 | `parse_config()` | 解析 configs.xml |
| UUID 验证 | `validate_uuid()` | 验证 UUID 格式 |
| 参数校验 | `validate_config()` | 校验配置值范围 |
| Manifest 生成 | `generate_manifest()` | 生成 TA Manifest |

**配置错误处理**:

| 错误类型 | 错误码 | 说明 |
|---------|--------|------|
| UUID 格式错误 | `MANIFEST_ERR_INVALID_UUID` | UUID 格式不正确 |
| 栈大小超限 | `MANIFEST_ERR_STACK_OVERFLOW` | stack_size > 1048576 |
| 堆大小超限 | `MANIFEST_ERR_HEAP_OVERFLOW` | heap_size > 10485760 |
| 名称超长 | `MANIFEST_ERR_NAME_TOO_LONG` | service_name > 64 字符 |

---

## TA 开发步骤

### 1. 创建 TA 目录结构

在 `sdk/src/TA/` 目录下创建新目录，参考 `helloworld_demo` 的结构：

```
my_ta/
├── ta_myta.c          # TA 源码文件 (必须)
├── configs.xml        # TA 属性配置文件 (必须)
├── Makefile           # Make 构建文件 (可选，使用 Make 时)
├── CMakeLists.txt     # CMake 构建文件 (可选，使用 CMake 时)
└── build_ta.sh        # 一键编译脚本 (可选)
```

**代码证据**: `sdk/build/TA_demo/` (完整示例目录)

### 2. 编写 TA 代码

TA 必须实现以下 GP TEE 标准入口函数：

**代码证据**: `sdk/src/TA/helloworld_demo/ta_demo.c:50-158`

#### 2.1 TA_CreateEntryPoint

```c
/**
 * 功能: TA 实例构造函数
 * 调用时机: TEE Framework 创建 TA 实例时
 * 用途: 初始化 TA 全局资源、添加 CA 白名单等
 */
TEE_Result TA_CreateEntryPoint(void)
{
    // 初始化代码
    return TEE_SUCCESS;
}
```

#### 2.2 TA_OpenSessionEntryPoint

```c
/**
 * 功能: 会话打开入口
 * 调用时机: CA 请求创建会话时
 * 参数:
 *   - parm_type: 参数类型
 *   - params: 参数数组
 *   - session_context: 会话上下文指针
 */
TEE_Result TA_OpenSessionEntryPoint(uint32_t parm_type,
    TEE_Param params[PARAM_COUNT], void** session_context)
{
    (void)parm_type;
    (void)params;
    (void)session_context;
    return TEE_SUCCESS;
}
```

#### 2.3 TA_InvokeCommandEntryPoint

```c
/**
 * 功能: 命令调用入口
 * 调用时机: CA 发送命令时
 * 参数:
 *   - session_context: 会话上下文
 *   - cmd: 命令 ID
 *   - parm_type: 参数类型
 *   - params: 参数数组
 */
TEE_Result TA_InvokeCommandEntryPoint(void* session_context,
    uint32_t cmd, uint32_t parm_type, TEE_Param params[PARAM_COUNT])
{
    // 命令处理逻辑
    switch (cmd) {
        case CMD_XXX:
            // 处理命令
            break;
        default:
            return TEE_ERROR_BAD_PARAMETERS;
    }
    return TEE_SUCCESS;
}
```

#### 2.4 TA_CloseSessionEntryPoint

```c
/**
 * 功能: 会话关闭入口
 * 调用时机: CA 请求关闭会话时
 */
void TA_CloseSessionEntryPoint(void* session_context)
{
    (void)session_context;
    // 清理会话资源
}
```

#### 2.5 TA_DestroyEntryPoint

```c
/**
 * 功能: TA 实例析构函数
 * 调用时机: TEE Framework 销毁 TA 实例时
 */
void TA_DestroyEntryPoint(void)
{
    // 清理全局资源
}
```

### 3. 编写 TA 配置

**代码证据**: `sdk/build/TA_demo/configs.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ConfigInfo>
  <TA_Basic_Info>
    <service_name>myta</service_name>
    <uuid>12345678-1234-1234-1234-123456789abc</uuid>
  </TA_Basic_Info>
  <TA_Manifest_Info>
    <instance_keep_alive>false</instance_keep_alive>
    <stack_size>8192</stack_size>
    <heap_size>81920</heap_size>
    <multi_session>false</multi_session>
    <single_instance>true</single_instance>
  </TA_Manifest_Info>
</ConfigInfo>
```

#### 配置项说明

| 配置项 | 类型 | 说明 | 默认值 |
|--------|------|------|--------|
| `service_name` | String | TA 名称，不超过 64 字符，仅支持数字、字母、`_`、`-` | 必须指定 |
| `uuid` | UUID | TA 唯一标识符 | 必须指定 |
| `instance_keep_alive` | Bool | 会话关闭后是否保留 TA 实例 | false |
| `stack_size` | Integer | 每会话栈大小 (bytes) | 8192 |
| `heap_size` | Integer | TA 堆大小 (bytes) | 0 |
| `multi_session` | Bool | 是否支持多会话 | false |
| `single_instance` | Bool | 是否单实例 (仅支持 true) | true |

### 4. 编写 Makefile

TA 使用 Makefile 进行构建，SDK 提供了完整的 Makefile 模板。

**代码证据**: `sdk/build/TA_demo/Makefile:1-50`, `sdk/build/mk/common.mk:1-60`

#### Makefile 模板

```makefile
# ========================================
# TA Makefile 模板
# ========================================

# ----------------------------------------
# 1. 工具链配置 (64位 vs 32位)
# ----------------------------------------
# 64位 TA: TARGET_S_SARM64=y
# 32位 TA: 不设置或 TARGET_S_SARM64=n
ifeq ($(TARGET_S_SARM64),y)
    $(info "Building 64-bit TA")
    TOOLCHAIN_PREFIX := aarch64-linux-gnu-
else
    $(info "Building 32-bit TA")
    TOOLCHAIN_PREFIX := arm-linux-gnueabi-
endif

# ----------------------------------------
# 2. SDK 路径配置
# ----------------------------------------
TEE_SDK_ROOT ?= $(abspath $(dir $(lastword $(MAKEFILE_LIST)))/../../..)
-include $(TEE_SDK_ROOT)/sdk/build/mk/common.mk

# ----------------------------------------
# 3. 编译配置
# ----------------------------------------
include $(TEE_SDK_ROOT)/sdk/build/mk/common_flags.mk
include $(TEE_SDK_ROOT)/sdk/build/mk/security_features.mk

# ----------------------------------------
# 4. 源文件配置
# ----------------------------------------
# 添加 TA 源文件
TA_SOURCES += ta_myta.c
# 添加其他源文件（如果有）
# TA_SOURCES += crypto_utils.c
# TA_SOURCES += secure_storage.c

# ----------------------------------------
# 5. 链接配置
# ----------------------------------------
# 添加额外的链接标志（如果有）
# LDFLAGS += -lsyslib

# ----------------------------------------
# 6. 输出配置
# ----------------------------------------
# 固定输出文件名
TARGET := libcombine.so

include $(TEE_SDK_ROOT)/sdk/build/mk/common_llvm.mk
```

#### Makefile 变量说明

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TEE_SDK_ROOT` | SDK 根目录 | 自动推导 |
| `TA_SOURCES` | TA 源文件列表 | 必需设置 |
| `TARGET` | 输出目标文件名 | `libcombine.so` |
| `TARGET_S_SARM64` | 64位编译标志 | - |
| `CFLAGS` | C 编译标志 | 由 `common_flags.mk` 定义 |
| `LDFLAGS` | 链接标志 | 由 `common_llvm.mk` 定义 |

#### 构建系统包含层次

```
Makefile (用户编写)
    │
    ├── common.mk (SDK 通用配置)
    │   ├── common_gcc.mk (GCC 配置)
    │   └── common_llvm.mk (LLVM 配置)
    │
    ├── common_flags.mk (编译标志)
    │   ├── security_features.mk (安全特性)
    │   └── 架构特定标志
    │
    └── security_features.mk (安全编译选项)
```

### 5. 编译 TA

#### 方式一：一键编译脚本

SDK 提供了 `build_ta.sh` 脚本，自动化完成编译和签名流程。

```bash
# 1. 复制 build_ta.sh 到 TA 目录
cd sdk/src/TA/my_ta/
cp ../../build/TA_demo/build_ta.sh .

# 2. 执行一键编译
./build_ta.sh
```

**代码证据**: `sdk/build/TA_demo/build_ta.sh:1-100`

脚本执行流程：

| 阶段 | 操作 | 说明 |
|------|------|------|
| 1 | 环境检查 | 验证 LLVM 工具链、Python 依赖 |
| 2 | 配置解析 | 解析 `configs.xml` |
| 3 | 编译 | 调用 `make` 编译 TA |
| 4 | 打包 | 生成 Manifest 和签名 |
| 5 | 输出 | 生成 `.sec` 安装包 |

#### 方式二：手动编译

```bash
# 1. 设置环境变量
export TEE_SDK_ROOT=/path/to/tee_dev_kit
export PATH=$TEE_SDK_ROOT/prebuilts/clang/ohos/linux-x86_64/15.0.4/llvm/bin:$PATH

# 2. 进入 TA 目录
cd sdk/src/TA/my_ta/

# 3. 执行编译
make -f Makefile

# 4. 签名（如果需要）
python3 $TEE_SDK_ROOT/sdk/build/script/signtool_sec.py \
    --in_path ./ \
    --out_path ./output \
    --publicCfg $TEE_SDK_ROOT/sdk/build/config/config_ta_public.ini \
    --privateCfg $TEE_SDK_ROOT/sdk/build/config/config_tee_private_sample.ini
```

### 6. 编译产物说明

#### 中间产物

| 产物 | 文件名 | 说明 |
|------|--------|------|
| **目标文件** | `*.o` | 编译生成的目标文件 |
| **TA 镜像** | `libcombine.so` | 链接后的 TA 共享库 |
| **符号表** | `*.map` | 符号表文件（调试用） |

**代码证据**: `sdk/build/mk/common.mk:100-150`

#### 最终产物

| 产物 | 文件名 | 说明 |
|------|--------|------|
| **TA 安装包** | `{uuid}.sec` | 签名后的 TA 安装包 |

**产物路径**:
- 中间产物: `sdk/src/TA/my_ta/out/`
- 最终产物: `sdk/src/TA/my_ta/` 或指定输出目录

#### .sec 文件格式

`.sec` 文件是 TA 的安装包格式，包含以下组件：

```
{sec 文件}
├── TA 镜像头部
│   ├── magic number: 0x5A3C9A27
│   ├── 版本信息
│   └── 镜像大小
├── Manifest 区域
│   ├── service_name
│   ├── uuid
│   ├── stack_size
│   └── 权限配置
├── 签名区域
│   ├── hash 值 (SHA256)
│   └── RSA 签名
└── TA 镜像数据
    └── libcombine.so
```

**签名流程**:

```
libcombine.so → SHA256 Hash → RSA 私钥签名 → 追加到 .sec 文件
```

### 7. 签名 TA

编译生成 `libcombine.so` 后，需要使用签名工具生成安装包。

**代码证据**: `sdk/build/script/signtool_sec.py:1-200`

#### 签名命令

```bash
python3 $TEE_SDK_ROOT/sdk/build/script/signtool_sec.py \
    --in_path ./my_ta \
    --out_path ./output \
    --publicCfg $TEE_SDK_ROOT/sdk/build/config/config_ta_public.ini \
    --privateCfg $TEE_SDK_ROOT/sdk/build/config/config_tee_private_sample.ini
```

#### 签名参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--in_path` | TA 源目录路径 | 必需 |
| `--out_path` | 输出目录路径 | 必需 |
| `--publicCfg` | 公共配置文件路径 | 必需 |
| `--privateCfg` | 私有配置文件路径 | 必需 |
| `--key_path` | 自定义密钥路径 | 可选 |
| `--algo` | 签名算法 | RSA |
| `--hash` | Hash 算法 | SHA256 |

#### 签名配置文件

**公共配置** (`config_ta_public.ini`):

```ini
[TA]
uuid = e3d37f4a-f24c-48d0-8884-3bdd6c44e988
service_name = demo-ta
```

**私有配置** (`config_tee_private_sample.ini`):

```ini
[Sdefault]
sign_algo = RSA
key_length = 4096
hash_algo = SHA256
```

#### 签名验证

签名工具会自动验证：

| 验证项 | 说明 |
|-------|------|
| UUID 有效性 | 验证 UUID 格式 |
| 镜像完整性 | 验证 libcombine.so 完整性 |
| Manifest 有效性 | 验证配置项范围 |

签名成功后会生成 `{uuid}.sec` 文件，即 TA 安装包。

---

## CA 开发指南

### CA 调用流程

**代码证据**: `sdk/src/CA/helloworld_demo/ca_demo.c`

```c
#include <tee_client_api.h>

int main(void)
{
    TEEC_Context context;
    TEEC_Session session;
    TEEC_Operation operation;
    TEEC_Result result;

    // 1. 初始化 Context
    TEEC_InitializeContext(NULL, &context);

    // 2. 打开会话
    result = TEEC_OpenSession(&context, &session, &UUID,
        TEEC_LOGIN_PUBLIC, NULL, NULL, NULL);
    if (result != TEEC_SUCCESS) {
        // 处理错误
    }

    // 3. 发送命令
    operation.paramTypes = TEEC_PARAM_TYPES(
        TEEC_NONE, TEEC_NONE, TEEC_NONE, TEEC_MEMREF_TEMP_OUTPUT);
    operation.params[3].tmpref.buffer = buffer;
    operation.params[3].tmpref.size = buffer_size;

    result = TEEC_InvokeCommand(&session, CMD_GET_TA_VERSION,
        &operation, NULL);
    if (result != TEEC_SUCCESS) {
        // 处理错误
    }

    // 4. 关闭会话
    TEEC_CloseSession(&session);

    // 5. 销毁 Context
    TEEC_FinalizeContext(&context);

    return 0;
}
```

---

## 最佳实践

### 1. 参数校验

**代码证据**: `sdk/src/TA/helloworld_demo/ta_demo.c:109-126`

```c
// 使用 check_param_type 验证参数类型
if (!check_param_type(parm_type,
    TEE_PARAM_TYPE_NONE,
    TEE_PARAM_TYPE_NONE,
    TEE_PARAM_TYPE_NONE,
    TEE_PARAM_TYPE_MEMREF_OUTPUT)) {
    return TEE_ERROR_BAD_PARAMETERS;
}

// 验证输出缓冲区
if (params[OUT_BUFFER_INDEX].memref.buffer == NULL ||
    params[OUT_BUFFER_INDEX].memref.size == 0) {
    return TEE_ERROR_BAD_PARAMETERS;
}
```

### 2. 安全编码

- 使用安全字符串函数 (`strncpy_s` 代替 `strncpy`)
- 使用 `TEE_Malloc`/`TEE_Free` 分配内存
- 避免使用不安全的 API

### 3. CA 白名单管理

**代码证据**: `sdk/src/TA/helloworld_demo/ta_demo.c:58-70`

```c
// 在 TA_CreateEntryPoint 中添加 CA 白名单
const uint8_t hash[] = {
    0xca, 0x9f, 0x5e, 0xd7, ... // CA 调用者哈希
};
AddCaller_CA(hash, sizeof(hash));
```

---

## 继续阅读

### 推荐阅读路径

| 文档 | 阅读时间 | 目标 |
|------|---------|------|
| [01_Architecture.md](01_Architecture.md) | 15 分钟 | 理解 CA ↔ TEE 通信架构 |
| [03_Build_System.md](03_Build_System.md) | 20 分钟 | 深入理解构建系统 |
| [05_Examples.md](05_Examples.md) | 20 分钟 | 参考完整示例 |
| [04_Security_Review.md](04_Security_Review.md) | 30 分钟 | 安全最佳实践 |

### 问题排查

遇到问题？请参考 [06_Troubleshooting.md](06_Troubleshooting.md)

---

## 变更历史

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 1.0.0 | 2026-02-07 | 初始版本 |

---

*最后更新: 2026-02-07*
