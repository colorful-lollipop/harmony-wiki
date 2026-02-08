# 05 - 安全风险评审

## 目的与适用范围

**目的**: 基于代码证据分析 `sys_installer_lite` 的安全风险，识别攻击面、信任边界和可被利用点。

**适用范围**: 安全工程师、需要评估系统安全性的架构师。

**评审范围**: 
- 代码路径: `interfaces/kits/`, `frameworks/source/`, `hals/`
- 排除: `test/` 目录（按规则忽略）

---

## 威胁模型

### 攻击者假设

| 能力级别 | 描述 |
|----------|------|
| **远程攻击者** | 可发送 OTA 升级包数据，但无法物理接触设备 |
| **本地攻击者** | 可物理接触设备，可能有调试接口访问权限 |
| **供应链攻击者** | 可篡改升级包生成流程 |

### 攻击面清单

```
外部输入
    │
    ├──► 升级包数据（网络/本地存储）
    │       ├──► PkgBasicInfo 结构
    │       ├──► Info Component
    │       ├──► Signature (RSA)
    │       └──► Data Components
    │
    ├──► HAL 层接口（厂商实现）
    │       ├──► HotaHalGetPubKey() → 公钥来源
    │       ├──► HotaHalRead/Write() → 存储访问
    │       └──► HotaHalSetBootSettings() → 启动控制
    │
    └──► 系统接口
            ├──► GetIncrementalVersion() → 当前版本
            └──► 系统参数接口

敏感操作
    ├──► Flash 分区写入（破坏固件）
    ├──► 系统重启（DoS）
    ├──► 版本降级（安全漏洞回退）
    └──► 公钥绕过（伪造签名）
```

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                    信任边界图                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────┐         ┌───────────────┐              │
│  │   升级服务器   │◄───────►│    设备端     │              │
│  │  (高信任区)   │  签名   │  (中信任区)   │              │
│  └───────────────┘         └───────┬───────┘              │
│                                    │                      │
│                         ┌──────────▼──────────┐          │
│                         │  sys_installer_lite │          │
│                         │   (本组件边界)      │          │
│                         └──────────┬──────────┘          │
│                                    │                      │
│                         ┌──────────▼──────────┐          │
│                         │   HAL 层 (厂商实现)  │          │
│                         │    (低信任区)        │          │
│                         └──────────┬──────────┘          │
│                                    │                      │
│                         ┌──────────▼──────────┐          │
│                         │  硬件/Flash/Bootloader│         │
│                         │    (硬件信任根)      │          │
│                         └───────────────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘

关键信任边界:
1. 升级包签名验证 → 阻止非法包进入
2. HAL 接口实现 → 依赖厂商正确实现
3. 公钥存储安全 → 防止公钥被篡改
```

---

## 可被利用点分析

### 风险 1: 缓冲区溢出（中等风险）

**位置**: `frameworks/source/updater/hota_updater.c:421-437`

**代码证据**:
```c
static int CopyToDloadCompBuffer(const unsigned char *buffer, unsigned int buffSize)
{
    if (g_currentDloadComp.currentSize + buffSize > MAX_BUFFER_SIZE) {  // 行424: 边界检查
        UpdateStatus(HOTA_FAILED);
        printf("Size is out of range.\r\n");
        return OHOS_FAILURE;
    }
    if (memcpy_s(g_infoCompBuff + g_currentDloadComp.currentSize,  // 行428
                 MAX_BUFFER_SIZE - g_currentDloadComp.currentSize,
                 buffer, buffSize) != EOK) {
        // ...
    }
    // ...
}
```

**分析**:
- ✅ 使用了 `memcpy_s` 安全函数（带长度检查）
- ✅ 有前置边界检查
- ⚠️ 但 `MAX_BUFFER_SIZE` 只有 1500 字节（hota_updater.c:38）
- ⚠️ 如果组件信息超过 1500 字节（如 10 个分区 + 大签名），会导致失败

**触发条件**:
- 构造包含超过 10 个分区的升级包
- 或签名数据异常大

**影响**:
- DoS（拒绝服务）：合法升级被阻止
- 无法利用为代码执行（有 `memcpy_s` 保护）

**修复建议**:
```c
// 建议: 动态分配或增大缓冲区
// 或: 在编译期检查最大可能大小
#define MAX_COMPONENTS 10
#define MAX_INFO_COMP_SIZE (sizeof(PkgBasicInfo) + SIGN_DATA_LEN + \
                            MAX_COMPONENTS * sizeof(ComponentInfo))
