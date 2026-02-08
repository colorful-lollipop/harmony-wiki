# 安全风险评审

本文档对 `security_cangjie_wrapper` 进行安全风险分析，包括攻击面、信任边界、可利用点和修复建议。

## 评审范围

| 范围 | 说明 |
|------|------|
| 模块 | `security_cangjie_wrapper` 所有代码 |
| 依赖 | `crypto_framework`、`huks` Native 实现 |
| SysCap | `SystemCapability.Security.*` |
| 排除 | `test/` 目录、mock 目录 |

**评审依据**：代码扫描 + 架构分析

---

## 攻击面分析

### 外部输入点

| 输入类型 | 来源 | 处理文件 |
|----------|------|----------|
| JS/Cangjie API 参数 | 上层应用 | `cipher.cj`、`huks_session.cj`、`huks_key_item.cj` |
| 文件路径（keyAlias） | 上层应用 | `huks_session.cj`、`huks_key_item.cj` |
| 二进制数据（DataBlob） | 上层应用 | `cj_crypto_native.cj` |
| 配置参数（HuksOptions） | 上层应用 | `huks_struct.cj`、`huks_session.cj` |

### 敏感操作

| 操作 | 风险等级 | 说明 |
|------|----------|------|
| 密钥生成 | 🔴 高 | 生成加密密钥 |
| 密钥存储 | 🔴 高 | HUKS 密钥存储 |
| 加密操作 | 🟡 中 | 数据加解密 |
| 摘要计算 | 🟢 低 | 消息摘要 |
| 随机数生成 | 🟢 低 | 安全随机数 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      信任边界边界                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Cangjie 应用层（不可信）                                 │   │
│  │  - 输入验证：API 参数校验                                 │   │
│  │  - 类型检查：Cangjie Runtime                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│              参数校验 / 类型转换 / 边界检查                      │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Wrapper 层（半可信）                                    │   │
│  │  - FFI 调用：foreign 声明                                │   │
│  │  - 数据转换：HcfBlob、OhosHksBlob                        │   │
│  │  - 错误码映射：BusinessException                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│              IPC / FFI 调用 / 序列化                            │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Native 层（可信）                                       │   │
│  │  - crypto_framework：加密实现                            │   │
│  │  - huks：密钥管理实现                                    │   │
│  │  - TEE/SE：硬件安全单元（可选）                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 可利用点分析

### 🔴 高风险

#### 1. 密钥别名注入（Key Alias Injection）

**证据**：`huks_key_item.cj:48-49`
```cj
if (keyAlias.isEmpty()) {
    throw BusinessException(HuksExceptionErrCode.HuksErrCodeIllegalArgument.getValue(), "key alias is empty")
}
```

**问题**：仅检查空值，未检查路径遍历字符

**触发**：`hasKeyItem(keyAlias: "../etc/passwd", options)`

**影响**：
- 访问非预期密钥
- 密钥枚举攻击

**修复建议**：
```cj
// 添加路径校验
if (keyAlias.contains("..") || keyAlias.contains("/") || keyAlias.contains("\\")) {
    throw BusinessException(HuksExceptionErrCode.HuksErrCodeIllegalArgument.getValue(), "invalid key alias")
}
```

**状态**：`TODO(需确认)` - 需验证 Native 层是否已有防护

---

#### 2. 内存复制越界（Memcpy Overflow）

**证据**：`cj_crypto_native.cj:40-41`
```cj
memcpy_s(this.head, this.size, cp.pointer, this.size)
```

**问题**：`this.size` 作为 `destMax`，若后续写入超出预期大小可能导致溢出

**触发**：构造超大 `DataBlob` 输入

**影响**：
- 堆溢出
- 远程代码执行

**修复建议**：
```cj
// 添加大小限制
const MAX_BLOB_SIZE: UInt32 = 1024 * 1024  // 1MB
if (this.size > MAX_BLOB_SIZE) {
    throw BusinessException(17620001, "blob too large")
}
```

**状态**：`TODO(需确认)` - 需确认 Native 层 `memcpy_s` 已有保护

---

#### 3. 空指针解引用（Null Pointer Dereference）

**证据**：`huks_session.cj:62-67`
```cj
try {
    handle = unsafe { OhosHksBlob.malloc(HANDLE_SIZE) }
} catch (e: Exception) {
    unsafe { LibC.free(keyAliasCstr) }
    throw e
}
```

**问题**：`malloc` 失败返回空指针，后续 FFI 调用可能崩溃

**触发**：`malloc` 分配失败（内存耗尽）

**影响**：
- 进程崩溃
- 拒绝服务

**修复建议**：
```cj
try {
    handle = unsafe { OhosHksBlob.malloc(HANDLE_SIZE) }
    if (handle.head.isNull()) {
        throw BusinessException(17620001, "malloc failed")
    }
} catch (e: Exception) {
    // ...
}
```

**状态**：`TODO(需确认)` - 需确认 `safeMalloc` 返回值处理

---

### 🟡 中风险

#### 4. 整数溢出（Integer Overflow）

