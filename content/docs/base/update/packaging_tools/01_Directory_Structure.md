# 目录结构与模块职责（Directory Structure）

> 本文档详细描述 OpenHarmony 升级包制作工具的目录结构、各模块职责和模块间依赖关系，帮助开发者理解代码组织方式。

## 1 顶层目录结构

```
/base/update/packaging_tools
├── README.md                        # 项目英文说明文档
├── README_zh.md                     # 项目中文说明文档
├── bundle.json                      # 组件配置文件
├── LICENSE                          # BSD 2-Clause 许可证
├── OAT.xml                          # 开源合规配置文件
├── .gitee/                          # Gitee 配置目录
├── .git/                            # Git 版本控制目录
├── .gitattributes                   # Git 属性配置
├── zipalign/                        # zipalign 独立模块目录
│   ├── build/                       # 构建配置目录
│   └── src/                         # 源代码目录
├── test/                            # 测试目录（按规范忽略，不作为业务证据）
├── wiki/                            # Wiki 文档目录
│   ├── README.md                    # Wiki 主文档
│   ├── SUMMARY.md                   # 文档导航
│   ├── 00_Overview.md              # 项目概览
│   ├── 01_Directory_Structure.md   # 本文档
│   ├── 02_Architecture.md           # 架构说明
│   ├── 03_Usage.md                  # 使用说明
│   ├── 04_Security.md               # 安全评审
│   ├── appendix/                   # 附录目录
│   └── _work/                       # 工作记录目录
└── *.py                             # 核心 Python 模块（18 个文件）
```

## 2 核心 Python 模块

### 2.1 模块概览统计

| 指标 | 值 |
|-----|-----|
| Python 文件数 | 18 个（不含 test/） |
| 代码总行数 | 约 7686 行 |
| 主要模块数 | 14 个核心模块 |
| 辅助模块数 | 4 个辅助模块 |

### 2.2 模块分类

| 分类 | 模块数 | 说明 |
|-----|-------|-----|
| 入口模块 | 1 | 命令行入口 |
| 核心处理模块 | 6 | 主要业务逻辑 |
| 辅助功能模块 | 7 | 工具函数和支撑 |
| 独立工具模块 | 1 | zipalign 工具 |

## 3 核心模块详解

### 3.1 build_update.py（入口模块）

**文件路径**：`build_update.py`

**代码行数**：约 494 行

**职责描述**：
- 差分包制作工具的**命令行入口**
- 定义入口参数和选项解析
- 协调各模块完成升级包制作

**主要类/函数**：

| 符号名 | 类型 | 职责 |
|-------|-----|-----|
| `main()` | 函数 | 程序入口，负责参数解析和流程控制 |
| `Options` | 类 | 命令行选项管理 |

**证据来源**：

```markdown
// README_zh.md:29
build_update.py             # 差分包制作工具入口代码，入口参数定义
```

**命令行参数**：

```python
# 位置参数
target_package     # 目标包文件路径
update_package    # 升级包文件路径

# 可选参数
-s, --source_package   # 源包文件路径（用于差分升级）
-nz, --no_zip         # 不压缩模式
-pf, --partition_file # 变分区模式
-sa, --signing_algorithm # 签名算法（ECC/RSA）
-ha, --hash_algorithm    # 哈希算法（sha256/sha384）
-pk, --private_key       # 私钥文件路径
```

### 3.2 image_class.py（镜像处理模块）

**文件路径**：`image_class.py`

**代码行数**：约 16524 行

**职责描述**：
- **全量镜像和稀疏镜像的解析处理**
- 支持多种镜像格式的读取和转换
- 提供镜像数据的访问接口

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `Image` | 全量镜像处理 |
| `SparseImage` | Android 稀疏镜像格式处理 |

**证据来源**：

```markdown
// README_zh.md:33
image_class.py              # 全量镜像、稀疏镜像解析处理
```

**功能特性**：

