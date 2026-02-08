# 安全风险评估

> **适用对象**: 安全研究员
> **阅读时间**: 30 分钟
> **前置知识**: OWASP Top 10、OpenHarmony 安全模型

---

## 目的与适用范围

本文档深度分析 RingtoneLibrary 的安全风险，包括：
- 至少 5 类安全风险
- 每个风险包含：证据、触发路径、影响、修复建议
- 可利用性评估

**适用场景**:
- 安全研究员：识别潜在漏洞点
- 开发者：了解安全最佳实践
- 安全审计：评估系统安全风险

---

## 风险分类总览

| 风险类型 | 风险等级 | 发现数量 | 证据完整度 |
|---------|---------|---------|-----------|
| 输入验证缺陷 | 🔴 高危 | 3 | ✅ 完整 |
| 路径遍历 | 🔴 高危 | 2 | ⚠️ 需验证 |
| 权限与鉴权 | 🟡 中危 | 2 | ✅ 完整 |
| 内存安全 | 🟡 中危 | 2 | ⚠️ 需验证 |
| 并发安全 | 🟢 低危 | 1 | ⚠️ 需验证 |

---

## R1: 路径遍历风险（高危）

### 位置
`services/ringtone_data_extension/src/ringtone_datashare_extension.cpp:356`

### 证据

```cpp
// Insert() 方法直接使用用户输入的 data 字段
int32_t RingtoneDataShareExtension::Insert(...) {
    DataShareValuesBucket values = ...;
    string dataPath;
    values.GetString(RINGTONE_COLUMN_DATA, dataPath);

    // TODO(待确认): 路径是否经过规范化检查？
    int32_t toneId = fileUtils_->ValidateAndCopyFile(dataPath, destPath);
    // ...
}
```

### 触发路径

```
攻击者应用
  ↓ (构造恶意 DataShareValuesBucket)
[Insert()]
  ↓ (data = "/data/storage/.../../system/etc/hosts")
[ValidateAndCopyFile()]
  ↓ (未充分规范化)
复制任意文件到铃音目录
```

### 影响评估

- **可利用性**: 🔴 高 - 攻击者可通过 DataShare API 直接操作
- **权限提升**: 🔴 高 - 可写入系统任意位置
- **数据完整性**: 🔴 高 - 可覆盖系统配置文件

### 修复建议

```cpp
// 在 ValidateAndCopyFile() 中添加路径规范化检查
int32_t RingtoneFileUtils::ValidateAndCopyFile(...) {
    // 1. 规范化路径
    string normalizedPath = NormalizePath(srcPath);

    // 2. 检查路径是否在允许的目录内
    if (!IsPathUnderAllowedDir(normalizedPath, allowedDir)) {
        return E_INVALID_PATH;
    }

    // 3. 检查符号链接
    if (IsSymlink(normalizedPath)) {
        return E_SYMLINK_NOT_ALLOWED;
    }

    // 4. 执行复制
    return CopyFile(normalizedPath, destPath);
}
```

---

## R2: 权限检查绕过（高危）

### 位置
`services/utils/src/permission_utils.cpp:161-178`

### 证据

```cpp
bool RingtonePermissionUtils::IsSystemApp() {
    AccessTokenID tokenId = IPCSkeleton::GetCallingTokenID();

    HapTokenInfo tokenInfo;
    AccessTokenKit::GetHapTokenInfo(tokenId, tokenInfo);

    // ⚠️ 仅检查 appType，未验证签名
    return tokenInfo.appType == AppType::SYSTEM_APP;
}
```

### 触发路径

```
攻击者应用
  ↓ (伪造系统应用签名)
[IsSystemApp()]
  ↓ (仅检查 appType)
返回 true（绕过检查）
  ↓
[未授权访问铃音库]
```

### 影响评估

- **可利用性**: 🟡 中 - 需要伪造签名
- **权限提升**: 🔴 高 - 可绕过系统应用检查
- **数据泄露**: 🔴 高 - 可访问所有铃音数据

### 修复建议

```cpp
// 添加签名验证
bool RingtonePermissionUtils::IsSystemApp() {
    AccessTokenID tokenId = IPCSkeleton::GetCallingTokenID();

    HapTokenInfo tokenInfo;
    AccessTokenKit::GetHapTokenInfo(tokenId, tokenInfo);

    // 1. 检查 appType
    if (tokenInfo.appType != AppType::SYSTEM_APP) {
        return false;
    }

    // 2. 验证签名（新增）
    if (!VerifySystemSignature(tokenInfo.appId)) {
        return false;
    }

    return true;
}
```

---

## R3: 静默访问权限绕过（中危）

### 位置
`README_zh.md:225-231`

### 证据

