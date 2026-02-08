# 项目概览（Overview）

> 本文档介绍 OpenHarmony 升级包制作工具的项目定位、功能边界、核心能力和技术栈，帮助读者快速建立对项目的整体认知。

## 1 项目定位

### 1.1 所属子系统

Packaging Tools 是 OpenHarmony **升级子系统（updater subsystem）** 的核心组件之一。根据 `bundle.json:14` 配置，该组件归属于 `updater` 子系统，主要负责**升级包的制作和管理**工作。

**证据来源**：

```json
// bundle.json:14
"subsystem": "updater"
```

### 1.2 项目定位声明

Packaging Tools 是用于制作 OpenHarmony 系统升级包的工具，提供以下核心功能：

- **全量升级包制作**：生成包含完整镜像数据的升级包
- **差分升级包制作**：生成只包含变化数据的增量升级包
- **变分区升级包制作**：生成分区表可变的升级包，支持分区调整场景

**证据来源**：

```markdown
// README_zh.md:13-19
升级包制作工具是用于制作升级包的工具，功能主要包括：全量升级包制作、差分升级包制作以及变分区升级包制作。

- 全量升级包制作：升级包中只包括镜像全量升级相关数据，用于镜像全量升级；
- 差分升级包制作：升级包中只包括镜像差分升级相关数据，用于镜像差分升级；
- 变分区升级包：升级包中包括分区表、镜像全量数据，用于变分区处理和变分区后的镜像恢复。
```

### 1.3 项目边界

**在边界内**：

- 升级包的格式设计和生成
- 差分算法的调用和封装
- 升级脚本的生成
- 签名和哈希计算
- 升级包的反解（用于验证）

**在边界外**：

- 升级包的实际安装和执行（由 updater 组件负责）
- 镜像的烧录和写入
- 系统启动和恢复流程
- 第三方差分工具的实现（bsdiff、imgdiff、e2fsdroid）

**证据来源**：

```markdown
// README_zh.md:21
更多升级子系统相关概念，请参考：[升级子系统](https://gitcode.com/openharmony/docs/blob/master/zh-cn/readme/%E5%8D%87%E7%BA%A7%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
```

## 2 核心能力

### 2.1 全量升级包制作

全量升级包制作功能生成包含**完整镜像数据**的升级包，适用于以下场景：

- 首次安装系统
- 镜像损坏需要完整恢复
- 大规模变更导致差分效率不高

**工作流程**：

1. 解析目标镜像文件（全量格式或稀疏格式）
2. 提取镜像数据块
3. 生成升级脚本
4. 计算签名和哈希
5. 打包生成升级包

**相关模块**：

- `image_class.py`：镜像解析
- `update_package.py`：升级包格式管理
- `script_generator.py`：升级脚本生成
- `build_pkcs7.py`：签名处理

### 2.2 差分升级包制作

差分升级包制作功能生成包含**增量数据**的升级包，适用于以下场景：

- 常规软件更新
- 补丁发布
- 网络带宽受限环境

**技术特点**：

- 基于 Block 的差分计算
- 支持多种差分算法（bsdiff、imgdiff）
- 显著减少传输数据量

**相关模块**：

- `blocks_manager.py`：Block 管理
- `patch_package_process.py`：差分处理
- `gigraph_process.py`：差分优化

### 2.3 变分区升级包制作

变分区升级包制作功能生成分区表**可调整**的升级包，适用于以下场景：

- 系统分区方案变更
- 存储容量调整
- OEM 分区定制

**工作流程**：

1. 解析分区表配置文件
2. 提取新分区方案
3. 生成镜像数据
4. 打包包含分区表和镜像的完整升级包

**相关模块**：

- `build_module_package.py`：模块包处理
- `create_update_package.py`：升级包创建

## 3 运行环境

### 3.1 系统要求

| 要求项 | 规格 | 说明 |
|-------|------|-----|
| 操作系统 | Ubuntu 18.04 或更高版本 | 运行环境 |
| Python 版本 | 3.5 及以上 | 编程语言 |
| 内存 | 建议 4GB 以上 | 处理大镜像时需要 |
| 磁盘空间 | 建议 20GB 以上 | 需要存储临时文件 |

**证据来源**：

```markdown
// README_zh.md:47-50
工具运行环境配置：

- Ubuntu18.04或更高版本系统；
- python3.5及以上版本；
```

### 3.2 外部依赖

#### Python 库依赖

| 库名 | 版本 | 用途 | 安装方式 |
|-----|------|-----|---------|
| xmltodict | 最新版本 | XML 文件解析 | `pip install xmltodict` |

**证据来源**：

```markdown
// README_zh.md:52
- python库xmltodict， 解析xml文件，需要单独安装；
```

#### 外部可执行程序

| 程序名 | 用途 | 来源 |
|-------|-----|-----|
| bsdiff | 通用差分计算，生成 patch | 需单独安装 |
| imgdiff | 针对压缩文件的差分计算 | 需单独安装 |
| e2fsdroid | 生成镜像 map 文件 | 需单独安装 |

**证据来源**：

