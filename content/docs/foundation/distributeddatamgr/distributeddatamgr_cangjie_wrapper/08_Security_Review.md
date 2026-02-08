# 安全风险评审

## 文档目的

本文档对 distributeddatamgr_cangjie_wrapper 项目进行全面的安全风险评审，识别潜在的安全威胁、攻击面和可被利用点。

## 适用范围

本文档覆盖：
- FFI 层的安全风险
- 输入验证安全
- 权限控制审计
- 内存安全分析
- 敏感数据处理

## 威胁模型

### 数据流分析

```
┌─────────────────────────────────────────────────────────────────┐
│                    Cangjie 应用                              │
├─────────────────────────────────────────────────────────────────┤
│  Public API Layer                                            │
│  - DistributedKVStore                                          │
│  - RdbStore                                                    │
│  - Preferences                                                  │
├─────────────────────────────────────────────────────────────────┤
│  FFI Layer (Foreign Function Interface)                   │
│  - 参数类型转换（Cangjie <-> C）                             │
│  - 内存管理（malloc/free）                                   │
│  - 字符串编码转换                                            │
├─────────────────────────────────────────────────────────────────┤
│  Native Layer (C++ System Services)                   │
│  - 数据加密/解密                                            │
│  - 文件 I/O                                                   │
│  - 数据库操作                                                │
└─────────────────────────────────────────────────────────────────┘
```

## 攻击面分析

### 1. FFI 边界

**攻击面**: FFI (Foreign Function Interface) 层

**风险描述**:
- Cangjie 与 C 之间的参数类型转换可能导致类型混淆或截断
- 内存分配和释放的不一致可能导致内存泄漏或双重释放
- 指针操作可能引入空指针解引用

**证据**:
- ohos/data/distributed_kv_store/distributed_kv_store_ffi.cj:229-316 - @C struct 和类型转换
- ohos/data/preferences/preferences_ffi.cj:64-172 - CPreferencesValueType 类型映射
- ohos/data/relational_store/relational_store_ffi.cj:430-535 - RetValueType 类型转换

**影响**: 中等

### 2. 输入验证攻击面

**攻击面**: 用户输入数据

**风险描述**:
- Key/Value 超长验证不充分可能导致缓冲区溢出
- SQL 注入（RDB 模块的 executeSql）
- 路径遍历（数据库文件路径）

**证据**:
- distributed_kv_store_common.cj:41-86 - MAX_KEY_LENGTH, MAX_VALUE_LENGTH 定义
- preferences_options.cj:141-150 - MAX_KEY_LENGTH, MAX_VALUE_LENGTH 定义
- relational_store.cj:295-361 - executeSql 方法未展示参数验证逻辑

**影响**: 高

### 3. 权限控制攻击面

**攻击面**: 访问控制机制

**风险描述**:
- Access Token 验证仅在测试代码中存在，实际应用可能未强制执行
- Bundle Name 验证依赖外部 C++ 层，Wrapper 层无二次验证
- 数据访问未进行细粒度权限检查

**证据**:
- test/*/unittest_engine.cj:924-962 - 仅测试代码中包含 getRequiredPermissions
- distributed_kv_store.cj:64-79 - KVManagerConfig 仅存储 bundleName，无验证
- preferences.cj:80 - Preferences 类无权限检查代码

**影响**: 高

### 4. 内存安全攻击面

**攻击面**: 内存管理

**风险描述**:
- FFI 层使用 unsafe 代码块进行指针操作
- C 结构体的 free() 方法可能未在所有代码路径中调用
- 资源管理依赖 RAII 模式，异常时可能导致资源泄漏

**证据**:
- distributed_kv_store_ffi.cj:298-311 - CKVValueType.free() 方法
- preferences_ffi.cj:143-171 - CPreferencesValueType.free() 方法
- relational_store_ffi.cj:512-534 - RetValueType.free() 方法

**影响**: 高

### 5. 数据加密攻击面

**攻击面**: 加密存储

**风险描述**:
- Security Level 依赖用户配置，未强制最小化原则
- 加密密钥通过参数传递，可能在日志或异常中泄露
- CryptoParam 初始化后未立即清零密钥

**证据**:
- distributed_kv_store_common.cj:217-278 - KVSecurityLevel 枚举定义
- relational_store_common.cj:78-171 - CryptoParam 类定义（encryptionKey 未清零）
- distributed_kv_store_ffi.cj:369-395 - COptions 结构体包含 encrypt 选项

**影响**: 中等

### 6. 敏感信息泄露攻击面

**攻击面**: 日志和异常信息

**风险描述**:
- HiLog 日志可能包含敏感数据（如 Key, Value）
- 异常消息可能泄露内部实现细节
- 错误码可能暴露系统状态

**证据**:
- distributed_kv_store.cj, preferences.cj, relational_store.cj - 大量使用 HiLog 记录错误
- 各模块的错误处理可能包含详细的内部状态信息

**影响**: 低

### 7. 并发安全攻击面

**攻击面**: 多线程访问

**风险描述**:
- 跨线程 FFI 调用可能导致竞态条件
- 资源生命周期管理未显式考虑并发场景
- 远程回调（如 onDataChange）可能在不同线程执行

**证据**:
- single_kvstore.cj:299-326 - onDataChange 回调注册（无线程安全说明）
- preferences.cj:238-330 - on 回调注册（无线程安全说明）

**影响**: 中等

## 可被利用点

### 1. SQL 注入漏洞（高危）

**位置**: ohos/data/relational_store/relational_store.cj:295-361

**问题描述**:
executeSql 和 querySql 方法直接执行用户提供的 SQL 语句，未展示明确的参数化查询或输入验证机制。

**证据**: relational_store.cj:295-361 - executeSql 函数实现
```cangjie
/**
 * Executes the SQL statement.
 */