```

---

### 风险 2: 整数溢出（低风险）

**位置**: `frameworks/source/updater/hota_updater.c:375-384`

**代码证据**:
```c
if (basicInfo.type == SIGN_ARITHMETIC_RSA2048) {
    g_signDataLen = SIGN_RSA2048_LEN;      // 256
    g_signStartAddr = SIGN_DATA_LEN;        // 640
} else if (basicInfo.type == SIGN_ARITHMETIC_RSA3072) {
    g_signDataLen = SIGN_RSA3072_LEN;      // 384
    g_signStartAddr = SIGN_RSA3072_LEN;    // 384
}
g_infoCompAndSignSize = basicInfo.infoCompSize + SIGN_DATA_LEN;  // 行375
// ...
g_currentDloadComp.remainSize = g_infoCompAndSignSize - sizeof(PkgBasicInfo);  // 行384
```

**分析**:
- ⚠️ `basicInfo.infoCompSize` 来自外部输入（升级包）
- ⚠️ 如果 `infoCompSize` 为 0，则 `remainSize` 为负数（无符号回绕）
- ⚠️ 如果 `infoCompSize` 极大，加法会溢出

**触发条件**:
- 构造恶意 PkgBasicInfo，设置 `infoCompSize = 0` 或极大值

**影响**:
- 可能导致后续缓冲区操作越界
- 或导致无限循环（如果 `remainSize` 回绕为极大值）

**修复建议**:
```c
// 增加范围检查
if (basicInfo.infoCompSize < sizeof(PkgBasicInfo) || 
    basicInfo.infoCompSize > MAX_BUFFER_SIZE - SIGN_DATA_LEN) {
    UpdateStatus(HOTA_FAILED);
    return OHOS_FAILURE;
}
```

---

### 风险 3: TOCTOU 竞态条件（中高风险）

**位置**: `frameworks/source/updater/hota_updater.c:90-103, 509-558`

**代码证据**:
```c
// 全局状态（非线程安全）
static HotaStatus g_otaStatus = HOTA_NOT_INIT;
static CurrentDloadComp g_currentDloadComp = { 0 };
static unsigned char *g_infoCompBuff = NULL;
// ... 其他全局变量

// HotaDefaultWrite 中检查和使用（非原子）
static int HotaDefaultWrite(unsigned char *buffer, unsigned int offset, unsigned int buffSize)
{
    // ...
    if (HotaIsRejected()) {  // 检查状态
        printf("Hota is canceled or verify failed.\r\n");
        return OHOS_FAILURE;
    }
    // 时间窗口：状态可能在此改变
    // ... 使用全局变量 ...
    do {
        // 长时间运行的循环，使用 g_currentDloadComp 等
    } while (tempBuffSize > 0);
}
```

**分析**:
- ❌ 全局变量无保护
- ❌ 检查和使用之间存在时间窗口
- ❌ `HotaCancel()` 可能在 `HotaWrite()` 执行期间被调用

**触发条件**:
- 多线程环境下同时调用 `HotaWrite()` 和 `HotaCancel()`
- 或中断处理程序调用相关函数

**影响**:
- 状态混乱，可能导致：
  - 写入到错误分区
  - 内存重复释放（`g_infoCompBuff`）
  - 验证状态错误

**修复建议**:
```c
// 方案1: 添加互斥锁（如果系统支持）
static pthread_mutex_t g_otaMutex = PTHREAD_MUTEX_INITIALIZER;

int HotaWrite(...) {
    pthread_mutex_lock(&g_otaMutex);
    // ... 原有逻辑 ...
    pthread_mutex_unlock(&g_otaMutex);
}

