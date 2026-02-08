# 关键配置与 Feature Flags

## 目的

本文档记录 ability_base 组件的关键配置选项、编译标志、Feature Flags 和常量定义，帮助开发者理解组件的可配置性和编译选项。

---

## 1. GN 构建配置

### 1.1 路径变量（ability_base.gni）

| 变量名 | 值 | 用途 |
|--------|-----|------|
| `ability_base_path` | `//foundation/ability/ability_base` | 组件根路径 |
| `ability_base_innerapi_path` | `${ability_base_path}/interfaces/inner_api` | Inner API 路径 |
| `ability_base_kits_native_path` | `${ability_base_path}/interfaces/kits/native` | Native Kits 路径 |
| `ability_base_ndk_path` | `${ability_base_path}/interfaces/kits/c` | NDK 路径 |
| `base_global_innerapi_path` | `//base/global/resource_management/interfaces/inner_api` | 全局资源管理路径 |
| `base_fuzz_output_path` | `ability_base/ability_base` | Fuzz 测试输出路径 |
| `ipc_native_path` | `//foundation/communication/ipc/ipc/native` | IPC Native 路径 |

**证据位置**：`ability_base.gni:14-22`

### 1.2 日志标签（ABILITYBASE_LOG_TAG）

| Target | 标签值 | 定义位置 |
|--------|---------|----------|
| configuration | `"Configuration"` | `BUILD.gn:96` |
| want | `"Want"` | `BUILD.gn:162` |

**用途**：区分不同模块的日志输出

**代码示例**：
```cpp
#define ABILITYBASE_LOG_TAG "Want"
ABILITYBASE_LOG_INFO("Want created successfully");
```

---

## 2. 架构相关 Flags

### 2.1 BINDER_IPC_32BIT

| 属性 | 值 |
|------|-----|
| **适用条件** | `target_cpu == "arm"` |
| **用途** | 启用 32 位 ARM IPC 支持 |
| **影响范围** | base, configuration, zuri, want, string_utils, extractortool, extractresourcemanager, ability_base_want |

**定义位置**：
- `BUILD.gn:99` (configuration)
- `BUILD.gn:120` (zuri)
- `BUILD.gn:159` (want)
- `BUILD.gn:197` (string_utils)
- `BUILD.gn:317` (extractortool)
- `BUILD.gn:358` (extractresourcemanager)
- `BUILD.gn:398` (ability_base_want)

**GN 配置**：
```gn
if (target_cpu == "arm") {
    cflags += [ "-DBINDER_IPC_32BIT" ]
}
```

### 2.2 平台特定 Flags

| Flag | 适用条件 | 值 | 用途 |
|------|----------|-----|------|
| `WINDOWS_PLATFORM` | `is_mingw` | - | Windows 平台标识 |
| `MAC_PLATFORM` | `!is_mingw` | - | macOS 平台标识 |

**定义位置**：`BUILD.gn:307-311` (string_utils)

**GN 配置**：
```gn
if (is_mingw) {
    defines = [ "WINDOWS_PLATFORM" ]
} else {
    defines = [ "MAC_PLATFORM" ]
}
```

---

## 3. 编译选项

### 3.1 异常支持（-fexceptions）

| 适用范围 | 选项 | 说明 |
|----------|------|------|
| base, configuration, zuri, want, extractortool, ability_base_want | `-fexceptions` | 启用 C++ 异常支持 |

**定义位置**：
- `BUILD.gn:31` (base_exceptions_config)
- `BUILD.gn:81` (configuration_exceptions_config)
- `BUILD.gn:125` (zuri_exceptions)
- `BUILD.gn:166` (want_exceptions_config)
- `BUILD.gn:210` (extractortool exceptions)

**GN 配置**：
```gn
config("base_exceptions_config") {
    cflags_cc = [ "-fexceptions" ]
}
```

### 3.2 严格编译警告

| 适用范围 | 选项 | 说明 |
|----------|------|------|
| want | `-Werror,-Wfloat-equal` | 浮点相等比较触发警告并视为错误 |