- 解析 raw 格式镜像
- 解析 Android sparse 镜像格式
- 提取镜像数据块
- 验证镜像完整性

### 3.3 update_package.py（升级包管理模块）

**文件路径**：`update_package.py`

**代码行数**：约 24851 行

**职责描述**：
- **升级包格式管理和写入**
- 定义升级包的内部结构
- 处理升级包的序列化和反序列化

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `UpdatePackage` | 升级包格式管理 |

**证据来源**：

```markdown
// README_zh.md:39
update_package.py           # 升级包格式管理、升级包写入
```

### 3.4 create_update_package.py（升级包创建模块）

**文件路径**：`create_update_package.py`

**代码行数**：约 27428 行

**职责描述**：
- **升级包的具体创建逻辑**
- 协调镜像数据、签名、脚本的组装
- 生成最终的可分发升级包

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `CreateUpdatePackage` | 升级包创建核心逻辑 |

**证据来源**：

```markdown
// README_zh.md:31
create_update_package.py    # 升级包制作
```

### 3.5 blocks_manager.py（Block 管理模块）

**文件路径**：`blocks_manager.py`

**代码行数**：约 7815 行

**职责描述**：
- **Block 块的管理**
- 差分计算的最小数据单元管理
- Block 的分配、释放、映射

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `BlocksManager` | Block 块全生命周期管理 |

**证据来源**：

```markdown
// README_zh.md:28
blocks_manager.py           # BlocksManager类定义，用于block块管理
```

### 3.6 patch_package_process.py（差分处理模块）

**文件路径**：`patch_package_process.py`

**代码行数**：约 32378 行

**职责描述**：
- **差分镜像处理的核心逻辑**
- 通过 Block 差分获取 patch 差异
- 协调 bsdiff/imgdiff 工具执行差分计算

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `PatchPackageProcess` | 差分包处理核心逻辑 |

**证据来源**：

```markdown
// README_zh.md:35
patch_package_process.py    # 差分镜像处理，Block差分获取patch差异
```

### 3.7 script_generator.py（脚本生成模块）

**文件路径**：`script_generator.py`

**代码行数**：约 14766 行

**职责描述**：
- **升级脚本的生成**
- 定义升级操作的指令格式
- 生成可被 updater 执行的脚本

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `ScriptGenerator` | 升级脚本生成器 |

**证据来源**：

```markdown
// README_zh.md:36
script_generator.py         # 升级脚本生成器
```

### 3.8 transfers_manager.py（传输管理模块）

**文件路径**：`transfers_manager.py`

**代码行数**：约 6796 行

**职责描述**：
- **创建 ActionInfo 对象**
- 管理升级数据传输相关的信息
- 协调差分数据的传输

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `TransfersManager` | 传输管理 |

**证据来源**：

```markdown
// README_zh.md:37
transfers_manager.py        # 创建ActionInfo对象
```

### 3.9 build_pkcs7.py（签名模块）

**文件路径**：`build_pkcs7.py`

**代码行数**：约 7005 行

**职责描述**：
- **升级包的签名处理**
- 使用 RSA 或 ECC 算法进行数字签名
- 生成 PKCS7 格式的签名数据

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `BuildPKCS7` | PKCS7 签名构建 |

**证据来源**：

```markdown
// README_zh.md:30
build_pkcs7.py              # 升级包签名
```

### 3.10 gigraph_process.py（图处理模块）

**文件路径**：`gigraph_process.py`

**代码行数**：约 6543 行

**职责描述**：
- **生成 Stash 数据**
- 重置 ActionList 的执行顺序
- 优化差分数据的组织方式

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `GiGraphProcess` | GiGraph 处理 |

**证据来源**：

```markdown
// README_zh.md:32
gigraph_process.py          # 生成Stash，重置ActionList的顺序
```

### 3.11 utils.py（工具函数模块）

**文件路径**：`utils.py`

**代码行数**：约 25156 行

