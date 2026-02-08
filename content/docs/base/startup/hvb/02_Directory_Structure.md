# 目录结构与模块职责

## 目的

本文档介绍 HVB 组件的目录结构、模块划分和职责分工，帮助读者快速定位代码。

---

## 适用范围

- 目标读者：代码贡献者、模块维护者、Bootloader 适配开发者
- 知识储备：C 项目结构、模块化设计原则

---

## 核心结论

1. **libhvb**：核心校验库，包含 8 个功能模块
2. **tools**：签名工具，仅 1 个 Python 脚本
3. **include/**：公共头文件，11 个对外 API
4. **src/**：实现文件，15 个 .c 文件按功能分组

---

## 顶层目录结构

```
base/startup/hvb/
├── BUILD.gn              # 根构建入口
├── bundle.json           # 部件元数据
├── LICENSE               # Apache 2.0 许可证
├── README.md            # 英文说明
├── README_zh.md         # 中文说明
├── figures/             # 架构图
│
├── libhvb/             # 核心校验库
│   ├── BUILD.gn          # libhvb 构建配置
│   ├── include/          # 公共头文件（11个）
│   └── src/             # 实现文件（15个）
│
├── tools/               # 签名工具
│   ├── hvbtool.py       # 主工具脚本
│   ├── readme.md        # 中文使用说明
│   └── vb_pub_key/      # 公钥说明
│
├── test/                # 测试目录（本文档不涉及）
└── wiki/                # 本文档
```

---

## libhvb 模块划分

### 1. include/ - 公共 API 头文件

| 文件 | 行数 | 职责 | 稳定性 |
|------|------|------|----------|
| `hvb.h` | 102 | 主 API：链式校验、数据初始化 | ✅ 稳定 |
| `hvb_ops.h` | 60 | Bootloader 适配接口 | ✅ 稳定 |
| `hvb_types.h` | 35 | 基础类型定义（hvb_buf） | ✅ 稳定 |
| `hvb_sysdeps.h` | 51 | 平台依赖抽象（内存、字符串） | ✅ 稳定 |
| `hvb_cert.h` | 170 | Verity 证书结构体与解析 | ✅ 稳定 |
| `hvb_footer.h` | 50 | Footer 解析与初始化 | ✅ 稳定 |
| `hvb_rvt.h` | 83 | RVT 结构体与解析 | ✅ 稳定 |
| `hvb_crypto.h` | 80 | 加密接口（RSA、哈希） | ✅ 稳定 |
| `hvb_cmdline.h` | 41 | 启动参数生成 | ✅ 稳定 |
| `hvb_util.h` | 65 | 工具函数宏定义 | ✅ 稳定 |
| `hvb_sm2.h` | 49 | SM2 国密接口 | ✅ 稳定 |
| `hvb_sm3.h` | （未在 sources） | SM3 国密接口 | ⚠️ 内部 |
| `libhvb.h` | 28 | 统一头文件 | ✅ 稳定 |

**证据**：`bundle.json:29-44`（inner_kits 头文件列表）

---

### 2. src/ - 实现文件按功能分组

#### auth/ - 链式校验逻辑

**文件**：`hvb.c`

**职责**：
- 初始化校验数据：`hvb_init_verified_data()`
- 链式校验入口：`hvb_chain_verify()`
- 释放校验数据：`hvb_chain_verify_data_free()`
- RVT 根校验：`hvb_rvt_verify_root()`
- 公钥解析：`hvb_pubkey_parser()`

**依赖**：所有其他模块

**证据**：`libhvb/src/auth/hvb.c:27-102`

---

#### cert/ - 证书解析与验证

**文件**：`hvb_cert.c`

**职责**：
- 解析 verity 证书：`cert_init_desc()`
- 解析证书结构：`hvb_cert_parser()`
- 验证证书签名
- 校验镜像哈希

**依赖**：crypto、util

**证据**：`libhvb/include/hvb_cert.h:160-163`

---

#### cmdline/ - 启动参数生成

**文件**：`hvb_cmdline.c`

**职责**：
- 生成内核启动参数：`hvb_creat_cmdline()`
- 格式化 HVB 配置为 bootargs

**生成的参数**：
- `ohos.boot.hvb.enable`：HVB 使能状态
- `ohos.boot.hvb.version`：HVB 版本
- `ohos.boot.hvb.hash_algo`：哈希算法
- `ohos.boot.hvb.digest`：校验摘要

**证据**：`libhvb/include/hvb_cmdline.h:28-33`

---

#### crypto/ - 加密算法实现

**文件**：
- `hvb_rsa.c` / `hvb_rsa.h` - RSA 加解密
- `hvb_rsa_verify.c` / `hvb_rsa_verify.h` - RSA 验签
- `hvb_sm2.c` - SM2 国密签名
- `hvb_sm2_bn.c` / `hvb_sm2_bn.h` - SM2 大数运算
- `hvb_sm3.c` / `hvb_sm3.h` - SM3 国密哈希
- `hvb_hash_sha256.c` / `hvb_hash_sha256.h` - SHA256 哈希
- `hvb_gm_common.h` - 国密通用定义
- `hvb_gm_log.c` / `hvb_gm_log.h` - 国密日志

**职责**：
- 实现加密算法（不依赖 OpenSSL）
- 支持 RSA-PSS 签名验证
- 支持 SM2/SM3 国密算法

**依赖**：无（纯数学实现）

**证据**：`libhvb/src/crypto/` 目录结构

---

#### deps/ - 平台依赖实现

**文件**：`hvb_sysdeps.c`

**职责**：
- 内存函数包装：`hvb_malloc()`、`hvb_free()`、`hvb_calloc()`
- 字符串函数：`hvb_strcmp()`、`hvb_strncmp()`、`hvb_strlen()`
- 内存操作：`hvb_memcpy()`、`hvb_memset()`
- 调试输出：`hvb_print()`、`hvb_printv()`

**安全实践**：包装系统内存函数，使用 securec 库进行边界检查

**证据**：`libhvb/include/hvb_sysdeps.h:29-44`

---

#### footer/ - Footer 解析

**文件**：`hvb_footer.c`

**职责**：
- 初始化 footer 描述：`footer_init_desc()`
- 解析 footer 魔数和结构
- 定位证书在镜像中的位置

**依赖**：cert、rvt

**证据**：`libhvb/include/hvb_footer.h:42-44`

---

#### rvt/ - RVT 处理

**文件**：`hvb_rvt.c`

**职责**：
- 解析 RVT 头部：`hvb_rvt_head_parser()`
- 获取公钥描述符：`hvb_rvt_get_pubk_desc()`
- 解析公钥描述符：`hvb_rvt_pubk_desc_parser()`
- 获取公钥数据：`hvb_rvt_get_pubk_buf()`
- 计算证书摘要：`hvb_calculate_certs_digest()`

**依赖**：cert

**证据**：`libhvb/include/hvb_rvt.h:71-76`

---

#### utils/ - 工具函数

**文件**：`hvb_util.c`

**职责**：
- 字节序转换：`hvb_be64toh()`、`hvb_htobe64()`
- 字符串操作：`hvb_strdup()`
- 类型转换：`hvb_uint64_to_base10()`
- 十六进制转换：`hvb_bin2hex()`
- ops 检查：`check_hvb_ops()`

**依赖**：无

**证据**：`libhvb/include/hvb_util.h:49-59`

---

## tools 模块

### hvbtool.py

**文件**：`tools/hvbtool.py`（1206+ 行）

**职责**：
- 镜像签名：生成 verity footer
- RVT 制作：制作 Root Verity Table
- 镜像解析：解析 HVB 格式镜像
- 镜像擦除：移除 footer 和签名

**支持命令**：
1. `make_hash_footer` - Hash 模式签名
2. `make_hashtree_footer` - Hashtree 模式签名
3. `make_rvt_image` - 制作 RVT
4. `parse_image` - 解析镜像
5. `erase_image` - 擦除签名

**依赖**：Python 3.0+、OpenSSL

**证据**：`tools/readme.md`（中文使用说明）

---

## 模块依赖关系

### 依赖方向

```
hvb.c (auth)
  ├── cert/
  │   ├── crypto/ (RSA, SM2, SM3, SHA256)
  │   └── footer/
  ├── rvt/
  │   └── cert/
  ├── cmdline/
  └── utils/
      └── deps/ (sysdeps)
```

### 依赖规则

1. **auth/** 依赖所有模块（顶层入口）
2. **cert/** 依赖 crypto/、footer/、rvt/
3. **crypto/** 独立模块（无依赖）
4. **rvt/** 依赖 cert/
5. **utils/** 独立模块（无依赖）
6. **deps/** 独立模块（平台底层）

**证据**：`libhvb/include/libhvb.h:15-26`（头文件包含顺序）

---

## 代码组织原则

### 1. 头文件与实现分离

- **include/**：所有公共头文件
- **src/**：所有实现文件
- 头文件中仅声明接口，不包含实现

### 2. 按功能分组

- **auth/**：校验逻辑
- **cert/**：证书处理
- **crypto/**：加密算法
- **deps/**：平台依赖
- 各模块职责清晰，避免循环依赖

### 3. 平台抽象

- **hvb_ops.h**：定义 Bootloader 适配接口
- **hvb_sysdeps.h**：定义平台依赖函数
- 实现可替换，便于移植

### 4. 国密支持

- **sm2**、**sm3**：独立模块，可选编译
- 与标准算法（RSA、SHA256）平级

---

## 相关跳转

- [项目定位与边界](01_Project_Scope.md) - HVB 能力范围
- [架构说明](03_Architecture.md) - 组件依赖图与数据流
- [内部 API](05_Internal_API.md) - 模块接口详情

---

*最后更新: 2026-02-06*
