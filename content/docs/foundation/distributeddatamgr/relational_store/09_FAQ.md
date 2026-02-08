# 常见问题与定位路径

## 目的

本文档收集 relational_store 组件在构建、运行、调试过程中的常见问题及解决方案。

## 适用范围

- 构建相关问题和解决方法
- 运行时问题定位
- 性能优化建议
- 调试技巧

## 关键结论

### 1. 构建问题

#### 1.1 GN 构建错误

| 问题 | 错误信息 | 原因 | 解决方案 | 证据 |
|------|---------|------|----------|------|
| **找不到头文件** | "No such file or directory" | 路径配置错误或未生成 | 1. 检查 `relational_store.gni` 中的路径变量<br/>2. 运行 `gn gen --check-system` 验证依赖 | `relational_store.gni` |
| **未定义符号** | "undefined reference to" | 缺少依赖或头文件 | 1. 确认 bundle.json 中声明了所有依赖<br/>2. 检查 BUILD.gn 的 deps 和 external_deps | `interfaces/inner_api/rdb/BUILD.gn` |
| **重复定义** | "redefinition of" | 头文件包含重复 | 1. 检查 include_dirs 中是否有重复路径<br/>2. 使用头文件保护宏 | 多个头文件 |
| **类型不匹配** | "cannot convert" | C++ 标准不一致 | 1. 统一使用 C++17 标准<br/>2. 检查跨平台代码的编译标志 | `frameworks/js/napi/relationalstore/BUILD.gn:103` |

#### 1.2 NDK 编译错误

| 问题 | 错误信息 | 原因 | 解决方案 | 证据 |
|------|---------|------|----------|------|
| **NDK 头文件缺失** | "relational_store.h: No such file" | 未安装 NDK 或路径错误 | 1. 安装 OpenHarmony NDK<br/>2. 检查 CPATH 环境变量 | `interfaces/ndk/BUILD.gn` |
| **符号未找到** | "undefined reference: OH_RdbStore" | 未链接 native_rdb_ndk 库 | 1. 检查 pkg-config 配置<br/>2. 添加 -lnative_rdb_ndk 链接选项 | `interfaces/ndk/BUILD.gn` |

### 2. 运行时问题

#### 2.1 数据库打开失败

| 错误码 | 错误消息 | 原因 | 解决方案 | 证据 |
|--------|---------|------|----------|------|
| **RDB_E_INVALID_DB** | "Invalid database" | 数据库文件损坏或不存在 | 1. 使用 `Restore()` 从备份恢复<br/>2. 检查文件权限 | `rdb_errno.h` |
| **RDB_E_CANNOT_OPEN_ERR** | "Cannot open database" | 路径错误或权限问题 | 1. 验证数据库路径在应用沙箱内<br/>2. 检查存储权限（READ、WRITE） | `rdb_store_config.cpp` |
| **加密失败** | "Encryption failed" | 密钥不匹配或 HUKS 不可用 | 1. 检查密钥文件完整性<br/>2. 确认设备支持硬件加密 | `rdb_security_manager.cpp` |

#### 2.2 SQL 执行失败

| 错误码 | 错误消息 | 原因 | 解决方案 | 证据 |
|--------|---------|------|----------|------|
| **RDB_E_INVALID_SQL** | "Invalid SQL statement" | SQL 语法错误或参数不匹配 | 1. 使用参数化查询 `Query()`<br/>2. 检查 SQL 语句在 SQLite 中可用 | `sqlite_statement.cpp` |
| **RDB_E_EXECUTE_FAIL** | "Execute failed" | 约束冲突或索引问题 | 1. 检查外键约束<br/>2. 检查唯一索引冲突 | `rdb_store_impl.cpp` |
| **RDB_E_EMPTY_VALUES_BUCKET** | "Empty values bucket" | 插入空数据 | 1. 检查 ValuesBucket 不为空<br/>2. 添加必填字段校验 | `rdb_store.h` |

#### 2.3 连接池问题

| 现象 | 原因 | 解决方案 | 证据 |
|------|------|----------|------|
| **连接耗尽** | 连接池达到最大值（4）且连接未释放 | 1. 及时关闭 ResultSet<br/>2. 释放事务对象<br/>3. 检查连接泄漏 | `connection_pool.cpp` |
| **写操作阻塞** | 读操作持有 EXCLUSIVE 锁导致写操作等待 | 1. 缩短事务持有时间<br/>2. 避免长事务 | `rdb_store.h:597-614` |
| **死锁** | 连接循环等待 | 1. 检查事务嵌套<br/>2. 使用超时机制 | `transaction_impl.cpp` |

### 3. 性能优化建议

#### 3.1 查询优化

