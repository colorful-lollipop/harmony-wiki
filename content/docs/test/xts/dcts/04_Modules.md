# 模块详解

## ability 模块 - 分布式能力框架测试

### 模块概述

**路径**: `ability/`
**GN Target**: `ability` (group)
**主要测试**: DMS (Distributed Mission Scheduler) 分布式任务调度框架

### 子模块清单

| 子模块 | GN Target | 测试内容 |
|--------|-----------|----------|
| `dmsfwk` | `DctsDmsHapTest` | 主测试 HAP |
| `dmsfwkfatest` | `DctsDmsFaTest` | FA 模式测试 |
| `dmsfwkserver` | `DctsDmsJsServer` | JS 服务端测试 |
| `dmsfwkstageserver` | `DctsDmsFwkStageServer` | Stage 模式服务端 |
| `dmsfwkstagetest` | `DctsDmsFwkStageTest` | Stage 模式测试 |
| `dmsfwkstagetestserver` | `DctsDmsFwkStageTestServer` | Stage 模式测试服务端 |
| `dmsfwkstagepermissiontest` | `DctsDmsFwkStagePermissionTest` | Stage 模式权限测试 |

### 测试的 API 命名空间

```javascript
// 测试用例中引用的主要 API
import featureAbility from '@ohos.ability.featureAbility';
import AbilityDelegatorRegistry from '@ohos.application.abilityDelegatorRegistry';
```

### 测试能力

- **Feature Ability 调用**: 跨设备 FA 启动和交互
- **分布式任务**: 任务迁移和同步
- **权限验证**: 跨设备权限检查

### 目录结构

```
ability/dmsfwk/
├── dmsfwk/
│   ├── entry/src/main/js/       # 主代码
│   ├── entry/src/ohosTest/js/   # 测试用例
│   └── BUILD.gn
├── dmsfwkfatest/                # FA 测试
├── dmsfwkserver/                # 服务端测试
├── dmsfwkstageserver/           # Stage 模式服务端
└── ...其他子模块
```

---

## communication 模块 - 软总线通信测试

### 模块概述

**路径**: `communication/`
**GN Target**: `communication` (group)
**主要测试**: 分布式软总线 (SoftBus) 和 RPC 通信

### 子模块清单

| 子模块 | GN Target | 测试内容 |
|--------|-----------|----------|
| `dsoftbus/rpc` | `DctsRpcJsTest` | RPC JS 测试 |
| `dsoftbus/rpcserver` | `DctsRpcJsServer` | RPC 服务端测试 |
| `dsoftbus_request/rpc` | `DctsRpcRequestJsTest` | RPC 请求测试 |
| `dsoftbus_request/rpcserver` | `DctsRpcRequestJsServer` | RPC 请求服务端 |
| `dsoftbus_rpcets/rpcclient` | `DctsRpcEtsTest` | RPC + ETS 客户端 |
| `dsoftbus_rpcets/rpcserver` | `DctsRpcEtsServer` | RPC + ETS 服务端 |
| `softbus_standard/dsoftbusTest` | `Softbustestserver` | 标准软总线服务端 |
| `softbus_standard/socket_trans` | `DctsSoftBusSoketTransFuncTest` | Socket 传输功能 |
| `softbus_standard/transmission/sendfile` | `DctsSoftBusTransFileFunTest` | 文件传输 |
| `softbus_standard/transmission/sendmsg` | `DctsSoftBusTransFunTest` | 消息传输 |
| `softbus_standard/transmission/sendstream` | `DctsSoftBusTransStreamFunTest` | 流传输 |
| `softbus_standard/transmission/sessionmgt` | `DctsSoftBusTransSessionFunTest` | 会话管理 |
| `softbus_standard/transmission/reliability` | `DctsSoftBusTransReliabilityTest` | 传输可靠性 |

### 测试的 API 命名空间

```javascript
import rpc from '@ohos.rpc';
import deviceManager from '@ohos.distributedDeviceManager';
import featureAbility from '@ohos.ability.featureAbility';
```

### 测试能力

- **RPC 通信**: 远程过程调用
- **Socket 传输**: 底层 Socket 通信
- **会话管理**: 建立/销毁分布式会话
- **文件传输**: 跨设备文件共享
- **流媒体传输**: 实时音视频流
- **可靠性测试**: 传输稳定性验证

---

## distributeddatamgr 模块 - 分布式数据管理测试

### 模块概述

**路径**: `distributeddatamgr/`
**GN Target**: `distributeddatamgr` (group)
**主要测试**: 分布式数据存储和同步

### 子模块清单

| 子模块 | 测试内容 |
|--------|----------|
| `distributed_kv_store` | KV 分布式存储测试 |
| `distributed_rdb_store` | RDB 关系型存储测试 |
| `distributed_kv_store_stage` | Stage 模式 KV 存储测试 |
| `distributed_rdb_stage_store` | Stage 模式 RDB 存储测试 |
| `distributed_data_object_stage` | Stage 模式分布式对象测试 |

### 测试的 API 命名空间

