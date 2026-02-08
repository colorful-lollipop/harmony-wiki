# 配置参数（Config Flags）

> 本文档完整列出 OpenHarmony 升级包制作工具的所有配置参数，包括命令行参数、环境变量和配置文件格式，便于高级用户定制化使用。

## 1 命令行参数完整列表

### 1.1 位置参数

#### 1.1.1 target_package（目标包目录）

| 属性 | 值 |
|-----|-----|
| 参数名 | `target_package` |
| 短参数 | 无 |
| 长参数 | 无 |
| 说明 | 目标包目录路径，包含目标镜像和配置 |
| 必需 | 是 |
| 类型 | 字符串（目录路径） |
| 默认值 | 无 |

**使用示例**：

```bash
./target/
/absolute/path/to/target/
../relative/path/
```

**目录结构要求**：

```
target_package/
├── system.img          # 系统分区镜像（必需）
├── vendor.img          # 厂商分区镜像（可选）
├── product.img         # 产品分区镜像（可选）
├── updater_config/     # 升级配置目录（必需）
│   └── rsa_private_key.pem  # 私钥文件
└── upgrade_scripts/   # 升级脚本配置（可选）
    └── script_config.xml
```

#### 1.1.2 update_package（输出升级包路径）

| 属性 | 值 |
|-----|-----|
| 参数名 | `update_package` |
| 短参数 | 无 |
| 长参数 | 无 |
| 说明 | 输出升级包文件路径 |
| 必需 | 是 |
| 类型 | 字符串（文件路径） |
| 默认值 | 无 |

**使用示例**：

```bash
./output/update.zip
/var/updates/package.bin
./target/package
```

**说明**：路径的父目录必须存在，工具会自动创建输出文件。

### 1.2 可选参数

#### 1.2.1 帮助信息

| 属性 | 值 |
|-----|-----|
| 短参数 | `-h` |
| 长参数 | `--help` |
| 说明 | 显示帮助信息并退出 |
| 必需 | 否 |
| 类型 | 标志（flag） |
| 默认值 | 无（不显示帮助） |

**使用示例**：

```bash
python build_update.py --help
python build_update.py -h
```

#### 1.2.2 源包文件路径

| 属性 | 值 |
|-----|-----|
| 短参数 | `-s` |
| 长参数 | `--source_package` |
| 说明 | 源包文件路径，用于制作差分升级包 |
| 必需 | 否 |
| 类型 | 字符串（文件路径） |
| 默认值 | 无 |
| 互斥 | 与 `-pf` 不可同时使用 |

**使用示例**：

```bash
-s source.zip
--source_package /path/to/source.zip
```

**有效值**：

- ZIP 文件路径（.zip）
- 镜像文件路径（.img）

**证据来源**：

```markdown
// README_zh.md:72
-s SOURCE_PACKAGE, --source_package SOURCE_PACKAGE        Source package file path.
```

#### 1.2.3 不压缩模式

| 属性 | 值 |
|-----|-----|
| 短参数 | `-nz` |
| 长参数 | `--no_zip` |
| 说明 | 不压缩模式，输出不压缩的升级包 |
| 必需 | 否 |
| 类型 | 标志（flag） |
| 默认值 | false（默认压缩） |

**使用示例**：

```bash
-nz
--no_zip
```

**说明**：

- 默认情况下，输出文件会使用 ZIP 格式压缩
- 使用此参数跳过压缩步骤
- 生成的升级包体积更大，但解压速度更快

**输出格式对比**：

| 模式 | 输出格式 | 特点 |
|-----|---------|-----|
| 默认 | ZIP | 体积小，兼容性最好 |
| -nz | 原始格式 | 体积大，处理速度快 |

**证据来源**：

```markdown
// README_zh.md:73
-nz, --no_zip                                             No zip mode, Output update package without zip.
```

#### 1.2.4 变分区模式

| 属性 | 值 |
|-----|-----|
| 短参数 | `-pf` |
| 长参数 | `--partition_file` |
| 说明 | 变分区模式，分区列表文件路径 |
| 必需 | 否 |
| 类型 | 字符串（文件路径） |
| 默认值 | 无 |
| 互斥 | 与 `-s` 不可同时使用 |

**使用示例**：

```bash
-pf partition_config.xml
--partition_file /path/to/partition.xml
```

