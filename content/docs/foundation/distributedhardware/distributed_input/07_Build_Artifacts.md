# 编译产物 (Build Artifacts)

## 目的

本文档描述 distributed_input 模块的编译产物，包括库文件、安装路径、运行时加载关系和依赖说明。

## 适用范围

本文档适用于以下场景：
- 理解模块的产物输出和安装位置
- 进行部署和集成测试
- 诊断运行时加载问题
- 进行产物大小和依赖分析

## 关键结论

1. **14 个共享库**: 生成 14 个 .so 共享库，安装在 system/lib/
2. **2 个 SA 库**: libdinput_source.z.so 和 libdinput_sink.z.so，由 SA 进程加载
3. **1 个配置文件**: dinput.cfg，安装到 /system/etc/init/
4. **依赖外部库**: libevdev、access_token、dsoftbus 等系统库

## 编译产物清单

### 共享库产物

#### 公共库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_sdk.so` | Inner SDK 公共接口，供多模输入模块调用 | `/system/lib/` | 无 |
| `libdinput_source.so` | Source Manager 实现，Source SA 进程主库 | `/system/lib/` | 无 |
| `libdinput_sink.so` | Sink Manager 实现，Sink SA 进程主库 | `/system/lib/` | 无 |

#### 服务支持库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_source_trans.so` | Source Transport 实现 | `/system/lib/` | 无 |
| `libdinput_inject.so` | 事件注入实现 | `/system/lib/` | libevdev |
| `libdinput_sink_trans.so` | Sink Transport 实现 | `/system/lib/` | 无 |
| `libdinput_collector.so` | 事件采集实现 | `/system/lib/` | libevdev |
| `libdinput_trans_base.so` | Transport 基类实现 | `/system/lib/` | 无 |

#### 状态和管理库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_sink_state.so` | Sink 状态管理实现 | `/system/lib/` | 无 |

#### 框架集成库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_source_handler.so` | Source Handler 实现 | `/system/lib/` | 无 |
| `libdinput_sink_handler.so` | Sink Handler 实现 | `/system/lib/` | 无 |

#### 设备处理库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_handler.so` | 设备能力查询实现 | `/system/lib/` | libevdev, dsoftbus |

#### DFX 工具库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_dfx_utils.so` | HiDumper 和 HiSysEvent 工具实现 | `/system/lib/` | 无 |

#### 基础工具库

| 库文件 | 说明 | 安装路径 | 运行时依赖 |
|---------|------|----------|---------|
| `libdinput_utils.so` | 基础工具函数实现 | `/system/lib/` | 无 |

---

### System Ability 库产物

| 库文件 | 说明 | SA ID | 加载方式 |
|---------|------|--------|---------|
| `libdinput_source.z.so` | Source SA 主库，由 System Ability Manager 按需加载 | 4809 | On-demand |
| `libdinput_sink.z.so` | Sink SA 主库，由 System Ability Manager 按需加载 | 4810 | On-demand |

**证据**: [sa_profile/4809.json:6](../sa_profile/4809.json:6), [sa_profile/4810.json:6](../sa_profile/4810.json:6)

---

### 配置文件产物

| 文件 | 说明 | 安装路径 | 证据 |
|------|------|----------|---------|
| `dinput.cfg` | 服务初始化配置 | `/system/etc/init/dinput.cfg` | [sa_profile/BUILD.gn:29-34](../sa_profile/BUILD.gn:29-34) |

**dinput.cfg 内容**:

```ini
{
    "services" : [{
        "name" : "4809-4810",
        "path" : ["system/lib/libdinput_source.z.so", "system/lib/libdinput_sink.z.so"]
    }],
    "permission": [
        "ohos.permission.DISTRIBUTED_DATASYNC",
        "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE"
    ]
}
```

---

## 运行时加载关系

### 进程架构

```
┌──────────────────────────────────────────────────────────┐
│           dinput 进程                                  │
│                                                        │
│           ┌────────────────────────────┐              │
│           │  Source SA (4809)         │              │
│           │  + Sink SA (4810)          │              │
│           └────────────────────────────┘              │
│                        │                               │
│                        │                               │
│                  ┌─────▼────────────────────┐               │
│                  │ libdinput_source.z.so   │               │
│                  │ libdinput_sink.z.so     │               │
│                  └──────────────────────────┘               │
│                                                        │
└────────────────────────────────────────────────────────────┘
```

**加载机制**:
- Source SA (4809) 和 Sink SA (4810) 都在同一个 `dinput` 进程中
- SA 通过 `run-on-create: false` 按需加载（on-demand）
- SA 由 System Ability Manager（samgr）管理和加载

