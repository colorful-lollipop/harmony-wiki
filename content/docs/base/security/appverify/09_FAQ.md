# 常见问题与排查 (FAQ)

## 目的

提供 appverify 模块的常见问题、错误码解释和排查路径。

## 适用范围

- 目标读者：开发者、运维人员、调试人员
- 涵盖内容：常见错误、排查路径、调试技巧

## 关键结论

1. **常见错误**：签名不匹配、证书过期、设备未授权
2. **排查工具**：Hilog、gdb、ldd
3. **调试模式**：启用测试证书和详细日志

## 常见错误码

| 错误码 | 含义 | 常见原因 | 解决方案 |
|--------|------|----------|----------|
| VERIFY_SUCCESS (0) | 验证成功 | - | - |
| FILE_PATH_INVALID (-1) | 文件路径无效 | 扩展名错误、路径为空 | 检查文件路径和扩展名 |
| OPEN_FILE_ERROR (-2) | 打开文件失败 | 文件不存在、无权限 | 检查文件权限 |
| SIGNATURE_NOT_FOUND (-3) | 未找到签名块 | HAP 未签名或损坏 | 重新签名 HAP |
| VERIFY_APP_PKCS7_FAIL (-4) | PKCS7 解析失败 | 签名格式错误 | 检查签名工具版本 |
| PROFILE_PARSE_FAIL (-5) | Profile 解析失败 | JSON 格式错误 | 检查 Profile 配置 |
| APP_SOURCE_NOT_TRUSTED (-6) | 不在可信源中 | 证书未配置到可信源 | 更新 trusted_apps_sources.json |
| GET_DIGEST_FAIL (-7) | 获取摘要失败 | 摘要算法不支持 | 检查签名算法 |
| VERIFY_INTEGRITY_FAIL (-8) | 完整性验证失败 | 文件被篡改 | 重新签名 HAP |
| FILE_SIZE_TOO_LARGE (-9) | 文件过大 | HAP > 2GB | 压缩或拆分 HAP |
| VERIFY_SIGNATURE_FAIL (-13) | 签名验证失败 | 签名不匹配 | 检查私钥和证书 |
| VERIFY_SOURCE_INIT_FAIL (-14) | 初始化失败 | 配置文件缺失 | 检查 /system/etc/security/ |
| DEVICE_UNAUTHORIZED (-15) | 设备未授权 | 设备 ID 不在列表 | 更新 Provision 或设备 |
| CERTIFICATE_EXPIRED (-16) | 证书过期 | 证书超过有效期 | 更新证书 |
| VERIFY_ENTERPRISE_RESIGN_FAIL (-17) | 企业验证失败 | 企业配置错误 | 检查企业证书和设备 |

---

## 常见问题

### 1. HAP 验证失败，错误码 -6

**症状**：
```
HapVerify failed: -6 (APP_SOURCE_NOT_TRUSTED)
```

**原因**：
- 应用签名证书不在 `trusted_apps_sources.json` 中

**排查步骤**：
1. 检查证书主题：
```bash
openssl x509 -in cert.pem -noout -subject
openssl x509 -in cert.pem -noout -issuer
```
2. 对比 `trusted_apps_sources.json`：
```bash
cat /system/etc/security/trusted_apps_sources.json | grep "subject"
```
3. 添加或更新可信源配置

**解决**：
```json
{
  "trusted-apps-sources": [
    {
      "name": "MY_SOURCE",
      "app-signing-certs": [
        {
          "subject": "CN=My Subject",
          "issuer-ca": "CN=My CA"
        }
      ]
    }
  ]
}
```

---

### 2. 证书过期错误

**症状**：
```
HapVerify failed: -16 (CERTIFICATE_EXPIRED)
```

**原因**：
- 证书超过有效期（notAfter）

**排查步骤**：
1. 检查证书有效期：
```bash
openssl x509 -in cert.pem -noout -dates
```
2. 检查系统时间：
```bash
date
```

**解决**：
- 更新证书
- 或调整系统时间（仅测试环境）

---

### 3. 设备未授权

**症状**：
```
HapVerify failed: -15 (DEVICE_UNAUTHORIZED)
```

**原因**：
- 设备 UDID 不在 Provision 的设备列表中（开发模式）

**排查步骤**：
1. 获取设备 UDID：
```bash
# 通过命令行
cat /sys/.../device_id

# 或通过 IPC API
```
2. 检查 Provision 配置：
```bash
# 解析 Profile 查看 debugInfo.deviceIds
```

**解决**：
- 在 Provision 中添加设备 ID
- 或使用空 `deviceIds` 列表（允许任意设备）

---

### 4. 初始化失败

**症状**：
```
HapVerify failed: -14 (VERIFY_SOURCE_INIT_FAIL)
```

**原因**：
- 配置文件缺失或格式错误

**排查步骤**：
1. 检查配置文件存在：
```bash
ls -l /system/etc/security/trusted_*.json
```
2. 检查 JSON 格式：
```bash
python -m json.tool /system/etc/security/trusted_root_ca.json
```
3. 检查文件权限：
```bash
ls -la /system/etc/security/
```

**解决**：
- 恢复配置文件
- 修复 JSON 语法错误
- 检查 root 权限

---

### 5. 签名块未找到

**症状**：
```
HapVerify failed: -3 (SIGNATURE_NOT_FOUND)
```

**原因**：
- HAP 文件未签名
- 签名块格式错误

**排查步骤**：
1. 检查 HAP 结构：
```bash
# 使用 zip 工具
unzip -l app.hap

# 查看签名块
hexdump -C app.hap | grep "HAPSigningBlock"
```
2. 检查签名工具版本