**定义位置**：`BUILD.gn:161`

**GN 配置**：
```gn
cflags = [ "-Werror,-Wfloat-equal" ]
```

**理由**：浮点数的精确相等比较通常是不安全的，应使用 epsilon 比较。

### 3.3 分支保护（pac_ret）

| 适用范围 | 保护类型 | 说明 |
|----------|----------|------|
| 所有 ohos_shared_library | `pac_ret` | 返回地址保护 |

**定义位置**：所有生产 target 的 `branch_protector_ret` 属性

**GN 配置**：
```gn
ohos_shared_library("base") {
    branch_protector_ret = "pac_ret"
}
```

---

## 4. Want Flags 常量

### 4.1 URI 权限 Flags

| 常量名 | 值 | 说明 | 证据位置 |
|----------|-----|------|----------|
| `FLAG_AUTH_READ_URI_PERMISSION` | 0x00000001 | URI 读取权限 | `want.h:39` |
| `FLAG_AUTH_WRITE_URI_PERMISSION` | 0x00000002 | URI 写入权限 | `want.h:43` |
| `FLAG_AUTH_PERSISTABLE_URI_PERMISSION` | 0x00000040 | URI 持久权限 | `want.h:63` |
| `FLAG_AUTH_PREFIX_URI_PERMISSION` | 0x00000080 | URI 前缀权限 | `want.h:67` |

### 4.2 Ability 启动 Flags

| 常量名 | 值 | 说明 | 证据位置 |
|----------|-----|------|----------|
| `FLAG_ABILITY_FORWARD_RESULT` | 0x00000004 | 返回结果到源 Ability | `want.h:47` |
| `FLAG_ABILITY_CONTINUATION` | 0x00000008 | 支持跨设备迁移 | `want.h:51` |
| `FLAG_NOT_OHOS_COMPONENT` | 0x00000010 | 非 OHOS 组件 | `want.h:55` |
| `FLAG_ABILITY_FORM_ENABLED` | 0x00000020 | Ability 卡片支持 | `want.h:59` |
| `FLAG_START_FOREGROUND_ABILITY` | 0x00000200 | 前台启动 Service Ability | `want.h:76` |
| `FLAG_INSTALL_ON_DEMAND` | 0x00000800 | 按需安装 | `want.h:86` |
| `FLAG_ABILITY_CONTINUATION_REVERSIBLE` | 0x00000400 | 可逆延续 | `want.h:81` |
| `FLAG_ABILITY_ON_COLLABORATE` | 0x00002000 | 协作请求生命周期回调 | `want.h:94` |
| `FLAG_ABILITY_NEW_MISSION` | 0x10000000 | 新建任务 | `want.h:110` |
| `FLAG_START_WITHOUT_TIPS` | 0x40000000 | 启动不显示提示 | `want.h:119` |
| `FLAG_INSTALL_WITH_BACKGROUND_MODE` | 0x80000000 | 后台模式安装 | `want.h:102` |

**证据位置**：`interfaces/kits/native/want/include/want.h:39-119`

### 4.3 Action 常量（部分）

| 常量名 | 值 | 说明 | 证据位置 |
|----------|-----|------|----------|
| `ACTION_PLAY` | `"ohos.action.play"` | 播放动作 | `want.h:846` |
| `ACTION_HOME` | `"ohos.action.home"` | 主页动作 | `want.h:847` |

**证据位置**：`interfaces/kits/native/want/include/want.h:845-852`

### 4.4 Entity 常量（部分）

| 常量名 | 值 | 说明 | 证据位置 |
|----------|-----|------|----------|
| `ENTITY_HOME` | `"entity.home"` | 主页实体 | `want.h:850` |
| `ENTITY_VIDEO` | `"entity.video"` | 视频实体 | `want.h:851` |
| `ENTITY_MUSIC` | `"entity.music"` | 音乐实体 | `want.h:853` |
| `ENTITY_EMAIL` | `"entity.email"` | 邮件实体 | `want.h:854` |
| `ENTITY_CONTACTS` | `"entity.contacts"` | 联系人实体 | `want.h:855` |
| `ENTITY_BROWSER` | `"entity.browser"` | 浏览器实体 | `want.h:857` |

**证据位置**：`interfaces/kits/native/want/include/want.h:849-861`

### 4.5 保留参数键（部分）

| 参数键 | 说明 | 证据位置 |
|--------|------|----------|
| `PARAM_RESV_WINDOW_MODE` | 窗口模式 | `want.h:867` |
| `PARAM_RESV_DISPLAY_ID` | 显示器 ID | `want.h:868` |
| `PARAM_RESV_CALLER_TOKEN` | 调用者 Token | `want.h:879` |
| `PARAM_RESV_CALLER_BUNDLE_NAME` | 调用者包名 | `want.h:880` |
| `PARAM_RESV_CALLER_UID` | 调用者 UID | `want.h:885` |
| `PARAM_RESV_CALLER_PID` | 调用者 PID | `want.h:886` |
| `PARAM_RESV_FOR_RESULT` | 返回结果标记 | `want.h:888` |

**证据位置**：`interfaces/kits/native/want/include/want.h:867-921`

---

## 5. WantParams 类型常量

### 5.1 值类型枚举

| 类型常量名 | 值 | 对应 C++ 类型 | 证据位置 |
|------------|-----|-------------|----------|
| `VALUE_TYPE_NULL` | -1 | null | `want_params.cpp:204` |
| `VALUE_TYPE_BOOLEAN` | 1 | bool | `want_params.cpp:205` |
| `VALUE_TYPE_BYTE` | 2 | int8_t | `want_params.cpp:206` |
| `VALUE_TYPE_CHAR` | 3 | zchar (32-bit char) | `want_params.cpp:207` |
| `VALUE_TYPE_SHORT` | 4 | int16_t | `want_params.cpp:208` |
| `VALUE_TYPE_INT` | 5 | int32_t | `want_params.cpp:209` |
| `VALUE_TYPE_LONG` | 6 | int64_t | `want_params.cpp:210` |
| `VALUE_TYPE_FLOAT` | 7 | float | `want_params.cpp:211` |
| `VALUE_TYPE_DOUBLE` | 8 | double | `want_params.cpp:212` |
| `VALUE_TYPE_STRING` | 9 | std::string | `want_params.cpp:213` |
| `VALUE_TYPE_ARRAY` | 102 | IArray | `want_params.cpp:214` |
| `VALUE_TYPE_WANTPARAMS` | 101 | WantParams (嵌套) | `want_params.cpp:215` |
| `VALUE_TYPE_FD` | 103 | int32_t (文件描述符) | `want_params.cpp:216` |
| `VALUE_TYPE_REMOTE_OBJECT` | 104 | IRemoteObject | `want_params.cpp:217` |

**证据位置**：`interfaces/kits/native/want/src/want_params.cpp:204-233`

### 5.2 大小限制常量

| 常量名 | 值 | 说明 | 证据位置 |
|--------|-----|------|----------|
| `MAX_RECURSION_DEPTH` | 100 | WantParams 递归深度限制 | `want_params.cpp:204` |
| `maxAllowedSize` | 1024 | 数组最大元素数 | `want_params.cpp:1305` |
| `maxAllowedSize` | 100MB (104857600 bytes) | Buffer 最大大小 | `want_params.cpp:1581` |

**证据位置**：`interfaces/kits/native/want/src/want_params.cpp`

---

## 6. Configuration 常量

### 6.1 连接符号

| 常量名 | 值 | 用途 | 证据位置 |
|--------|-----|------|----------|
| `CONNECTION_SYMBOL` | `"#"` | displayId 和 key 之间的分隔符 | `configuration.h:31` |
| `EMPTY_STRING` | `""` | 空字符串常量 | `configuration.h:32` |

### 6.2 配置键前缀

