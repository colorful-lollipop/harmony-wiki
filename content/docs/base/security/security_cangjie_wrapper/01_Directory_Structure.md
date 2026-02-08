# 目录结构

## 顶层结构

```
security_cangjie_wrapper/
├── figures/                    # 架构图资源
├── kit/                         # Kit 导出层（Cangjie 应用入口）
│   ├── CryptoArchitectureKit/   # 加密算法 Kit
│   └── UniversalKeystoreKit/   # 密钥管理 Kit
├── ohos/                        # Wrapper 实现层
│   └── security/                # 安全模块
│       ├── crypto_framework/    # Crypto Framework 封装
│       └── huks/                # HUKS 封装
├── mock/                        # Mock 实现（跨平台编译）
├── test/                        # 测试代码（不纳入 Wiki 分析）
├── wiki/                        # 工程文档
├── BUILD.gn                     # 根构建配置
├── bundle.json                  # 组件配置
├── LICENSE                      # Apache 2.0
└── README.md                    # 项目说明
```

**证据来源**：`README.md:84-99`（Directory Structure 章节）

---

## Kit 导出层（kit/）

Kit 层是 Cangjie 应用的直接入口，提供符合 OpenHarmony 规范的模块导出。

### CryptoArchitectureKit/

| 文件 | 职责 |
|------|------|
| `index.cj` | 导出 crypto_framework 模块 |

**证据来源**：`kit/CryptoArchitectureKit/index.cj:18-20`
```cj
package kit.CryptoArchitectureKit
public import ohos.security.crypto_framework.*
```

### UniversalKeystoreKit/

| 文件 | 职责 |
|------|------|
| `index.cj` | 导出 huks 模块 |

**证据来源**：`kit/UniversalKeystoreKit/index.cj:18-20`
```cj
package kit.UniversalKeystoreKit
public import ohos.security.huks.*
```

---

## Wrapper 实现层（ohos/）

Wrapper 层是核心实现，处理 FFI 桥接、类型转换和错误映射。

### crypto_framework/

| 文件 | 职责 | 稳定性 |
|------|------|--------|
| `cj_crypto_interface.cj` | 核心接口定义（Key、ParamsSpec） | Stable |
| `cj_crypto_native.cj` | Native 数据结构（HcfBlob） | Stable |
| `cj_crypto_enum.cj` | 枚举与错误码 | Stable |
| `cipher.cj` | 加密/解密实现 | Stable |
| `sym_key.cj` | 对称密钥实现 | Stable |
| `sym_key_generator.cj` | 对称密钥生成器 | Stable |
| `md.cj` | 消息摘要实现 | Stable |
| `random.cj` | 随机数生成器 | Stable |
| `mac.cj` | MAC 实现 | Stable |
| `cj_crypto_common.cj` | 公共数据结构 | Stable |
| `cj_crypto_log.cj` | 日志接口（依赖 hiviewdfx） | Stable |

**证据来源**：`ohos/security/crypto_framework/BUILD.gn:14-27`

### huks/

| 文件 | 职责 | 稳定性 |
|------|------|--------|
| `huks_session.cj` | 会话操作（init/update/finish） | Stable |
| `huks_key_item.cj` | 密钥项操作（生成/导入/删除） | Stable |
| `huks_ffi.cj` | FFI 函数声明 | Stable |
| `huks_struct.cj` | HUKS 数据结构 | Stable |
| `huks_struct_ffi.cj` | FFI 结构体定义 | Stable |
| `huks_enum.cj` | 枚举与常量定义 | Stable |
| `huks_err.cj` | 错误码映射 | Stable |
| `huks_limit.cj` | 常量限制（MAX_KEY_SIZE 等） | Stable |

**证据来源**：`ohos/security/huks/BUILD.gn:14-24`

### security/

| 文件 | 职责 |
|------|------|
| `security.cj` | 包声明（`package ohos.security`） |

**证据来源**：`ohos/security/security.cj:18`

---

## 构建配置

### 根目录

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | 定义 Cangjie SDK 复制任务 |
| `bundle.json` | 组件元数据、依赖、构建配置 |

**证据来源**：`BUILD.gn:1-24`、`bundle.json:1-50`

### 模块构建

| 路径 | 构建文件 | 目标类型 |
|------|----------|----------|
| `kit/CryptoArchitectureKit/` | `BUILD.gn` | `ohos_cangjie_shared_library` |
| `kit/UniversalKeystoreKit/` | `BUILD.gn` | `ohos_cangjie_shared_library` |
| `ohos/security/` | `BUILD.gn` | `ohos_cangjie_shared_library` |
| `ohos/security/crypto_framework/` | `BUILD.gn` | `ohos_cangjie_shared_library` |
| `ohos/security/huks/` | `BUILD.gn` | `ohos_cangjie_shared_library` |

**证据来源**：各目录 `BUILD.gn` 文件

---

## 模块职责归类

| 层级 | 模块 | 对外接口 | 依赖方向 |
|------|------|----------|----------|
| Kit | CryptoArchitectureKit | `kit.CryptoArchitectureKit` | → crypto_framework |
| Kit | UniversalKeystoreKit | `kit.UniversalKeystoreKit` | → huks |
| Wrapper | crypto_framework | `ohos.security.crypto_framework` | → crypto_framework native |
| Wrapper | huks | `ohos.security.huks` | → huks native |

**依赖原则**：
- Kit 层只依赖 Wrapper 层
- Wrapper 层依赖 Native 实现（`crypto_framework:cj_cryptoframework_ffi`、`huks:cj_huks_ffi`）
- 不允许反向依赖

---

## Mock 目录

| 路径 | 用途 |
|------|------|
| `mock/` | 跨平台编译时使用的 stub 实现 |

**说明**：在 Mingw/Mac 环境下，使用 `mock/*.cj` 文件作为源码，避免 Native 依赖导致编译失败。

**证据来源**：`ohos/security/BUILD.gn:10-13`、`ohos/security/crypto_framework/BUILD.gn:10-13`