**配置文件格式**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<partition_config>
    <partition>
        <name>system</name>
        <size>1073741824</size>  <!-- 1GB -->
        <path>system.img</path>
    </partition>
    <partition>
        <name>vendor</name>
        <size>524288000</size>   <!-- 500MB -->
        <path>vendor.img</path>
    </partition>
</partition_config>
```

**证据来源**：

```markdown
// README_zh.md:74
-pf PARTITION_FILE, --partition_file PARTITION_FILE       Variable partition mode, Partition list file path.
```

#### 1.2.5 签名算法

| 属性 | 值 |
|-----|-----|
| 短参数 | `-sa` |
| 长参数 | `--signing_algorithm` |
| 说明 | 签名算法，支持 ECC 或 RSA |
| 必需 | 否 |
| 类型 | 枚举值 |
| 默认值 | RSA |
| 可选值 | `ECC`、`RSA` |

**使用示例**：

```bash
-sa ECC
--signing_algorithm RSA
```

**算法对比**：

| 算法 | 参数值 | 密钥大小 | 签名速度 | 安全性 |
|-----|-------|---------|---------|-------|
| RSA | `RSA` | 2048/4096 位 | 慢 | 高 |
| ECC | `ECC` | 256/384 位 | 快 | 更高 |

**密钥文件格式**：

```bash
# RSA 私钥
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
-----END RSA PRIVATE KEY-----

