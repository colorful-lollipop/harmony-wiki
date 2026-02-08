# 常见问题与定位

> 证书管理模块的构建、运行、调试问题及解决方案

## 文档目的

帮助开发者快速定位和解决证书管理模块的常见问题。

## 适用范围

- 构建问题
- 运行时问题
- 调试技巧
- 性能问题
- 常见错误码

## 构建问题

### 问题 1：N-API 编译失败

**症状**：
```
error: 'cm_napi_common.h' file not found
```

**原因**：
- 头文件路径配置错误
- include_dirs 未正确设置

**解决方案**：
```gn
# 检查 BUILD.gn 中的 include_dirs 配置
include_dirs = [ "include" ]

# 确保 include 目录下有所需头文件
ls interfaces/kits/napi/include/
```

**证据**：interfaces/kits/napi/BUILD.gn:28

---

### 问题 2：HUKS 依赖未找到

**症状**：
```
error: dependency "huks" not found
```

**原因**：
- HUKS 模块未编译或路径错误
- `certificate_manager_deps_huks_enabled` 配置不当

**解决方案**：
```bash
# 1. 检查 HUKS 是否存在
find out/ -name "*huks*"

# 2. 检查 cert_manager.gni 配置
cat cert_manager.gni | grep huks

# 3. 重新构建 HUKS
cd ../security_huks
./build.sh --product-name <product>

# 4. 重新编译证书管理器
cd ../certificate_manager
./build.sh --product-name <product> --args="huks_path=../out/huks"
```

**证据**：cert_manager.gni:20, bundle.json:53

---

### 问题 3：RDB 编译失败

**症状**：
```
error: "relational_store:native_rdb" not found
```

**原因**：
- RDB 模块未启用
- 系统类型不支持 RDB

**解决方案**：
```bash
# 1. 检查系统类型
cat out/<product>/args.gn | grep "is_standard_system"

# 2. 确认 relational_store 编译
find out/ -name "*relational_store*"

# 3. 如果是 mini/small 系统，可能不支持 RDB
# 只构建不支持的部分功能
ninja -C out/<product> cert_manager_type_base
```

**证据**：services/cert_manager_standard/cert_manager_engine/main/rdb/BUILD.gn

---

### 问题 4：对话框功能未编译

**症状**：
```
note: certmanagerdialog not built (feature disabled)
```

**原因**：
- `certificate_manager_feature_dialog_enabled` 为 false
- ace_engine 未找到

**解决方案**：
```bash
# 1. 启用对话框功能
gn gen out/<product> --args="certificate_manager_feature_dialog_enabled=true"

# 2. 或者确保 ace_engine 存在
find out/ -name "*ace_engine*"

# 3. 检查 bundle.json
cat bundle.json | grep ace_engine
```

**证据**：cert_manager.gni:26-30, interfaces/kits/napi/BUILD.gn:78-139

---

### 问题 5：链接错误 - 未定义符号

**症状**：
```
undefined reference to `CmClientGetCertList`
```

**原因**：
- 静态库链接顺序问题
- 符号未正确导出（`CM_API_EXPORT`）

**解决方案**：
```gn
# 检查 cert_manager_api.h 中的导出宏
grep "CM_API_EXPORT" interfaces/innerkits/cert_manager_standard/main/include/cert_manager_api.h

# 确保 Inner SDK 目标正确设置 visibility
check BUILD.gn:
  deps = [ ":cert_manager_sdk", ... ]

# 清理并重新构建
rm -rf out/<product>
./build.sh --product-name <product> --build-type release
```

**证据**：interfaces/innerkits/cert_manager_standard/main/include/cert_manager_api.h:26-34

---

## 运行时问题

### 问题 6：服务未启动

**症状**：
```
JS Error: CertificateManager service not available
```

**原因**：
- cert_manager_service 未运行
- SA 未发布

**解决方案**：
```bash
# 1. 检查服务状态
hdc shell ps -A | grep cert_manager

# 2. 检查 SA 状态
hdc shell samgr list -p security | grep 3512

# 3. 启动服务（如果未启动）
hdc shell "sa start 3512"

# 4. 检查服务日志
hdc shell "cd /data/log/cert_manager_service/ && tail -f cert_manager.log"
```

**证据**：cm_sa.h:54, services/cert_manager_standard/cert_manager_service/main/os_dependency/sa/sa_profile/cert_manager_service.json:5-6

---

### 问题 7：权限被拒绝

**症状**：
```
JS Error: Permission denied (CM_ERROR_NO_PERMISSION: -23)
```

