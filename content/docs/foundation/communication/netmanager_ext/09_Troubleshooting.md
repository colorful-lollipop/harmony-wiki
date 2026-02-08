# 常见问题与定位

## 目的

本文档提供 Net Manager Ext 的常见构建、运行、调试问题及定位路径，帮助开发者快速解决问题。

---

## 构建相关问题

### 问题 1：构建失败 - 找不到 netmanager_base

**现象**：
```
ninja: error: unknown dependency: //foundation/communication/netmanager_base:net_conn_manager_if
```

**原因**：
- `netmanager_base` 仓库未检出或配置错误

**定位步骤**：
```bash
# 1. 检查 netmanager_base 是否存在
ls -l ../../../foundation/communication/netmanager_base/

# 2. 检查 .gn 文件中的依赖
grep -r "netmanager_base" BUILD.gn

# 3. 检查构建配置
cat out/default/args.gn | grep netmanager
```

**解决方案**：
1. 检出 `netmanager_base` 仓库到同级目录
2. 确保 `ohos.build` 配置正确
3. 重新构建：`./build.sh --product-name <product>`

**代码证据**：BUILD.gn:18-23 - 依赖 netmanager_base

---

### 问题 2：N-API 编译错误

**现象**：
```
error: 'napi_value' was not declared in this scope
```

**原因**：
- N-API 头文件未正确包含
- 编译器版本不支持 N-API 接口

**定位步骤**：
```bash
# 1. 检查头文件包含
grep "napi/native_api.h" frameworks/js/napi/ethernet/ethernet_module.cpp

# 2. 检查编译器版本
gcc --version

# 3. 查看完整错误信息
grep -A 5 "napi_value" build.log
```

**解决方案**：
1. 检查头文件路径：`//foundation/arkui/napi/native_api.h`
2. 确保 `bundle.json` 中依赖 `napi` 组件正确
3. 检查 OHOS SDK 版本兼容性

**代码证据**：ethernet_module.cpp:16 - 包含 `napi/native_api.h`

---

### 问题 3：IPC Stub/Proxy 编译错误

**现象**：
```
error: undefined reference to 'HDF_INVOKE'
```

**原因**：
- HDI/IPC 框架头文件缺失
- 依赖的 HDI 组件未包含

**定位步骤**：
```bash
# 1. 检查 HDI 头文件
grep -r "hdf_base" frameworks/native/ethernetclient/

# 2. 检查 IPC 依赖
grep "ipc" bundle.json
```

**解决方案**：
1. 确保 `bundle.json` 中 `ipc` 和 `safwk` 组件存在
2. 检查 HDI 生成器是否正确运行
3. 清理构建缓存后重新编译

---

## 运行时问题

### 问题 4：SA 启动失败

**现象**：
```
SystemAbility start failed: 8300 (NetFirewall)
```

**日志**：
```bash
hdc shell hilog -T | grep NetFirewall
```

**常见原因**：
1. **依赖库缺失**：`libnetfirewall_manager.z.so` 依赖未满足
2. **权限不足**：进程无法访问所需资源
3. **配置文件错误**：`sa_profile/8300.json` 格式错误

**定位步骤**：
```bash
# 1. 查看 SA 日志
hdc shell hilog -x | grep -i "netfirewall"

# 2. 检查进程状态
ps -A | grep netmanager

# 3. 查看 dmesg
hdc shell dmesg | grep -i "net"
```

**解决方案**：
```bash
# 1. 检查库依赖
readelf -d /system/lib64/libnetfirewall_manager.z.so | grep NEEDED

# 2. 手动启动 SA（测试）
hdc shell "sa_start 8300 /system/lib64/libnetfirewall_manager.z.so"

# 3. 检查 SELinux
hdc shell "getenforce"
hdc shell "cat /proc/<pid>/attr/current"
```

**代码证据**：sa_profile/8300.json - SA 配置

---

### 问题 5：N-API 加载失败

**现象**：
```
Error: module 'net.ethernet' not found
```

**日志**：
```bash
hdc shell hilog -T | grep napi
```

**常见原因**：
1. **产物未安装**：`libnet_ethernet.z.so` 不在系统路径
2. **N-API 版本不匹配**：ABI 不兼容
3. **模块注册失败**：构造函数异常

**定位步骤**：
```bash
# 1. 检查产物是否存在
ls -l /system/lib64/libnet_*.z.so

# 2. 检查应用日志
hdc shell "hilog -t | grep <pid>"

# 3. 查看 N-API 加载日志
hdc shell "cat /data/log/ace/*.log | grep napi"
```

**解决方案**：
```javascript
// 添加错误处理
import ethernet from '@ohos.net.ethernet';

try {
    await ethernet.getIfaceConfig('eth0');
} catch (error) {
    console.error('N-API error:', error.code, error.message);
}
```

---

### 问题 6：网络配置不生效

**现象**：
- 调用 `setIfaceConfig()` 后，接口配置未改变
- `getIfaceConfig()` 返回旧配置

**常见原因**：
1. **权限不足**：`CONNECTIVITY_INTERNAL` 未授予
2. **参数错误**：配置格式不正确
3. **DHCP 冲突**：尝试配置 DHCP 但 DHCP 未停止

**定位步骤**：
```bash
# 1. 检查权限
hdc shell "bm dump -a | grep <package_name>"

# 2. 检查网络状态
hdc shell "ifconfig eth0"

# 3. 查看 SA 日志
hdc shell hilog -x | grep SetIfaceConfig
```

**解决方案**：
1. 验证权限：`ohos.permission.CONNECTIVITY_INTERNAL`
2. 检查参数格式：IP、网关、掩码
3. 确认接口状态：`isIfaceActive()`

