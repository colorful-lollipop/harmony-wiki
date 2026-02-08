# 故障排查

本文档汇总 Preferences 模块的常见问题、错误码速查和调试方法。

## 1 错误码速查

### 1.1 通用错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 0 | `E_OK` | 成功 | - |
| 1 | `E_STALE` | 资源已停止/销毁 | 检查实例是否已关闭 |
| 2 | `E_INVALID_ARGS` | 输入参数无效 | 检查参数合法性 |
| 3 | `E_OUT_OF_MEMORY` | 内存不足 | 释放内存或减少数据量 |
| 4 | `E_NOT_PERMIT` | 操作不允许 | 检查权限配置 |

### 1.2 参数错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 5 | `E_KEY_EMPTY` | Key 为空 | 提供非空 Key |
| 6 | `E_KEY_EXCEED_MAX_LENGTH` | Key 过长 | Key 不超过 1024 字符 |
| 14 | `E_VALUE_EXCEED_MAX_LENGTH` | Value 过长 | Value 不超过 16MB |
| 15 | `E_KEY_EXCEED_LENGTH_LIMIT` | Key 超过限制 | Key 不超过 1024 字符 |
| 16 | `E_VALUE_EXCEED_LENGTH_LIMIT` | Value 超过限制 | Value 不超过 16MB |
| 17 | `E_DEFAULT_EXCEED_LENGTH_LIMIT` | 默认值过长 | 默认值不超过 16MB |

### 1.3 文件错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 8 | `E_RELATIVE_PATH` | 相对路径 | 使用绝对路径 |
| 9 | `E_EMPTY_FILE_PATH` | 空路径 | 提供有效路径 |
| 11 | `E_EMPTY_FILE_NAME` | 空文件名 | 提供有效文件名 |
| 12 | `E_INVALID_FILE_PATH` | 无效路径 | 检查路径格式 |
| 13 | `E_PATH_EXCEED_MAX_LENGTH` | 路径过长 | 缩短路径长度 |
| 10 | `E_DELETE_FILE_FAIL` | 删除失败 | 检查文件权限 |

### 1.4 权限错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 18 | `PERMISSION_DENIED` | 权限拒绝 | 检查应用权限配置 |

### 1.5 状态错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 7 | `E_PTR_EXIST_ANOTHER_HOLDER` | 指针被占用 | 等待其他操作完成 |
| 19 | `E_GET_DATAOBSMGRCLIENT_FAIL` | 获取客户端失败 | 重试或检查系统服务 |
| 20 | `E_OBSERVER_RESERVE` | 观察者保留 | 释放观察者资源 |
| 21 | `E_ALREADY_CLOSED` | 已关闭 | 重新打开实例 |
| 22 | `E_NO_DATA` | 数据不存在 | 检查 Key 是否存在 |
| 27 | `E_OBJECT_NOT_ACTIVE` | 对象未激活 | 检查对象状态 |

### 1.6 锁/并发错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 24 | `E_OPERAT_IS_LOCKED` | 文件被锁定 | 等待锁释放 |
| 25 | `E_OPERAT_IS_CROSS_PROESS` | 跨进程操作 | 避免并发冲突 |

### 1.7 其他错误码

| 错误码 | 宏名 | 说明 | 解决方案 |
|--------|------|------|----------|
| 23 | `E_XML_RESTORED_FROM_BACKUP_FILE` | 从备份恢复 | 检查数据完整性 |
| 26 | `E_SUBSCRIBE_FAILED` | 订阅失败 | 检查观察者配置 |

---

## 2 常见问题

### Q1: getPreferences 返回 Promise 挂起

**问题描述**：`getPreferences()` 调用后 Promise 一直 pending

**可能原因**：
1. 应用上下文无效
2. 内存加载阻塞

**排查步骤**：

```typescript
// 1. 检查 context 是否有效
try {
  let preferences = await preferences.getPreferences(context, 'test');
} catch (e) {
  console.error('Error:', e);
}
```

**解决方案**：
- 确保 context 有效且属于当前应用
- 使用 `flushSync()` 代替 `flush()` 确保数据同步

**相关日志**：搜索 `LoadFromDisk` 相关日志

---

### Q2: 数据写入后读取不到

**问题描述**：`put()` 成功但 `get()` 返回默认值

**可能原因**：
1. 未调用 `flush()` 持久化
2. Key 大小写不匹配

**排查步骤**：