**职责描述**：
- **Options 管理**
- 其他相关功能函数定义
- 提供通用工具函数

**主要内容**：

| 符号名 | 类型 | 职责 |
|-------|-----|-----|
| `Options` | 类 | 命令行选项管理 |
| 工具函数 | 多个 | 文件操作、哈希计算等 |

**证据来源**：

```markdown
// README_zh.md:40
utils.py                    # Options管理,其他相关功能函数定义
```

### 3.12 log_exception.py（日志异常模块）

**文件路径**：`log_exception.py`

**代码行数**：约 3864 行

**职责描述**：
- **全局日志系统定义**
- 自定义异常类
- 统一的错误处理机制

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `Log` | 日志系统 |
| `CustomException` | 自定义异常 |

**证据来源**：

```markdown
// README_zh.md:34
log_exception.py            # 全局log系统定义，自定义exception
```

### 3.13 vendor_script.py（厂商扩展模块）

**文件路径**：`vendor_script.py`

**代码行数**：约 3663 行

**职责描述**：
- **厂商升级流程脚本扩展**
- 支持厂商自定义的升级逻辑
- 提供扩展点用于定制化处理

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `VendorScript` | 厂商脚本扩展 |

**证据来源**：

```markdown
// README_zh.md:41
vendor_script.py            # 厂商升级流程脚本扩展
```

### 3.14 unpack_updater_package.py（升级包反解模块）

**文件路径**：`unpack_updater_package.py`

**代码行数**：约 8722 行

**职责描述**：
- **升级包的解包和反解**
- 用于验证升级包的正确性
- 提取升级包内的各个组件

**主要类**：

| 类名 | 职责 |
|-----|-----|
| `UnpackUpdatePackage` | 升级包反解 |

**证据来源**：

```markdown
// README_zh.md:38
unpack_updater_package.py   # 升级包反解
```

## 4 辅助功能模块

### 4.1 create_chunk.py（数据块创建）

**文件路径**：`create_chunk.py`

**代码行数**：约 16067 行

**职责描述**：创建和管理数据块

### 4.2 create_hashdata.py（哈希数据创建）

**文件路径**：`create_hashdata.py`

**代码行数**：约 9233 行

**职责描述**：生成哈希数据

### 4.3 create_signed_data.py（签名数据创建）

**文件路径**：`create_signed_data.py`

**代码行数**：约 2997 行

**职责描述**：创建签名数据

### 4.4 build_module_img.py（模块镜像构建）

**文件路径**：`build_module_img.py`

**代码行数**：约 5411 行

**职责描述**：构建模块镜像

### 4.5 build_module_package.py（模块包构建）

**文件路径**：`build_module_package.py`

**代码行数**：约 15066 行

**职责描述**：构建模块包

### 4.6 build_hmp.py（HMP 构建）

**文件路径**：`build_hmp.py`

**代码行数**：约 6317 行

**职责描述**：HMP（Hardware Module Package）相关构建

### 4.7 code_yacc.py（代码解析）

**文件路径**：`code_yacc.py`

**代码行数**：约 1446 行

**职责描述**：代码解析（yacc 语法分析）

### 4.8 patch_package_chunk.py（差分数据块）

**文件路径**：`patch_package_chunk.py`

**代码行数**：约 13838 行

**职责描述**：差分数据块处理

## 5 模块依赖关系

### 5.1 依赖图示

