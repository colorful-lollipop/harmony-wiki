# 附录：错误码参考

## 目的

本文档汇总 `telephony_core_service` 使用的所有错误码。

---

## 通用错误码

**文件**: `interfaces/innerkits/include/telephony_errors.h:42-91`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `TELEPHONY_ERR_SUCCESS` | 0 | 成功 |
| `TELEPHONY_ERR_FAIL` | ErrCodeOffset | 通用失败 |
| `TELEPHONY_ERR_ARGUMENT_MISMATCH` | +1 | 参数不匹配 |
| `TELEPHONY_ERR_ARGUMENT_INVALID` | +2 | 参数无效 |
| `TELEPHONY_ERR_ARGUMENT_NULL` | +3 | 参数为空 |
| `TELEPHONY_ERR_MEMCPY_FAIL` | +4 | 内存拷贝失败 |
| `TELEPHONY_ERR_MEMSET_FAIL` | +5 | 内存设置失败 |
| `TELEPHONY_ERR_STRCPY_FAIL` | +6 | 字符串拷贝失败 |
| `TELEPHONY_ERR_LOCAL_PTR_NULL` | +7 | 本地指针为空 |
| `TELEPHONY_ERR_PERMISSION_ERR` | +8 | 权限错误 |
| `TELEPHONY_ERR_DESCRIPTOR_MISMATCH` | +9 | 描述符不匹配 |
| `TELEPHONY_ERR_WRITE_DESCRIPTOR_TOKEN_FAIL` | +10 | 写入描述符令牌失败 |
| `TELEPHONY_ERR_WRITE_DATA_FAIL` | +11 | 写入数据失败 |
| `TELEPHONY_ERR_WRITE_REPLY_FAIL` | +12 | 写入回复失败 |
| `TELEPHONY_ERR_READ_DATA_FAIL` | +13 | 读取数据失败 |
| `TELEPHONY_ERR_IPC_CONNECT_STUB_FAIL` | +14 | IPC 连接失败 |
| `TELEPHONY_ERR_ADD_DEATH_RECIPIENT_FAIL` | +15 | 添加死亡监听失败 |
| `TELEPHONY_ERR_REGISTER_CALLBACK_FAIL` | +16 | 注册回调失败 |
| `TELEPHONY_ERR_CALLBACK_ALREADY_REGISTERED` | +17 | 回调已注册 |
| `TELEPHONY_ERR_UNINIT` | +18 | 未初始化 |
| `TELEPHONY_ERR_UNREGISTER_CALLBACK_FAIL` | +19 | 注销回调失败 |
| `TELEPHONY_ERR_SLOTID_INVALID` | +20 | 卡槽 ID 无效 |
| `TELEPHONY_ERR_SUBSCRIBE_BROADCAST_FAIL` | +21 | 订阅广播失败 |
| `TELEPHONY_ERR_PUBLISH_BROADCAST_FAIL` | +22 | 发布广播失败 |
| `TELEPHONY_ERR_STRTOINT_FAIL` | +23 | 字符串转整数失败 |
| `TELEPHONY_ERR_NO_SIM_CARD` | +24 | 无 SIM 卡 |
| `TELEPHONY_ERR_DATABASE_WRITE_FAIL` | +25 | 数据库写入失败 |
| `TELEPHONY_ERR_DATABASE_READ_FAIL` | +26 | 数据库读取失败 |
| `TELEPHONY_ERR_RIL_CMD_FAIL` | +27 | RIL 命令失败 |
| `TELEPHONY_ERR_UNKNOWN_NETWORK_TYPE` | +28 | 未知网络类型 |
| `TELEPHONY_ERR_ILLEGAL_USE_OF_SYSTEM_API` | +29 | 非法使用系统 API |
| `TELEPHONY_ERR_AIRPLANE_MODE_ON` | +30 | 飞行模式开启 |
| `TELEPHONY_ERR_NETWORK_NOT_IN_SERVICE` | +31 | 网络不在服务中 |
| `TELEPHONY_ERR_MMS_PS_NOT_ATTACHED` | +32 | MMS PS 未附着 |
| `TELEPHONY_ERR_MMS_FAIL_APN_INVALID` | +33 | APN 无效 |
| `TELEPHONY_ERR_MMS_FAIL_HTTP_ERROR` | +34 | HTTP 错误 |
| `TELEPHONY_ERR_MMS_FAIL_DATA_NETWORK_ERROR` | +35 | 数据网络错误 |
| `TELEPHONY_ERR_VCARD_FILE_INVALID` | +36 | vCard 文件无效 |
| `TELEPHONY_ERR_NOT_SUPPORT_ESIM` | +37 | 不支持 eSIM |
| `TELEPHONY_ERR_ESIM_GET_RESULT_TIMEOUT` | +38 | eSIM 获取结果超时 |
| `TELEPHONY_ERR_RAW_PARCEL_CALLBACK_TIMEOUT` | +39 | 原始 Parcel 回调超时 |
| `TELEPHONY_ERR_DATABASE_READ_EMPTY` | +40 | 数据库读取为空 |
| `TELEPHONY_ERR_POLICY_DISABLED` | +41 | 策略禁用 |
| `TELEPHONY_ERR_ARRAY_OUT_OF_BOUNDS` | +42 | 数组越界 |
| `TELEPHONY_ERR_CORE_SERVICE_NOT_SUPPORTED_ESIM` | +43 | 核心服务不支持 eSIM |