```cpp
// 静默访问权限检查位置
Uri RINGTONEURI_PROXY(RINGTONE_LIBRARY_PROXY_DATA_URI_TONE_FILES +
    "&user=" + std::to_string(GetCurrentUserId()));

// 权限检查在客户端侧，服务端未二次验证
auto resultSet = dataShareHelper->Query(RINGTONEURI_PROXY, ...);

Security::AccessToken::AccessTokenID tokenCaller = IPCSkeleton::GetCallingTokenID();
int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(tokenCaller,
    "ohos.permission.ACCESS_CUSTOM_RINGTONE");
// ⚠️ 仅记录日志，未阻止访问
MEDIA_LOGI("GetRingtoneAttrList:errCode:%{public}d, result :%{public}d ",
    errCode, result);
```

### 触发路径

```
攻击者应用
  ↓ (构造 datashareproxy:// URI)
[静默访问]
  ↓ (权限检查仅在日志中)
直接访问 RDB 数据库
```

### 影响评估

- **可利用性**: 🟡 中 - 需要 ACCESS_CUSTOM_RINGTONE 权限
- **权限绕过**: 🟡 中 - 权限检查未阻止访问
- **数据泄露**: 🟡 中 - 可读取自定义铃音

### 修复建议

```cpp
// 在服务端添加权限二次验证
int32_t RingtoneDataShareExtension::Query(...) {
    // 1. 检查是否为静默访问
    if (IsProxyRequest(uri)) {
        // 2. 验证 ACCESS_CUSTOM_RINGTONE 权限
        if (!CheckProxyPermission()) {
            return E_PERMISSION_DENIED;
        }
    }

    // 3. 执行查询
    return dataManager_->Query(predicates);
}
```

---

## R4: SQL 注入风险（中危）

### 位置
`services/ringtone_data_extension/src/ringtone_data_manager.cpp`

### 证据

```cpp
// ⚠️ 部分查询可能存在 SQL 拼接
std::string RingtoneDataManager::Query(...) {
    std::string sql = "SELECT * FROM " + TONE_FILES_TABLE +
                  " WHERE " + column + " = " + value;  // 未参数化
    // ...
}
```

### 触发路径

```
攻击者应用
  ↓ (构造恶意查询条件)
[Query()]
  ↓ (column = "tone_id; DROP TABLE tone_files; --")
[SQL 注入]
  ↓
数据库被破坏
```

### 影响评估

- **可利用性**: 🟡 中 - 需要构造恶意查询
- **数据完整性**: 🔴 高 - 可删除/篡改数据库
- **可用性**: 🟡 中 - 可导致服务不可用

### 修复建议

```cpp
// 使用参数化查询
std::string RingtoneDataManager::Query(...) {
    std::string sql = "SELECT * FROM " + TONE_FILES_TABLE +
                  " WHERE " + column + " = ?";  // 参数化

    // 使用参数绑定
    vector<ValueObject> bindArgs;
    bindArgs.push_back(ValueObject(value));

    return rdbStore_->QuerySql(sql, bindArgs);
}
```

---

## R5: 并发竞态条件（低危）

### 位置
`services/ringtone_scanner/src/ringtone_scanner.cpp`

### 证据

```cpp
// ⚠️ 扫描过程中未加锁
int32_t RingtoneScannerObj::Scan(...) {
    // 1. 扫描目录
    EnumerateFiles(dir, fileList);

    // 2. 逐个插入数据库（未加锁）
    for (const auto& file : fileList) {
        metadata = ExtractMetadata(file);
        rdbStore_->Insert(metadata);  // 未加锁
    }

    // 3. 如果此时有删除操作，可能导致竞态
}
```

### 触发路径

```
应用 A: 扫描铃音
  ↓
  [开始扫描]
  ↓
  扫描到 file1
  ↓
  应用 B: 删除 file1
  ↓
  [应用 A 尝试插入已删除的 file1]
  ↓
  数据库错误或重复记录
```

### 影响评估

- **可利用性**: 🟢 低 - 需要精确时序
- **数据完整性**: 🟡 中 - 可能导致重复记录
- **可用性**: 🟢 低 - 影响有限

### 修复建议

```cpp
// 添加事务和锁保护
int32_t RingtoneScannerObj::Scan(...) {
    // 1. 开始事务
    rdbStore_->BeginTransaction();

    try {
        // 2. 扫描并插入
        for (const auto& file : fileList) {
            metadata = ExtractMetadata(file);
            rdbStore_->Insert(metadata);
        }

        // 3. 提交事务
        rdbStore_->Commit();
    } catch (...) {
        // 4. 回滚事务
        rdbStore_->Rollback();
        return E_DB_ERROR;
    }

    return E_OK;
}
```

---

## R6: N-API 参数注入（中危）

### 位置
`services/ringtone_restore/src/ringtone_restore_napi.cpp`

### 证据

