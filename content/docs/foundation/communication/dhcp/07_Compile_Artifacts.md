# 编译产物与运行时

## 文档目的

本文档说明 DHCP 组件的编译产物、安装路径、运行时加载关系和产物验证方法。

---

## 适用范围

- 构建系统: GN + Ninja
- 组件版本: 3.1.0
- 系统类型: small, standard

---

## 编译产物清单

### 共享库（.so）

| 产物名称 | 生成 Target | 安装路径 | 大小估计 | 说明 |
|----------|-------------|----------|----------|------|
| libdhcp_sdk.z.so | dhcp_sdk | /system/lib64/ | ~300KB | 应用接口 SDK |
| libdhcp_client.z.so | dhcp_client | /system/lib64/ | ~500KB | DHCP Client SA |
| libdhcp_server.z.so | dhcp_server | /system/lib64/ | ~400KB | DHCP Server SA |
| libdhcp_utils.z.so | dhcp_utils | /system/lib64/ | ~100KB | 工具库 |
| libdhcp_updater_client.z.so | dhcp_updater_client | /updater/lib/ | ~200KB | 升级专用 Client |

### 静态库（.a）

| 产物名称 | 生成 Target | 大小估计 | 说明 |
|----------|-------------|----------|------|
| libdhcp_client_static.a | dhcp_client_static | ~800KB | Client 静态链接库 |
| libdhcp_server_static.a | dhcp_server_static | ~600KB | Server 静态链接库 |

### 配置文件

| 产物名称 | 生成 Target | 安装路径 | 大小 | 说明 |
|----------|-------------|----------|------|------|
| 1126.json | wifi_standard_sa_profile | /system/profile/ | ~200B | DHCP Client SA 配置 |
| 1127.json | wifi_standard_sa_profile | /system/profile/ | ~200B | DHCP Server SA 配置 |
| dhcpd.conf | - | /etc/dhcp/ | ~1KB | Server 配置示例 |

---

## SA 配置文件详情

### DHCP Client SA (1126.json)

```json
// 证据: services/sa_profile/1126.json
{
  "process": "wifi_manager_service",
  "systemability": [
    {
      "name": 1126,
      "libpath": "libdhcp_client.z.so",
      "run-on-create": false,
      "distributed": false,
      "dump_level": 1,
      "auto-restart": true
    }
  ]
}
```

### DHCP Server SA (1127.json)

```json
// 证据: services/sa_profile/1127.json
{
  "process": "wifi_manager_service",
  "systemability": [
    {
      "name": 1127,
      "libpath": "libdhcp_server.z.so",
      "run-on-create": false,
      "distributed": false,
      "dump_level": 1,
      "auto-restart": true
    }
  ]
}
```

### 配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| process | wifi_manager_service | SA 运行进程 |
| name | 1126 / 1127 | SA ID |
| libpath | libdhcp_client.z.so | SO 文件名 |
| run-on-create | false | 按需启动 |
| auto-restart | true | 异常后自动重启 |
| dump_level | 1 | Dump 信息级别 |

---

## 运行时加载关系

### 启动时序

```
1. 系统启动
   ↓
2. SA Manager 读取 /system/profile/1126.json, 1127.json
   ↓
3. SA Manager 解析 SA 配置
   ↓
4. SA Manager 注册 SA (不立即启动，run-on-create=false)
   ↓
5. 应用调用 DHCP API
   ↓
6. SA Manager 按需启动对应 SA
   ↓
7. 动态链接器加载依赖库
   └─→ libdhcp_client.z.so
       └─→ libdhcp_utils.z.so
           └─→ libhilog.z.so
           └─→ libipc_core.z.so
           └─→ ...
```

### 依赖加载顺序

当 `libdhcp_client.z.so` 被加载时，动态链接器会按以下顺序加载依赖：

1. **直接依赖**:
   - libdhcp_utils.z.so
   - libhilog.z.so
   - libipc_core.z.so
   - libsystem_ability_fwk.z.so
   - libsamgr_proxy.z.so

2. **间接依赖** (通过 libdhcp_utils.z.so):
   - libaccesstoken_sdk.z.so
   - libtokenid_sdk.z.so
   - libappexecfwk_base.z.so

### 运行时进程视图

```
wifi_manager_service 进程
├── libdhcp_client.z.so (SA ID: 1126)
│   ├── DhcpClientServiceImpl
│   ├── State Machine
│   └── 回调线程池
├── libdhcp_server.z.so (SA ID: 1127)
│   ├── DhcpServerServiceImpl
│   ├── Address Pool
│   └── DHCPd 线程
└── libdhcp_utils.z.so (共享)
    ├── Permission Utils
    ├── SA Manager
    ├── ARP Checker
    ├── Thread Pool
    └── Timer
```

