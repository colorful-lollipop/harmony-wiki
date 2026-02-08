# 01_Directory_Structure - 目录结构

## 目的

本文档介绍 `syscap_codec` 项目的目录组织结构和各模块职责。

## 适用范围

- 新加入项目的开发人员
- 需要定位代码的开发者

## 顶层目录结构

```
syscap_codec/
├── include/                    # 公共头文件
├── interfaces/                 # 接口定义
│   └── inner_api/              # 内部API接口
├── napi/                       # N-API接口实现（传统JS接口）
├── taihe/                      # Taihe/ANI接口实现（新JS接口）
├── src/                        # 核心源码
├── tools/                      # Python工具脚本
├── test/                       # 测试代码（本Wiki不深入）
├── BUILD.gn                    # 主构建文件
├── bundle.json                 # 组件配置
├── config.gni                  # 构建配置
├── libsyscap_interface_shared.versionscript  # 动态库导出符号
├── LICENSE                     # Apache 2.0许可证
├── README.md                   # 英文README
├── README_ZH.md                # 中文README
├── RELEASE-NOTE.txt            # 发布说明
└── OAT.xml                     # 开源合规检查配置
```

## 各目录职责

### 1. include/ - 公共头文件

**路径**: `include/`

**职责**: 定义对外暴露的接口和数据结构

**文件清单**:

| 文件 | 行数 | 职责 |
|------|------|------|
| `codec_config/syscap_define.h` | 805 | 系统能力枚举定义和映射表 |
| `context_tool.h` | 54 | 上下文工具接口（文件操作、JSON创建） |
| `create_pcid.h` | 44 | PCID创建和解码接口 |
| `syscap_tool.h` | 51 | 主工具接口（PCID/RPCID编解码） |

**关键符号**:
- `SyscapNum` 枚举 (`syscap_define.h:36`)
- `g_arraySyscap[]` 映射表 (`syscap_define.h:414`)
- `RPCIDHead` 结构 (`context_tool.h:36`)
- `PCIDMain` 结构 (`create_pcid.h:23`)

### 2. interfaces/inner_api/ - 内部API

**路径**: `interfaces/inner_api/`

**职责**: 提供给其他部件调用的C/C++接口

**文件清单**:

| 文件 | 行数 | 职责 |
|------|------|------|
| `syscap_interface.h` | 79 | 内部API头文件 |
| `syscap_interface.c` | 703 | 内部API实现 |

**导出接口** (见 `libsyscap_interface_shared.versionscript`):
- `EncodeOsSyscap` - 编码OS系统能力
- `DecodeOsSyscap` - 解码OS系统能力
- `EncodePrivateSyscap` - 编码私有系统能力
- `DecodePrivateSyscap` - 解码私有系统能力
- `ComparePcidString` - 比较PCID字符串
- `DecodeRpcidToStringFormat` - 解码RPCID为字符串
- `FreeCompareError` - 释放比较错误信息

### 3. napi/ - N-API接口

**路径**: `napi/`

**职责**: 提供JavaScript可调用的N-API接口（传统方式）

**文件清单**:

| 文件 | 行数 | 职责 |
|------|------|------|
| `BUILD.gn` | 68 | N-API模块构建配置 |
| `napi_query_syscap.cpp` | 261 | N-API实现 |
| `query_syscap.js` | 22 | JS接口封装 |

**关键符号**:
- `QuerySystemCapability()` - 查询系统能力 (`napi_query_syscap.cpp:177`)
- `g_systemCapabilityModule` - N-API模块定义 (`napi_query_syscap.cpp:244`)
- `SystemCapabilityRegisterModule()` - 模块注册函数 (`napi_query_syscap.cpp:257`)

### 4. taihe/ - Taihe/ANI接口

**路径**: `taihe/`

**职责**: 提供JavaScript可调用的ANI接口（新方式，ArkCompiler）

**文件清单**:

| 文件 | 行数 | 职责 |
|------|------|------|
| `BUILD.gn` | 23 | Taihe组构建配置 |
| `syscap/BUILD.gn` | 87 | Taihe模块构建配置 |
| `syscap/idl/ohos.systemCapability.taihe` | 24 | IDL接口定义 |
| `syscap/src/ani_constructor.cpp` | 30 | ANI构造函数 |
| `syscap/src/ohos.systemCapability.impl.cpp` | 169 | ANI接口实现 |

**关键符号**:
- `ANI_Constructor()` - ANI构造函数 (`ani_constructor.cpp:17`)
- `querySystemCapabilitie()` - 查询系统能力 (`ohos.systemCapability.impl.cpp:144`)

### 5. src/ - 核心源码

**路径**: `src/`

**职责**: 实现核心编解码逻辑