// 方案2: 文档明确说明非线程安全，要求调用者同步
// 在 hota_updater.h 中添加注释：
/**
 * @warning This function is not thread-safe. 
 *          Caller must ensure no concurrent calls to OTA APIs.
 */
```

---

### 风险 4: 版本校验绕过（中等风险）

**位置**: `frameworks/source/updater/hota_updater.c:163-225`

**代码证据**:
```c
static bool IsLatestVersion(const char *pkgVersion, const char *currentVersion)
{
    char pkgVerCopy[PKG_VERSION_LENGTH] = {0};       // 64 bytes
    char currentVerCopy[PKG_VERSION_LENGTH] = {0};   // 64 bytes
    // ...
    int ret = strcpy_s(pkgVerCopy, PKG_VERSION_LENGTH, pkgVersion);       // 行172
    ret += strcpy_s(currentVerCopy, PKG_VERSION_LENGTH, currentVersion);  // 行173
    if (ret != 0) {
        return false;
    }
    // 使用 strtok_s 解析版本号
    char split[] = ".| ";  // 分隔符包含空格！
    currentVerSplit = strtok_s(currentVerCopy, split, &currentVerTemp);
    // ...
}

static int CheckPkgVersionValid(const char *pkgVersion)
{
    if (pkgVersion == NULL) {
        return OHOS_FAILURE;
    }
    const char *currentVersion = GetIncrementalVersion();
    if (currentVersion == NULL) {
        return OHOS_FAILURE;
    }

    if (!IsLatestVersion(pkgVersion, currentVersion)) {
        printf("pkgVersion is valid\r\n");
        return OHOS_FAILURE;  // 逻辑反了？应该是 invalid
    }
    return OHOS_SUCCESS;
}
```

**分析**:
- ⚠️ 分隔符 `".| "` 包含空格，可能导致解析异常
- ⚠️ 注释与逻辑矛盾：`IsLatestVersion` 返回 false 时打印 "valid"
- ⚠️ `HotaHalCheckVersionValid` 可覆盖默认检查，如果厂商实现不当可能绕过

**触发条件**:
- 构造特殊版本号字符串（如包含空格）
- 或利用 HAL 覆盖实现

**影响**:
- 版本回滚（降级到旧版本，可能利用已知漏洞）

**修复建议**:
```c
// 1. 明确分隔符，避免包含空格
static const char VERSION_DELIMITERS[] = ".";

// 2. 修正日志输出
if (!IsLatestVersion(pkgVersion, currentVersion)) {
    printf("pkgVersion is NOT valid (older or same as current)\r\n");
    return OHOS_FAILURE;
}

// 3. 在 HAL 层要求严格版本检查
```

---

### 风险 5: 签名验证前数据暴露（中等风险）

**位置**: `frameworks/source/updater/hota_updater.c:227-283`

**代码证据**:
```c
static int ParseHotaInfoComponent(unsigned char *infoCompBuffer, unsigned short bufLen)
{
    // ...
    // 1. 先分配内存并拷贝数据
    unsigned char *infoHeaderBuf = (unsigned char *)malloc(bufLen - SIGN_DATA_LEN);
    if (memcpy_s(infoHeaderBuf, bufLen - SIGN_DATA_LEN, infoCompBuffer, 
                 bufLen - SIGN_DATA_LEN) != EOK) {
        // ...
    }
    
    // 2. 验证签名
    if (HotaSignVerify(infoHeaderBuf, bufLen - SIGN_DATA_LEN,
        infoCompBuffer + (bufLen - g_signStartAddr), g_signDataLen)) {
        UpdateStatus(HOTA_FAILED);
        ReportErrorCode(HOTA_DATA_SIGN_CHECK_ERR);
        printf("Verify file failed.\r\n");
        free(infoHeaderBuf);
        return OHOS_FAILURE;
    }
    
    // 3. 签名验证通过后才解析组件表
    // ...
}
```

**分析**:
- ⚠️ 在验证签名之前，数据已经写入 `PARTITION_INFO_COMP` 分区
- ⚠️ 如果 HAL 层的写入操作不可逆，恶意数据可能残留在 Flash

**代码流**:
```
HotaHalWrite(PARTITION_INFO_COMP, infoCompBuffer, 0, bufLen)  // 行275-280
    └── 数据写入 Flash（即使后续验证失败）