---

## 安装路径结构

### 标准系统布局

```
/system/
├── lib64/
│   ├── libdhcp_sdk.z.so          # SDK (符号导出)
│   ├── libdhcp_client.z.so       # Client SA
│   ├── libdhcp_server.z.so       # Server SA
│   └── libdhcp_utils.z.so        # Utils (符号导出)
├── profile/
│   ├── 1126.json                 # Client SA 配置
│   └── 1127.json                 # Server SA 配置
└── etc/
    └── dhcp/
        └── dhcpd.conf            # Server 配置示例

/updater/                          # 升级镜像
└── lib/
    └── libdhcp_updater_client.z.so
```

### Lite 系统布局

```
/lib/
├── libdhcp_client.so
├── libdhcp_server.so
└── libdhcp_utils.so

/etc/
├── 1126.json
└── 1127.json
```

---

## 符号导出控制

### SDK 符号表

```map
# 证据: frameworks/native/libdhcp_sdk.map
{
  global:
    DhcpClient*;
    DhcpServer*;
    *RegisterDhcp*CallBack*;
    *StartDhcp*;
    *StopDhcp*;
    *SetDhcp*;
    *GetDhcp*;
    *DealWifiDhcp*;
    *UpdateLeasesTime*;
    *StopDhcpd*Sa;
  local:
    *;
};
```

### Utils 符号表

```map
# 证据: services/utils/libdhcp_util.map
{
  global:
    *DhcpPermissionUtils*;
    *DhcpSaManager*;
    *DhcpArpChecker*;
    *DhcpThread*;
    *DhcpSystemTimer*;
    *DhcpCommonUtils*;
  local:
    *;
};
```

---

## 产物验证

### 编译验证

```bash
# 检查产物是否生成
ls -lh out/rk3568/system/lib64/libdhcp_*.so

# 检查 SA 配置是否生成
ls -lh out/rk3568/system/profile/1126.json
ls -lh out/rk3568/system/profile/1127.json
```

### 符号验证

```bash
# 查看 SDK 导出符号
readelf -sW out/rk3568/system/lib64/libdhcp_sdk.z.so | grep "FUNC.*GLOBAL"

# 查看依赖库
readelf -d out/rk3568/system/lib64/libdhcp_client.z.so | grep NEEDED
```

### SA 配置验证

```bash
# 验证 SA 配置格式
cat out/rk3568/system/profile/1126.json | python3 -m json.tool
cat out/rk3568/system/profile/1127.json | python3 -m json.tool
```

---

## 运行时验证

### SA 状态检查

```bash
# 查看 SA 状态
hidumper -s 1126 -a -h  # DHCP Client
hidumper -s 1127 -a -h  # DHCP Server

# 查看 SA 列表
hidumper -ls
```

### 日志验证

```bash
# 查看日志
hdc shell hilog | grep DHCP

# 过滤 Client 日志
hdc shell hilog | grep "DhcpClient"

# 过滤 Server 日志
hdc shell hilog | grep "DhcpServer"
```

### 功能验证

```bash
# 启动 DHCP Client（通过系统 API）
# （需要通过 WiFi 服务调用）

# 检查 IP 获取
ifconfig wlan0
```

---

## 常见问题

### 问题1: SA 启动失败

**现象**: SA Manager 日志提示 "load SA failed"

**原因**:
1. SO 文件路径错误
2. SO 文件依赖缺失
3. 权限不足

**定位**:
```bash
# 查看 SA Manager 日志
hdc shell hilog | grep SA

# 检查 SO 文件
hdc shell ls -l /system/lib64/libdhcp_client.z.so

# 检查依赖
hdc shell readelf -d /system/lib64/libdhcp_client.z.so | grep NEEDED
```

### 问题2: 符号未定义

**现象**: 应用链接 SDK 时提示 "undefined reference"

**原因**:
1. 符号表未正确配置
2. SDK 版本不匹配

**定位**:
```bash
# 检查 SDK 符号
readelf -sW libdhcp_sdk.z.so | grep <symbol>
```

### 问题3: 动态链接失败

**现象**: 运行时提示 "dlopen failed"

**原因**:
1. 依赖库缺失
2. 架构不匹配

**定位**:
```bash
# 检查依赖
hdc shell ldd /system/lib64/libdhcp_client.z.so
```

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 架构说明
- [06_Build_System](06_Build_System.md) - 构建配置
- [09_Common_Issues](09_Common_Issues.md) - 常见问题
