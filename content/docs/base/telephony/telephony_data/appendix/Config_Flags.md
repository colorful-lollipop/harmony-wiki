# 配置标志与宏定义

## B.1 构建配置标志

### B.1.1 GN 构建变量

| 变量名 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `use_cfi` | boolean | false | 启用 Control Flow Integrity |
| `enable_heap_profiler` | boolean | false | 启用堆内存分析 |
| `enable_allocator` | string | "system" | 内存分配器选择 |
| `debuggable` | boolean | false | 调试模式开关 |

### B.1.2 编译器标志

| 标志 | 类别 | 描述 |
|------|------|------|
| `-Wall` | 警告 | 启用所有警告 |
| `-Wextra` | 警告 | 启用额外警告 |
| `-Werror` | 警告 | 将警告视为错误 |
| `-fstack-protector-strong` | 安全 | 堆栈保护 |
| `-fsanitize=cfi` | 安全 | CFI 防护 (可选) |
| `-Wl,-z,relro` | 安全 | 只读重定位 |
| `-Wl,-z,now` | 安全 | 立即绑定符号 |
| `-fno-omit-frame-pointer` | 调试 | 保留帧指针 |

---

## B.2 功能开关

### B.2.1 编译时功能标志

| 宏定义 | 定义位置 | 描述 |
|--------|----------|------|
| `DATA_STORAGE_LOG_DOMAIN` | `data_storage_log_wrapper.h` | 日志领域 ID |
| `DATA_STORAGE_LOG_TAG` | `data_storage_log_wrapper.h` | 日志标签 |

### B.2.2 日志级别

| 级别 | 值 | 用途 |
|------|-----|------|
| `DEBUG` | 0 | 调试信息 |
| `INFO` | 1 | 普通信息 |
| `WARN` | 2 | 警告信息 |
| `ERROR` | 3 | 错误信息 |
| `FATAL` | 4 | 致命错误 |

**代码证据**: `common/include/data_storage_log_wrapper.h`

```cpp
#define DATA_STORAGE_LOG_DOMAIN 0xD002900

#define DATA_STORAGE_LOG_TAG "TelephonyData"

#define DATA_STORAGE_LOGD(...) \
    HiLogPrintLog(DOMAIN, LEVEL_DEBUG, TAG, __VA_ARGS__)

#define DATA_STORAGE_LOGI(...) \
    HiLogPrintLog(DOMAIN, LEVEL_INFO, TAG, __VA_ARGS__)

#define DATA_STORAGE_LOGW(...) \
    HiLogPrintLog(DOMAIN, LEVEL_WARN, TAG, __VA_ARGS__)

#define DATA_STORAGE_LOGE(...) \
    HiLogPrintLog(DOMAIN, LEVEL_ERROR, TAG, __VA_ARGS__)
```

---

## B.3 权限常量

### B.3.1 权限定义

| 常量 | 值 | 保护级别 | 用途 |
|------|-----|----------|------|
| `SET_TELEPHONY_STATE` | `"ohos.permission.SET_TELEPHONY_STATE"` | system_basic | 修改电话状态 |
| `GET_TELEPHONY_STATE` | `"ohos.permission.GET_TELEPHONY_STATE"` | system_basic | 查询电话状态 |
| `READ_MESSAGES` | `"ohos.permission.READ_MESSAGES"` | system_basic | 读取短信 |

**代码证据**: `common/include/permission_util.h`

```cpp
namespace Permission {
    static constexpr const char *SET_TELEPHONY_STATE = 
        "ohos.permission.SET_TELEPHONY_STATE";
    static constexpr const char *GET_TELEPHONY_STATE = 
        "ohos.permission.GET_TELEPHONY_STATE";
    static constexpr const char *READ_MESSAGES = 
        "ohos.permission.READ_MESSAGES";
}
```

---

## B.4 错误码定义

### B.4.1 统一错误码

| 错误码 | 常量 | 描述 |
|--------|------|------|
| `0` | `ERR_OK` | 操作成功 |
| `-1` | `ERR_UNKNOWN` | 未知错误 |
| `-2` | `ERR_PERMISSION` | 权限错误 |
| `-3` | `ERR_INVALID_PARAM` | 参数无效 |
| `-4` | `ERR_DATABASE` | 数据库错误 |
| `-5` | `ERR_NOT_FOUND` | 数据不存在 |

**代码证据**: `common/include/data_storage_errors.h`

```cpp
class DataStorageErrors {
public:
    static constexpr int32_t ERR_OK = 0;
    static constexpr int32_t ERR_UNKNOWN = -1;
    static constexpr int32_t ERR_PERMISSION = -2;
    static constexpr int32_t ERR_INVALID_PARAM = -3;
    static constexpr int32_t ERR_DATABASE = -4;
    static constexpr int32_t ERR_NOT_FOUND = -5;
};
```

---

## B.5 DataShare URI 常量

### B.5.1 URI 定义

| 模块 | URI | 常量定义位置 |
|------|-----|--------------|
| SIM | `datashare:///com.ohos.simability` | `sim_ability.h` |
| SMS/MMS | `datashare:///com.ohos.smsmmsability` | `sms_mms_ability.h` |
| PDP | `datashare:///com.ohos.pdpprofileability` | `pdp_profile_ability.h` |
| OpKey | `datashare:///com.ohos.opkeyability` | `opkey_ability.h` |
| GlobalParams | `datashare:///com.ohos.globalparamsability` | `global_params_ability.h` |

---

## B.6 数据库表结构标志

### B.6.1 表名常量

| 表名 | 用途 |
|------|------|
| `sim_info` | SIM 卡信息表 |
| `sms_mms_info` | 短信/多媒体消息表 |
| `pdp_profile` | PDP/APN 配置表 |
| `opkey_info` | 运营商密钥表 |
| `global_params` | 全局参数表 |

### B.6.2 列名字段常量

**SIM 表字段** (`interfaces/innerkits/include/sim_data.h`):

| 字段 | 类型 | 描述 |
|------|------|------|
| `SIM_ID` | INTEGER | 主键 |
| `ICC_ID` | TEXT | ICCID |
| `CARD_ID` | INTEGER | 卡槽 ID |
| `SLOT_INDEX` | INTEGER | 槽位索引 |
| `SHOW_NAME` | TEXT | 显示名称 |
| `PHONE_NUMBER` | TEXT | 电话号码 |

**SMS/MMS 表字段** (`interfaces/innerkits/include/sms_mms_data.h`):

| 字段 | 类型 | 描述 |
|------|------|------|
| `MSG_ID` | INTEGER | 主键 |
| `SENDER_NUMBER` | TEXT | 发送方 |
| `RECEIVER_NUMBER` | TEXT | 接收方 |
| `MSG_CONTENT` | TEXT | 消息内容 |
| `MSG_TITLE` | TEXT | 标题 |
| `GROUP_ID` | INTEGER | 会话组 ID |

---

## B.7 特征标志使用场景

### B.7.1 条件编译

```cpp
// 使用 CFI 时的特殊处理
#if defined(USE_CFI)
    // CFI 相关代码
#endif

// 调试模式下的额外检查
#if DEBUGGABLE
    // 调试断言
#endif
```

### B.7.2 运行时特性检测

| 检测项 | 用途 |
|--------|------|
| CFI 支持 | 控制流完整性检查 |
| 堆分析器 | 内存分配追踪 |
| 分配器类型 | 选择内存分配策略 |

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 构建系统 | [06_Build](06_Build.md) |
| 对外 API | [04_DataShare_API](04_DataShare_API.md) |
| 安全评审 | [07_Security](07_Security.md) |

---

*最后更新: 2024-02-06*