public func executeSql(sql: String): Unit {
    unsafe {
        let code = FfiOHOSRelationalStoreExecuteSql(getID(), sqlCString.value, errCodePtr)
        throwIfNotSuccess(code)
    }
}
```

**利用场景**:
1. 恶意应用构造包含 SQL 注入的语句（如 `; DROP TABLE users; --`）
2. 通过 executeSql 执行注入的 SQL
3. 可能导致数据库被删除、篡改或敏感数据泄露

**影响**:
- 数据完整性破坏
- 数据泄露
- 拒绝服务

**修复建议**:
1. 提供参数化查询 API（如 executeSqlWithParams）
2. 在 Wrapper 层添加 SQL 注入检测和过滤
3. 文档中明确警告用户不要拼接 SQL 字符串
4. 考虑使用 querySql 而非 executeSql 作为推荐方式

### 2. 缓冲区溢出漏洞（高危）

**位置**: ohos/data/distributed_kv_store/distributed_kv_store_ffi.cj:239-276 (CKVValueType)

**问题描述**:
String 类型值转换为 CString 时使用 LibC.mallocCString，未验证输入字符串长度是否超过分配的缓冲区大小。

**证据**: distributed_kv_store_ffi.cj:248-250
```cangjie
case StringValue(v) =>
    string = unsafe { LibC.mallocCString(v).getChars() }
    tag = 0
```

**利用场景**:
1. 恶意应用传入超长字符串（超过 MAX_VALUE_LENGTH = 4194303）
2. LibC.mallocCString 分配的缓冲区可能不足
3. 写入时导致缓冲区溢出，可能覆盖相邻内存

**影响**:
- 内存损坏
- 程序崩溃
- 可能的任意代码执行

**修复建议**:
1. 在 Cangjie 层验证字符串长度 <= MAX_VALUE_LENGTH
2. 在 FFI 层使用安全的字符串复制函数（如 strncpy）
3. 在 @C struct 的 init 中添加长度检查
4. 考虑使用安全的字符串处理库（如 libsafe_strings）

### 3. 空指针解引用漏洞（中危）

**位置**: ohos/data/preferences/preferences_ffi.cj:64-108 (CPreferencesValueType)

**问题描述**:
toValueType() 方法在解析 tag 时，如果 tag 值不在预期范围内（0-6），返回 Integer(-1)，可能导致后续解引用错误值。

**证据**: preferences_ffi.cj:112-137
```cangjie
static func parse(cPreferencesValueType: CPreferencesValueType): PreferencesValueType {
    match (cPreferencesValueType.tag) {
        case 0 => return PreferencesValueType.Integer(cPreferencesValueType.integer)
        // ...
        case _ =>
            PREFERENCES_LOG.info("WARNING: Shouldn't have walked here")
            return PreferencesValueType.Integer(-1)  // 危险：无效的 Integer 值
    }
}
```

**利用场景**:
1. 内存损坏导致 tag 值异常（如值为 255）
2. 解析返回无效的 PreferencesValueType
3. 应用使用该值时可能导致逻辑错误或崩溃

**影响**:
- 程序异常
- 逻辑错误
- 潜在的拒绝服务

**修复建议**:
1. 为异常 tag 值抛出 BusinessException 而非返回默认值
2. 使用 Option<PreferencesValueType> 表示可能失败的解析
3. 在 @C struct 的 init 中验证 tag 值的有效范围
4. 添加单元测试覆盖所有异常情况

### 4. 密钥泄露漏洞（中危）

**位置**: ohos/data/relational_store/relational_store_common.cj:78-171 (CryptoParam)

**问题描述**:
CryptoParam 的 encryptionKey 字段在传递给底层后未立即清零，可能因日志、异常处理或内存 dump 导致密钥泄露。

**证据**: relational_store_common.cj:78-107
```cangjie
public class CryptoParam {
    public var encryptionKey: Array<UInt8>
    