```javascript
import factory from '@ohos.data.distributedData';      // 分布式数据工厂
import data_Rdb from '@ohos.data.relationalStore';     // 关系型存储
import deviceManager from '@ohos.distributedDeviceManager';
import distributedObject from '@ohos.data.distributedDataObject';
```

### 测试能力

- **KV 存储操作**: 键值对读写、监听
- **RDB 存储操作**: 关系型数据库 CRUD
- **数据同步**: 设备间数据同步
- **安全级别**: 数据安全标签管理
- **冲突处理**: 多设备数据冲突解决

---

## distributedhardware 模块 - 分布式硬件设备测试

### 模块概述

**路径**: `distributedhardware/`
**GN Target**: `distributedhardware` (group)
**主要测试**: 分布式硬件设备管理

### 子模块清单

| 子模块 | GN Target | 测试内容 |
|--------|-----------|----------|
| `devicemanagernotest` | `DctsSubDeviceJsTest` | 设备管理器 JS 测试 |
| `devicemanagerteststatic` | `DctsDeviceManagerAPITestStatic` | 静态 API 测试 |
| `devicemanagernoteststatic` | `DctsDeviceManagerTestStatic` | 静态设备管理测试 |
| `distributeddevicejstest` | `DctsSubdisDeviceJsTest` | 分布式设备 JS 测试 |
| `distributeddevicejstestservice` | `DctsSubdisDeviceJsTestserver` | 分布式设备测试服务 |
| `distributedaudiotest` | `DctsSubAudioTest` | 分布式音频测试 |
| `distributedcameratest` | `DctsSubdisCameraTest` | 分布式相机测试 |
| `distributedinputtest` | `DctsSubDistributedInputTest` | 分布式输入测试 |
| `distributedscreentest` | `DctsSubdisScreenTest` | 分布式屏幕测试 |

### 测试的 API 命名空间

```javascript
import deviceManager from '@ohos.distributedDeviceManager';
import hilog from '@ohos.hilog';
```

### 测试能力

- **设备发现**: 发现网络中可用设备
- **设备认证**: 设备间信任关系建立
- **设备属性**: 读取设备信息
- **分布式音频**: 跨设备音频播放控制
- **分布式相机**: 跨设备相机访问
- **分布式输入**: 跨设备输入设备共享
- **分布式屏幕**: 屏幕投射和共享

---

## filemanagement 模块 - 文件管理测试

### 模块概述

**路径**: `filemanagement/`
**GN Target**: `filemanagement` (group)
**主要测试**: 分布式文件管理

### 子模块清单

| 子模块 | GN Target | 测试内容 |
|--------|-----------|----------|
| `fileio/client` | `DctsFileioClientTest` | 客户端文件 IO 测试 |
| `fileio/server` | `DctsFileioServer` | 服务端文件测试 |

### 测试的 API 命名空间

```javascript
import fs from '@ohos.file.fs';
import rpc from '@ohos.rpc';
import securityLabel from '@ohos.file.securityLabel';
import featureAbility from '@ohos.ability.featureAbility';
```

### 测试能力

- **文件读写**: 分布式文件系统操作
- **安全标签**: 文件安全标签设置和读取
- **文件加密**: 分布式文件加密支持

---

## multimedia 模块 - 多媒体测试

### 模块概述

**路径**: `multimedia/`
**GN Target**: `multimedia` (group)
**主要测试**: 音视频会话管理

### 子模块清单

| 子模块 | GN Target | 测试内容 |
|--------|-----------|----------|
| `avsession` | `DctsAVSessionClientTest` | AV 会话客户端测试 |
| `avsessionserver` | `DctsAVSessionServerTest` | AV 会话服务端测试 |

### 测试的 API 命名空间

```javascript
// 测试 AVSessionManager 相关 API
```

### 测试能力

- **会话控制**: 音视频会话创建和控制
- **跨设备播放**: 多设备媒体同步播放
- **会话管理**: 会话生命周期管理

---

## testtools 模块 - 测试工具

### 模块概述

**路径**: `testtools/`
**GN Target**: `testtools` (group)
**职责**: 提供测试辅助工具

### 子模块清单

| 子模块 | 职责 |
|--------|------|
| `disetsTest/` | ETS 分布式测试工具 |
| `disjsTest/` | JS 分布式测试工具 |
| `config/` | 测试配置 |

### 测试辅助组件

```javascript
// 常用测试工具
import ApiMessage from '../common/apiMessage.js';
import ApiResult from '../common/apiResult.js';
import rpc from '@ohos.rpc';
import process from '@ohos.process';
```

---

## common 模块 - 公共模块

### 模块概述

**路径**: `common/`
**职责**: 提供测试用例间的共享内存通信机制

### 文件清单

| 文件 | 描述 |
|------|------|
| `shm_utils.h` | 共享内存工具头文件 |
| `shm_utils.cpp` | 共享内存工具实现 |

### 功能说明

- **跨进程通信**: 测试用例间通过共享内存交换数据
- **同步机制**: 提供进程同步原语

---

## 相关文档

- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构设计](03_Architecture.md)
- [构建系统](05_Build_System.md)
- [测试框架](appendix/Test_Frameworks.md)