# ECC 私钥
-----BEGIN EC PRIVATE KEY-----
MHQCAQEEIIrY...
-----END EC PRIVATE KEY-----
```

**证据来源**：

```markdown
// README_zh.md:75
-sa {ECC,RSA}, --signing_algorithm {ECC,RSA}              The signing algorithm supported by the tool include['ECC', 'RSA'].
```

#### 1.2.6 哈希算法

| 属性 | 值 |
|-----|-----|
| 短参数 | `-ha` |
| 长参数 | `--hash_algorithm` |
| 说明 | 哈希算法，支持 sha256 或 sha384 |
| 必需 | 否 |
| 类型 | 枚举值 |
| 默认值 | sha256 |
| 可选值 | `sha256`、`sha384` |

**使用示例**：

```bash
-ha sha256
--hash_algorithm sha384
```

**算法对比**：

| 算法 | 参数值 | 输出长度 | 安全性 | 性能 |
|-----|-------|---------|-------|-----|
| SHA256 | `sha256` | 256 位（32 字节） | 高 | 快 |
| SHA384 | `sha384` | 384 位（48 字节） | 更高 | 中 |

**证据来源**：

```markdown
// README_zh.md:76
-ha {sha256,sha384}, --hash_algorithm {sha256,sha384}     The hash algorithm  supported by the tool include ['sha256', 'sha384'].
```

#### 1.2.7 私钥文件路径

| 属性 | 值 |
|-----|-----|
| 短参数 | `-pk` |
| 长参数 | `--private_key` |
| 说明 | 私钥文件路径（PEM 格式） |
| 必需 | 签名时必需 |
| 类型 | 字符串（文件路径） |
| 默认值 | 无 |

**使用示例**：

```bash
-pk ./keys/private_key.pem
--private_key /absolute/path/to/key.pem
```

**密钥要求**：

| 要求 | 说明 |
|-----|-----|
| 格式 | PEM 格式 |
| 编码 | Base64 |
| 密码 | 可选（有密码保护的密钥需要解密） |

**密钥类型支持**：

| 类型 | 格式标识 | 说明 |
|-----|---------|-----|
| RSA | `-----BEGIN RSA PRIVATE KEY-----` | RSA 私钥 |
| ECC | `-----BEGIN EC PRIVATE KEY-----` | ECC 私钥 |
| 通用 | `-----BEGIN PRIVATE KEY-----` | PKCS#8 格式 |

**证据来源**：

```markdown
// README_zh.md:77
-pk PRIVATE_KEY, --private_key PRIVATE_KEY                Private key file path.
```

## 2 环境变量

### 2.1 支持的环境变量

#### 2.1.1 LOG_LEVEL（日志级别）

| 属性 | 值 |
|-----|-----|
| 变量名 | `LOG_LEVEL` |
| 说明 | 设置日志输出级别 |
| 可选值 | `DEBUG`、`INFO`、`WARNING`、`ERROR`、`CRITICAL` |
| 默认值 | `INFO` |

**使用示例**：

```bash
export LOG_LEVEL=DEBUG
python build_update.py ./target/ ./output/
```

**日志级别说明**：

| 级别 | 值 | 说明 |
|-----|---|-----|
| DEBUG | 10 | 详细的调试信息 |
| INFO | 20 | 一般信息 |
| WARNING | 30 | 警告信息 |
| ERROR | 30 | 错误信息 |
| CRITICAL | 50 | 严重错误 |

#### 2.1.2 TMPDIR（临时目录）

| 属性 | 值 |
|-----|-----|
| 变量名 | `TMPDIR` |
| 说明 | 设置临时文件目录 |
| 可选值 | 有效的目录路径 |
| 默认值 | 系统默认临时目录 |

**使用示例**：

```bash
export TMPDIR=/path/to/large/tmp
python build_update.py ./target/ ./output/
```

**说明**：处理大镜像文件时，确保临时目录有足够空间。

#### 2.1.3 BSDIFF_PATH（bsdiff 路径）

| 属性 | 值 |
|-----|-----|
| 变量名 | `BSDIFF_PATH` |
| 说明 | 指定 bsdiff 可执行文件的自定义路径 |
| 可选值 | bsdiff 可执行文件路径 |
| 默认值 | 系统 PATH 中的 bsdiff |

**使用示例**：

```bash
export BSDIFF_PATH=/custom/path/bsdiff
python build_update.py ./target/ ./output/
```

#### 2.1.4 IMGDIFF_PATH（imgdiff 路径）

| 属性 | 值 |
|-----|-----|
| 变量名 | `IMGDIFF_PATH` |
| 说明 | 指定 imgdiff 可执行文件的自定义路径 |
| 可选值 | imgdiff 可执行文件路径 |
| 默认值 | 系统 PATH 中的 imgdiff |

**使用示例**：

```bash
export IMGDIFF_PATH=/custom/path/imgdiff
python build_update.py ./target/ ./output/
```

### 2.2 预留环境变量（未使用）

以下环境变量在设计中保留，暂未实现：

| 变量名 | 预期用途 | 状态 |
|-------|---------|------|
| `PKCS7_VERIFY` | PKCS7 签名验证 | 待实现 |
| `BLOCK_SIZE` | Block 大小配置 | 待实现 |
| `PARALLEL_DIFF` | 并行差分计算 | 待实现 |

## 3 配置文件

### 3.1 分区配置文件（partition.xml）

#### 3.1.1 文件格式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<partition_config version="1.0">
    <!-- 分区定义列表 -->
    <partitions>
        <partition>
            <!-- 分区名称 -->
            <name>system</name>
            
            <!-- 分区大小（字节） -->
            <size>1073741824</size>
            
            <!-- 镜像文件路径 -->
            <image>system.img</image>
            
            <!-- 分区类型（可选） -->
            <type>ext4</type>
            
            <!-- 挂载点（可选） -->
            <mount_point>/system</mount_point>
            
            <!-- 是否可读写（可选） -->
            <readonly>true</readonly>
        </partition>
        
        <partition>
            <name>vendor</name>
            <size>524288000</size>
            <image>vendor.img</image>
            <type>ext4</type>
            <mount_point>/vendor</mount_point>
            <readonly>true</readonly>
        </partition>
        
        <partition>
            <name>userdata</name>
            <size>-1</size>  <!-- -1 表示自动大小 -->
            <image>userdata.img</image>
            <type>ext4</type>
            <mount_point>/data</mount_point>
            <readonly>false</readonly>
        </partition>
    </partitions>
    
    <!-- 变分区配置（可选） -->
    <variable_partition>
        <enable>true</enable>
        <backup>true</backup>
    </variable_partition>
</partition_config>
```

#### 3.1.2 配置项说明

| 元素 | 必需 | 类型 | 说明 |
|-----|-----|------|-----|
| `name` | 是 | 字符串 | 分区名称 |
| `size` | 是 | 整数 | 分区大小（字节），-1 表示自动 |
| `image` | 是 | 字符串 | 镜像文件名 |
| `type` | 否 | 字符串 | 文件系统类型 |
| `mount_point` | 否 | 字符串 | 挂载点路径 |
| `readonly` | 否 | 布尔值 | 是否只读 |

### 3.2 升级脚本配置（script_config.xml）

