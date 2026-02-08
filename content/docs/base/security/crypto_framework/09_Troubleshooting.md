# 常见问题与定位 (Troubleshooting)

## 目的

本文档提供 crypto_framework 的常见问题、排查路径和调试技巧，帮助开发者快速定位和解决问题。

## 适用范围

- **目标读者**: 开发者、测试人员、支持工程师
- **阅读时长**: 20 分钟

## 构建问题

### 问题 1: 编译错误 - 找不到头文件

**现象**:
```
fatal error: 'hcf_key.h' file not found
```

**可能原因**:
1. 依赖顺序错误，`frameworks_inc_path` 未正确设置
2. `.gni` 文件中的路径配置错误

**定位步骤**:
1. 检查 `frameworks/frameworks.gni` 中的 `frameworks_inc_path`
2. 确认包含路径是否正确：`interfaces/inner_api/`
3. 检查 BUILD.gn 中的 `include_dirs`

**解决方法**:
```gn
# 在 frameworks/frameworks.gni 中确认
frameworks_inc_path = [
  "//base/security/crypto_framework/interfaces/inner_api",
  "//base/security/crypto_framework/interfaces/inner_api/common",
  "//base/security/crypto_framework/interfaces/inner_api/crypto_operation",
  "//base/security/crypto_framework/interfaces/inner_api/key",
]
```

**证据位置**: `frameworks/frameworks.gni`

---

### 问题 2: 链接错误 - 找不到符号

**现象**:
```
undefined reference to 'HcfCipherInit'
```

**可能原因**:
1. 缺少对 `crypto_framework_lib` 的依赖
2. 插件未正确链接

**定位步骤**:
1. 检查 BUILD.gn 中的 `deps` 是否包含 `frameworks:crypto_framework_lib`
2. 使用 `nm -D` 检查符号是否在框架库中导出
3. 检查链接顺序

**解决方法**:
```gn
# 确保依赖正确
ohos_shared_library("my_crypto_app") {
  deps = [
    "//base/security/crypto_framework/frameworks:crypto_framework_lib",
  ]
}
```

---

### 问题 3: 系统类型条件编译错误

**现象**:
```
error: 'crypto_framework_napi' not found in dependencies
```

**可能原因**:
1. `os_level` 参数未正确设置
2. 在错误的系统类型下构建

**定位步骤**:
1. 检查 `--args='os_level="standard"'` 是否正确
2. 查看 `BUILD.gn` 中的条件分支

**解决方法**:
```bash
# 明确指定系统类型
gn gen out --args='os_level="standard"'

# 或
gn gen out --args='os_level="mini"'
```

**证据位置**: `BUILD.gn:19-34`

---

## 运行时问题

### 问题 4: N-API 模块加载失败

**现象**:
```
Error: Module not found: No such file or directory
```

**可能原因**:
1. HAP 包中缺少 `.so` 文件
2. 安装路径错误
3. 符号链接损坏

**定位步骤**:
1. 检查 HAP 包中是否包含 `libcryptoframework_napi.so`
2. 使用 `readelf -h` 检查 .so 文件完整性
3. 查看 hilog 日志

**解决方法**:
```bash
# 检查 HAP 包内容
unzip -l MyApp.hap | grep libcryptoframework

# 检查安装后的库
adb shell ls -l /system/lib/ | grep crypto
```

---

### 问题 5: 算法不支持

**现象**:
```
Error code: 801 (CRYPTO_NOT_SUPPORTED)
```

**可能原因**:
1. 使用了不支持的算法名称
2. 插件未正确加载
3. 算法参数组合不支持

**定位步骤**:
1. 检查算法名称拼写
2. 确认当前系统类型（Standard/Mini）
3. 查看插件支持的算法列表

**解决方法**:
```javascript
// 检查算法名称是否正确
// 支持的算法：AES, SM4, RSA, ECC, ECDSA, SHA256, SM3, etc.
let cipher = cryptoFramework.createCipher("AES256", "GCM");  // 正确
// let cipher = cryptoFramework.createCipher("AES256-GCM", "");  // 错误格式

// 检查系统类型
// Standard: 支持 OpenSSL 插件（完整算法）
// Mini: 支持 MbedTLS 插件（基础算法）
```

---

### 问题 6: 密钥格式错误

**现象**:
```
Error code: 401 (CRYPTO_INVALID_PARAMS)
```

**可能原因**:
1. PEM 格式错误（缺少头尾标记）
2. DER 解析失败
3. Base64 编码错误

**定位步骤**:
1. 检查 PEM 文件格式
2. 使用 `openssl asn1parse` 验证 DER
3. 查看 hilog 详细错误信息

**解决方法**:
```bash
# 验证 PEM 格式
openssl pkey -in mykey.pem -text -noout

# 检查是否有正确的头尾标记
-----BEGIN PRIVATE KEY-----
... base64 data ...
-----END PRIVATE KEY-----
```

---

### 问题 7: 内存不足

**现象**:
```
Error code: 17620001 (CRYPTO_MEMORY_ERROR)
```

**可能原因**:
1. 处理过大的数据
2. 系统内存不足
3. 内存泄漏

**定位步骤**:
1. 使用 `top` 或 `free` 检查系统内存
2. 使用 Valgrind 或 AddressSanitizer 检查内存泄漏
3. 查看 hilog 日志

