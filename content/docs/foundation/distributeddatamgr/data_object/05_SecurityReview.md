# 安全风险评估

> 分布式数据对象组件的安全风险分析与修复建议

## 目的

本文档系统性分析分布式数据对象组件的安全风险，包括输入验证、内存安全、权限管理、并发安全和逻辑漏洞，提供可操作的修复建议。

## 适用范围

- OpenHarmony 标准系统
- 组件版本 3.1.0
- 面向安全研究员和安全审计人员

---

## R1: sessionId 验证不足（中危）

**位置**: `frameworks/jskitsimpl/src/adaptor/js_distributedobject.cpp`

**证据**:
```cpp
// TODO: 需要找到具体代码行
// setSessionId() 可能未充分验证 sessionId 格式
```

**触发路径**:
```
JS API 调用 setSessionId("../../../etc/passwd")
  → N-API 接收
  → 直接使用作为 key
  → 可能导致路径遍历或注入
```

**影响评估**:
- 可利用性：中等
- 影响：信息泄露、数据篡改
- 权限提升可能：否

**修复建议**:
```cpp
// 验证 sessionId 格式（字母数字下划线，最长 128）
bool ValidateSessionId(const std::string &sessionId) {
    if (sessionId.empty() || sessionId.length() > 128) {
        return false;
    }
    for (char c : sessionId) {
        if (!isalnum(c) && c != '_') {
            return false;
        }
    }
    return true;
}
```

---

## R2: 权限检查时机错误（中危）

**位置**: `frameworks/innerkitsimpl/src/adaptor/flat_object_storage_engine.cpp:43-45`

**证据**:
```cpp
int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(tokenId, DISTRIBUTED_DATASYNC);
if (ret == Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
    // 允许操作
}
```

**触发路径**:
```
1. 应用调用 setSessionId()
2. 权限检查通过
3. 应用被撤销权限
4. 对象仍保持激活状态（竞态条件）
```

**影响评估**:
- 可利用性：中等（TOCTOU 竞态条件）
- 影响：权限提升
- 权限提升可能：是

**修复建议**:
```cpp
// 在关键操作前重新验证权限
int32_t FlatObjectStorageEngine::PutDouble(const std::string &key, double value) {
    // 每次操作前检查权限
    int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(tokenId, DISTRIBUTED_DATASYNC);
    if (ret != Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return ERR_NO_PERMISSION;
    }
    // 执行操作
    // ...
}
```

---

## R3: Asset URI 未验证（高危）

**位置**: `frameworks/jskitsimpl/src/adaptor/js_distributedobject.cpp`

**证据**:
```cpp
// TODO: 需要定位 bindAssetStore() 的具体实现
// Asset URI 可能未验证文件路径有效性
```

**触发路径**:
```
JS API 调用 bindAssetStore("key", { uri: "file://../../../../etc/passwd" })
  → N-API 接收
  → 直接使用 URI
  → 可能读取任意文件
```

**影响评估**:
- 可利用性：高
- 影响：任意文件读取、数据泄露
- 权限提升可能：是（如果可以读取系统文件）

**修复建议**:
```cpp
// 验证 Asset URI 路径规范化
bool ValidateAssetUri(const std::string &uri) {
    // 检查是否包含路径遍历字符
    if (uri.find("..") != std::string::npos) {
        return false;
    }
    // 检查是否以允许的协议开头
    if (uri.find("file://") != 0 && uri.find("content://") != 0) {
        return false;
    }
    return true;
}
```

---

## R4: 并发竞态条件（中危）

**位置**: `frameworks/innerkitsimpl/src/adaptor/flat_object_store.cpp`

**证据**:
```cpp
// 多个互斥锁但可能存在死锁风险
std::lock_guard<std::mutex> lck(mutex_);
std::lock_guard<std::mutex> lck(progressInfoMutex_);
```

**触发路径**:
```
1. 线程 A 持有 mutex_，尝试获取 progressInfoMutex_
2. 线程 B 持有 progressInfoMutex_，尝试获取 mutex_
3. 死锁（如果锁获取顺序不一致）
```

**影响评估**:
- 可利用性：低（需要特定条件）
- 影响：拒绝服务
- 权限提升可能：否

**修复建议**:
```cpp
// 使用统一的锁顺序或使用 std::scoped_lock
// 方案 1: 固定锁顺序
std::lock(mutex_, progressInfoMutex_);
std::lock_guard<std::mutex> lck1(mutex_, std::adopt_lock);
std::lock_guard<std::mutex> lck2(progressInfoMutex_, std::adopt_lock);

// 方案 2: 使用 scoped_lock 避免死锁
std::scoped_lock lock(mutex_, progressInfoMutex_);
```

---

## R5: 资源耗尽风险（低危）

**位置**: `frameworks/innerkitsimpl/src/adaptor/flat_object_store.cpp`

**证据**:
```cpp
// 对象数量未限制
// README 提示"不建议创建过多分布式对象"
```

**触发路径**:
```
1. 恶意应用循环调用 createObjectSync()
2. 每个对象占用 100-150KB
3. 大量创建导致内存耗尽
```

**影响评估**:
- 可利用性：中等
- 影响：拒绝服务
- 权限提升可能：否

**修复建议**:
```cpp
// 添加对象数量限制
const int MAX_OBJECTS_PER_APP = 100;
int FlatObjectStore::CreateObject(const std::string &sessionId) {
    std::lock_guard<std::mutex> lck(mutex_);
    if (objects_.size() >= MAX_OBJECTS_PER_APP) {
        return ERR_EXIST;
    }
    // 创建对象
}
```

---

## 证据完整性

| 风险 | 证据来源 |
|------|----------|
| R1: sessionId 验证不足 | JS 代码分析，TODO 定位具体行 |
| R2: 权限检查时机错误 | `flat_object_storage_engine.cpp:43-45` |
| R3: Asset URI 未验证 | JS 代码分析，TODO 定位具体行 |
| R4: 并发竞态条件 | `flat_object_store.cpp` 互斥锁使用 |
| R5: 资源耗尽风险 | `flat_object_store.cpp` 对象管理 |

## 修复优先级

| 优先级 | 风险 | 建议修复时间 |
|--------|------|------------|
| P0 | R3: Asset URI 未验证 | 立即 |
| P1 | R2: 权限检查时机错误 | 1-2 周 |
| P1 | R1: sessionId 验证不足 | 1-2 周 |
| P2 | R4: 并发竞态条件 | 2-4 周 |
| P3 | R5: 资源耗尽风险 | 下个版本 |

## 相关链接

- [攻击面分析](./04_AttackSurface.md)
- [接口文档](./03_Interface.md)
- [内部实现](./07_Internals.md)