**证据**：`cj_crypto_native.cj:38`
```cj
this.head = safeMalloc<UInt8>(count: blob.data.size)
```

**问题**：`blob.data.size` 为 `Int64`，转换为 `UIntNative` 可能溢出

**触发**：`blob.data.size > UIntNative.MAX`

**影响**：
- 分配极小内存
- 堆溢出

**修复建议**：
```cj
const MAX_ALLOC_SIZE: Int64 = 1024 * 1024 * 1024  // 1GB
if (blob.data.size > MAX_ALLOC_SIZE) {
    throw BusinessException(401, "data too large")
}
```

**状态**：`TODO(需确认)` - 需验证实际场景下数据大小限制

---

#### 5. 资源未释放（Resource Leak）

**证据**：`huks_session.cj:91-96`
```cj
try {
    if (retCode == HKS_SUCCESS) {
        // ...
    } else {
        throw hksCodeToException(retCode)
    }
} finally {
    unsafe {
        LibC.free(keyAliasCstr)
        handle.free()
        token.free()
    }
}
```

**问题**：虽然有 `finally` 块，但异常场景下可能跳过清理

**触发**：复杂嵌套异常

**影响**：
- 内存泄漏
- 文件句柄泄漏

**修复建议**：
- 当前代码结构合理，建议添加日志确认清理执行

**状态**：`✅ 低风险` - 代码已正确使用 try-finally

---

#### 6. 随机数熵不足（Insufficient Entropy）

**证据**：`random.cj:104-121`
```cj
public func setSeed(seed: DataBlob): Unit {
    // ...
    FfiOHOSSetSeed(getID(), cp, inout errCode)
}
```

**问题**：用户可设置随机数种子，可能降低随机性

**触发**：应用设置弱种子

**影响**：
- 可预测随机数
- 密钥泄露

**修复建议**：
- 不建议应用调用 `setSeed()`
- 文档明确警告：仅用于测试用途

**状态**：`✅ 已标注` - 建议在 API 文档中明确警告

---

### 🟢 低风险

#### 7. 信息泄露（Information Disclosure）

**证据**：`cj_crypto_enum.cj:83-92`
```cj
public func toString(): String {
    return match (this) {
        case InvalidParams => "Parameter error."
        case NotSupport => "Capability not supported."
        // ...
    }
}
```

**问题**：错误信息过于详细，可能泄露内部状态

**触发**：捕获异常并显示消息

**影响**：
- 内部路径泄露
- 系统信息暴露

**评估**：`✅ 低风险` - 错误信息已泛化，无敏感信息

---

#### 8. 竞态条件（Race Condition）

**证据**：`huks_key_item.cj:364-391`（generateKeyItem）
```cj
public func generateKeyItem(keyAlias: String, options: HuksOptions): Unit {
    // ...
}
```

**问题**：并发生成相同 keyAlias 密钥可能覆盖

**触发**：多线程并发调用

**影响**：
- 密钥意外覆盖
- 状态不一致

**评估**：`✅ 低风险` - HUKS Native 层应有互斥保护

---

## 安全建议汇总

### 立即修复

| 优先级 | 问题 | 建议 |
|--------|------|------|
| P0 | 密钥别名未校验路径字符 | 添加 `../` `/` `\` 检查 |
| P0 | 内存复制无大小限制 | 添加 MAX_BLOB_SIZE 检查 |
| P0 | malloc 失败未检查空指针 | 添加 null 检查 |

### 长期改进

| 优先级 | 问题 | 建议 |
|--------|------|------|
| P1 | 缺少安全编码规范 | 添加 SAST 检查 |
| P1 | 无安全测试用例 | 添加模糊测试 |
| P2 | 文档缺少安全警告 | 完善 API 文档安全章节 |

---

## 依赖安全评估

### crypto_framework

| 组件 | 版本 | 漏洞状态 |
|------|------|----------|
| OpenSSL | 依赖决定 | 需单独评估 |

**建议**：使用 `hb env` 查看具体依赖版本

### huks

| 组件 | 版本 | 漏洞状态 |
|------|------|----------|
| HUKS Core | 系统组件 | 需单独评估 |

---

## 审计结论

### 整体评估

| 维度 | 评级 | 说明 |
|------|------|------|
| 输入验证 | 🟡 中 | 需增强 keyAlias 校验 |
| 内存安全 | 🟡 中 | 需确认边界检查 |
| 错误处理 | 🟢 良好 | 异常机制完善 |
| 依赖安全 | ⚪ 待评估 | 需检查 Native 依赖 |

### 风险总结

| 风险等级 | 数量 | 处理建议 |
|----------|------|----------|
| 🔴 高 | 3 | 需立即修复 |
| 🟡 中 | 3 | 计划修复 |
| 🟢 低 | 2 | 接受 |

---

## 参考标准

- [OpenHarmony 安全开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/security-guidelines.md)
- [CWE-119](https://cwe.mitre.org/data/definitions/119.html) - 内存错误
- [CWE-20](https://cwe.mitre.org/data/definitions/20.html) - 输入验证错误
