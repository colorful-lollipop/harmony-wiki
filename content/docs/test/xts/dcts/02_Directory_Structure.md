# 目录结构与模块职责

## 顶层目录结构

```
/Volumes/lexar/code/d/work/oh/test/xts/dcts/
├── ability/              # 分布式能力框架测试 (DMS)
├── common/               # 共享内存工具模块
├── communication/        # 软总线通信测试
├── distributeddatamgr/   # 分布式数据管理测试
├── distributedhardware/  # 分布式硬件设备测试
├── filemanagement/        # 文件管理测试
├── multimedia/           # 多媒体会话测试
├── testtools/            # 测试工具
├── BUILD.gn              # 主构建配置
├── bundle.json           # 组件配置
├── test_packages.gni     # 测试包配置
├── README.md             # 项目说明文档
└── LICENSE               # Apache 2.0 许可证
```

## 模块职责说明

### 1. ability - 分布式能力框架测试

**路径**: `ability/`
**GN Target**: `ability` (group) - `ability/BUILD.gn:15-30`

**子模块清单与证据**:

| 子模块 | GN Target | 代码位置 | 测试内容 |
|--------|-----------|----------|----------|
| `dmsfwk` | `DctsDmsHapTest` | `ability/dmsfwk/dmsfwk/entry/src/ohosTest/` | 主测试 HAP |
| `dmsfwkfatest` | `DctsDmsFaTest` | `ability/dmsfwk/dmsfwkfatest/entry/src/ohosTest/` | FA 模式测试 |
| `dmsfwkserver` | `DctsDmsJsServer` | `ability/dmsfwk/dmsfwkserver/entry/src/main/` | JS 服务端测试 |
| `dmsfwkstageserver` | `DctsDmsFwkStageServer` | `ability/dmsfwk/dmsfwkstageserver/entry/src/main/` | Stage 模式服务端 |
| `dmsfwkstagetest` | `DctsDmsFwkStageTest` | `ability/dmsfwk/dmsfwkstagetest/entry/src/ohosTest/` | Stage 模式测试 |
| `dmsfwkstagetestserver` | `DctsDmsFwkStageTestServer` | `ability/dmsfwk/dmsfwkstagetestserver/entry/src/main/` | Stage 测试服务端 |
| `dmsfwkstagepermissiontest` | `DctsDmsFwkStagePermissionTest` | `ability/dmsfwk/dmsfwkstagepermissiontest/entry/src/ohosTest/` | 权限测试 |

**测试的 API 命名空间**:

```javascript
// 证据：ability/dmsfwk/dmsfwkstagetest/entry/src/ohosTest/ets/test/DmsFwkStageTest.ets:69-86
import deviceManager from '@ohos.distributedDeviceManager';
import featureAbility from '@ohos.ability.featureAbility';
```

**测试能力**:
- **Feature Ability 调用**: 跨设备 FA 启动和交互
- **分布式任务**: 任务迁移和同步
- **权限验证**: 跨设备权限检查


### 2. common - 公共模块

**路径**: `common/`
**文件**:
- `shm_utils.h` - 共享内存工具头文件
- `shm_utils.cpp` - 共享内存工具实现

**职责**: 提供测试用例间的共享内存通信机制

### 3. communication - 通信测试

**路径**: `communication/`
**子模块**:

| 子模块 | 职责 |
|--------|------|
| `dsoftbus/rpc/` | RPC 通信测试 |
| `dsoftbus/rpcserver/` | RPC 服务端测试 |
| `dsoftbus_request/rpc/` | RPC 请求测试 |
| `dsoftbus_request/rpcserver/` | RPC 请求服务端测试 |
| `dsoftbus_rpcets/rpcclient/` | RPC + ETS 客户端测试 |
| `dsoftbus_rpcets/rpcserver/` | RPC + ETS 服务端测试 |
| `softbus_standard/` | 标准软总线测试 |
| ├── `dsoftbusTest/` | 服务端测试 |
| ├── `socket_trans/` | Socket 传输测试 |
| ├── `transmission/` | 传输可靠性测试 |

**测试能力**:
- 分布式软总线连接
- RPC 远程调用
- 会话管理
- 文件传输
- 流媒体传输

### 4. distributeddatamgr - 分布式数据管理测试

**路径**: `distributeddatamgr/`
**子模块**:
- `jstest/` - JS 测试主目录
  - `distributed_kv_store/` - KV 存储测试
  - `distributed_rdb_store/` - RDB 关系型存储测试
  - `distributed_kv_store_stage/` - Stage 模式 KV 存储测试
  - `distributed_rdb_stage_store/` - Stage 模式 RDB 存储测试
  - `distributed_data_object_stage/` - Stage 模式分布式对象测试

**测试能力**:
- 分布式键值存储
- 分布式关系型数据库
- 数据同步
- 安全级别管理

### 5. distributedhardware - 分布式硬件设备测试

**路径**: `distributedhardware/`
**子模块**:

| 子模块 | 职责 |
|--------|------|
| `devicemanagernotest/` | 设备管理器 JS 测试 |
| `devicemanagerteststatic/` | 静态测试 |
| `devicemanagernoteststatic/` | Node.js 静态测试 |
| `distributeddevicejstest/` | 分布式设备 JS 测试 |
| `distributeddevicejstestservice/` | 分布式设备测试服务 |
| `distributedaudiotest/` | 分布式音频测试 |
| `distributedcameratest/` | 分布式相机测试 |
| `distributedinputtest/` | 分布式输入测试 |
| `distributedscreentest/` | 分布式屏幕测试 |