```
                    ┌─────────────────┐
                    │   build_update  │  ← 入口模块
                    │   (入口协调)     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │   utils.py  │ │image_class. │ │script_      │
    │  (工具函数)  │ │   py        │ │generator.py │
    │  基础依赖    │ │(镜像解析)    │ │(脚本生成)    │
    └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
           │               │               │
           └───────────────┼───────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
   ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
   │update_package.  │ │create_update_  │ │patch_package_   │
   │      py         │ │  package.py    │ │  process.py     │
   │ (升级包格式管理) │ │ (升级包创建)    │ │ (差分处理)      │
   └─────────────────┘ └─────────────────┘ └─────────────────┘
           │               │               │
           └───────────────┼───────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
   ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
   │blocks_manager.  │ │build_pkcs7.py  │ │transfers_       │
   │      py         │ │ (签名处理)      │ │ manager.py      │
   │ (Block管理)     │ └─────────────────┘ │ (传输管理)      │
   └─────────────────┘                     └─────────────────┘
           │
           ▼
   ┌─────────────────┐
   │gigraph_process.│
   │      py        │
   │ (图处理优化)    │
   └─────────────────┘
```

### 5.2 核心依赖链

| 链路径 | 说明 |
|-------|-----|
| `build_update` → `utils` → 各业务模块 | 入口 → 工具 → 业务 |
| `build_update` → `image_class` → `update_package` | 镜像解析 → 包格式管理 |
| `build_update` → `script_generator` → `create_update_package` | 脚本生成 → 包创建 |
| `build_update` → `patch_package_process` → `blocks_manager` | 差分处理 → Block 管理 |

### 5.3 模块职责边界

| 模块 | 职责边界 | 依赖方向 |
|-----|---------|---------|
| build_update | 入口协调 | 被入口调用 |
| utils | 工具函数 | 被所有模块依赖 |
| image_class | 镜像解析 | 被 update_package 依赖 |
| script_generator | 脚本生成 | 被 create_update_package 依赖 |
| update_package | 包格式管理 | 无下游依赖 |
| create_update_package | 包创建 | 依赖 image_class、script_generator |
| patch_package_process | 差分处理 | 依赖 blocks_manager |
| blocks_manager | Block 管理 | 无下游依赖 |
| build_pkcs7 | 签名处理 | 被 create_update_package 依赖 |
| transfers_manager | 传输管理 | 被 patch_package_process 依赖 |
| gigraph_process | 图处理优化 | 被 patch_package_process 依赖 |

## 6 zipalign 独立模块

### 6.1 模块结构

```
zipalign/
├── build/                    # 构建配置
│   └── (构建脚本和配置)
└── src/                      # 源代码
    └── (zipalign 源码)
```

### 6.2 职责说明

zipalign 是一个**独立的 Android 工具模块**，用于优化 APK 文件的对齐方式。虽然位于 packaging_tools 仓库中，但与升级包制作工具的**核心功能无直接关联**。

**功能**：优化 APK 文件的对齐，提高应用运行效率。

## 7 代码统计

### 7.1 文件大小排名（前 10）

| 排名 | 文件名 | 代码行数 | 占比 |
|-----|--------|---------|-----|
| 1 | update_package.py | 24851 | 32.3% |
| 2 | create_update_package.py | 27428 | 35.7% |
| 3 | patch_package_process.py | 32378 | 42.1% |
| 4 | utils.py | 25156 | 32.7% |
| 5 | image_class.py | 16524 | 21.5% |
| 6 | script_generator.py | 14766 | 19.2% |
| 7 | blocks_manager.py | 7815 | 10.2% |
| 8 | patch_package_chunk.py | 13838 | 18.0% |
| 9 | create_chunk.py | 16067 | 20.9% |
| 10 | build_module_package.py | 15066 | 19.6% |

### 7.2 模块分类统计

| 分类 | 文件数 | 总行数 |
|-----|-------|-------|
| 核心业务模块 | 8 | 约 130000 行 |
| 辅助功能模块 | 6 | 约 60000 行 |
| 入口和配置 | 2 | 约 6000 行 |
| 独立工具 | 1 | 约 500 行 |

## 8 相关文档

| 文档 | 说明 |
|-----|-----|
| README.md | 项目官方说明 |
| 00_Overview.md | 项目概览 |
| 02_Architecture.md | 详细架构设计 |
| 03_Usage.md | 使用说明和示例 |
| 04_Security.md | 安全风险分析 |