**文件清单**:

| 文件 | 行数 | 职责 |
|------|------|------|
| `main.c` | 226 | 命令行工具入口 |
| `syscap_tool.c` | 770 | 核心编解码实现 |
| `create_pcid.c` | 926 | PCID创建详细实现 |
| `context_tool.c` | 212 | 上下文工具实现 |
| `endian_internal.c` | 54 | 大小端转换实现 |
| `endian_internal.h` | 38 | 大小端转换头文件 |
| `common_method.c` | 25 | 通用方法实现 |
| `common_method.h` | 23 | 通用方法头文件 |

**关键函数**:
- `main()` - 命令行入口 (`main.c:71`)
- `RPCIDEncode()` / `RPCIDDecode()` - RPCID编解码 (`syscap_tool.c:145`, `257`)
- `CreatePCID()` / `DecodePCID()` - PCID创建解码 (`create_pcid.c:247`, `478`)
- `GetFileContext()` - 读取文件内容 (`context_tool.c:40`)

### 6. tools/ - Python工具

**路径**: `tools/`

**职责**: 提供Syscap一致性检查和收集工具

**文件清单**:

| 文件 | 行数 | 职责 |
|------|------|------|
| `syscap_check.py` | 311 | Syscap一致性检查 |
| `syscap_collector.py` | 545 | Syscap收集器 |
| `syscap_config_merge.py` | 158 | 配置合并工具 |
| `requirements.txt` | - | Python依赖 |

**使用场景**:
- 检查部件Syscap定义与codec定义是否一致
- 检查SDK接口Syscap标注是否一致
- 收集产品Syscap配置

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                        应用层 (JS)                           │
│  ┌─────────────────┐  ┌─────────────────────────────────┐  │
│  │   N-API接口     │  │      Taihe/ANI接口               │ │
│  │  napi/*.js      │  │  idl/*.taihe                    │ │
│  └────────┬────────┘  └──────────────┬──────────────────┘  │
└───────────┼──────────────────────────┼──────────────────────┘
            │                          │
            ▼                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Native实现层                              │
│  ┌─────────────────┐  ┌─────────────────────────────────┐  │
│  │ napi_query_     │  │  ohos.systemCapability.impl.cpp │  │
│  │ syscap.cpp      │  │  ani_constructor.cpp            │  │
│  └────────┬────────┘  └──────────────┬──────────────────┘  │
└───────────┼──────────────────────────┼──────────────────────┘
            │                          │
            └──────────┬───────────────┘
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    内部API层                                 │
│              interfaces/inner_api/                           │
│              syscap_interface.c/h                            │
└────────────────────────┬────────────────────────────────────┘
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
┌─────────────────┐ ┌──────────┐ ┌──────────────┐
│   核心编解码     │ │  工具函数 │ │   平台抽象    │
│  src/syscap_    │ │  src/    │ │  src/endian_ │
│  tool.c         │ │  context_│ │  internal.c  │
│  src/create_    │ │  tool.c  │ │              │
│  pcid.c         │ │          │ │              │
└─────────────────┘ └──────────┘ └──────────────┘
```

## 代码统计

| 目录 | 文件数 | 代码行数(估算) | 主要语言 |
|------|--------|----------------|----------|
| include/ | 4 | ~950 | C/C++ |
| interfaces/inner_api/ | 2 | ~780 | C |
| napi/ | 3 | ~350 | C++/JS |
| taihe/ | 4 | ~310 | C++ |
| src/ | 8 | ~2,200 | C |
| tools/ | 3 | ~1,000 | Python |
| **总计** | **24** | **~5,600** | - |

## 关键配置文件

### bundle.json

**路径**: `bundle.json`

**职责**: 定义组件元数据、依赖、构建目标

**关键配置**:
- 组件名: `syscap_codec`
- 子系统: `developtools`
- 系统能力: `SystemCapability.Developtools.Syscap`
- 依赖: `napi`, `bounds_checking_function`, `cJSON`, `runtime_core`

### BUILD.gn

**路径**: `BUILD.gn`

**职责**: 定义GN构建目标

**主要Targets**:
- `syscap_tool_bin` - 命令行可执行文件
- `syscap_interface_shared` - 内部API动态库
- `generate_pcid` - PCID生成动作
- `pcid.sc` - PCID预构建产物

### config.gni

**路径**: `config.gni`

**职责**: 定义构建配置参数

**可配置项**:
- `syscap_codec_config_path` - Syscap定义头文件路径
- `syscap_codec_config_extern_path` - 外部Syscap定义路径（扩展用）

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位与核心能力
- [架构说明](02_Architecture.md) - 技术架构详情
- [GN构建目标](05_GN_Targets.md) - 构建系统详细说明