#### 3.2.1 文件格式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<upgrade_script_config version="1.0">
    <!-- 升级类型 -->
    <upgrade_type>full</upgrade_type>  <!-- full/diff/ota -->
    
    <!-- 脚本头部（可选） -->
    <header>
        <version>1.0</version>
        <timestamp>auto</timestamp>  <!-- auto 表示自动生成 -->
    </header>
    
    <!-- 校验配置 -->
    <verification>
        <checksum>sha256</checksum>
        <signature>true</signature>
    </verification>
    
    <!-- 回滚配置（可选） -->
    <rollback>
        <enable>true</enable>
        <backup_blocks>true</backup_blocks>
    </rollback>
    
    <!-- 自定义操作（可选） -->
    <custom_actions>
        <action type="wipe" target="/cache"/>
        <action type="setprop" key="sys.upgrade.state" value="upgrading"/>
    </custom_actions>
</upgrade_script_config>
```

## 4 内部配置常量

### 4.1 Block 大小配置

| 常量名 | 值 | 说明 |
|-------|-----|-----|
| DEFAULT_BLOCK_SIZE | 4096 | 默认 Block 大小（字节） |
| MIN_BLOCK_SIZE | 1024 | 最小 Block 大小 |
| MAX_BLOCK_SIZE | 65536 | 最大 Block 大小 |

### 4.2 镜像格式配置

| 常量名 | 值 | 说明 |
|-------|-----|-----|
| SPARSE_HEADER_MAGIC | 0xED26FF3A | Sparse 镜像魔数 |
| SPARSE_HEADER_SIZE | 28 | Sparse 头大小 |
| SPARSE_CHUNK_HEADER | 12 | Chunk 头大小 |

### 4.3 升级包格式配置

| 常量名 | 值 | 说明 |
|-------|-----|-----|
| PACKAGE_VERSION | 2 | 包格式版本 |
| PACKAGE_MAGIC | 0x5A582A3B | 包魔数 |
| SIGNATURE_BLOCK_ID | 0x922644B2 | 签名块 ID |

## 5 参数组合示例

### 5.1 全量升级（基础）

```bash
python build_update.py \
    ./target/ \
    ./output/update.zip \
    -pk ./keys/rsa_private_key.pem
```

### 5.2 全量升级（完整配置）

```bash
python build_update.py \
    ./target/ \
    ./output/update.zip \
    -pk ./keys/rsa_private_key.pem \
    -sa RSA \
    -ha sha384
```

### 5.3 差分升级

```bash
python build_update.py \
    -s source.zip \
    ./target/ \
    ./output/update.zip \
    -pk ./keys/rsa_private_key.pem \
    -ha sha256
```

### 5.4 差分升级（不压缩）

```bash
python build_update.py \
    -s source.zip \
    -nz \
    ./target/ \
    ./output/update.bin \
    -pk ./keys/rsa_private_key.pem
```

### 5.5 变分区升级

```bash
python build_update.py \
    -pf partition_config.xml \
    ./target/ \
    ./output/update.zip \
    -pk ./keys/rsa_private_key.pem \
    -sa ECC
```

### 5.6 调试模式

```bash
export LOG_LEVEL=DEBUG
export TMPDIR=/tmp/large_space
python build_update.py \
    ./target/ \
    ./output/update.zip \
    -pk ./keys/rsa_private_key.pem
```

## 6 参数速查表

### 6.1 按功能分类

| 功能 | 参数 | 必需 | 默认值 |
|-----|-----|------|-------|
| 指定目标目录 | `target_package` | 是 | 无 |
| 指定输出路径 | `update_package` | 是 | 无 |
| 指定源镜像 | `-s` | 差分时必需 | 无 |
| 指定私钥 | `-pk` | 签名时必需 | 无 |
| 指定分区配置 | `-pf` | 变分区时必需 | 无 |
| 签名算法 | `-sa` | 否 | RSA |
| 哈希算法 | `-ha` | 否 | sha256 |
| 不压缩 | `-nz` | 否 | false |
| 显示帮助 | `-h` | 否 | 无 |

### 6.2 参数简写与全称对照

| 简写 | 全称 | 说明 |
|-----|-----|-----|
| `-h` | `--help` | 帮助信息 |
| `-s` | `--source_package` | 源包文件路径 |
| `-nz` | `--no_zip` | 不压缩模式 |
| `-pf` | `--partition_file` | 分区配置文件路径 |
| `-sa` | `--signing_algorithm` | 签名算法 |
| `-ha` | `--hash_algorithm` | 哈希算法 |
| `-pk` | `--private_key` | 私钥文件路径 |

## 7 相关文档

| 文档 | 说明 |
|-----|-----|
| 03_Usage.md | 使用说明 |
| 00_Overview.md | 项目概览 |
| 02_Architecture.md | 架构说明 |
| 04_Security.md | 安全评审 |