```typescript
// 1. 检查是否 flush
await preferences.put('key', 'value');
await preferences.flush();  // 必须调用
let value = await preferences.get('key', 'default');

// 2. 检查 Key 是否完全匹配
```

**解决方案**：
- `put()` 后必须调用 `flush()` 或 `flushSync()`
- 确保 Key 大小写与写入时一致

---

### Q3: 跨进程访问失败

**问题描述**：`PERMISSION_DENIED` 错误

**可能原因**：
1. 跨应用访问未授权
2. 文件权限不足

**排查步骤**：

```bash
# 检查文件权限
ls -la /data/app/el*/[bundleName]/pref/
```

**解决方案**：
- 确认应用有访问目标文件的权限
- 使用 `removePreferencesFromCache()` 后重新获取

**相关错误码**：18 (`PERMISSION_DENIED`)

---

### Q4: 内存占用过高

**问题描述**：Preferences 占用内存过大

**可能原因**：
1. 存储数据量超过建议值
2. Value 值过大
3. 实例未及时释放

**排查步骤**：

```typescript
// 1. 检查数据量
let all = await preferences.getAll();
console.log('Keys count:', Object.keys(all).length);

// 2. 移除不使用实例
await preferences.removePreferencesFromCache(context, 'myPrefs');
```

**解决方案**：
- 建议总数据量不超过 10000 条
- 单个 Value 不超过 16MB
- 及时 `removePreferencesFromCache()`

---

### Q5: XML 解析失败

**问题描述**：读取数据时解析错误

**可能原因**：
1. XML 文件损坏
2. 磁盘空间不足
3. 文件权限被修改

**排查步骤**：

```bash
# 检查 XML 文件
cat /data/app/el*/[bundleName]/pref/[name].xml

# 检查文件权限
ls -la /data/app/el*/[bundleName]/pref/
```

**解决方案**：
- 使用 `deletePreferences()` 删除后重建
- 检查磁盘空间

**相关错误码**：23 (`E_XML_RESTORED_FROM_BACKUP_FILE`)

---

### Q6: 文件被锁定

**问题描述**：`E_OPERAT_IS_LOCKED` 或 `E_OPERAT_IS_CROSS_PROESS`

**可能原因**：
1. 其他进程正在访问
2. 上次操作未正常关闭

**排查步骤**：

```typescript
// 1. 检查是否有未关闭实例
// 2. 使用 flushSync() 同步操作
await preferences.flushSync();
```

**解决方案**：
- 避免跨进程并发访问同一 Preferences
- 确保每次操作后正确 Close

**相关错误码**：24, 25

---

### Q7: Key/Value 长度超限

**问题描述**：`E_KEY_EXCEED_MAX_LENGTH` 或 `E_VALUE_EXCEED_MAX_LENGTH`

**可能原因**：
1. Key 超过 1024 字符
2. Value 超过 16MB

**排查解决方案**：

```typescript
// 检查 Key 长度
const MAX_KEY_LENGTH = 1024;
if (key.length > MAX_KEY_LENGTH) {
  throw new Error('Key too long');
}

// 检查 Value 长度
const MAX_VALUE_LENGTH = 16 * 1024 * 1024;
if (typeof value === 'string' && value.length > MAX_VALUE_LENGTH) {
  throw new Error('Value too long');
}
```

---

### Q8: 观察者不触发

**问题描述**：`on('change')` 注册的监听器不触发

**可能原因**：
1. 未调用 `flush()`
2. 监听器注册错误
3. 实例被关闭

**排查步骤**：

```typescript
// 1. 检查监听器注册
preferences.on('change', (pref) => {
  console.log('Changed!');
});

// 2. 确保 flush
await preferences.put('key', 'value');
await preferences.flush();
```

**解决方案**：
- 数据变化后必须调用 `flush()` 才能触发通知
- 确保监听器在实例有效期内注册

---

### Q9: XML 解析失败（XXE 注入相关）⚠️

**问题描述**：加载 Preferences 时出现 XML 解析错误或安全问题

**可能原因**：
1. 数据文件包含恶意 XML 实体
2. 文件损坏
3. 非法 XML 字符

**排查步骤**：

```typescript
// 1. 检查错误日志
// 搜索 "xml" 或 "parse" 相关错误
// 2. 验证文件完整性
// 3. 检查是否有异常大的文件
```