HotaSignVerify(...)  // 行248-255
    └── 如果失败，数据已写入
```

**触发条件**:
- 发送恶意升级包，在签名验证失败后查看 Flash

**影响**:
- 信息泄露（组件表结构暴露）
- Flash 磨损（DoS）
- 如果 Bootloader 使用 INFO 分区，可能触发未定义行为

**修复建议**:
```c
// 方案1: 先验证再写入
static int ParseHotaInfoComponent(unsigned char *infoCompBuffer, unsigned short bufLen)
{
    // 1. 先验证签名
    if (HotaSignVerify(...)) {
        return OHOS_FAILURE;
    }
    
    // 2. 验证通过后再写入
    if (HotaHalWrite(PARTITION_INFO_COMP, ...) != OHOS_SUCCESS) {
        return OHOS_FAILURE;
    }
    // ...
}

// 方案2: 使用临时缓冲区，最后原子提交
```

---

### 风险 6: 公钥来源不可信（高风险）

**位置**: `hals/hal_hota_board.h:148-149`, `frameworks/source/verify/hota_verify.c:112,155-159`

**代码证据**:
```c
// hota_verify.c:112
uint8 *keyBuf = HotaHalGetPubKey(&length);
if (keyBuf == NULL) {
    return OHOS_FAILURE;
}

// hota_verify.c:155-159
uint8 *HotaGetPubKey(uint32 *length)
{
    return HotaHalGetPubKey(length);  // 直接透传
}
```

**分析**:
- ⚠️ 公钥由 HAL 层提供，框架层无校验
- ⚠️ 如果厂商 HAL 实现从可写存储读取公钥，攻击者可能替换公钥
- ⚠️ 公钥格式无校验（应为 DER 编码 RSA 公钥）

**攻击场景**:
```
1. 攻击者获取设备 root 权限
2. 修改存储公钥的分区/文件
3. 植入攻击者公钥
4. 使用攻击者私钥签名恶意升级包
5. 设备接受恶意升级
```

**影响**:
- 完全绕过签名验证机制
- 可刷入任意恶意固件

**修复建议**:
```c
// 1. 在框架层增加公钥格式校验
static int ValidatePublicKey(const uint8 *key, uint32 length)
{
    // 检查是否为有效 DER 编码 RSA 公钥
    // 检查密钥长度（256B/384B）
    // 可选：与编译期内置公钥哈希比对
}

// 2. 要求厂商从安全存储读取公钥（OTP/eFuse）
// 3. 考虑支持公钥证书链验证
```

---

### 风险 7: 内存泄漏（低风险）

**位置**: `frameworks/source/updater/hota_updater.c:227-283`

**代码证据**:
```c
static int ParseHotaInfoComponent(unsigned char *infoCompBuffer, unsigned short bufLen)
{
    // ...
    unsigned char *infoHeaderBuf = (unsigned char *)malloc(bufLen - SIGN_DATA_LEN);
    if (infoHeaderBuf == NULL) {
        printf("malloc infoHeaderBuf failed.\r\n");
        return OHOS_FAILURE;
    }
    if (memcpy_s(...) != EOK) {
        free(infoHeaderBuf);  // ✅ 释放
        return OHOS_FAILURE;
    }

    if (HotaSignVerify(...)) {
        // ...
        free(infoHeaderBuf);  // ✅ 释放
        return OHOS_FAILURE;
    }

    free(infoHeaderBuf);  // ✅ 释放
    // ...
    // 后续错误返回可能遗漏释放？
}
```

**分析**:
- ✅ 当前代码中 `infoHeaderBuf` 的释放是正确的
- ⚠️ `g_infoCompBuff` 在失败时通过 `UpdateStatus` 释放（行128-131）
- ⚠️ 但某些错误路径可能绕过 `UpdateStatus`

**潜在问题路径**:
```c
if (memcpy_s(&g_allComponentSize, ...) != EOK) {
    return OHOS_FAILURE;  // 未调用 UpdateStatus，g_infoCompBuff 可能泄漏
}
```

**修复建议**:
```c
// 使用 goto 或 RAII 模式确保释放
static int ParseHotaInfoComponent(...)
{
    int result = OHOS_FAILURE;
    unsigned char *infoHeaderBuf = malloc(...);
    if (infoHeaderBuf == NULL) goto cleanup;
    
    // ... 处理逻辑 ...
    result = OHOS_SUCCESS;
    
cleanup:
    free(infoHeaderBuf);
    if (result != OHOS_SUCCESS) {
        UpdateStatus(HOTA_FAILED);  // 确保释放 g_infoCompBuff
    }
    return result;
}
```

---

### 风险 8: 缺少降级保护配置（中等风险）

**位置**: `frameworks/source/updater/hota_updater.c:163-225`, `hals/hal_hota_board.h:260`

**代码证据**:
```c
// HAL 层提供了覆盖接口
int HotaHalCheckVersionValid(const char *currentVersion, 
                             const char *pkgVersion, 
                             unsigned int pkgVersionLength);
