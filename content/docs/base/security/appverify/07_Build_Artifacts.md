# 编译产物与安装

## 目的

说明 appverify 模块的编译产物、安装路径和运行时加载关系。

## 适用范围

- 目标读者：部署人员、系统维护人员
- 涵盖内容：产物清单、安装路径、运行时加载、依赖解析

## 关键结论

1. **核心产物**：libhapverify.so（标准系统）/ libverify.so（Lite 系统）
2. **配置文件**：5 个 JSON 配置，安装到 /system/etc/security/
3. **动态链接**：运行时动态加载依赖库（OpenSSL、cJSON 等）
4. **测试产物**：单元测试可执行文件

## 产物清单

### 标准系统产物

| 产物类型 | 文件名 | Target | 安装路径 |
|----------|--------|---------|----------|
| 动态库 | libhapverify.so | libhapverify | /usr/lib/ 或 /system/lib/ |
| 配置文件 | trusted_apps_sources.json | trusted_apps_sources | /system/etc/security/ |
| 配置文件 | trusted_root_ca.json | trusted_root_ca | /system/etc/security/ |
| 配置文件 | trusted_tickets_sources.json | trusted_tickets_sources | /system/etc/security/ |
| 测试配置 | trusted_apps_sources_test.json | trusted_apps_sources_test | /system/etc/security/ |
| 测试配置 | trusted_root_ca_test.json | trusted_root_ca_test | /system/etc/security/ |
| 测试可执行 | appverify | verify_test | - |
| 测试可执行 | app_verify_test | app_verify_test | - |

### Lite 系统产物

| 产物类型 | 文件名 | Target | 安装路径 |
|----------|--------|---------|----------|
| 动态库 | libverify.so | verify | /usr/lib/ 或 /lib/ |
| 静态库 | libverify_base.a | verify_base（LiteOS-M） | - |
| 动态库 | libverify_base.so | verify_base（非 LiteOS-M） | /usr/lib/ 或 /lib/ |
| 测试可执行 | app_verify_test.bin | app_verify_test | $root_out_dir/test/unittest/security/ |

---

## 安装路径详解

### 核心库路径

**标准系统**：
```
/usr/lib/libhapverify.so          # 用户空间
/system/lib/libhapverify.so        # 系统空间（某些设备）
```

**Lite 系统**：
```
/lib/libverify.so                 # 小型系统
/usr/lib/libverify.so            # 标准
```

### 配置文件路径

**统一路径**：
```
/system/etc/security/
├── trusted_apps_sources.json
├── trusted_root_ca.json
├── trusted_tickets_sources.json
├── trusted_apps_sources_test.json
└── trusted_root_ca_test.json
```

**说明**：
- `trusted_apps_sources.json`：正式可信源配置
- `trusted_root_ca.json`：正式根证书列表
- `trusted_tickets_sources.json`：OpenTest Ticket 可信源
- `*_test.json`：测试环境配置（需启用 Debug 模式）

### 测试产物路径

**标准系统测试**：
```
out/.../tests/unittest/appverify/appverify              # verify_test
out/.../tests/unittest/appverify/app_verify_test        # app_verify_test
```

**Lite 系统测试**：
```
out/.../test/unittest/security/app_verify_test.bin
```

---

## 运行时加载关系

### libhapverify 动态依赖

**使用 ldd 查看**（标准系统）：
```
$ ldd libhapverify.so
libhapverify.so:
    libcrypto.so.1.1 => /usr/lib/libcrypto.so.1.1
    libcjson.so => /usr/lib/libcjson.so
    libhilog.so => /usr/lib/libhilog.so
    libbegetutil.so => /usr/lib/libbegetutil.so
    libc.so => /usr/lib/libc.so
    libsec_shared.so => /usr/lib/libsec_shared.so
    libz.so => /usr/lib/libz.so
```

### verify 动态依赖（Lite 系统）

```
$ ldd libverify.so
libverify.so:
    libmbedtls.so => /usr/lib/libmbedtls.so
    libcjson.so => /usr/lib/libcjson.so
    libsec_shared.so => /usr/lib/libsec_shared.so
    libhilog.so => /usr/lib/libhilog.so
    libbegetutil.so => /usr/lib/libbegetutil.so
    libc.so => /usr/lib/libc.so
```

---

## 配置文件加载

### 加载时序

```
1. 调用 HapVerify() 首次
   ↓
2. HapVerifyInit() 被调用
   ↓
3. TrustedRootCa::Init()
   ├─ 解析 /system/etc/security/trusted_root_ca.json
   └─ 加载所有根证书到内存
   ↓
4. TrustedSourceManager::Init()
   ├─ 解析 /system/etc/security/trusted_apps_sources.json
   └─ 加载所有可信源规则
   ↓
5. TrustedTicketManager::Init()
   ├─ 解析 /system/etc/security/trusted_tickets_sources.json
   └─ 加载 Ticket 可信源
   ↓
6. HapCrlManager::Init()
   └─ 初始化 CRL 管理器（如有）
   ↓
7. DeviceTypeManager::GetDeviceTypeInfo()
   └─ 获取设备类型信息
```

### 配置文件格式

#### trusted_root_ca.json

