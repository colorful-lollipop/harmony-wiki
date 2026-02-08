# 错误码

> **注意**: 本文档仅包含服务端常见错误码。完整错误码请参考各模块源码。

---

## 通用错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| 0 | `Status::SUCCESS` | 成功 |
| -1 | `Status::ERROR` | 通用错误 |

---

## KVDB 错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| -1 | `DB_ERROR` | 数据库错误 |
| -2 | `DB_NOT_FOUND` | 数据库不存在 |
| -3 | `DB_ALREADY_EXISTS` | 数据库已存在 |
| -4 | `DB_INVALID_ARGS` | 无效参数 |

**证据**: `service/kvdb/kvdb_service_impl.cpp` 及相关头文件

---

## RDB 错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| -1 | `INVALID_ARGS` | 无效参数 |
| -2 | `NOT_FOUND` | 未找到 |
| -3 | `ALREADY_EXISTS` | 已存在 |
| -4 | `EXECUTE_FAIL` | 执行失败 |

**证据**: `service/rdb/rdb_service_impl.cpp` 及相关头文件

---

## 权限错误码

| 错误码 | 常量名 | 说明 |
|-------|--------|------|
| -401 | `CheckSyncPermission` | 无同步权限 |
| -402 | `INVALID_TOKEN_ID` | 无效 Token ID |

**证据**: `service/permission/include/permission_validator.h`

---

## IPC 错误码

| 错误码 | 说明 |
|-------|------|
| 0 | 成功 |
| 负值 | 各模块定义 |

**证据**: `app/src/kvstore_data_service_stub.h`

---

## 错误码范围对应表

| 模块 | 错误码范围 |
|-----|----------|
| 通用 | -1 |
| KVDB | -1 ~ -100 |
| RDB | -1 ~ -100 |
| 权限 | -401 ~ -500 |
| IPC | 0, 负值 |

---

> **提示**: 完整错误码定义请查看各模块头文件中的 `Status` 或 `ErrorCode` 枚举。
