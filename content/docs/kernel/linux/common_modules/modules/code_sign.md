# code_sign - 代码签名

## 1. 概述

### 1.1 模块定位

code_sign 提供 ELF 文件签名验证和证书链管理能力，确保只有经过合法签名的代码才能执行。

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/code_sign/
├── code_sign_elf.c/h          # ELF 文件签名处理
├── code_sign_ext.c/h          # 签名描述符验证
├── code_sign_ioctl.c/h        # 用户空间 ioctl 接口
├── code_sign_misc.c           # 杂项工具
├── code_sign_log.h            # 日志宏
├── verify_cert_chain.c/h      # 证书链验证
├── Kconfig                     # 配置: CONFIG_SECURITY_CODE_SIGN
├── Makefile                    # 构建5个目标文件
└── apply_code_sign.sh         # 部署脚本
```

---

## 3. 核心数据结构

| 数据类型 | 说明 |
|----------|------|
| `sign_block_t` | 签名块结构 |
| `sign_head_t` | 签名头结构 |
| `merkle_tree_t` | Merkle 树结构 |
| `CODE_SIGNING_DATA_TYPE` | 代码签名数据类型 |
| `BLOCK_TYPE` | 块类型枚举 |

---

## 4. 主要 API

### 4.1 ELF 签名

| 函数 | 功能 |
|------|------|
| `elf_file_enable_fs_verity()` | 启用文件系统完整性验证 |

### 4.2 证书链管理

| 函数 | 功能 |
|------|------|
| `cert_chain_search()` | 在 RB-tree 中搜索证书 |
| `cert_chain_insert()` | 插入证书到链 |
| `cert_chain_remove()` | 从链中移除证书 |
| `find_match()` | 匹配证书（主题/颁发者） |

### 4.3 ioctl 接口

| 函数 | 功能 |
|------|------|
| `code_sign_ioctl()` | 主 ioctl 处理器 |
| `code_sign_avc_has_perm()` | SELinux 权限检查 |

### 4.4 描述符验证

| 函数 | 功能 |
|------|------|
| `code_sign_check_descriptor()` | 验证签名描述符 |
| `code_sign_before_measurement()` | 测量前检查 |
| `code_sign_after_measurement()` | 测量后处理 |
| `code_sign_init_salt()` | 初始化盐值 |
| `code_sign_set_ownerid()` | 设置所有者 ID |

---

## 5. 相关文档

| 文档 | 说明 |
|------|------|
| [modules/xpm.md](xpm.md) | 可执行权限管理（依赖代码签名） |
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |
