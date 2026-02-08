# 安全风险评审 (Security Review)

## 目的

本文档基于代码证据分析 crypto_framework 的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- **目标读者**: 安全审计人员、开发者、安全工程师
- **阅读时长**: 30 分钟

## 攻击面分析

### 1. N-API 接口层

**暴露路径**: `frameworks/js/napi/crypto/`

**攻击面**:
- 参数注入：通过 JS 参数传入恶意数据
- 类型混淆：传递错误类型导致未定义行为
- 内存耗尽：通过大参数触发内存分配失败

**证据**: 
- N-API 绑定文件：`frameworks/js/napi/crypto/src/` 目录
- 参数校验：各 napi 文件中的参数解析逻辑

### 2. Native API 层

**暴露路径**: `frameworks/native/src/`

**攻击面**:
- 缓冲区溢出：通过 `Crypto_DataBlob` 越界访问
- 整数溢出：通过长度参数触发
- 释放后使用：使用已释放的密钥对象

**证据**:
- Native API 实现：`frameworks/native/src/` 目录
- 使用 SecureC：`bounds_checking_function` 依赖

### 3. 框架核心层

**暴露路径**: `frameworks/crypto_operation/`, `frameworks/key/`

**攻击面**:
- 密钥泄露：敏感密钥数据在内存中停留
- 侧信道攻击：通过时间分析推断密钥
- 竞态条件：多线程并发访问同一对象

**证据**:
- 框架实现：`frameworks/crypto_operation/` 目录
- 密钥管理：`frameworks/key/` 目录

### 4. 插件层

**暴露路径**: `plugin/openssl_plugin/`, `plugin/mbedtls_plugin/`

**攻击面**:
- 底层库漏洞：OpenSSL/MbedTLS 的已知漏洞
- 插件加载劫持：替换恶意插件
- 不安全的随机数：弱随机数源

**证据**:
- OpenSSL 插件：`plugin/openssl_plugin/` 目录
- MbedTLS 插件：`plugin/mbedtls_plugin/` 目录

## 信任边界

### 1. 外部输入边界

```
不可信数据
    │
    ├── JS 应用程序
    │   └── N-API 接口
    │       └── 框架核心层
    │           └── 插件层
    │               └── OpenSSL/MbedTLS
    │
    └── 可信操作（加密结果）
```

### 2. 密钥材料边界

```
不可信输入
    │
密钥生成器
    │
    ├── 生成的密钥材料 ← 不可信
    │
密钥操作
    │
    ├── 加密/签名结果 ← 可信（如果密钥可信）
    │
密钥存储
    │
    └── 需要安全存储（HuKS）
```

### 3. 内存边界

```
应用程序内存（不可信）
    │
    ↓
    ↓
框架内存（半可信）
    │
    ├── 密钥内存（敏感）← 需要保护
    │   ├── 清零机制
    │   └── 内存锁定
    │
    ↓
第三方库内存（不可信）
    │
    └── OpenSSL/MbedTLS 堆
```

## 潜在利用点

### 风险 1: 参数校验不足

**严重性**: 高

**位置**: 各 N-API 绑定文件

**描述**: 部分参数校验可能不完整，导致未定义行为。

**证据**:
- 文件：`frameworks/js/napi/crypto/src/napi_cipher.cpp`
- 问题：`HcfBlob` 长度可能为负数（size_t 在某些平台上可能是有符号数）

**可利用路径**:
1. 攻击者传入超大的 `length` 参数
2. 导致 `malloc` 分配失败或整数溢出
3. 触发缓冲区溢出或内存耗尽

**影响**:
- 拒绝服务
- 可能的代码执行（如果有溢出）

**修复建议**:
```c
// 在 frameworks/js/napi/crypto/src/napi_utils.cpp 添加
HcfResult ValidateBlob(HcfBlob *blob) {
    if (blob == NULL || blob->data == NULL) {
        return HCF_INVALID_PARAMS;
    }
    // 检查长度是否合理
    if (blob->len > MAX_BLOB_SIZE) {
        return HCF_INVALID_PARAMS;
    }
    return HCF_SUCCESS;
}
```

### 风险 2: 敏感数据未清除

**严重性**: 高

**位置**: 密钥对象销毁逻辑

**描述**: 密钥对象销毁时，敏感密钥数据可能未完全从内存清除。

**证据**:
- 文件：`interfaces/inner_api/key/sym_key.h`
- 方法：`HcfSymKey.clearMem`
- 实现：`frameworks/key/sym_key_generator.c`

**可利用路径**:
1. 应用生成对称密钥
2. 使用密钥加密数据
3. 销毁密钥对象
4. 攻击者从内存堆转储中提取密钥

**影响**:
- 密钥泄露
- 后续通信解密

**修复建议**:
```c
// 确保密钥清除使用安全内存擦除
HcfResult SecureClearMemory(uint8_t *data, size_t len) {
    volatile uint8_t *p = data;
    while (len--) {
        *p++ = 0;
    }
    // 确保编译器不优化掉
    __sync_synchronize();
    return HCF_SUCCESS;
}

// 在 destroy 中使用
HcfResult HcfSymKeyDestroy(HcfSymKey *self) {
    SecureClearMemory(self->data, self->len);
    free(self->data);
    return HCF_SUCCESS;
}
```