### 库加载顺序

#### Source SA 启动加载顺序

```
[系统启动]
      │
      ▼
[samgr 读取 dinput.cfg]
      │
      ├──────────────────┐
      ▼                 │
[发现服务 4809-4810]  │
      │                 │
      ├──────────────────┘  │
      ▼                 │
[按需加载 libdinput_source.z.so]  │
      │                 │
      ├──────────────────┐  │
      ▼                 │
[加载依赖库]            │
      │                 │
      ├──────────────────┘  │
      ▼                 │
[DistributedInputSourceManager.OnStart()]
      │
      ├──────────────────┐
      ▼                 │
[初始化组件]
      │
      └──────────────────┘
[Source SA 就绪]
```

**依赖加载**（根据 [services/source/sourcemanager/BUILD.gn:64-95](../services/source/sourcemanager/BUILD.gn:64-95)）:

1. `libdinput_dfx_utils.so` - DFX 工具
2. `libdinput_trans_base.so` - Transport 基类
3. `libdinput_sdk.so` - Inner SDK（如果内部使用）
4. `libdinput_inject.so` - 事件注入
5. `libdinput_source_trans.so` - Source Transport

#### Sink SA 启动加载顺序

```
[系统启动]
      │
      ▼
[samgr 读取 dinput.cfg]
      │
      ├──────────────────┐
      ▼                 │
[发现服务 4809-4810]  │
      │                 │
      ├──────────────────┘  │
      ▼                 │
[按需加载 libdinput_sink.z.so]  │
      │                 │
      ├──────────────────┐  │
      ▼                 │
[加载依赖库]            │
      │                 │
      ├──────────────────┘  │
      ▼                 │
[DistributedInputSinkManager.OnStart()]
      │
      ├──────────────────┐
      ▼                 │
[初始化组件]
      │
      └──────────────────┘
[Sink SA 就绪]
```

**依赖加载**（根据 [services/sink/sinkmanager/BUILD.gn:64-95](../services/sink/sinkmanager/BUILD.gn:64-95)）：

1. `libdinput_dfx_utils.so` - DFX 工具
2. `libdinput_sink_state.so` - 状态管理
3. `libdinput_trans_base.so` - Transport 基类
4. `libdinput_sdk.so` - Inner SDK（如果内部使用）
5. `libdinput_collector.so` - 事件采集
6. `libdinput_sink_trans.so` - Sink Transport

---

## 运行时依赖分析

### 外部库依赖

| 库 | 依赖的外部库 | 依赖用途 |
|------|---------------|----------|
| libdinput_sdk.so | access_token:libaccesstoken_sdk, libtokenid_sdk<br/>c_utils:utils<br/>dsoftbus:softbus_client<br/>eventhandler:libeventhandler<br/>hilog:libhilog<br/>ipc:ipc_core<br/>libevdev:libevdev<br/>safwk:system_ability_fwk, samgr:samgr_proxy<br/>distributed_hardware_fwk:distributed_av_receiver, distributed_av_sender, distributedhardwareutils, libdhfwk_sdk<br/>json:nlohmann_json_static | 权限验证、事件处理、IPC、输入驱动、SA 框架、传输 |
| libdinput_source.so | access_token:libaccesstoken_sdk, libtokenid_sdk<br/>c_utils:utils<br/>dsoftbus:softbus_client<br/>eventhandler:libeventhandler<br/>hicollie:libhicollie<br/>hilog:libhilog<br/>hisysevent:libhisysevent<br/>hitrace:hitrace_meter<br/>ipc:ipc_core<br/>json:nlohmann_json_static<br/>libevdev:libevdev<br/>safwk:system_ability_fwk, samgr:samgr_proxy<br/>distributed_hardware_fwk:distributed_av_receiver, distributed_av_sender, distributedhardwareutils, libdhfwk_sdk | Source 端所有依赖 |
| libdinput_sink.so | access_token:libaccesstoken_sdk, libtokenid_sdk<br/>c_utils:utils<br/>dsoftbus:softbus_client<br/>eventhandler:libeventhandler<br/>hilog:libhilog<br/>ipc:ipc_core<br/>json:nlohmann_json_static<br/>libevdev:libevdev<br/>safwk:system_ability_fwk, samgr:samgr_proxy<br/>distributed_hardware_fwk:distributed_av_receiver, distributed_av_sender, distributedhardwareutils, libdhfwk_sdk<br/>graphic_2d:librender_service_base, surface<br/>window_manager:libdm | Sink 端所有依赖（包含图形相关库） |
| libdinput_inject.so | libevdev:libevdev<br/>openssl:libcrypto_shared<br/>libdinput_sink_state | 虚拟驱动写入、加密、状态管理 |
| libdinput_collector.so | libevdev:libevdev<br/>openssl:libcrypto_shared | 本地驱动读取、加密 |
| libdinput_trans_base.so | device_manager<br/>dsoftbus:softbus_client<br/>os_account<br/>c_utils:utils | 设备管理、软总线、账号、C 工具 |
| libdinput_sink_state.so | libevdev:libevdev<br/>dsoftbus:softbus_client<br/>libdinput_collector<br/>libdinput_sink_trans<br/>libdinput_dfx_utils<br/>libdinput_utils | 输入驱动、软总线、其他内部库 |
| libdinput_source_handler.so | c_utils:utils<br/>dsoftbus:softbus_client<br/>eventhandler:libeventhandler<br/>hilog:libhilog<br/>ipc:ipc_core<br/>libevdev:libevdev<br/>safwk:system_ability_fwk, samgr:samgr_proxy<br/>distributed_hardware_fwk:distributedhardwareutils, libdhfwk_sdk | Source Handler 所有依赖 |
| libdinput_sink_handler.so | c_utils:utils<br/>dsoftbus:softbus_client<br/>eventhandler:libeventhandler<br/>hilog:libhilog<br/>ipc:ipc_core<br/>libevdev:libevdev<br/>safwk:system_ability_fwk, samgr:samgr_proxy<br/>distributed_hardware_fwk:distributedhardwareutils, libdhfwk_sdk | Sink Handler 所有依赖 |
| libdinput_handler.so | libevdev:libevdev<br/>dsoftbus:softbus_client | 设备能力查询所有依赖 |
| libdinput_dfx_utils.so | c_utils:utils<br/>hisysevent:libhisysevent | DFX 工具所有依赖 |
| libdinput_utils.so | c_utils:utils<br/>dsoftbus:softbus_client<br/>distributed_hardware_fwk:distributedhardwareutils<br/>libevdev:libevdev<br/>openssl:libcrypto_shared | 基础工具所有依赖 |