| 键前缀 | 值 | 说明 | 证据位置 |
|--------|-----|------|----------|
| `APPLICATION_DIRECTION` | `"ohos.application.direction"` | 应用方向 | `configuration.h:33` |
| `APPLICATION_DENSITYDPI` | `"ohos.application.densitydpi"` | 显示密度 | `configuration.h:34` |
| `APPLICATION_DISPLAYID` | `"ohos.application.displayid"` | 显示器 ID | `configuration.h:35` |
| `APPLICATION_FONT` | `"ohos.application.font"` | 字体 | `configuration.h:36` |

### 6.3 配置值常量

| 配置 | 可选值 | 证据位置 |
|------|--------|----------|
| Color Mode | `"light"`, `"dark"`, `"auto"` | `configuration.h:38-40` |
| Device Type | `"default"` | `configuration.h:41` |
| Direction | `"vertical"`, `"horizontal"` | `configuration.h:42-43` |

**证据位置**：`interfaces/kits/native/configuration/include/configuration.h:31-51`

---

## 7. SessionInfo 常量

### 7.1 CallToState 枚举

| 常量名 | 值 | 说明 | 证据位置 |
|--------|-----|------|----------|
| `CallToState::UNKNOW` | 0 | 未知状态 | `session_info.h:CallToState` |
| `CallToState::FOREGROUND` | 1 | 前台 | `session_info.h:CallToState` |
| `CallToState::BACKGROUND` | 2 | 后台 | `session_info.h:CallToState` |

### 7.2 UIExtensionUsage 枚举

| 常量名 | 值 | 说明 | 证据位置 |
|--------|-----|------|----------|
| `UIExtensionUsage::MODAL` | 0 | 模态对话框 | `session_info.h:57` |
| `UIExtensionUsage::EMBEDDED` | 1 | 嵌入式 | `session_info.h:57` |
| `UIExtensionUsage::CONSTRAINED_EMBEDDED` | 2 | 约束嵌入式 | `session_info.h:57` |
| `UIExtensionUsage::PRE_VIEW_EMBEDDED` | 3 | 预览嵌入式 | `session_info.h:57` |

**证据位置**：`interfaces/kits/native/session_info/include/session_info_constants.h`

---

## 8. ZipFile 常量

### 8.1 ZIP 签名常量

| 常量名 | 值（hex） | 说明 | 证据位置 |
|--------|-----------|------|----------|
| `EOCD_SIGNATURE` | 0x06054b50 | 中央目录结束记录签名 | `zip_file.cpp` |
| `CENTRAL_SIGNATURE` | 0x02014b50 | 中央目录文件头签名 | `zip_file.cpp` |
| `DATA_DESC_SIGNATURE` | 0x08074b50 | 数据描述符签名 | `zip_file.cpp` |
| `LOCAL_HEADER_SIGNATURE` | 0x04034b50 | 本地文件头签名 | `zip_file.cpp` |

**证据位置**：`interfaces/kits/native/extractortool/src/zip_file.cpp` (签名常量定义)

### 8.2 CacheMode 枚举

| 模式 | 说明 | 证据位置 |
|------|------|----------|
| SAFE_ABC | 安全映射模式（避免内存拷贝） | `zip_file.h:164-168` |

**证据位置**：`interfaces/kits/native/extractortool/include/zip_file.h`

---

## 9. 安全相关配置

### 9.1 ability_base_want Sanitize 选项

| 选项 | 值 | 说明 | 证据位置 |
|------|-----|------|----------|
| `integer_overflow` | true | 整数溢出检查 | `BUILD.gn:425-428` |
| `ubsan` | true | 未定义行为检查 | `BUILD.gn:426` |
| `boundary_sanitize` | true | 边界检查 | `BUILD.gn:427` |
| `cfi` | true | 控制流完整性 | `BUILD.gn:428` |
| `cfi_cross_dso` | true | 跨 DSO CFI | `BUILD.gn:429` |
| `cfi_vcall_icall_only` | true | 仅虚调用_icall | `BUILD.gn:430` |