| 问题 | 优化建议 | 证据 |
|------|----------|------|
| **全表扫描** | 1. 为常用查询条件创建索引<br/>2. 避免 SELECT * | `rdb_predicates.h` |
| **N+1 查询** | 1. 使用批量操作 `BatchInsert()`<br/>2. 使用事务包装多个操作 | `rdb_store.h:197-210` |
| **LIKE 查询慢** | 1. 使用 FTS（全文检索）索引<br/>2. 添加前置通配符优化 | `rdb_store.h:829-836` |

#### 3.2 内存优化

| 问题 | 优化建议 | 证据 |
|------|----------|------|
| **结果集过大** | 1. 使用 `QueryByStep()` 分页读取<br/>2. 设置合理的 LIMIT<br/>3. 及时关闭 ResultSet | `step_result_set.cpp` |
| **BLOB 内存占用** | 1. 使用流式读取大 BLOB<br/>2. 及时释放 BLOB 数据 | `value_object.cpp` |
| **连接泄漏** | 1. 使用 RAII 模式管理连接<br/>2. 及时释放 ResultSet 和 Transaction | `connection_pool.cpp` |

#### 3.3 云同步优化

| 问题 | 优化建议 | 证据 |
|------|----------|------|
| **频繁同步** | 1. 合并多个表到单个同步请求<br/>2. 合理设置同步间隔 | `cloud_manager.cpp` |
| **同步冲突多** | 1. 选择合适的冲突解决策略<br/>2. 优化数据设计避免冲突 | `rdb_store.h:152-167` |

### 4. 调试技巧

#### 4.1 日志开启

| 组件 | 日志开关 | 启用方法 | 证据 |
|------|---------|----------|------|
| **RDB** | HILOG_ENABLE | 1. 编译时添加 `define=HILOG_ENABLE`<br/>2. 运行时设置 `hilog.debug.enable=true` | `frameworks/native/rdb/src/rdb_sql_log.cpp` |
| **Cloud Data** | CLOUD_DEBUG | 同上 | `cloud_manager.cpp` |
| **NAPI** | NAPI_DEBUG | 同上 | `js_utils.cpp` |

#### 4.2 日志过滤

| 过滤器 | 说明 | 示例 |
|--------|------|------|
| **SQL 日志** | 记录执行的 SQL 语句 | `hilog -t Rdb:SELECT * FROM users` |
| **性能统计** | 记录查询耗时 | `hilog -t Perf:Query took 100ms` |
| **错误日志** | 记录错误信息 | `hilog -e Error:Insert failed` |

#### 4.3 工具使用

| 工具 | 用途 | 命令示例 |
|------|------|----------|
| **sqlite3** | 直接操作数据库文件 | `sqlite3 /data/el2/com.example/app/rdb/test.db` |
| **hilog** | 查看日志 | `hilog -b Rdb` |
| **hdc shell** | 进入设备 shell | `hdc shell` |

### 5. 常见错误码速查表

| 错误码 | 含义 | 快速检查 | 证据 |
|--------|------|----------|------|
| **RDB_OK (0)** | 成功 | 操作正常完成 | `relational_store_error_code.h:18` |
| **RDB_E_INVALID_ARGS (-1)** | 无效参数 | 检查参数类型和范围 | 同上 |
| **RDB_E_INVALID_COLUMN_NAME (-2)** | 无效列名 | 检查列名拼写和存在性 | 同上 |
| **RDB_E_EMPTY_TABLE_NAME (-5)** | 空表名 | 检查表名是否为空 | 同上 |
| **RDB_E_INVALID_SQL (-7)** | 无效 SQL | 检查 SQL 语法 | 同上 |
| **RDB_E_NOT_IN_TRANS (-8)** | 不在事务中 | 确保在事务中调用 | 同上 |
| **RDB_E_IN_TRANS (-9)** | 已在事务中 | 避免嵌套事务 | 同上 |
| **RDB_E_INVALID_DB (-10)** | 无效数据库 | 检查数据库路径和文件 | 同上 |

### 6. 跨平台开发注意事项

| 平台 | 注意事项 | 证据 |
|------|----------|------|
| **Windows (MinGW)** | 使用模拟实现，不支持完整功能 | `frameworks/js/napi/relationalstore/BUILD.gn:91-106` |
| **Mac** | 使用模拟实现，不支持 IPC | `frameworks/js/napi/relationalstore/BUILD.gn:132-174` |
| **Android/iOS** | 使用模拟实现，仅用于单元测试 | `frameworks/js/napi/relationalstore/BUILD.gn:176-256` |

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [03_Architecture.md](./03_Architecture.md) - 架构与数据流
- [06_GN_Build.md](./06_GN_Build.md) - GN 构建详解

---

**文档版本**: 1.0
**生成日期**: 2026-02-06
**主要证据来源**: 错误码定义、README_zh.md、实际开发经验