**测试能力**:
- 设备发现与认证
- 设备属性管理
- 跨设备音频播放
- 跨设备相机访问
- 分布式输入控制
- 屏幕投射

### 6. filemanagement - 文件管理测试

**路径**: `filemanagement/`
**子模块**:
- `fileio/` - 文件 IO 测试
  - `server/` - 服务端测试
  - `client/` - 客户端测试

**测试能力**:
- 分布式文件系统访问
- 文件安全标签
- 文件加密

### 7. multimedia - 多媒体测试

**路径**: `multimedia/`
**子模块**:
- `avsession/` - AV 会话测试
- `avsessionserver/` - AV 会话服务端测试

**测试能力**:
- 音视频会话控制
- 跨设备媒体播放
- 会话管理

### 8. testtools - 测试工具

**路径**: `testtools/`
**子模块**:
- `disetsTest/` - ETS 分布式测试工具
- `disjsTest/` - JS 分布式测试工具
- `config/` - 测试配置

**职责**: 提供测试辅助工具和通用测试组件

## 典型测试模块结构

```
module_name/
├── entry/              # 测试入口模块
│   ├── src/
│   │   ├── main/       # 主代码
│   │   │   ├── js/     # JS 代码
│   │   │   └── ets/    # ETS (方舟) 代码
│   │   └── ohosTest/   # 测试代码
│   │       ├── js/     # JS 测试
│   │       └── ets/    # ETS 测试
│   └── resources/      # 资源文件
├── BUILD.gn           # 构建配置
└── hvigor*            # HVigor 构建配置
```

## 模块依赖关系与代码证据

```
testtools (工具)
      ↓
common (共享内存)
      ↓
ability → distributeddatamgr → filemanagement
      ↓           ↓              ↓
communication ← distributedhardware
      ↓
multimedia
```

**证据**:
- `ability/BUILD.gn`: 依赖 `featureAbility` (分布式任务调度)
- `communication/BUILD.gn`: 依赖 `dsoftbus` (软总线)
- `distributeddatamgr/BUILD.gn`: 依赖 `distributedData` (分布式数据)
- `distributedhardware/BUILD.gn`: 依赖 `distributedDeviceManager` (设备管理)

**关键依赖代码示例**:

```javascript
// 证据：distributedhardware/distributeddevicejstest/entry/src/ohosTest/js/test/distributedDevice.test.js:67
// ability 模块依赖 communication 模块
dmInstance = deviceManager.createDeviceManager('com.ohos.distributedscreenjstest');
deviceList = dmInstance.getAvailableDeviceListSync();

// 证据：filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:199-239
// filemanagement 模块依赖 communication 模块
await fs.connectDfs(networkId, listeners);
```

## 代码导航图

### 功能 → 文件路径映射

| 功能 | 文件路径 | 测试用例示例 | API 模块 |
|------|----------|--------------|----------|
| **分布式任务调度** | `ability/dmsfwk/dmsfwkstagetest/entry/src/ohosTest/ets/test/DmsFwkStageTest.ets` | `SUB_DMS_StandardOs_Collaboration_Bycall_*` | `@ohos.ability.featureAbility` |
| **RPC 通信** | `communication/dsoftbus_rpcets/rpcclient/entry/src/ohosTest/ets/test/RpcRequestEtsUnit.test.ets` | `SUB_DSoftbus_RPC_API_NEW_MessageSequence_*` | `@ohos.rpc` |
| **设备发现** | `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets` | `SUB_DH_DeviceManager_Dcts_*` | `@ohos.distributedDeviceManager` |
| **文件 IO** | `filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js` | `test_filefs_*` | `@ohos.file.fs` |
| **KV 存储** | `distributeddatamgr/jstest/distributed_kv_store/client/hap/entry/src/ohosTest/js/test/KvStoreSecurityLevelS1Jsunit.test.js` | `SUB_DistributedData_KVStoreSecurityLevelS1_*` | `@ohos.data.distributedKVStore` |
| **分布式数据对象** | `distributeddatamgr/jstest/distributed_data_object_stage/server/entry/src/main/ets/serviceability/disetsTest/server/ReflectCallApi.js` | `SUB_DistributedData_DistributedObject_*` | `@ohos.data.distributedDataObject` |

**关键代码入口**:

```javascript
// RPC 通信测试入口
// 证据：communication/dsoftbus_rpcets/rpcclient/entry/src/ohosTest/ets/test/RpcRequestEtsUnit.test.ets:16-20
import { describe, beforeAll, beforeEach, afterEach, afterAll, it, expect, TestType, Size, Level } from '@ohos/hypium'
import rpc from '@ohos.rpc';

export default function rpcRequestTest() {
    describe('RPC Request', function() {
        // 包含 100+ 个 RPC 测试用例
    })
}

// 设备管理测试入口
// 证据：distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:48-55
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import { Permissions, PermissionRequestResult } from 'permissions';
import { Context } from '@ohos.app.ability.common';
import distributedDeviceManager from '@ohos.distributedDeviceManager';
import hilog from '@ohos.hilog';
```
