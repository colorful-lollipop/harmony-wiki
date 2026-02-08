# Cellular Data 模块概览

## 项目定位

Cellular Data 是 OpenHarmony Telephony 子系统的核心组件，提供以下能力：

- **蜂窝数据激活与管理**：开关控制、状态监控
- **数据漫游管理**：漫游开关、漫游状态查询
- **APN 配置管理**：APN 查询、设置、属性获取
- **数据流类型检测**：上下行流量状态
- **网络能力查询**：Internet 能力、供应商注册状态

## 核心依赖

```
cellular_data
├── telephony_core_service  # 核心服务依赖
├── ril_adapter             # RIL 适配层
├── safwk                   # System Ability Framework
├── samgr                   # Service Manager
└── netmanager_base         # 网络管理基础
```

## 目录结构

```
cellular_data/
├── frameworks/              # 框架层
│   ├── native/             # Native 框架（C++）
│   │   ├── cellular_data_client.cpp    # IPC 客户端
│   │   └── BUILD.gn                     # Native 构建配置
│   ├── js/napi/            # JS N-API 绑定
│   │   ├── include/        # N-API 头文件
│   │   └── src/            # N-API 实现
│   ├── ets/ani/            # ETS/ANI 绑定（ArkUI）
│   └── cj/                 # C++ FFI 绑定
├── interfaces/             # 接口层
│   ├── kits/js/            # JS API 声明（.d.ts）
│   └── innerkits/          # 内部 C++ 接口
├── services/               # 服务层
│   ├── include/            # 服务头文件
│   │   ├── cellular_data_service.h      # 主服务
│   │   ├── state_machine/              # 状态机
│   │   ├── apn_manager/                 # APN 管理
│   │   └── utils/                       # 工具类
│   └── src/               # 服务实现
│       ├── cellular_data_service.cpp    # 服务主逻辑
│       ├── state_machine/              # 状态机实现
│       ├── apn_manager/                 # APN 管理实现
│       └── utils/                       # 工具实现
├── sa_profile/            # SA 配置文件
├── BUILD.gn               # 根构建配置
├── bundle.json            # Bundle 配置
└── README.md              # 项目说明
```

## API 快速参考

### 数据开关状态

```typescript
import data from "@ohos.telephony.data";

// 检查蜂窝数据是否开启
data.isCellularDataEnabled().then((enabled) => {
    console.log(`Cellular data enabled: ${enabled}`);
});

// 同步获取（需要权限）
let enabled = data.isCellularDataEnabledSync();

// 获取数据连接状态
data.getCellularDataState().then((state) => {
    // DataConnectState: UNKNOWN(-1), DISCONNECTED(0), CONNECTING(1), CONNECTED(2), SUSPENDED(3)
});
```

### 漫游管理

```typescript
// 检查漫游是否开启（需要 slotId 参数）
data.isCellularDataRoamingEnabled(0).then((enabled) => {
    console.log(`Roaming enabled: ${enabled}`);
});

// 开启漫游
data.enableCellularDataRoaming(0);
```

### 默认卡管理

```typescript
// 获取默认数据卡
let slotId = data.getDefaultCellularDataSlotIdSync();

// 设置默认数据卡（需要权限）
data.setDefaultCellularDataSlotId(1);
```

## 权限要求

| 权限 | 用途 | APL 要求 |
|------|------|----------|
| `ohos.permission.GET_NETWORK_INFO` | 查询网络状态 | normal |
| `ohos.permission.SET_TELEPHONY_STATE` | 修改 telephony 状态 | system_basic |

## 下一步

- [架构设计](02_Architecture.md) - 深入了解系统架构
- [N-API 参考](04_NAPI_Reference.md) - 完整 API 清单
- [构建配置](06_Build.md) - 编译与构建说明
