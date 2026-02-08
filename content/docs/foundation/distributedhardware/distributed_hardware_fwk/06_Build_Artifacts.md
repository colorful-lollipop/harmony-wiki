# 编译产物

本文档描述分布式硬件管理框架的编译产物、输出路径和运行时加载关系。

> **适用范围**: 需要了解产物安装位置、依赖关系或进行部署的开发者

---

## 产物清单

### 核心库 (.so)

| 产物名称 | 源 Target | 安装路径 | 说明 |
|----------|-----------|----------|------|
| `libdistributedhardwarefwksvr.so` | `distributedhardwarefwksvr` | `/system/lib/distributedhardware/` | 核心 SA 服务 |
| `libdistributedhardwareutils.so` | `distributedhardwareutils` | `/system/lib/distributedhardware/` | 工具库 |
| `libdhfwk_sdk.so` | `libdhfwk_sdk` | `/system/lib/distributedhardware/` | Inner Kit SDK |
| `libhardwaremanager.so` | `hardwaremanager` | `/system/lib/module/distributedhardware/` | N-API 绑定 |
| `libdistributed_av_sender.so` | `distributed_av_sender` | `/system/lib/distributedhardware/` | AV 发送引擎 |
| `libdistributed_av_receiver.so` | `distributed_av_receiver` | `/system/lib/distributedhardware/` | AV 接收引擎 |
| `libdistributed_av_pipeline_fwk.so` | `distributed_av_pipeline_fwk` | `/system/lib/distributedhardware/` | AV 传输框架 |
| `libhardware_taihe.so` | `hardware_taihe` | `/system/lib/distributedhardware/` | Taihe 绑定 |

---

### 配置文件

| 产物名称 | 源 Target | 安装路径 | 说明 |
|----------|-----------|----------|------|
| `dhardware.cfg` | `dhardware.cfg` | `/system/etc/init/` | SA 初始化配置 |
| `dhardware.json` | SA profile | `/system/profile/` | SA 描述文件 |
| `4801.json` | `dhfwk_sa_profile` | `/system/profile/` | SA 能力描述 |

**证据**: `sa_profile/BUILD.gn`
```gn
ohos_prebuilt_etc("dhardware.cfg") {
  source = "dhardware.cfg"
  relative_install_dir = "init"
}

ohos_sa_profile("dhfwk_sa_profile") {
  source = "4801.json"
}
```

---

### 应用程序 (.hap)

| 产物名称 | 源 Target | 安装路径 | 说明 |
|----------|-----------|----------|------|
| `DHardware_UI.hap` | `DHardware_UI` | `/system/app/` | 系统应用 |

**证据**: `bundle.json:69`
```json
"//foundation/.../application:DHardware_UI"
```

---

### 字节码 (.abc)

| 产物名称 | 源 Target | 安装路径 | 说明 |
|----------|-----------|----------|------|
| `hardware_taihe_abc.abc` | `hardware_taihe_abc` | `/system/framework/` | Taihe 静态字节码 |

**证据**: `taihe/BUILD.gn`
```gn
generate_static_abc("hardware_taihe_abc") {
  ark_options = "--safe-point=1000"
  output = "$root_out_dir/system/framework/hardware_taihe_abc.abc"
}
```

---

## 产物依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                         DHardware_UI.hap                         │
│                     (JS/ETS 字节码 + 资源)                       │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              │ 依赖 N-API
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  libhardwaremanager.so                           │
│                      (N-API 绑定层)                              │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              │ 依赖 Inner Kit
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    libdhfwk_sdk.so                               │
│                    (C++ SDK 接口)                                │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              │ IPC 调用
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              libdistributedhardwarefwksvr.so                      │
│                   (核心 SA 服务)                                 │
└─────────────────────────────┬───────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│libdistributed_  │ │libdistributed_  │ │libdistributed_  │
│hardwareutils.so │ │av_sender.so    │ │av_receiver.so  │
│  (工具库)        │ │  (AV 发送)      │ │  (AV 接收)      │
└─────────────────┘ └─────────────────┘ └─────────────────┘
         │                    │
         │                    ▼
         │           ┌─────────────────┐
         │           │libdistributed_  │
         │           │av_pipeline_    │
         │           │fwk.so          │
         │           │  (AV 框架)      │
         │           └─────────────────┘
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                       系统库                                      │
│  softbus, device_manager, kv_store, ipc, safwk, hilog...       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 运行时加载关系

### SA 服务加载

```
系统启动
    │
    ▼
┌─────────────────────────┐
│  samgr (服务管理)       │
└───────────┬─────────────┘
            │
            │ 加载 SA
            ▼
┌─────────────────────────────────────────┐
│  /system/bin/sa_main                    │
│       │                                 │
│       ▼                                 │
│  /system/profile/dhardware.json         │
│       │                                 │
│       ▼                                 │
│  /system/lib/distributedhardware/       │
│  libdistributedhardwarefwksvr.so        │
│       │                                 │
│       ▼                                 │
│  DistributedHardwareService (SA 4801)   │
└─────────────────────────────────────────┘
```

**证据**: `sa_profile/dhardware.cfg`
```json
{
  "services": [{
    "name": "dhardware",
    "path": ["/system/bin/sa_main", "/system/profile/dhardware.json"]
  }]
}
```

---

### N-API 加载

```
应用加载 (@ohos/distributedHardware.hardwareManager)
    │
    ▼
┌─────────────────────────────────────────┐
│  ACE 框架加载 N-API 模块                 │
│       │                                 │
│       ▼                                 │
│  /system/lib/module/distributedhardware/│
│  libhardwaremanager.so                  │
└─────────────────────────────────────────┘
```

---

### 库加载顺序

1. `libc.so` - C 标准库
2. `libace_napi.so` - N-API 框架
3. `libhilog.so` - 日志库
4. `libipc.so` - IPC 库
5. `libsafwk.so` - SA 框架
6. `libdistributedhardwarefwksvr.so` - 核心服务
7. `libdhfwk_sdk.so` - SDK
8. `libhardwaremanager.so` - N-API

---

## 产物验证

### 检查核心服务

```bash
# 检查库文件是否存在
ls -la /system/lib/distributedhardware/libdistributedhardwarefwksvr.so

# 检查 SA 配置
ls -la /system/profile/dhardware.json

# 检查 SA 配置
cat /system/etc/init/dhardware.cfg
```

---

## 后续文档

- 安全评审 → [07_Security_Review.md](07_Security_Review.md)
- GN 构建配置 → [05_GN_Build.md](05_GN_Build.md)
- 架构说明 → [02_Architecture.md](02_Architecture.md)