**原因**：
- 应用缺少必需权限
- 权限声明缺失
- 权限级别不足

**解决方案**：
```json
// 1. 检查 app.json 中的权限声明
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCESS_CERT_MANAGER"
      },
      // 根据需要添加
      {
        "name": "ohos.permission.ACCESS_CERT_MANAGER_INTERNAL"
      }
    ]
  }
}

// 2. 确认应用类型
// 系统应用需要特殊处理

// 3. 重新签名并安装应用
```

**权限对照表**：
| 操作 | 所需权限 | 错误码 |
|------|-----------|-------|
| 安装应用证书 | `ACCESS_CERT_MANAGER` | -23 |
| 安装系统证书 | `ACCESS_CERT_MANAGER` + 系统应用 | -23, -26 |
| 安装用户 CA | `ACCESS_USER_TRUSTED_CERT` 或 `ACCESS_ENTERPRISE_USER_TRUSTED_CERT` | -23 |
| 授权证书 | `ACCESS_CERT_MANAGER` + `ACCESS_CERT_MANAGER_INTERNAL` | -23 |

**证据**：cert_manager_permission_check.h:25-32, cm_type.h:138-162

---

### 问题 8：证书数量达到上限

**症状**：
```
JS Error: Maximum certificate count reached (CM_ERROR_MAX_CERT_COUNT_REACHED: -27)
```

**原因**：
- 应用安装证书超过系统限制（512）
- 证书未正确清理

**解决方案**：
```javascript
// 1. 删除旧证书
await uninstallAllAppCert();

// 2. 检查当前数量
const list = await getAllAppPrivateCertificates();
console.log(`Current count: ${list.length}`);

// 3. 批量删除旧证书
for (const cert of list) {
    if (!cert.isRecent) {
        await uninstallPrivateCertificate(cert.keyUri);
    }
}
```

**系统限制**：
- 证书总数：512（`MAX_COUNT_CERTIFICATE_ALL`）
- 应用证书数：256（`MAX_COUNT_CERTIFICATE`）
- 单个应用：无硬限制

**证据**：cm_type.h:43-44

---

### 问题 9：别名长度超限

**症状**：
```
JS Error: Alias length reached limit (CM_ERROR_ALIAS_LENGTH_REACHED_LIMIT: -28)
```

**原因**：
- 证书别名超过 128 字节
- 未进行长度验证

**解决方案**：
```javascript
// 1. 缩短别名
const shortAlias = cert.alias.substring(0, 128);

// 2. 使用哈希或自动生成别名
const autoAlias = `cert_${Date.now()}`;

// 3. 客户端验证
if (alias.length > 128) {
    console.error('Alias too long, max 128 bytes');
    return;
}
```

**系统限制**：
- 最大别名长度：128 字节（`MAX_LEN_CERT_ALIAS`）

**证据**：cm_type.h:47

---

### 问题 10：证书格式错误

**症状**：
```
JS Error: Incorrect certificate format (CM_ERROR_INCORRECT_FORMAT: -20)
```

**原因**：
- 证书文件格式不支持或损坏
- PEM/DER/PKCS#12 格式错误

**解决方案**：
```bash
# 1. 验证证书文件格式
openssl x509 -in cert.pem -text -noout

# 2. 转换为正确格式
openssl x509 -in cert.pem -out cert.der -outform DER

# 3. 验证 P12 文件
openssl pkcs12 -in cert.p12 -info

# 4. 使用正确的 API 参数
// 确保使用 CertType.PEM_DER 或 CertType.P7B
```

**支持的格式**：
- `CertFileFormat.PEM_DER` (0) - PEM 或 DER
- `CertFileFormat.P7B` (1) - P7B（PKCS#7）

**证据**：cm_type.h:523-526

---

## 调试技巧

### 启用详细日志

```bash
# 1. 设置 HiLog 级别
hdc shell "hilog -b cert_manager -v"

# 2. 查看 N-API 日志
hdc shell "hilog -b cert_manager | grep CMNapi"

# 3. 查看 Service 日志
hdc shell "hilog -b cert_manager_service | grep cert_manager"

# 4. 实时跟踪
hdc shell "hilog -b cert_manager -v | tail -f"
```

**证据**：hisysevent.yaml, frameworks/cert_manager_standard/main/common/include/cm_log.h

---

### IPC 调试