---

## 安装路径汇总

### system/lib/

| 库文件 | 大小（预估） | 功能 |
|---------|------------|------|
| libdinput_sdk.so | ~200KB | 公共 API，供多模输入调用 |
| libdinput_source.so | ~300KB | Source SA 主库 |
| libdinput_sink.so | ~300KB | Sink SA 主库 |
| libdinput_source_trans.so | ~150KB | Source Transport 实现 |
| libdinput_inject.so | ~150KB | 事件注入实现 |
| libdinput_sink_trans.so | ~150KB | Sink Transport 实现 |
| libdinput_collector.so | ~150KB | 事件采集实现 |
| libdinput_trans_base.so | ~150KB | Transport 基类 |
| libdinput_sink_state.so | ~100KB | 状态管理 |
| libdinput_source_handler.so | ~150KB | Source Handler |
| libdinput_sink_handler.so | ~150KB | Sink Handler |
| libdinput_handler.so | ~100KB | 设备能力查询 |
| libdinput_dfx_utils.so | ~100KB | DFX 工具 |
| libdinput_utils.so | ~100KB | 基础工具 |

**总计**: 约 1.75MB

### system/etc/init/

| 文件 | 大小 | 功能 |
|------|------|------|
| dinput.cfg | ~1KB | 服务配置，定义 SA 名称、库路径和权限 |

---

## 运行时行为

### SA 生命周期

#### Source SA (4809)

| 生命周期阶段 | 方法 | 说明 |
|------------|------|------|
| **初始化** | `OnStart()` | 注册 SA 到 samgr，初始化组件 |
| **就绪** | `Publish(this)` | SA 可被外部发现和使用 |
| **运行** | 处理外部 IPC 调用 | 处理 Prepare/Start/Stop 等操作 |
| **停止** | `OnStop()` | 注销 SA，释放资源 |

**实现位置**: [distributed_input_source_manager.cpp:599](../services/source/sourcemanager/src/distributed_input_source_manager.cpp:599)

#### Sink SA (4810)

| 生命周期阶段 | 方法 | 说明 |
|------------|------|------|
| **初始化** | `OnStart()` | 注册 SA 到 samgr，初始化组件 |
| **就绪** | `Publish(this)` | SA 可被外部发现和使用 |
| **运行** | 处理外部 IPC 调用 | 响应 Source 的请求，管理状态 |
| **停止** | `OnStop()` | 注销 SA，释放资源 |

**实现位置**: [distributed_input_sink_manager.cpp:108](../services/sink/sinkmanager/src/distributed_input_sink_manager.cpp:108)

