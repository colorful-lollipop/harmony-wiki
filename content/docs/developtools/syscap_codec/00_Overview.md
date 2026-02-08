# 00_Overview - 项目概览

## 目的

本文档介绍 `syscap_codec` 项目的定位、核心能力、运行环境和关键概念。

## 适用范围

- 新加入项目的开发人员
- 需要了解系统能力机制的开发者
- IDE/工具链集成开发者

## 项目定位

### 在 OpenHarmony 中的位置

```
OpenHarmony 生态
├── 应用层 (Applications)
├── 框架层 (Framework)
├── 系统服务层 (System Services)
└── 开发工具层 (Developtools) ← syscap_codec 所在层
    ├── syscap_codec          ← 本模块
    ├── hdc
    └── ...
```

**syscap_codec** 属于 `developtools` 子系统，是系统能力（System Capability）的编解码工具。

### 核心职责

1. **编码**: 将 JSON 格式的系统能力描述转换为二进制格式（.sc文件）
2. **解码**: 将二进制格式还原为 JSON 格式
3. **比较**: 验证设备能力是否满足应用需求
4. **转换**: SC格式与字符串格式互转

## 核心能力

### 1. PCID 编解码

**PCID** (Product Compatibility ID) 描述设备提供的系统能力。

- **编码**: `include/syscap_tool.h:31` - `PCIDEncode()`
  - 输入: JSON文件 (包含api_version, system_type, manufacturer_id, syscap列表)
  - 输出: `pcid.sc` 二进制文件
  
- **解码**: `include/syscap_tool.h:33` - `PCIDDecode()`
  - 输入: `pcid.sc` 二进制文件
  - 输出: `pcid.json` JSON文件

### 2. RPCID 编解码

**RPCID** (Required Product Compatibility ID) 描述应用所需的系统能力。

- **编码**: `include/syscap_tool.h:35` - `RPCIDEncode()`
  - 输入: JSON文件 (包含api_version, syscap列表)
  - 输出: `rpcid.sc` 二进制文件
  
- **解码**: `include/syscap_tool.h:37` - `RPCIDDecode()`
  - 输入: `rpcid.sc` 二进制文件
  - 输出: `rpcid.json` JSON文件

### 3. 字符串格式转换

- **RPCID转字符串**: `include/syscap_tool.h:39` - `EncodeRpcidscToString()`
- **PCID转字符串**: `include/create_pcid.h:43` - `EncodePcidscToString()`
- **字符串PCID解码**: `include/create_pcid.h:42` - `DecodeStringPCIDToJson()`

### 4. 兼容性比较

- **文件比较**: `include/syscap_tool.h:41` - `ComparePcidWithRpcidString()`
- **字符串比较**: `interfaces/inner_api/syscap_interface.h:70` - `ComparePcidString()`

比较维度：
- API版本是否满足
- OS系统能力是否包含
- 私有系统能力是否包含

## 运行环境

### 编译环境

| 平台 | 支持状态 | 说明 |
|------|----------|------|
| Windows x86_64 | ✅ | 通过MinGW交叉编译 |
| Linux x86_64 | ✅ | 原生编译 |
| Darwin x86_64 | ✅ | 原生编译（需在macOS上编译）|

### 运行时依赖

**命令行工具** (`syscap_tool`):
- 标准C库
- cJSON库（静态链接）
- bounds_checking_function（安全函数库）

**N-API模块** (`systemcapability.so`):
- OpenHarmony 运行时
- N-API框架
- 依赖 `/system/etc/pcid.sc` 文件

**Taihe/ANI模块** (`systemCapability_taihe_native.z.so`):
- OpenHarmony 运行时
- ANI框架
- 依赖 `/system/etc/pcid.sc` 文件

### 部署位置

```
设备文件系统
├── /system/etc/pcid.sc          ← 设备PCID文件（运行时读取）
├── /system/framework/
│   ├── systemCapability.abc     ← Taihe ABC文件
│   └── ...
└── /system/lib/module/
    └── libsystemcapability.so   ← N-API模块（传统）
    └── systemCapability_taihe_native.z.so ← ANI模块（新）
```

## 关键概念

### System Capability (Syscap)

系统能力是 OpenHarmony 描述设备功能特性的机制。

**命名规范**: `SystemCapability.{Subsystem}.{Module}.{Feature}`

示例：
- `SystemCapability.Multimedia.Media.Core`
- `SystemCapability.Communication.WiFi.Core`
- `SystemCapability.Security.AccessToken`

完整列表见：`include/codec_config/syscap_define.h`

### OS Syscap vs Private Syscap

| 类型 | 定义位置 | 存储方式 | 数量限制 |
|------|----------|----------|----------|
| OS Syscap | `syscap_define.h` | 位图 (120 bytes) | 960个 |
| Private Syscap | 动态定义 | 字符串列表 | 无明确限制 |

**OS Syscap**: 在 `syscap_define.h` 中预定义的系统能力，通过位图索引。

**Private Syscap**: 厂商自定义的系统能力，以字符串形式存储。

### PCID 文件格式

```c
// include/create_pcid.h:23-30
typedef struct ProductCompatibilityID {
    uint16_t apiVersion : 15;      // API版本
    uint16_t apiVersionType : 1;   // 版本类型 (0=PCID)
    uint16_t systemType : 3;       // 系统类型 (mini/small/standard)
    uint16_t reserved : 13;        // 保留位
    uint32_t manufacturerID;       // 厂商ID
    uint8_t osSyscap[120];         // OS系统能力位图
    // 后跟私有系统能力字符串 (逗号分隔)
} PCIDMain;
```

### RPCID 文件格式

```c
// include/context_tool.h:36-39
typedef struct RequiredProductCompatibilityIDHead {
    uint16_t apiVersion : 15;      // API版本
    uint16_t apiVersionType : 1;   // 版本类型 (1=RPCID)
} RPCIDHead;
// 后跟: syscap类型(2字节) + syscap长度(2字节) + syscap字符串数组
```

## 使用场景

### 场景1: IDE开发

1. 开发者创建应用时，IDE收集所需Syscap
2. IDE调用 `syscap_tool -Rei` 生成 `rpcid.sc`
3. 应用打包时包含 `rpcid.sc`

### 场景2: 设备兼容性检查

1. 应用商店获取应用的 `rpcid.sc`
2. 获取设备的 `pcid.sc`
3. 调用 `syscap_tool -C` 比较兼容性
4. 只向兼容设备分发应用

### 场景3: 运行时查询

1. 应用通过 N-API/ANI 接口查询系统能力
2. 框架读取 `/system/etc/pcid.sc`
3. 返回系统能力列表给应用

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 代码组织方式
- [架构说明](02_Architecture.md) - 技术架构详情
- [N-API接口](03_NAPI_Interface.md) - JS接口说明
- [安全风险分析](06_Security_Analysis.md) - 安全注意事项

## 参考文档

- `README_ZH.md` - 项目中文README
- `include/codec_config/syscap_define.h` - 系统能力定义
- `include/syscap_tool.h` - 核心接口头文件