**适用 Target**：`ability_base_want` (C NDK)

**证据位置**：`BUILD.gn:424-433`

### 9.2 大小限制

| 限制 | 值 | 适用范围 | 证据位置 |
|------|-----|----------|----------|
| WantParams 数组最大元素 | 1024 | WantParams | `want_params.cpp:1305` |
| WantParams Buffer 最大大小 | 100MB | WantParams | `want_params.cpp:1581` |
| WantParams 递归最大深度 | 100 | WantParams | `want_params.cpp:204` |

---

## 10. InnerAPI 标签

### 10.1 标签定义

| 标签 | 含义 | 说明 |
|------|------|------|
| `platformsdk` | 平台 SDK | 对应用开发者公开 |
| `sasdk` | 系统 Ability SDK | 对系统 Ability 开发者公开 |
| `platformsdk_indirect` | 间接平台 SDK | 通过其他组件间接导出 |
| `chipsetsdk_indirect` | 芯片 SDK 间接 | 对芯片厂商间接导出 |
| `ndk` | Native Development Kit | 对 NDK 开发者公开 |

### 10.2 Target 标签映射

| Target | 标签 | 证据位置 |
|--------|------|----------|
| base | `platformsdk`, `sasdk` | `BUILD.gn:65-68` |
| configuration | `platformsdk` | `BUILD.gn:110` |
| zuri | `platformsdk`, `sasdk` | `BUILD.gn:145-148` |
| want | `platformsdk`, `sasdk` | `BUILD.gn:229-232` |
| view_data | `platformsdk_indirect` | `BUILD.gn:262` |
| session_info | `platformsdk_indirect` | `BUILD.gn:295` |
| string_utils | `chipsetsdk_indirect`, `platformsdk_indirect` | `BUILD.gn:323-326` |
| extractortool | `chipsetsdk_indirect`, `platformsdk_indirect` | `BUILD.gn:377-380` |
| extractresourcemanager | `platformsdk_indirect` | `BUILD.gn:406` |
| ability_base_want | `ndk` | `BUILD.gn:455` |

---

## 11. 配置文件总结

### 11.1 配置文件清单

| 文件 | 作用 | 关键内容 |
|------|------|----------|
| `BUILD.gn` | 主构建配置 | Target 定义、依赖、Flags |
| `ability_base.gni` | 构建变量 | 路径变量 |
| `bundle.json` | 组件元数据 | 组件信息、依赖、导出接口 |

### 11.2 配置层次

```
系统配置
    ↓
bundle.json (组件级配置）
    ↓
ability_base.gni (路径变量）
    ↓
BUILD.gn (Target 配置)
    ↓
Config (include_dirs, cflags, defines)
    ↓
编译选项和 Feature Flags
```

---

## 12. 调整配置

### 12.1 修改日志标签

**场景**：需要区分不同模块的日志

**方法**：修改 `BUILD.gn` 中的 `ABILITYBASE_LOG_TAG` 定义

```gn
# 配置模块
defines = [ "ABILITYBASE_LOG_TAG = \"Configuration\"" ]
```

### 12.2 启用/禁用 Feature Flags

**场景**：为特定架构或平台启用功能

**方法**：修改 `BUILD.gn` 中的条件编译

```gn
# 启用 32 位 ARM IPC
if (target_cpu == "arm") {
    cflags += [ "-DBINDER_IPC_32BIT" ]
}
```

### 12.3 调整大小限制

**场景**：根据设备内存限制调整 WantParams 大小限制

**方法**：修改源代码中的常量定义

```cpp
// want_params.cpp
constexpr int maxAllowedSize = 512;  // 降低到 512
```

---

## 相关跳转

- ⚙️ **GN 构建**：[05_GN_Build.md](05_GN_Build.md)
- 🔒 **安全评审**：[06_Security_Review.md](06_Security_Review.md)

---

**返回导航**：[SUMMARY.md](SUMMARY.md)