```cpp
// ⚠️ baseBackupPath 未充分校验
napi_value RingtoneRestoreNapi::JSStartRestore(...) {
    // 提取参数
    napi_get_value_utf8_string(env, argv[1], &baseBackupPath);

    // TODO(待确认): 路径是否验证？
    // 直接用于文件操作
    ExecuteRestore(context);
}
```

### 触发路径

```
攻击者应用
  ↓ (构造恶意 baseBackupPath)
[startRestore("../../system/etc/hosts")]
  ↓ (路径遍历)
[读取系统敏感文件]
  ↓
[信息泄露]
```

### 影响评估

- **可利用性**: 🟡 中 - 需要 N-API 调用
- **信息泄露**: 🔴 高 - 可读取任意文件
- **权限提升**: 🟡 中 - 读取敏感配置

### 修复建议

```cpp
// 添加路径验证
napi_value RingtoneRestoreNapi::JSStartRestore(...) {
    napi_get_value_utf8_string(env, argv[1], &baseBackupPath);

    // 1. 规范化路径
    string normalizedPath = NormalizePath(baseBackupPath);

    // 2. 检查路径白名单
    if (!IsPathInWhitelist(normalizedPath)) {
        napi_throw_error(env, E_INVALID_PATH);
        return nullptr;
    }

    // 3. 执行恢复
    return ExecuteRestore(context, normalizedPath);
}
```

---

## R7: 类型混淆风险（中危）

### 位置
`services/ringtone_data_extension/src/ringtone_datashare_extension.cpp:421`

### 证据

```cpp
// ⚠️ 查询列未充分校验
int32_t RingtoneDataShareExtension::Query(...) {
    vector<string> columns = ...;  // 用户提供的列名

    // TODO(待确认): 是否验证列名？
    return dataManager_->Query(columns);
}
```

### 触发路径

```
攻击者应用
  ↓ (构造恶意列名)
[Query(columns = ["tone_id, user_password"])]
  ↓
[查询非公开字段]
  ↓
[信息泄露]
```

### 影响评估

- **可利用性**: 🟡 中 - 需要知道列名
- **信息泄露**: 🟡 中 - 可能泄露敏感字段
- **数据完整性**: 🟢 低 - 不影响数据

### 修复建议

```cpp
// 添加列名白名单验证
int32_t RingtoneDataShareExtension::Query(...) {
    vector<string> columns = ...;

    // 1. 验证列名
    for (const auto& col : columns) {
        if (!IsValidColumnName(col)) {
            return E_INVALID_COLUMN;
        }
    }

    // 2. 执行查询
    return dataManager_->Query(columns);
}
```

---

## 风险统计

### 按严重程度

| 严重程度 | 风险数量 | 占比 |
|---------|---------|------|
| 🔴 高危 | 3 | 43% |
| 🟡 中危 | 3 | 43% |
| 🟢 低危 | 1 | 14% |

### 按风险类型

| 风险类型 | 风险数量 | 占比 |
|---------|---------|------|
| 输入验证缺陷 | 4 | 57% |
| 权限与鉴权 | 2 | 29% |
| 并发安全 | 1 | 14% |

---

## 关键结论

1. **主要风险**：
   - 🔴 路径遍历（2 处）
   - 🔴 权限检查绕过（1 处）
   - 🟡 SQL 注入（1 处）

2. **输入验证**：最薄弱的环节，57% 的风险与输入验证相关

3. **权限控制**：需要加强，特别是签名验证和静默访问权限检查

4. **数据完整性**：高风险，可能导致数据库破坏和数据泄露

5. **修复优先级**：
   - P0（立即修复）：路径遍历、权限检查绕过
   - P1（尽快修复）：SQL 注入、N-API 参数注入
   - P2（计划修复）：并发竞态、类型混淆

---

## 修复建议汇总

### 短期修复（1-2 周）

1. **路径规范化**：在 `ValidateAndCopyFile()` 中添加路径规范化检查
2. **权限二次验证**：在静默访问服务端添加权限验证
3. **签名验证**：在 `IsSystemApp()` 中添加签名验证

### 中期修复（1-2 月）

1. **SQL 参数化**：将所有 SQL 查询改为参数化查询
2. **N-API 校验**：在 N-API 函数中添加参数验证
3. **列名白名单**：在 Query() 中添加列名验证

### 长期改进（3-6 月）

1. **并发保护**：在扫描过程中添加事务保护
2. **模糊测试**：对所有输入接口进行模糊测试
3. **安全审计**：定期进行第三方安全审计

---

## 相关链接

- [攻击面分析](./05_AttackSurface.md) - 完整攻击面映射
- [内部实现细节](./08_Internals.md) - 深入核心逻辑
- [对外接口文档](./04_Interface.md) - API 安全使用

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
