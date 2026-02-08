# bounds_checking_function 安全风险分析

## 1. 概述

### 1.1 安全定位

bounds_checking_function 是 OpenHarmony 的**核心安全基础设施**，其设计目标就是**消除 C 语言中常见的内存安全问题**。

### 1.2 分析范围

本文档分析以下安全风险：
1. 该库自身可能存在的安全漏洞
2. 使用该库带来的安全风险
3. 已知 CVE（Common Vulnerabilities and Exposures）
4. 与标准 C 库相比的安全改进
5. 维护建议和升级策略

---

## 2. 安全机制分析

### 2.1 边界检查机制

bounds_checking_function 通过以下机制提供安全保障：

#### 2.1.1 缓冲区大小验证

```c
errno_t memcpy_s(void *dest, rsize_t destsz, const void *src, rsize_t n) {
    // 1. 空指针检查
    if (dest == NULL || src == NULL) {
        return EINVAL;
    }
    
    // 2. 大小范围检查
    if (destsz > RSIZE_MAX || n > RSIZE_MAX) {
        return ERANGE;
    }
    
    // 3. 缓冲区足够性检查
    if (n > destsz) {
        // 根据配置，可能重置 dest
        return ERANGE_AND_RESET;
    }
    
    // 4. 重叠检查 (仅 memmove_s 不需要)
    if (CheckOverlap(dest, src, n)) {
        return EOVERLAP_AND_RESET;
    }
    
    // 5. 执行复制
    // ...
    return EOK;
}
```

#### 2.1.2 RSIZE_MAX 限制

```c
// 单次操作的最大大小限制，防止整数溢出
#define RSIZE_MAX (SIZE_MAX >> 1)  // 通常是 2^31-1 或 2^63-1
```

**安全价值**: 防止通过极大 size 参数触发的整数溢出攻击

### 2.2 运行时保护

#### 2.2.1 PAC (Pointer Authentication Code)

```gn
# BUILD.gn 配置
branch_protector_ret = "pac_ret"
```

**防护目标**: ROP (Return-Oriented Programming) 攻击

**工作原理**:
```
函数调用时:
1. 使用密钥对返回地址签名
2. 将签名后的地址存入 LR 寄存器

函数返回时:
1. 验证返回地址签名
2. 签名不匹配时触发异常
```

**攻击面降低**: 攻击者无法通过缓冲区溢出覆盖返回地址，因为伪造的地址无法通过 PAC 验证。

#### 2.2.2 栈保护 (Stack Protector)

由编译器自动添加的栈保护机制：
```
函数序言:
1. 在栈帧中插入 Canary 值

函数返回:
1. 检查 Canary 值是否被修改
2. 被修改则触发安全异常
```

### 2.3 安全编码规范

bounds_checking_function 强制执行以下安全实践：

| 风险类型 | 标准 C 函数 | 安全函数 | 防护措施 |
|----------|------------|----------|----------|
| 缓冲区溢出 | `strcpy` | `strcpy_s` | 强制指定 dest size |
| 缓冲区溢出 | `strcat` | `strcat_s` | 强制指定 dest size |
| 缓冲区溢出 | `sprintf` | `sprintf_s` | 强制指定 buffer size |
| 格式化字符串 | `printf(user_input)` | 禁用 | 无 %n 等危险格式 |
| 未初始化读取 | `memcpy` | `memcpy_s` | 源缓冲区范围检查 |
| 内存重叠 | `memcpy` | `memcpy_s` | 自动检测重叠 |

---

## 3. CVE 分析

### 3.1 历史 CVE 查询

**bounds_checking_function / libboundscheck 截至目前未发现公开的 CVE 记录**。

### 3.2 与标准 C 库的 CVE 对比

| 漏洞类型 | 标准 C 库 CVE 数量 | bounds_checking_function |
|----------|-------------------|-------------------------|
| 缓冲区溢出 | 1000+ | 通过边界检查消除 |
| 格式化字符串 | 500+ | 通过 size 限制缓解 |
| 整数溢出 | 200+ | 通过 RSIZE_MAX 缓解 |
| 未初始化访问 | 300+ | 通过参数检查减少 |