**代码证据**：ethernet_service.cpp:226 - 权限检查

---

### 问题 7：网络共享失败

**现象**：
- `startSharing()` 返回错误
- 共享状态保持未开启

**常见原因**：
1. **上游网络未连接**：WiFi/以太网未连接
2. **硬件不支持**：设备不支持 USB 共享
3. **权限不足**：`CONNECTIVITY_INTERNAL` 未授予

**定位步骤**：
```bash
# 1. 检查上游网络
hdc shell "ifconfig wlan0"

# 2. 检查共享状态
hdc shell "cat /data/service/el2/share_state"

# 3. 查看 Sharing SA 日志
hdc shell hilog -x | grep NetworkShare
```

**解决方案**：
1. 确认上游网络已连接
2. 检查 `isSharingSupported()` 返回值
3. 验证设备支持

---

## 调试技巧

### 日志收集

#### 启用详细日志

```bash
# 修改编译标志
export NETMGR_EXT_DEBUG=true

# 重新编译
./build.sh --product-name <product>

# 运行时启用
hdc shell "param set debug.netmanager_ext 1"
hdc shell "param set persist.debug.netmanager_ext 1"
```

**代码证据**：netmanager_ext_config.gni:60 - `enable_netmgr_ext_debug = true`

#### 查看 N-API 日志

```bash
# 查看 JavaScript 应用日志
hdc shell "hilog -T | grep JsApp"

# 查看 N-API 层日志
hdc shell "hilog -T | grep NAPI"
```

#### 查看 SA 日志

```bash
# 查看特定 SA 日志
hdc shell "hilog -x | grep -E 'Ethernet|Sharing|MDNS|VPN|Firewall'"

# 实时跟踪
hdc shell hilog -T | grep --color=always "netmanager"
```

### IPC 调试

#### 查看 IPC 通信

```bash
# 查看 Binder 统计
hdc shell "cat /sys/kernel/debug/binder/stats/state"

# 查看 IPC 事务
hdc shell "cat /sys/kernel/debug/binder/transaction_log"
```

### 网络调试

#### 查看网络接口

```bash
# 查看所有接口
hdc shell "ip link show"

# 查看接口配置
hdc shell "ip addr show eth0"

# 查看路由
hdc shell "ip route show"
```

#### 查看流量统计

```bash
# 查看共享流量
hdc shell "cat /proc/net/dev"

# 查看 iptables 规则
hdc shell "iptables -L -n -v"
```

### 性能分析

#### CPU 使用率

```bash
# 查看进程 CPU 使用
hdc shell "top -n 1 | grep netmanager"

# 查看 SA 进程 CPU
hdc shell "ps -A | grep -E 'mdns|netmanager'"
```

#### 内存使用

```bash
# 查看进程内存
hdc shell "cat /proc/<pid>/status | grep -E 'VmRSS|VmSize'"
```

---

## 常见错误码

### 以太网模块错误

| 错误码 | 说明 | 解决方案 |
|-------|------|---------|
| ERR_PERMISSION_DENIED | 权限不足 | 检查 `CONNECTIVITY_INTERNAL` 权限 |
| ERR_INVALID_PARAM | 参数无效 | 验证接口名、配置格式 |
| ERR_IFACE_NOT_EXIST | 接口不存在 | 检查接口名是否正确 |
| ERR_OPERATION_FAILED | 操作失败 | 检查网络状态、系统日志 |

### 网络共享模块错误

| 错误码 | 说明 | 解决方案 |
|-------|------|---------|
| ERR_SHARE_NOT_SUPPORTED | 不支持共享 | 检查设备硬件支持 |
| ERR_NO_UPSTREAM | 无上游网络 | 连接 WiFi/以太网 |
| ERR_SHARE_ALREADY_ACTIVE | 共享已开启 | 先停止再启动 |

### VPN 模块错误

| 错误码 | 说明 | 解决方案 |
|-------|------|---------|
| ERR_VPN_NOT_PREPARED | VPN 未准备 | 调用 `Prepare()` |
| ERR_VPN_CONNECTION_FAILED | 连接失败 | 检查 VPN 服务器、网络 |
| ERR_INVALID_VPN_CONFIG | 配置无效 | 验证 VPN 参数 |

---

## 开发环境配置

### 本地构建

```bash
# 1. 设置环境
source ./build/envsetup.sh

# 2. 选择产品
hb set -p <product_name>

# 3. 构建特定模块
hb build -b netmanager_ext
```

### 交叉编译

```bash
# 为不同架构编译
./build.sh --product-name <product> --ccache
./build.sh --product-name <product> --build-type release
```

### 单元测试

```bash
# 运行以太网测试
./test/ethernetmanager/test_ethernet

# 运行共享测试
./test/networksharemanager/test_sharing

# 查看覆盖率
gcovr -r .
```

---

## 工具推荐

### 日志分析

- `hilog` - 系统日志查看
- `grep` / `rg` - 日志搜索
- `awk` / `sed` - 日志过滤

### 性能分析

- `top` - CPU 使用率
- `vmstat` - 内存统计
- `iostat` - I/O 统计

### 网络调试

- `tcpdump` - 抓包分析
- `netstat` - 网络连接
- `ss` - socket 统计

---

## 相关跳转

- [项目定位](01_Project_Positioning.md) - 特性开关
- [GN Build 文档](06_GN_Build.md) - 构建配置
- [安全风险评审](08_Security_Review.md) - 安全问题定位
- [编译产物](07_Build_Artifacts.md) - 产物验证