```

**分析**:
- ⚠️ 版本检查逻辑可被 HAL 层完全覆盖
- ⚠️ 如果厂商实现返回始终成功，版本防回滚失效
- ⚠️ 无编译期开关强制启用版本检查

**修复建议**:
```c
// 在框架层添加强制检查，HAL 仅作为补充
static int CheckPkgVersionValid(const char *pkgVersion)
{
    // 强制版本检查（不可绕过）
    if (!FrameworkVersionCheck(pkgVersion)) {
        return OHOS_FAILURE;
    }
    
    // 可选的 HAL 扩展检查
    return HotaHalCheckVersionValid(...);
}
```

---

## 安全加固建议

### 立即修复（高优先级）

1. **修复整数溢出检查**（风险 2）
   - 添加 `infoCompSize` 范围校验

2. **修复签名验证顺序**（风险 5）
   - 先验证签名再写入 Flash

3. **添加公钥来源要求**（风险 6）
   - 文档明确要求从 OTP/eFuse 读取
   - 添加公钥格式校验

### 中期改进（中优先级）

4. **添加线程安全说明**（风险 3）
   - 文档明确标注非线程安全
   - 或添加互斥锁保护

5. **修复版本校验逻辑**（风险 4）
   - 修正日志输出
   - 明确分隔符定义

6. **内存管理改进**（风险 7）
   - 统一错误处理路径

### 长期规划（低优先级）

7. **支持安全启动链**
   - 与 Bootloader 安全启动整合
   - 支持证书链验证

8. **安全增强配置**
   - 编译期强制启用安全选项
   - 安全配置审计

---

## 安全测试建议

### 模糊测试（Fuzzing）

针对以下输入进行模糊测试：

1. **PkgBasicInfo 结构**
   - 变异 type、infoCompSize 等字段
   - 测试边界值（0, MAX, MAX+1）

2. **升级包数据流**
   - 随机数据包
   - 畸形边界数据

3. **组件表**
   - 超过 10 个组件
   - 超大组件大小

### 代码审计

1. 所有 `memcpy_s` 返回值检查
2. 所有整数运算溢出检查
3. 所有错误处理路径的资源释放

### 渗透测试

1. **签名绕过测试**
   - 篡改签名数据
   - 替换公钥
   - 重放旧版本包

2. **版本回滚测试**
   - 尝试降级到旧版本
   - 测试 HAL 覆盖场景

---

## 局限性说明

本次安全评审基于代码静态分析，存在以下局限：

1. **未包含 HAL 层实现**: 厂商实现的安全性无法评估
2. **未包含 Bootloader**: 启动链安全不在本次评审范围
3. **未动态测试**: 未进行实际的模糊测试或渗透测试
4. **依赖链未完全分析**: mbedtls、系统库等依赖的安全性需要单独评估

---

## 相关跳转

- [项目架构详解 → 01_Architecture.md](01_Architecture.md)
- [对外 API 说明 → 02_Public_API.md](02_Public_API.md)
- [HAL 接口适配 → 03_Inner_API.md](03_Inner_API.md)
- [构建系统说明 → 04_GN_Build.md](04_GN_Build.md)