    public init(encryptionKey: Array<UInt8>, ...) {
        this.encryptionKey = encryptionKey
        // 问题：encryptionKey 未在 init 后清零
    }
}
```

**利用场景**:
1. 应用使用加密数据库
2. 发生异常或日志记录时，encryptionKey 内容可能被捕获
3. 攻击者通过日志、核心转储或调试接口获取密钥
4. 解密数据库内容

**影响**:
- 数据机密性破坏
- 数据泄露

**修复建议**:
1. 在 CryptoParam 的 ~析构函数中清零 encryptionKey
2. 使用 safeZero() 或类似的内存清零函数
3. 避免在日志中输出加密相关数据
4. 考虑使用密钥派生（KDF）而非直接传递密钥

### 5. 权限绕过漏洞（高危）

**位置**: ohos/data/distributed_kv_store/distributed_kv_store.cj:64-79 (KVManagerConfig)

**问题描述**:
KVManagerConfig 接受任意 bundleName 字符串，Wrapper 层未验证该 bundleName 是否与调用应用的包名一致。

**证据**: distributed_kv_store.cj:163-196
```cangjie
public class KVManagerConfig {
    public var bundleName: String
    public var context: BaseContext
    
    public init(context: BaseContext, bundleName: String) {
        this.context = context
        this.bundleName = bundleName
        // 问题：无验证逻辑
    }
}
```

**利用场景**:
1. 恶意应用传入他人的 bundleName（如 "com.victim.app"）
2. 成功创建 KVManager，可能访问受害者的数据
3. 读取、修改或删除受害者的数据

**影响**:
- 数据泄露
- 数据篡改
- 跨应用数据访问

**修复建议**:
1. 在 KVManagerConfig.init 中验证 bundleName 与 context 对应的包名一致
2. 从 context 中获取真实的 bundleName 而非接受用户输入
3. 在底层 FFI 调用前进行二次验证
4. 添加单元测试验证权限检查逻辑

## 检查范围与局限性

### 已检查的模块

✅ **已检查**:
- ohos/data/distributed_kv_store/
- ohos/data/relational_store/
- ohos/data/preferences/
- ohos/data/data_share_predicates/
- ohos/data/values_bucket/

### 未检查的范围

❌ **未检查**:
- 底层 C++ 组件实现（distributeddatamgr_kv_store, distributeddatamgr_relational_store 等）
- Ability 框架的权限验证机制（ability_cangjie_wrapper）
- IPC 通信的安全性（Binder/HIDL）
- 文件系统的访问控制（数据目录权限）

### 局限性说明

1. **静态分析**: 本评审基于静态代码分析，未包含动态测试或模糊测试
2. **FFI 假设**: 假设底层 C++ 函数正确实现，但未审查其内部实现
3. **测试代码排除**: 未审查 test/ 目录下的测试代码，虽然其中包含一些安全最佳实践
4. **Mock 实现**: 未审查 Mock 实现的安全性（不用于生产环境）

## 安全建议

### 开发者最佳实践

1. **输入验证**:
   - 始终验证输入参数的长度和格式
   - 使用 API 提供的常量（如 MAX_KEY_LENGTH）进行边界检查

2. **使用安全 API**:
   - 优先使用参数化查询 API（如 querySql）而非直接 SQL（executeSql）
   - 使用类型安全的 Query/Predicates 构造器

3. **错误处理**:
   - 不要在日志中输出敏感信息（Key, Value, 密钥）
   - 使用通用的错误消息，避免泄露内部实现细节

4. **权限管理**:
   - 遵循最小权限原则
   - 定期审查应用的权限声明

### 维护者行动项

1. **短期（1-2 周）**:
   - [ ] 添加 KVManagerConfig 的 bundleName 验证
   - [ ] 修复 parse() 方法的异常 tag 处理（使用异常而非默认值）
   - [ ] 清零加密密钥（CryptoParam.free()）

2. **中期（1-2 月）**:
   - [ ] 审查所有 FFI 类型转换，添加长度检查
   - [ ] 实现 SQL 注入检测和过滤
   - [ ] 添加线程安全文档和测试

3. **长期（3-6 月）**:
   - [ ] 进行安全模糊测试
   - [ ] 进行渗透测试
   - [ ] 建立安全 CI/CD 检查

## 相关跳转

- [01_Project_Boundaries.md](01_Project_Boundaries.md) - 项目边界和约束
- [05_Internal_API.md](05_Internal_API.md) - FFI 层实现
- [04_Public_API.md](04_Public_API.md) - API 使用指南

## 参考资料

- [OpenHarmony 安全指南](https://docs.openharmony.cn/security/)
- [Cangjie 语言安全指南](https://developer.huawei.com/consumer/cn/doc/cangjie-guidelines-V5)
- [OWASP 移动应用安全测试指南](https://owasp.org/www-project-mobile-app-security/)