**解决方法**:
```c
// 处理大文件时分块
#define MAX_CHUNK_SIZE (1024 * 1024)  // 1MB

HcfResult ProcessLargeFile(HcfCipher *cipher, FILE *fp) {
    HcfBlob chunk = { .data = malloc(MAX_CHUNK_SIZE), .len = MAX_CHUNK_SIZE };
    
    while (!feof(fp)) {
        size_t read = fread(chunk.data, 1, MAX_CHUNK_SIZE, fp);
        chunk.len = read;
        
        HcfResult ret = HcfCipherUpdate(cipher, &chunk, &out);
        if (ret != HCF_SUCCESS) {
            free(chunk.data);
            return ret;
        }
    }
    
    free(chunk.data);
    return HCF_SUCCESS;
}
```

---

## 调试技巧

### 启用详细日志

**位置**: `common/inc/log.h`

```c
// 设置日志级别
#define LOG_LEVEL_DEBUG  0
#define LOG_LEVEL_INFO   1
#define LOG_LEVEL_WARN   2
#define LOG_LEVEL_ERROR  3

// 在代码中使用
#define LOGD(...)  // Debug 日志
#define LOGI(...)  // Info 日志
#define LOGE(...)  // Error 日志

// 运行时设置日志级别
// 通过 hilog 命令
hilog -b crypto_framework -D DEBUG
```

### 使用 GDB 调试

```bash
# 附加到进程
adb shell ps -A | grep crypto_framework
adb forward tcp:1234 tcp:1234
gdb /system/lib/libcryptoframework_napi.so
(gdb) target remote :1234
(gdb) b HcfCipherInit
(gdb) c
```

### 查看 hilog 日志

```bash
# 查看所有日志
hilog -r

# 过滤特定模块
hilog -b crypto_framework

# 查看错误日志
hilog -b crypto_framework -e

# 保存日志到文件
hilog -r > crypto_framework.log
```

### 内存调试

使用 AddressSanitizer 或 Valgrind 检测内存问题：

```bash
# 启用 AddressSanitizer
gn gen out --args='os_level="standard" use_asan=true'
ninja -C out

# 或使用 Valgrind
adb push libcryptoframework_napi.so /data/local/tmp/
adb shell valgrind --leak-check=full /data/local/tmp/libcryptoframework_napi.so
```

### 符号调试

```bash
# 检查库中的符号
nm -D libcryptoframework_napi.so | grep "Hcf"

# 检查动态符号
readelf -d libcryptoframework_napi.so

# 检查依赖
readelf -d libcryptoframework_napi.so | grep NEEDED
```

## 性能问题

### 问题 8: 加密性能慢

**可能原因**:
1. 使用了不合适的算法或模式
2. 数据频繁的小块更新
3. 硬件加速未启用

**解决方法**:
```javascript
// 使用硬件加速的算法
let cipher = cryptoFramework.createCipher("AES256", "GCM");  // GCM 支持硬件加速
// let cipher = cryptoFramework.createCipher("AES256", "CBC");  // 软件实现

// 合并小块数据
let bigData = concatenateChunks(chunks);
await cipher.update(bigData);  // 一次性处理
// 避免：
// for (chunk of chunks) {
//     await cipher.update(chunk);  // 频繁的小块处理
// }
```

---

### 问题 9: 内存占用高

**可能原因**:
1. 大密钥对象未释放
2. 密钥数据在内存中停留过久
3. 缓冲区未复用

**解决方法**:
```javascript
// 及时释放对象
let key = await generator.generateKeyPair();
try {
    let cipher = await cryptoFramework.createCipher(...);
    // ... 使用 cipher
} finally {
    cipher.destroy();  // 销毁对象
    key.destroy();      // 销毁密钥
}

// 使用 clearMem 清除敏感数据
await symKey.clearMem();
```

---

## 定位工具

### 文件定位

| 文件 | 职责 | 快速定位命令 |
|------|------|-------------|
| `interfaces/kits/native/` | 对外 Native API | `grep -r "OH_Crypto_" interfaces/kits/native/` |
| `interfaces/inner_api/` | 内部 API | `grep -r "Hcf" interfaces/inner_api/` |
| `frameworks/js/napi/` | N-API 绑定 | `find frameworks/js/napi -name "*.cpp"` |
| `frameworks/native/src/` | Native 实现 | `grep -r "Hcf" frameworks/native/src/` |
| `plugin/openssl_plugin/` | OpenSSL 实现 | `find plugin/openssl_plugin -name "*.c"` |

### 代码搜索

```bash
# 查找特定函数的定义
grep -r "HcfCipherInit" frameworks/

# 查找特定函数的调用
grep -r "HcfCipherInit\|HcfCipherUpdate" frameworks/native/src/

# 查找错误码的使用
grep -r "HCF_INVALID_PARAMS" frameworks/
```

### 依赖分析

```bash
# 查看目标依赖
gn deps out/ //base/security/crypto_framework:crypto_framework_component --all

# 查看依赖树
gn desc out/ libs //base/security/crypto_framework:crypto_framework_component

# 生成依赖图
gn analyze --out=out/ dep-graph.json //base/security/crypto_framework:crypto_framework_component
dot -Tpng dep-graph.json > dep-graph.png
```

## 相关跳转

- **架构设计**: [03_Architecture.md](03_Architecture.md)
- **GN Targets**: [06_GN_Targets.md](06_GN_Targets.md)
- **编译产物**: [07_Build_Artifacts.md](07_Build_Artifacts.md)

## 更新记录

- **2026-02-06**: 创建文档，基于常见问题整理生成

## TODO

- [ ] 补充更多常见问题和解决方案
- [ ] 添加性能优化建议
- [ ] 补充调试工具使用示例