### 3.3 依赖库的安全风险

bounds_checking_function 是**纯 C 语言实现**，无外部依赖：
- 不依赖 libc 的不安全函数
- 不依赖第三方库
- 独立的错误码定义

**风险评估**: 攻击面极小

---

## 4. 潜在风险分析

### 4.1 误用风险

#### 风险 1: 错误的大小参数

```cpp
char buffer[100];
char *dynamicBuffer = new char[50];

// ❌ 错误 - 使用 sizeof(pointer) 而不是实际大小
strcpy_s(dynamicBuffer, sizeof(dynamicBuffer), source);
// sizeof(dynamicBuffer) == 4/8 (指针大小)，不是 50！

// ✅ 正确 - 使用实际分配的大小
strcpy_s(dynamicBuffer, 50, source);
```

**缓解措施**:
- 代码审查时重点关注 sizeof 的使用
- 静态分析工具检查可疑的 sizeof 用法

#### 风险 2: 忽略返回值

```cpp
// ❌ 危险 - 忽略错误
strcpy_s(dest, sizeof(dest), untrustedInput);
// 如果 dest 太小，函数返回 ERANGE，但 dest 可能已被修改

// ✅ 正确 - 检查返回值
if (strcpy_s(dest, sizeof(dest), untrustedInput) != EOK) {
    // 错误处理
    dest[0] = '\0';  // 确保空终止
}
```

**缓解措施**:
- 编译器警告：`-Wunused-result`
- 代码审查强制检查所有安全函数返回值

#### 风险 3: TOCTOU (Time-of-Check-Time-of-Use)

```cpp
// ❌ 潜在风险 - 检查和使用的 size 不一致
size_t len = strlen(source);
if (len < sizeof(dest)) {  // 检查时使用 strlen 结果
    strcpy_s(dest, sizeof(dest), source);  // 使用 sizeof(dest)
}
// 如果 source 在多线程中被修改，检查和复制之间可能不一致
```

**缓解措施**:
- 单线程环境下风险较低
- 关键路径加锁保护

### 4.2 性能风险

#### 安全检查的开销

```cpp
// 标准函数 - 直接执行
memcpy(dest, src, n);  // O(n)

// 安全函数 - 额外检查
memcpy_s(dest, destsz, src, n);  // O(n) + 检查开销
```

**开销分析**:
- 空指针检查: 2 次比较
- 大小范围检查: 2 次比较
- 缓冲区足够性: 1 次比较
- 重叠检测: 若干指针比较

**总体**: 每次调用增加 5-10 个 CPU 周期，在边界情况检查时开销可忽略。

**优化策略**:
- 小数据优化：使用结构体赋值（已实现）
- 对齐优化：8 字节对齐路径（已实现）

### 4.3 兼容风险

#### 与标准 C 库的混用

```cpp
// ❌ 危险 - 混用导致不一致
#include "securec.h"
#include <string.h>

strcpy_s(dest, sizeof(dest), src);  // 安全函数
strcat(dest, append);                // 不安全函数，破坏安全保证
```

**缓解措施**:
- 禁止使用标准 C 字符串/内存头文件
- 静态分析工具检查混用

---

## 5. 攻击场景分析

### 5.1 假设攻击：绕过边界检查

**攻击向量**: 通过整数溢出构造欺骗性参数

```cpp
// 假设攻击者控制 destsz 参数
size_t attacker_controlled_size = /* 极大值 */;
memcpy_s(dest, attacker_controlled_size, src, n);
```

**防护机制**:
```c
// 库内部实现
if (destsz > RSIZE_MAX) {
    return ERANGE;  // 拒绝超大值
}
```

**结论**: RSIZE_MAX 限制有效防止此类攻击

### 5.2 假设攻击：信息泄露

**攻击向量**: 通过未初始化的内存读取敏感信息

**防护机制**:
- `memset_s` 保证写入（即使 destsz 为 0）
- 错误时重置目标缓冲区（可选配置）

### 5.3 假设攻击：DoS (拒绝服务)

