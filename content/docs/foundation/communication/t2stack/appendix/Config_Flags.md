# T2Stack 关键配置开关

> 本文档记录 t2stack 的关键编译时配置开关（Feature Flags）和运行时配置参数。

## 目录

- [1. 编译时 Feature Flags](#1-编译时-feature-flags)
- [2. Fillp 运行时配置](#2-fillp-运行时配置)
- [3. DFile 运行时配置](#3-dfile-运行时配置)
- [4. NStackX 运行时配置](#4-nstackx-运行时配置)

---

## 1. 编译时 Feature Flags

### 1.1 全局 Feature 配置

**文件**: `t2stack.gni`

```gn
declare_args() {
  # 协议特性开关
  t2stack_feature_vtp = true           # VTP/Fillp 协议
  t2stack_feature_dfile = true          # DFile 文件传输
  t2stack_feature_coap = true           # CoAP 协议
  
  # 内部特性开关
  t2stack_feature_inner_coap = true     # 内部 CoAP 实现
  t2stack_feature_deps_wifi = true      # WiFi 依赖
}
```

### 1.2 Fillp 编译标志

**文件**: `fillp/BUILD.gn`

| 标志 | 值 | 说明 |
|------|-----|------|
| `PDT_MIRACAST` | - | 支持 Miracast PDT |
| `FILLP_SERVER_SUPPORT` | - | 支持服务端模式 |
| `FILLP_LITTLE_ENDIAN` | - | 小端序编译 |
| `FILLP_LINUX` | - | Linux 平台 |
| `FILLP_POWER_SAVE` | - | 电源管理支持 |
| `FILLP_POWER_SAVING_LINUX` | - | Linux 电源管理 |
| `FILLP_ENABLE_DFX_HIDUMPER` | - | DFX hidump 支持 |
| `FILLP_MGT_MSG_LOG` | - | 管理消息日志 |
| `FILLP_MMSG_SUPPORT` | - | 支持 sendmmsg/recvmmsg |

**代码证据**：`fillp/BUILD.gn:25-34`

### 1.3 DFile 编译标志

**文件**: `nstackx_core/dfile/BUILD.gn`

| 标志 | 值 | 说明 |
|------|-----|------|
| `NSTACKX_WITH_LINUX` | - | Linux 平台 |
| `DFILE_ENABLE_HIDUMP` | - | 启用 hidump |
| `ENABLE_USER_LOG` | - | 用户日志支持 |
| `SSL_AND_CRYPTO_INCLUDED` | - | SSL 加密已包含 |
| `NSTACKX_WITH_LITEOS` | - | LiteOS 平台 |

### 1.4 NStackX 编译标志

**文件**: `nstackx_ctrl/BUILD.gn`

| 标志 | 值 | 说明 |
|------|-----|------|
| `ENABLE_USER_LOG` | - | 用户日志支持 |
| `NSTACKX_EXTEND_BUSINESSDATA` | - | 扩展业务数据 |
| `DFINDER_SAVE_DEVICE_LIST` | - | 保存设备列表 |
| `DFINDER_SUPPORT_SET_SCREEN_STATUS` | - | 屏幕状态支持 |
| `DFINDER_DISTINGUISH_ACTIVE_PASSIVE_DISCOVERY` | - | 主动/被动发现区分 |

---

## 2. Fillp 运行时配置

### 2.1 配置项枚举

**文件**: `fillp/include/fillpinc.h` (FillpConfigItemList)

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `FT_CONF_INIT_APP` | - | - | 应用初始化 |
| `FT_CONF_INIT_STACK` | - | - | 栈初始化 |
| `FT_CONF_TX_BURST` | uint16 | - | TX burst |
| `FT_CONF_SEND_CACHE` | uint32 | 8192 | 发送缓存 |
| `FT_CONF_RECV_CACHE` | uint32 | 8192 | 接收缓存 |
| `FT_CONF_MAX_RATE` | uint32 | 20 * 1000 | 最大速率 (20Mbps) |
| `FT_CONF_INITIAL_RATE` | uint32 | 2 * 1000 | 初始速率 (2Mbps) |
| `FT_CONF_SLOW_START` | bool | false | 慢启动 |
| `FT_CONF_FEC_REDUNDANCY_LEVEL` | enum | - | FEC 冗余级别 |
| `FT_CONF_KEEP_ALIVE_TIME` | uint32 | 10000 | 保活时间 (10s) |
| `FT_CONF_CONNECT_TIMEOUT` | uint32 | 10000 | 连接超时 (10s) |
| `FT_CONF_USE_FEC` | bool | false | 启用 FEC |

### 2.2 FEC 冗余级别

```c
typedef enum FillpFecRedundancyLevelStrcut {
    FILLP_FEC_REDUNDANCY_LEVEL_INVLAID = 0,
    FILLP_FEC_REDUNDANCY_LEVEL_LOW,      // 低冗余
    FILLP_FEC_REDUNDANCY_LEVEL_MID,       // 中冗余
    FILLP_FEC_REDUNDANCY_LEVEL_HIGH,      // 高冗余
    FILLP_FEC_REDUNDANCY_LEVEL_REAL,      // 实时冗余
    FILLP_FEC_REDUNDANCY_LEVEL_AUTO,      // 自动
} FillpFecRedundancyLevel;
```

**代码证据**：`fillp/include/fillpinc.h:647-655`

### 2.3 配置示例

```c
// 1. 基本配置
uint32_t maxRate = 50 * 1000 * 1000;  // 50 Mbps
FtConfigSet(FT_CONF_MAX_RATE, &maxRate);

// 2. 启用 FEC
FillpFecRedundancyLevel fecLevel = FILLP_FEC_REDUNDANCY_LEVEL_MID;
FtConfigSet(FT_CONF_FEC_REDUNDANCY_LEVEL, &fecLevel);

// 3. 慢启动配置
FILLP_BOOL slowStart = FILLP_TRUE;
FtConfigSet(FT_CONF_SLOW_START, &slowStart);
```

---

## 3. DFile 运行时配置

### 3.1 能力配置

**文件**: `nstackx_core/dfile/interface/nstackx_dfile.h`

| 能力位 | 值 | 说明 |
|--------|-----|------|
| `CAPS_UDP_GSO` | 0 | UDP GSO 支持 |
| `CAPS_LINK_SEQUENCE` | 1 | 链路顺序 |
| `CAPS_WLAN_CATAGORY` | 2 | WLAN 类别 |
| `CAPS_NO_RTT` | 3 | 无 RTT |
| `CAPS_ALG_NORATE` | 5 | NoRate 算法 |
| `CAPS_RESUMABLE_TRANS` | 6 | 断点续传 |
| `CAPS_ZEROCOPY` | 7 | 零拷贝 |
| `CAPS_MULTIPATH_MTP` | 8 | 多路径 MTP |

### 3.2 能力配置示例

```c
// 获取当前能力
uint32_t caps = NSTACKX_DFileGetCapabilities();

// 启用 UDP GSO
if (caps & NSTACKX_CAPS_UDP_GSO) {
    NSTACKX_DFileSetCapabilities(NSTACKX_CAPS_UDP_GSO, 1);
}

// 启用多路径
if (caps & NSTACKX_CAPS_MULTIPATH) {
    NSTACKX_DFileSetCapabilities(NSTACKX_CAPS_MULTIPATH, 1);
}
```

**代码证据**：`nstackx_core/dfile/interface/nstackx_dfile.h:60-112`

---

## 4. NStackX 运行时配置

### 4.1 发现配置

**文件**: `nstackx_ctrl/interface/nstackx.h`

```c
/**
 * @brief 设备发现配置
 */
typedef struct {
    uint16_t advertiseCount;      // 广播次数
    uint16_t advertiseDuration;   // 广播持续时间
    uint16_t responseDelay;       // 响应延迟
    // ... 其他配置
} NSTACKX_DiscoverySettings;
```

### 4.2 配置示例

```c
// 1. 启动发现
NSTACKX_DiscoverySettings settings = {
    .advertiseCount = 12,
    .advertiseDuration = 5000,  // 5秒
};
NSTACKX_StartDeviceDiscovery(&settings);

// 2. 设置设备列表老化时间
NSTACKX_SetDeviceListAgingTime(5);  // 5秒

// 3. 设置最大设备数
NSTACKX_SetMaxDeviceNum(50);
```

**代码证据**：`nstackx_ctrl/interface/nstackx.h:127-133`

---

## 配置优先级

| 优先级 | 配置来源 | 说明 |
|--------|----------|------|
| 1 | 运行时 API | 最高优先级，可覆盖编译时配置 |
| 2 | Feature Flags | 编译时开关，运行时不可修改 |
| 3 | 默认值 | 未配置时使用 |

---

## 相关文档

- [构建系统](../05_Build_System.md) - GN 构建配置
- [API 参考](../03_CAPI_Reference.md) - 配置 API
- [故障排查](../08_Troubleshooting.md) - 配置相关问题

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
*代码证据来源：头文件配置枚举和 BUILD.gn 分析*