**解决方案**：
- 代码位置：`preferences_xml_utils.cpp:427, 528`（`xmlReadFile()` 函数）
- 确认 XML 解析器配置了 `XML_PARSE_NOENTITIES` 标志以防止 XXE
- 如果遇到 XXE 相关错误，检查数据来源是否可信
- 定期备份关键配置文件

**相关风险**：详见 [安全评审 - SEC-006](./04_Security_Review.md#风险-6xxe-注入xml-外部实体注入-高危)

---

### Q10: 动态库加载失败

**问题描述**：启动时出现库加载错误或模块初始化失败

**可能原因**：
1. `libarkdata_db_core.z.so` 未找到或损坏
2. 动态库路径错误
3. 库版本不匹配

**排查步骤**：

```bash
# 1. 检查库文件是否存在
ls -l /usr/lib/libarkdata_db_core.z.so

# 2. 检查库依赖
ldd libohpreferences.so

# 3. 查看加载日志
# 搜索 "dlopen" 或 "LoadFunction" 相关错误
```

**解决方案**：
- 代码位置：`preferences_db_adapter.cpp:72`（`dlopen()` 调用）
- 确认 `dlopen()` 使用绝对路径加载库文件，避免路径劫持
- 验证库文件签名和完整性
- 如果是环境变量导致的问题，确保 `LD_LIBRARY_PATH` 未被篡改

**相关风险**：详见 [安全评审 - SEC-006](./04_Security_Review.md#风险-6xxe-注入xml-外部实体注入-高危)

---

## 3 调试方法

### 3.1 日志输出

**Hilog 标签**：`DistributedDataMgr_Preferences`

**日志级别**：
- DEBUG：详细调试信息
- INFO：普通运行信息
- WARN：警告信息
- ERROR：错误信息

**日志查看**：

```bash
# 过滤 Preferences 相关日志
hilog | grep -i "Preferences"
```

### 3.2 HiSysEvent

**上报事件**：`PREFERENCES`

**事件参数**：
- `BUNDLE_NAME`：应用包名
- `STORE_NAME`：存储名称
- `MODULE_NAME`：模块名
- `ERR_CODE`：错误码

**查看方法**：

```bash
# 查看 HiSysEvent
hiappevent -f PREFERENCES
```

### 3.3 文件位置

| 数据类型 | 默认路径 |
|----------|----------|
| Preferences XML | `/data/app/el*/[bundleName]/pref/[name].xml` |
| 备份文件 | `/data/app/el*/[bundleName]/pref/[name].bak` |
| 配置文件 | `$HOME/.preferencesrc` |

### 3.4 调试技巧

#### 检查实例状态

```typescript
// 检查是否已加载
console.log('Loaded:', preferences.loaded);
```

#### 检查缓存数据

```typescript
// 获取所有数据
let all = await preferences.getAll();
console.log('All data:', JSON.stringify(all));
```

---

## 4 构建问题

### 4.1 GN 构建失败

**错误**：`unknown variable "preferences_base_path"`

**解决**：确保正确 import `preferences.gni`

```gn
import("//foundation/distributeddatamgr/preferences/preferences.gni")
```

### 4.2 依赖缺失

**错误**：`deps not found`

**解决**：检查 bundle.json 中的 external_deps 配置

```json
"external_deps": [
  "ability_runtime:abilitykit_native",
  "ipc:ipc_single"
]
```

### 4.3 编译选项错误

**错误**：`sanitize` 配置不正确

**解决**：OHOS 平台才支持 sanitizer

```gn
if (is_ohos) {
  sanitize = {
    boundary_sanitize = true
    ubsan = true
    cfi = true
  }
}
```

---

## 5 性能问题

### 5.1 优化建议

| 问题 | 建议 |
|------|------|
| 读取慢 | 避免频繁打开/关闭，使用实例缓存 |
| 写入慢 | 使用批量操作，减少 flush 次数 |
| 内存高 | 及时 removePreferencesFromCache |

### 5.2 性能指标

| 操作 | 预期耗时 |
|------|----------|
| getPreferences | < 10ms（缓存命中） |
| put | < 1ms（内存操作） |
| flush | < 100ms（单次写入） |

---

## 6 相关文档

| 文档 | 说明 |
|------|------|
| [API 参考](./02_API_Reference.md) | 完整 API 清单 |
| [架构设计](./01_Architecture.md) | 内部架构详解 |
| [构建配置](./03_Build.md) | 构建流程说明 |
| [安全评审](./04_Security_Review.md) | 安全风险分析 |