```markdown
// README_zh.md:58-62
- bsdiff可执行程序，差分计算，比较生成patch；
- imgdiff可执行程序，差分计算，针对zip、gz、lz4类型的文件，对比生成patch；
- e2fsdroid可执行程序，差分计算，用于生成镜像的map文件。
```

### 3.3 项目依赖关系

Packaging Tools 作为独立工具运行，不依赖于 OpenHarmony 的其他组件。但生成的升级包会被以下组件使用：

- **updater**：升级包执行组件
- **recovery**：恢复模式组件
- **bootloader**：引导加载程序

## 4 技术栈

### 4.1 编程语言

| 语言 | 版本 | 用途 |
|-----|------|-----|
| Python | 3.5+ | 主要开发语言，所有功能模块 |

**说明**：本项目为纯 Python 实现，无 N-API 或 C/C++ 组件。

### 4.2 核心数据格式

| 格式 | 用途 | 相关模块 |
|-----|-----|---------|
| Raw Image | 原始镜像格式 | image_class.py |
| Sparse Image | Android 稀疏镜像格式 | image_class.py |
| ZIP | 升级包压缩格式 | create_update_package.py |
| PKCS7 | 数字签名格式 | build_pkcs7.py |
| XML | 分区表和配置格式 | xmltodict 解析 |
| JSON | 测试数据格式 | test/ 目录 |

### 4.3 差分算法

| 算法 | 特点 | 适用场景 |
|-----|-----|---------|
| bsdiff | 通用差分算法，适合二进制文件 | 普通差分升级 |
| imgdiff | 针对压缩文件优化 | 镜像差分升级 |
| e2fsdroid | 生成 ext4 镜像的 map | ext4 镜像处理 |

### 4.4 签名算法

| 算法 | 用途 | 配置参数 |
|-----|-----|---------|
| RSA | 数字签名 | `--signing_algorithm RSA` |
| ECC | 数字签名 | `--signing_algorithm ECC` |
| SHA256 | 哈希计算 | `--hash_algorithm sha256` |
| SHA384 | 哈希计算 | `--hash_algorithm sha384` |

**证据来源**：

```markdown
// README_zh.md:75-76
-sa {ECC,RSA}, --signing_algorithm {ECC,RSA}              签名算法
-ha {sha256,sha384}, --hash_algorithm {sha256,sha384}     哈希算法
```

## 5 关键概念

### 5.1 Block

Block 是差分计算的**最小数据单元**，在 `blocks_manager.py` 中定义和管理。

**关键类**：`BlocksManager`

**相关文件**：`blocks_manager.py:1-100`（待验证）

### 5.2 Sparse Image

Sparse Image 是 Android 提出的**稀疏镜像格式**，用于减少镜像文件大小。

**关键类**：`SparseImage`

**相关文件**：`image_class.py:1-100`（待验证）

### 5.3 Action

Action 是升级脚本中的**操作单元**，定义了对分区执行的具体操作。

**相关文件**：`script_generator.py:1-100`（待验证）

### 5.4 Upgrade Package Format

升级包采用**自定义格式**，包含以下组成部分：

1. **包头**：版本、类型、大小等信息
2. **签名块**：PKCS7 签名数据
3. **数据块**：镜像数据或差分数据
4. **脚本块**：升级操作指令

**相关文件**：`update_package.py:1-100`（待验证）

## 6 版本信息

| 版本 | 日期 | 主要变更 |
|-----|------|---------|
| 1.0 | 2026-02-06 | 初始文档版本 |
| 1.1 | - | 项目当前版本（基于 bundle.json） |

## 7 相关文档

| 文档 | 路径 | 说明 |
|-----|-----|-----|
| README.md | 项目根目录 | 官方 README |
| README_zh.md | 项目根目录 | 中文 README |
| bundle.json | 项目根目录 | 组件配置文件 |
| 架构说明 | 02_Architecture.md | 详细架构设计 |
| 使用说明 | 03_Usage.md | 命令行使用指南 |
| 安全评审 | 04_Security.md | 安全风险分析 |

## 8 快速开始

### 8.1 环境准备

```bash
# 1. 安装 Python 依赖
pip install xmltodict

# 2. 安装外部工具（以 Ubuntu 为例）
sudo apt-get install bsdiff

# 3. 克隆项目
git clone https://gitee.com/openharmony/update_packaging_tools.git
```

### 8.2 制作全量升级包

```bash
cd update_packaging_tools
python build_update.py ./target/ ./target/package -pk ./target/updater_config/rsa_private_key2048.pem
```

### 8.3 制作差分升级包

```bash
python build_update.py -s source.zip ./target/ ./target/package -pk ./target/updater_config/rsa_private_key2048.pem
```

**证据来源**：

```markdown
// README_zh.md:80-89
全量升级包制作命令示例：
python build_update.py ./target/ ./target/package -pk ./target/updater_config/rsa_private_key2048.pem

差分升级包制作命令示例：
python build_update.py -s source.zip ./target/ ./target/package -pk./target/updater_config/rsa_private_key2048.pem
```