```json
{
  "trusted-root-ca": [
    {
      "rootId": "OpenHarmony Root CA",
      "rootCa": "-----BEGIN CERTIFICATE-----\n...-----END CERTIFICATE-----"
    }
  ]
}
```

**证据**：config/trusted_root_ca.json

#### trusted_apps_sources.json

```json
{
  "trusted-apps-sources": [
    {
      "name": "APP_GALLERY",
      "app-signing-certs": [
        {
          "subject": "CN=HMS Application Signing",
          "issuer-ca": "CN=HMS Root CA"
        }
      ],
      "profile-signing-certificates": [
        {
          "profile-signing-certificate": "CN=HMS Profile Signing",
          "issuer-ca": "CN=HMS Root CA"
        }
      ]
    }
  ]
}
```

**证据**：config/trusted_apps_sources.json

#### trusted_tickets_sources.json

```json
{
  "trusted-tickets-sources": [
    {
      "subject": "CN=OpenTest Ticket Signing",
      "issuer-ca": "CN=OpenTest Root CA"
    }
  ]
}
```

**证据**：config/trusted_tickets_sources.json

---

## 运行时行为

### 调用方链接

**Bundle Manager Service (BMS)**：
```cpp
// CMakeLists.txt 或 BUILD.gn
target_link_libraries(bms
    PRIVATE
        libhapverify.so
)
```

**加载方式**：动态链接（dlopen 不需要）

### 初始化行为

**全局初始化**（hap_verify.cpp:39）：
```cpp
static bool g_isInit = false;  // 全局状态

bool HapVerifyInit() {
    // 加载配置文件
    TrustedRootCa::GetInstance().Init();
    TrustedSourceManager::GetInstance().Init();
    TrustedTicketManager::GetInstance().Init();
    HapCrlManager::GetInstance().Init();
    DeviceTypeManager::GetInstance().GetDeviceTypeInfo();

    g_isInit = true;
    return g_isInit;
}
```

**时机**：首次调用 `HapVerify()` 时自动初始化

### 调试模式切换

**调用方式**：
```cpp
// 启用调试模式（加载 *_test.json）
EnableDebugMode();

// 验证测试应用
HapVerify(testHapPath, result);

// 禁用调试模式
DisableDebugMode();
```

**影响**：
- `TrustedRootCa::EnableDebug()` - 加载 `trusted_root_ca_test.json`
- `TrustedSourceManager::EnableDebug()` - 加载 `trusted_apps_sources_test.json`

---

## 测试产物运行

### 标准系统测试

**运行单元测试**：
```bash
# 运行所有测试
./tests/unittest/appverify/appverify

# 运行特定测试
./tests/unittest/appverify/appverify --gtest_filter=HapVerifyTest.*

# 查看帮助
./tests/unittest/appverify/appverify --help
```

**测试输出**：
- GoogleTest 格式
- 日志输出到标准输出或 Hilog

### Lite 系统测试

**运行单元测试**：
```bash
# 直接运行测试二进制
./out/.../test/unittest/security/app_verify_test.bin

# 或使用自动化测试框架
hb test -f appverify
```

---

## 产物大小参考

### libhapverify.so 大小（估计）

| 配置 | 大小（大约） | 说明 |
|------|--------------|------|
| Release 构建 | 2-3 MB | 优化后 |
| Debug 构建 | 5-8 MB | 包含符号表 |
| 包含测试符号 | 8-10 MB | 链接了测试 |

### 配置文件大小

| 文件 | 大小（大约） |
|------|--------------|
| trusted_root_ca.json | 2-5 KB |
| trusted_apps_sources.json | 1-3 KB |
| trusted_tickets_sources.json | 0.5-1 KB |

### 总磁盘占用

- 标准系统：约 5-10 MB（库 + 配置）
- Lite 系统：约 1-2 MB（精简库）

---

## 依赖解析

### 依赖库版本要求

| 依赖库 | 最低版本 | 说明 |
|--------|----------|------|
| OpenSSL | 1.1.1 | libcrypto |
| cJSON | 1.7+ | JSON 解析 |
| bounds_checking_function | - | 边界检查库 |
| Hilog | - | 日志库 |

### 编译时依赖 vs 运行时依赖

| 依赖类型 | 库名 | 说明 |
|----------|------|------|
| **编译时** | c_utils, bounds_checking_function | 头文件和链接符号 |
| **运行时** | openssl, cJSON, hilog, libc | 动态加载 |

---

## 故障排查

### 常见问题

#### 1. 库加载失败

**症状**：`error while loading shared libraries: libhapverify.so`

**原因**：库路径不在 LD_LIBRARY_PATH 中

**解决**：
```bash
export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
```

#### 2. 配置文件缺失

**症状**：`VERIFY_SOURCE_INIT_FAIL`

**原因**：`/system/etc/security/` 下配置文件缺失

**解决**：检查文件权限和路径

```bash
ls -l /system/etc/security/trusted_*.json
```

#### 3. OpenSSL 版本不兼容

**症状**：签名验证失败

**原因**：OpenSSL 版本不支持算法

**解决**：升级 OpenSSL 到 1.1.1+

---

## 相关跳转

- [GN 构建目标](06_GN_Targets.md) - Targets 配置
- [对外 API](04_Public_API.md) - 接口使用
- [常见问题](09_FAQ.md) - 排查指南