### 风险 3: 弱随机数

**严重性**: 高

**位置**: 随机数生成实现

**描述**: 随机数生成器可能使用不够强的熵源。

**证据**:
- 文件：`plugin/openssl_plugin/crypto_operation/rand/src/rand_openssl.c`
- 方法：`HcfRandSpi.generateRandom`

**可利用路径**:
1. 应用依赖随机数生成器生成 IV、Nonce
2. 攻击者预测随机数
3. 重放攻击或密钥恢复

**影响**:
- 加密被破解
- 签名被伪造

**修复建议**:
```c
// 启用硬件熵源
HcfResult HcfRandEnableHardwareEntropy(HcfRand *self) {
    // 检查系统是否支持硬件熵
    if (IsHardwareEntropyAvailable()) {
        self->useHardwareEntropy = true;
        return HCF_SUCCESS;
    }
    // 否则使用系统熵池
    return HCF_NOT_SUPPORT;
}
```

### 风险 4: 竞态条件

**严重性**: 中

**位置**: 密钥和加密操作对象

**描述**: 多线程并发访问同一对象时可能存在竞态。

**证据**:
- 文件：`frameworks/crypto_operation/cipher.c`
- 问题：`HcfCipher` 对象可能被多个线程并发访问

**可利用路径**:
1. 线程 A 调用 `cipher.init()`
2. 线程 B 同时调用 `cipher.update()`
3. 导致未定义行为或崩溃

**影响**:
- 崩溃
- 数据损坏

**修复建议**:
```c
// 添加互斥锁保护
typedef struct HcfCipher {
    HcfObjectBase base;
    pthread_mutex_t lock;  // 添加锁
    // ... 其他字段
} HcfCipher;

HcfResult HcfCipherInit(HcfCipher *self, ...) {
    pthread_mutex_lock(&self->lock);
    // ... 初始化逻辑
    pthread_mutex_unlock(&self->lock);
    return HCF_SUCCESS;
}
```

### 风险 5: 不安全的密钥派生

**严重性**: 中

**位置**: 密钥派生函数

**描述**: 密钥派生可能使用不安全的默认参数。

**证据**:
- 文件：`plugin/openssl_plugin/crypto_operation/kdf/src/kdf_openssl.c`
- 方法：`PBKDF2`, `HKDF`

**可利用路径**:
1. 应用使用弱口令
2. 使用低迭代次数
3. 攻击者进行离线暴力破解

**影响**:
- 密钥被暴力破解

**修复建议**:
```c
// 强制最小迭代次数
#define PBKDF2_MIN_ITERATIONS 10000

HcfResult ValidateKdfParams(HcfKdfParams *params) {
    if (params->iterations < PBKDF2_MIN_ITERATIONS) {
        LOGE("Iterations too small: %d", params->iterations);
        return HCF_INVALID_PARAMS;
    }
    // 检查盐值长度
    if (params->salt->len < 16) {
        LOGE("Salt too short: %d", params->salt->len);
        return HCF_INVALID_PARAMS;
    }
    return HCF_SUCCESS;
}
```

## 检查范围

### 已检查的内容

- ✅ N-API 参数校验
- ✅ 密钥内存管理
- ✅ 随机数生成
- ✅ 密钥派生参数
- ✅ 插件层接口
- ✅ 缓冲区边界检查（通过 SecureC）
- ✅ 错误处理和传播

### 未检查的内容

- ❌ OpenSSL/MbedTLS 库内部实现（假设这些库是安全的）
- ❌ 硬件加密加速（由 HuKS 负责）
- ❌ 应用层密钥存储（由 HuKS 负责）
- ❌ 侧信道攻击防护（需要专门的密码学分析）
- ❌ 模糊测试结果（根据要求忽略 test 目录）

## 安全加固建议

### 1. 参数校验强化

- 所有外部输入参数必须验证
- 检查数据范围（长度、大小）
- 检查数据格式（PEM、DER）
- 检查空值和无效值

### 2. 内存安全管理

- 使用安全内存擦除函数
- 密钥对象销毁时必须清除敏感数据
- 避免敏感数据在堆上停留过久
- 考虑使用内存锁定机制

### 3. 线程安全

- 对共享对象添加互斥锁保护
- 避免全局变量
- 使用线程安全的随机数生成器

### 4. 随机数强化

- 默认启用硬件熵源
- 检查随机数质量
- 避免可预测的随机数

### 5. 密钥派生安全

- 强制最小迭代次数
- 使用足够的盐值长度
- 避免默认参数
- 提供安全配置指南

### 6. 依赖库安全

- 使用最新版本的 OpenSSL/MbedTLS
- 及时更新已知漏洞修复
- 配置安全的编译选项

## 相关跳转

- **架构设计**: [03_Architecture.md](03_Architecture.md)
- **对外 API**: [04_External_API.md](04_External_API.md)
- **目录结构**: [02_Directory_Structure.md](02_Directory_Structure.md)

## 更新记录

- **2026-02-06**: 创建文档，基于代码安全分析生成

## TODO

- [ ] 补充侧信道攻击分析
- [ ] 添加更详细的风险评估（CVSS）
- [ ] 补充模糊测试结果分析
- [ ] 添加密钥生命周期安全建议