---

## JS API 错误码

**文件**: `frameworks/js/napi/napi_util.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERROR_NONE` | 0 | 无错误 |
| `ERROR_SERVICE_UNAVAILABLE` | 8300001 | 服务不可用 |
| `ERROR_NO_SIM_CARD` | 8300002 | 无 SIM 卡 |
| `ERROR_SLOT_ID_INVALID` | 8300003 | 卡槽 ID 无效 |
| `ERROR_PARAMETER_TYPE_INVALID` | 401 | 参数类型无效 |
| `ERROR_PARAMETER_VALUE_INVALID` | 402 | 参数值无效 |
| `ERROR_PERMISSION_DENIED` | 201 | 权限被拒绝 |
| `ERROR_SYSTEM_IN_PROGRESS` | 8300999 | 系统处理中 |

---

## SIM 状态码

**文件**: `interfaces/innerkits/include/sim_state_type.h`

| 状态 | 值 | 说明 |
|------|-----|------|
| `SIM_STATE_UNKNOWN` | -1 | 未知状态 |
| `SIM_STATE_NOT_PRESENT` | 0 | 无 SIM 卡 |
| `SIM_STATE_LOCKED_OUT` | 1 | 已锁定 |
| `SIM_STATE_READY` | 2 | 就绪 |
| `SIM_STATE_LOADED` | 3 | 已加载 |

## 锁类型

| 类型 | 值 | 说明 |
|------|-----|------|
| `PIN_LOCK` | 0 | PIN 锁 |
| `PIN2_LOCK` | 1 | PIN2 锁 |
| `PUK_LOCK` | 2 | PUK 锁 |
| `PUK2_LOCK` | 3 | PUK2 锁 |
| `PPHN_LOCK` | 4 | 网络个人化锁 |

---

## 网络注册状态

**文件**: `interfaces/innerkits/include/network_search_types.h`

| 状态 | 值 | 说明 |
|------|-----|------|
| `REG_STATE_UNKNOWN` | 0 | 未知 |
| `REG_STATE_NOT_REG` | 1 | 未注册 |
| `REG_STATE_HOME_ONLY` | 2 | 仅本地网络 |
| `REG_STATE_ROAMING` | 3 | 漫游中 |

---

## 射频技术类型

| 制式 | 值 | 说明 |
|------|-----|------|
| `RADIO_TECHNOLOGY_UNKNOWN` | 0 | 未知 |
| `RADIO_TECHNOLOGY_GSM` | 1 | GSM |
| `RADIO_TECHNOLOGY_WCDMA` | 2 | WCDMA |
| `RADIO_TECHNOLOGY_LTE` | 3 | LTE |
| `RADIO_TECHNOLOGY_NR` | 4 | 5G NR |

---

## 相关链接

- [N-API 接口](../03_NAPI_API.md)
- [内部 API](../04_Inner_API.md)
