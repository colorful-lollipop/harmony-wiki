# fsverity-utils

## 库概述

fsverity-utils 是 Linux kernel **fs-verity** 功能的用户空间工具集，提供文件完整性验证的完整解决方案。

**核心功能**：
- 计算文件的 Merkle 树根哈希（摘要）
- 使用 X.509 证书对文件摘要进行 PKCS#7 签名
- 通过 ioctl 接口启用文件的 fs-verity 属性
- 支持 SHA-256 和 SHA-512 哈希算法

**许可证**：MIT License

**上游地址**：https://git.kernel.org/pub/scm/fs/fsverity/fsverity-utils.git

---

## OpenHarmony 适配概述

### 适配策略

该库在 OpenHarmony 中采用**干净导入**策略，无任何 OH 特定代码修改：

| 适配维度 | 状态 |
|----------|------|
| Patch 文件 | 无 |
| OH 特有代码 | 无 |
| 构建适配 | 仅 BUILD.gn 配置 |
| API 差异 | 无 |

### OH 集成点

```
third_party/fsverity-utils/
├── BUILD.gn              # OH 构建配置
├── lib/                  # 库实现（无修改）
│   ├── compute_digest.c
│   ├── enable.c
│   ├── hash_algs.c
│   ├── sign_digest.c
│   └── utils.c
├── include/
│   └── libfsverity.h     # 公共 API
└── common/
    └── fsverity_uapi.h   # 内核接口定义
```

### 依赖关系

- **运行时依赖**：OpenSSL (libcrypto)
- **构建依赖**：OH Build System (GN)

---

## 文档导航

| 文档 | 说明 | 受众 |
|------|------|------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 | 所有开发者 |
| [01_Overview.md](01_Overview.md) | 原始库功能介绍 | 需要了解背景的开发者 |
| [02_Patches.md](02_Patches.md) | Patch 分析 | 维护者、升级工程师 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建配置说明 | 构建系统工程师 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 使用场景 | 应用开发者 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异 | 需要迁移代码的开发者 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

---

## 快速开始

### 在 OH 应用中使用

```c
#include <libfsverity.h>

// 计算文件摘要
struct libfsverity_digest *digest = NULL;
libfsverity_compute_digest(fd, read_fn, &params, &digest);

// 对摘要进行签名
uint8_t *signature = NULL;
size_t sig_size = 0;
libfsverity_sign_digest(digest, &sig_params, &signature, &sig_size);

// 启用 fs-verity
libfsverity_enable_with_sig(fd, &params, signature, sig_size);
```

### 构建产物

| 目标 | 产物 | 安装位置 |
|------|------|----------|
| libfsverity_utils | 共享库 (.so) | system/lib64/ |
| libfsverity_utils_static | 静态库 (.a) | - |

---

## 版本信息

| 版本 | 说明 |
|------|------|
| 上游 v1.6 | 原始版本 |
| OH 3.1 | OH 适配版本 |

---

## 维护指南

### 升级上游版本

由于无 OH 特定 Patch，升级流程相对简单：

1. 替换上游源码
2. 更新 `README.OpenSource` 和 `bundle.json` 版本号
3. 验证 BUILD.gn 兼容性
4. 运行测试验证

### 相关资源

- [Linux fs-verity 文档](https://www.kernel.org/doc/html/latest/filesystems/fsverity.html)
- [上游项目](https://git.kernel.org/pub/scm/fs/fsverity/fsverity-utils.git)
- [OH 第三方组件规范](../docs/third_party_guidelines.md)