**解决**：
- 使用正确的签名工具重新签名
- 确保签名块格式正确

---

### 6. 完整性验证失败

**症状**：
```
HapVerify failed: -8 (VERIFY_INTEGRITY_FAIL)
```

**原因**：
- 文件内容被篡改
- 摘要不匹配

**排查步骤**：
1. 计算文件摘要：
```bash
sha256sum app.hap
```
2. 对比签名中的摘要
3. 检查文件传输过程

**解决**：
- 重新签名 HAP
- 检查文件传输完整性

---

### 7. 库加载失败

**症状**：
```
error while loading shared libraries: libhapverify.so: cannot open shared object file
```

**原因**：
- 库路径不在 LD_LIBRARY_PATH 中

**排查步骤**：
1. 检查库存在：
```bash
ls -l /usr/lib/libhapverify.so
ls -l /system/lib/libhapverify.so
```
2. 检查动态链接：
```bash
ldd <your_app>
```

**解决**：
```bash
export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
```
- 或将库安装到系统路径

---

## 调试技巧

### 启用调试模式

```cpp
#include "interfaces/hap_verify.h"

using namespace OHOS::Security::Verify;

// 启用调试模式（使用测试证书）
if (!EnableDebugMode()) {
    printf("Failed to enable debug mode\n");
    return -1;
}

// 验证测试应用
HapVerifyResult result;
int32_t ret = HapVerify(testHapPath, result);

// 验证完成后禁用调试模式
DisableDebugMode();
```

### 获取详细日志

**设置 Hilog 级别**：
```bash
# 设置为 DEBUG
hdc shell hilog -b D0001 -v DEBUG

# 清除历史日志
hdc shell hilog -r

# 查看实时日志
hdc shell hilog | grep appverify
```

**日志标签**：
- `appverify`：主日志标签

### 使用 GDB 调试

```bash
# 启动 GDB
gdb --args ./your_app

# 设置断点
b HapVerify

# 运行
run

# 查看变量
p hapVerifyResult
```

### 追踪调用链

```bash
# 使用 strace 追踪系统调用
strace -e trace=open,read,write ./your_app

# 追踪文件访问
strace -e trace=file ./your_app
```

---

## 问题定位路径

### 验证失败定位

```
错误码 → 查阅错误码表 → 确定失败阶段 → 检查对应配置 → 解决
    ↓
示例：-6 (APP_SOURCE_NOT_TRUSTED)
    ↓
失败阶段：可信源匹配
    ↓
检查：trusted_apps_sources.json
    ↓
解决：添加或更新可信源
```

### 文件问题定位

```
文件不存在/无权限 → 检查路径和权限 → 修复权限 → 重试
    ↓
示例：OPEN_FILE_ERROR (-2)
    ↓
检查：ls -l /path/to/app.hap
    ↓
解决：chmod 644 app.hap
```

### 配置问题定位

```
初始化失败 → 检查配置文件 → 验证 JSON 格式 → 恢复配置
    ↓
示例：VERIFY_SOURCE_INIT_FAIL (-14)
    ↓
检查：cat /system/etc/security/trusted_root_ca.json
    ↓
验证：python -m json.tool trusted_root_ca.json
    ↓
解决：修复 JSON 语法
```

---

## 性能问题

### 验证缓慢

**症状**：
- 大文件验证耗时过长（> 10s）

**原因**：
- 文件大小过大（接近 2GB）
- 证书链过长
- 串行计算摘要

**优化建议**：
1. 降低文件大小限制
2. 使用并行计算（已启用）
3. 优化证书链长度

**检查并行计算**：
```cpp
// hap_signing_block_utils.cpp:667
bool HapSigningBlockUtils::HapVerifyParallelizationSupported()
{
    // 检查是否支持并行计算
}
```

### 内存占用高

**症状**：
- 验证过程中内存占用高

**原因**：
- 大文件签名块加载
- 多个并发验证

**优化建议**：
1. 流式读取签名块
2. 限制并发验证数量

---

## 编译相关问题

### 链接错误

**症状**：
```
undefined reference to `HapVerify'
```

**原因**：
- 未链接 libhapverify.so

**解决**：
```cmake
# CMakeLists.txt
target_link_libraries(your_app PRIVATE libhapverify)
```
或
```bash
# 命令行
gcc your_app.cpp -o your_app -lhapverify
```

### 找不到头文件

**症状**：
```
fatal error: hap_verify.h: No such file or directory
```

**原因**：
- 头文件路径不在搜索路径

**解决**：
```bash
# 设置 CPATH
export CPATH=/path/to/appverify/include:$CPATH

# 或编译时指定
gcc -I/path/to/appverify/include your_app.cpp
```

---

## 运行时问题

### 版本不匹配

**症状**：
```
libhapverify.so: version `OPENSSL_1_1_1' not found
```

**原因**：
- OpenSSL 版本不匹配

**解决**：
```bash
# 检查 OpenSSL 版本
openssl version

# 升级 OpenSSL
# (根据系统包管理器）
```

### 配置文件冲突

**症状**：
- 调试模式和正式模式冲突

**原因**：
- EnableDebugMode() 后未禁用

**解决**：
```cpp
// 验证完成后禁用调试模式
DisableDebugMode();
```

---

## 附录：完整错误码表

参见 [04_Public_API.md](04_Public_API.md#返回值) 获取完整错误码说明。

---

## 相关跳转

- [对外 API](04_Public_API.md) - API 使用和错误码
- [架构详解](03_Architecture.md) - 验证流程
- [安全评审](08_Security_Review.md) - 安全机制
- [编译产物](07_Build_Artifacts.md) - 依赖和库加载