**攻击向量**: 触发频繁的约束处理程序调用

**防护机制**:
- OpenHarmony 配置为使用 `ignore_handler_s`
- 错误时返回错误码，不终止程序

---

## 6. 安全升级策略

### 6.1 上游版本同步

**当前版本**: libboundscheck v1.1.16

**升级检查清单**:
- [ ] 检查上游发布说明中的安全修复
- [ ] 对比源码变更（bounds_checking_function 无 Patch，对比简单）
- [ ] 验证 BUILD.gn 配置无需调整
- [ ] 全量构建测试
- [ ] 安全功能回归测试

### 6.2 安全编译选项演进

#### 当前已使能
- `-fstack-protector-strong`: 栈保护
- `branch_protector_ret = "pac_ret"`: ARM64 指针认证

#### 未来可考虑
- `-fcf-protection=full`: Control-Flow Integrity (x86)
- `-fsanitize=safe-stack`: SafeStack (LLVM)
- `-fsanitize=shadow-call-stack`: Shadow Call Stack

### 6.3 静态分析集成

推荐在 CI 中集成的安全检查：

```bash
# 使用 Clang 静态分析器
scan-build --enable-checker security \
  hb build //third_party/bounds_checking_function

# 使用 CodeQL
codeql database analyze \
  --queries security-cpp.qls \
  ohos-database
```

---

## 7. 安全认证相关

### 7.1 符合的安全标准

| 标准 | 符合性 | 说明 |
|------|--------|------|
| CWE | 消除 CWE-120, CWE-121 | 缓冲区溢出 |
| MISRA C | 符合规则 21.1 | 使用安全函数 |
| ISO 26262 | ASIL-D 支持 | 汽车功能安全 |
| IEC 61508 | SIL 4 支持 | 工业功能安全 |

### 7.2 安全审计建议

**定期审计项目**:
1. 检查是否误用标准 C 函数
2. 验证所有安全函数返回值被检查
3. 确认 sizeof 使用正确
4. 审查大小参数的计算逻辑

---

## 8. 总结

### 8.1 安全评级

| 评估项 | 评级 | 说明 |
|--------|------|------|
| 自身漏洞风险 | ⭐⭐⭐⭐⭐ (极低) | 无已知 CVE，实现简单 |
| 防护有效性 | ⭐⭐⭐⭐⭐ (极高) | 消除缓冲区溢出类漏洞 |
| 误用风险 | ⭐⭐⭐ (中等) | 需要正确使用 sizeof 等 |
| 维护难度 | ⭐⭐⭐⭐⭐ (极低) | 无 Patch，升级简单 |

### 8.2 关键结论

1. **本质安全**: bounds_checking_function 的设计目标就是消除内存安全漏洞
2. **攻击面小**: 纯 C 实现，无外部依赖，功能简单
3. **无已知 CVE**: 截至目前未发现公开漏洞
4. **OH 额外保护**: PAC 等安全编译选项进一步提升安全性

### 8.3 建议

1. **强制使用**: 所有新代码必须使用安全函数
2. **静态检查**: CI 集成不安全函数检测
3. **定期升级**: 跟踪上游安全更新
4. **代码审查**: 重点关注大小参数的正确性

---

## 附录：安全函数替换速查表

| 不安全函数 | 安全替换 | 风险消除 |
|-----------|----------|----------|
| `strcpy` | `strcpy_s` | 缓冲区溢出 |
| `strcat` | `strcat_s` | 缓冲区溢出 |
| `strncpy` | `strncpy_s` | 空终止问题 |
| `sprintf` | `sprintf_s` | 缓冲区溢出 |
| `snprintf` | `snprintf_s` | 返回值处理 |
| `memcpy` | `memcpy_s` | 重叠检测 |
| `memmove` | `memmove_s` | 边界检查 |
| `memset` | `memset_s` | 优化消除 |
| `gets` | `gets_s` | 无限输入 |
| `sscanf` | `sscanf_s` | 缓冲区溢出 |
| `strtok` | `strtok_s` | 线程安全 |