---

### 动态库加载

### 按需加载（On-Demand）

**SA 加载模式**: `run-on-create: false`

**含义**:
- SA 不会在系统启动时自动加载
- SA 在第一次被客户端调用时按需加载
- 不使用的 SA 可以节省系统资源

**证据**: [4809.json:7](../sa_profile/4809.json:7), [4810.json:7](../sa_profile/4810.json:7)

---

## 依赖解析

### libdinput_sdk.so 依赖图

```
libdinput_sdk.so
      │
      ├── libdinput_utils.so (工具类)
      │
      ├── IPC 连接
      │
      ├─────▼
      │   distributed_input_source_proxy (调用 Source SA)
      │
      └───▼
          distributed_input_sink_proxy (调用 Sink SA)
      │
      ├───┐ (依赖 Source/Sink Stub 实现)
      │
      ▼
      分布式硬件框架提供的 libdhfwk_sdk.so
      │
      ├───┐
      ▼
      samgr:samgr_proxy (SA 管理器)
      │
      └───▼
          IPC:ipc_core (IPC 通信)
```

### libevdev 使用场景

| 库 | 使用方 | 用途 |
|------|--------|------|
| libdinput_inject.so | 写入模式 | 创建虚拟输入设备节点，写入事件数据 |
| libdinput_collector.so | 读取模式 | 读取本地输入设备节点，读取原始事件 |
| libdinput_handler.so | 查询模式 | 查询输入设备信息和能力 |

**设备节点路径**:
- 虚拟设备节点：`/dev/input/eventX`（动态创建）
- 本地设备节点：`/dev/input/eventX`（实际硬件）

---

## 运行时诊断

### HiDumper 使用

**库**: `libdinput_dfx_utils.so`

**Dump 命令**（根据 [hidumper.h:53-78](../dfx_utils/include/hidumper.h:53-78)）：

| 命令 | 说明 |
|------|------|
| `-h` | 显示帮助信息 |
| `-a` | 显示所有节点信息 |
| `-s <sessionId>` | 显示指定会话信息 |
| `-d <dhId>` | 显示指定设备信息 |

**使用示例**:

```bash
# 显示所有信息
hdc shell dump -a 4809
hdc shell dump -a 4810

# 显示指定会话
hdc shell dump -s 4809 12345

# 显示指定设备
hdc shell dump -d 4809 network_id_of_device
```

### 日志查询

**日志域**: `LOG_DOMAIN=0xD004120`

**日志标签**:
| 模块 | 标签 | 位置 |
|------|------|------|
| Inner SDK | `distributedinputclient` | [interfaces/inner_kits/BUILD.gn:94-95](../interfaces/inner_kits/BUILD.gn:94-95) |
| Source Manager | `distributedinputmanagerkit` | [services/source/sourcemanager/BUILD.gn:91-92](../services/source/sourcemanager/BUILD.gn:91-92) |
| Sink Manager | `distributedinputmanagerkit` | [services/sink/sinkmanager/BUILD.gn:54-55](../services/sink/sinkmanager/BUILD.gn:54-55) |

**日志查询示例**:

```bash
# 查询 Source Manager 日志
hilog -T distributedinputmanagerkit | grep "DistributedInput"

# 查询 Sink Manager 日志
hilog -T distributedinputmanagerkit | grep "DistributedInputSink"

# 查询 SDK 日志
hilog -T distributedinputclient | grep "DistributedInputClient"
```

---

## 构建产物验证

### 验证清单

| 验证项 | 说明 | 方法 |
|---------|------|------|
| 库文件完整性 | 检查所有 14 个 .so 文件是否生成 | `ls out/{product}/system/lib/libdinput_*.so` |
| 库文件符号 | 检查导出符号是否正确 | `readelf -sW out/{product}/system/lib/libdinput_*.so` |
| SA Profile 正确性 | 检查 SA ID 和库路径是否匹配 | `cat out/{product}/system/profile/distributed_input/*.xml` |
| 配置文件安装 | 检查 dinput.cfg 是否安装到正确路径 | `cat out/{product}/system/etc/init/dinput.cfg` |
| 权限配置 | 检查 dinput.cfg 中权限是否正确 | `cat out/{product}/system/etc/init/dinput.cfg` |

---

## 相关跳转

- [GN 目标](06_GN_Targets.md) - 详细的构建目标和依赖关系
- [目录结构](02_Directory_Structure.md) - BUILD.gn 文件组织
- [公共 API](04_Public_API.md) - Inner SDK 使用方式

---

*更新时间: 2026-02-06 15:08:55*