```bash
# 1. 检查 SA 状态
hdc shell "samgr list -p security | grep 3512"

# 2. 查看服务线程数
hdc shell "ps -ef | grep cert_manager"

# 3. 查看 IPC 统计
hdc shell "cat /proc/net/rpc/xprt/tcp"

# 4. 测试 IPC 连通性
hdc shell "cd /data/service/el1/public/cert_manager_service && cat cert_manager_test_ipc"
```

---

### 证书调试

```bash
# 1. 查看证书目录结构
hdc shell "ls -laR /data/service/el1/public/cert_manager_service/certificates/"

# 2. 查看证书内容
hdc shell "cat /data/service/el1/public/cert_manager_service/certificates/<userId>/<uid>/certificates/<certFile>"

# 3. 验证证书
hdc shell "openssl x509 -in <cert_file> -text -noout"

# 4. 检查 HUKS 密钥
hdc shell "huksd list"
```

---

### 错误码快速查询

| 错误码 | 错误名 | 常见原因 | 快速检查 |
|--------|--------|----------|----------|
| -23 | NO_PERMISSION | 权限缺失 | 检查 app.json 权限 |
| -26 | NOT_SYSTEM_APP | 非系统应用 | 检查应用类型签名 |
| -20 | INCORRECT_FORMAT | 格式错误 | 使用 openssl 验证证书 |
| -27 | MAX_CERT_COUNT_REACHED | 达到上限 | 删除旧证书 |
| -28 | ALIAS_LENGTH_REACHED_LIMIT | 别名过长 | 缩短别名 |
| -24 | NO_AUTHORIZATION | 未授权 | 调用 grantPublicCertificate |
| -5 | NOT_FOUND | 证书不存在 | 检查证书列表 |
| -1 | GENERIC | 通用错误 | 查看详细日志 |

**完整错误码**：参见 [N-API 文档](03_N-API.md#cmerrorcode---错误码)

---

## 性能问题

### 问题：证书列表查询缓慢

**症状**：
- `getAllAppPrivateCertificates()` 耗时过长
- 返回大量证书列表时应用卡顿

**原因**：
- 文件系统 I/O 阻塞
- 大量证书时扫描慢
- RDB 查询效率低

**解决方案**：
```javascript
// 1. 分页查询
const pageSize = 50;
let offset = 0;
const allCerts = [];
while (true) {
    const certs = await getAppPrivateCertificatesByRange(offset, pageSize);
    allCerts.push(...certs);
    if (certs.length < pageSize) break;
    offset += pageSize;
}

// 2. 使用索引查询而非全量查询
// 3. 缓存结果
const cachedList = await getCachedCertList();
```

---

### 问题：服务启动延迟

**症状**：
- 首次调用证书 API 时响应慢
- 应用超时错误

**原因**：
- 服务按需启动（60 秒无活动后卸载）
- 首次调用需要加载服务

**解决方案**：
```javascript
// 1. 使用预加载
// 在应用启动时先调用轻量级 API 确保服务运行
await getSystemTrustedCertificateList();

// 2. 重试机制
async function callWithRetry(api, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await api();
        } catch (e) {
            if (i === maxRetries - 1) throw e;
            await sleep(1000); // 等待 1 秒
        }
    }
}

// 3. 设置合理超时
const result = await Promise.race([
    api(),
    new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), 5000)
]);
```

**证据**：cm_sa.h:58, cert_manager_service.json:10-12

---

## 日志分析

### HiSysEvent 事件类型

```bash
# 查看所有证书管理事件
hdc shell "hilog -b cert_manager_service -T | grep -E 'CERT_.*_EVENT'"

# 常见事件
# CERT_INSTALL_SUCCESS
# CERT_INSTALL_FAILED
# CERT_UNINSTALL_SUCCESS
# CERT_UNINSTALL_FAILED
# CERT_GRANT_SUCCESS
# CERT_GRANT_FAILED
# CERT_REMOVE_GRANT_SUCCESS
# CERT_REMOVE_GRANT_FAILED
# CERT_AUTH_FAILURE
```

**证据**：hisysevent.yaml

---

### 日志文件位置

```
/data/log/cert_manager_service/cert_manager_service.log  # 主服务日志
/data/log/cert_manager_service/                          # 其他日志
```

---

## 相关跳转

- [项目概述](00_Overview.md)
- [架构说明](02_Architecture.md)
- [N-API 接口文档](03_N-API.md)
- [内部 API 文档](04_Inner_API.md)
- [GN 构建目标](05_GN_Targets.md)
- [安全风险评审](07_Security_Review.md)

---

*更新时间：2026-02-06*
