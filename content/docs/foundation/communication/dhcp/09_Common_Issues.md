# 常见问题与调试指南

## 文档目的

本文档说明 DHCP 组件的常见问题、定位路径和调试方法，基于代码证据和实际使用经验。

---

## 适用范围

- 组件: @ohos/dhcp
- 版本: 3.1.0
- 场景: 构建、运行、调试

---

## 常见问题分类

| 类别 | 问题数量 | 关键词 |
|------|----------|--------|
| 构建问题 | 5 | 编译、链接、依赖 |
| 运行时问题 | 8 | 启动失败、崩溃、功能异常 |
| 功能问题 | 6 | IP 获取失败、Server 启动失败 |
| 性能问题 | 3 | 延迟高、内存泄漏 |
| 安全问题 | 4 | 权限拒绝、敏感信息泄露 |

---

## 构建问题

### 问题1: 编译错误 - 未定义符号

**现象**:
```
undefined reference to 'DhcpClient::RegisterDhcpClientCallBack'
```

**原因**:
- 链接库顺序错误
- SDK 头文件版本不匹配

**定位路径**:
1. 检查 BUILD.gn 中的 `deps` 顺序
2. 确认链接了 `dhcp_sdk`

**解决方案**:
```gn
# 确保先链接 dhcp_sdk
deps = [
  "//foundation/communication/dhcp/frameworks/native:dhcp_sdk",
  # 其他依赖
]
```

**代码证据**: `frameworks/native/BUILD.gn:20`

---

### 问题2: 链接错误 - 找不到库

**现象**:
```
error: cannot find -ldhcp_utils
```

**原因**:
- dhcp_utils 未编译
- 链接路径错误

**定位路径**:
1. 检查 `out/` 目录产物
2. 查看 BUILD.gn 中 `external_deps`

**解决方案**:
```bash
# 重新编译 dhcp_utils
hb build dhcp_utils

# 检查产物
ls -lh out/rk3568/system/lib64/libdhcp_utils.z.so
```

**代码证据**: `services/utils/BUILD.gn:20`

---

### 问题3: 符号可见性错误

**现象**:
```
warning: symbol 'xxx' is not exported
```

**原因**:
- 符号未在 `.map` 文件中声明
- 使用了内部函数

**定位路径**:
1. 检查 `libdhcp_sdk.map`
2. 确认使用的是公开 API

**解决方案**:
```bash
# 查看导出符号
readelf -sW libdhcp_sdk.z.so | grep <symbol>

# 使用公开 API（参考 04_C_API_Reference.md）
```

**代码证据**: `frameworks/native/libdhcp_sdk.map`

---

### 问题4: 编译配置错误

**现象**:
```
error: 'OHOS_ARCH_LITE' is not defined
```

**原因**:
- 编译目标与配置不匹配
- Lite/Standard 版本混用

**定位路径**:
1. 检查 `bundle.json` 中的 `adapted_system_type`
2. 确认编译命令

**解决方案**:
```bash
# 标准系统
hb build dhcp

# Lite 系统
hb build -f foundation/communication/dhcp
```

**代码证据**: `bundle.json:44-47`

---

### 问题5: ASan/Ubsan 报告

**现象**:
```
runtime error: unsigned integer overflow
```

**原因**:
- 整数溢出（已知安全问题）
- 边界检查未充分实现

**定位路径**:
1. 查看堆栈信息
2. 定位到具体代码行

**解决方案**:
- 参考 `08_Security_Review.md` 修复整数溢出问题
- 暂时禁用 ASan 进行调试

**代码证据**: `services/dhcp_client/src/dhcp_options.cpp:335`

---

## 运行时问题

### 问题1: SA 启动失败

**现象**:
```
hidumper: SA 1126 load failed
```

**原因**:
- SO 文件路径错误
- SO 文件依赖缺失
- 权限不足

**定位路径**:
```bash
# 查看 SA Manager 日志
hdc shell hilog | grep SA

# 检查 SO 文件
hdc shell ls -l /system/lib64/libdhcp_client.z.so

# 检查依赖
hdc shell readelf -d /system/lib64/libdhcp_client.z.so | grep NEEDED
```

**解决方案**:
1. 确保 SO 文件正确安装
2. 检查所有依赖库是否存在
3. 验证文件权限（644）

**代码证据**: `services/sa_profile/1126.json`

---

### 问题2: IPC 调用超时

**现象**:
```
IPC timeout: failed to call StartDhcpClient
```

**原因**:
- SA 未启动
- 网络接口不存在
- 权限不足

**定位路径**:
```bash
# 检查 SA 状态
hidumper -s 1126 -a -h

# 检查网络接口
hdc shell ifconfig

# 检查日志
hdc shell hilog | grep "DhcpClient"
```

**解决方案**:
1. 确保 SA 已启动
2. 确认接口名称正确（如 "wlan0"）
3. 检查应用是否持有 `NETWORK_DHCP` 权限

**代码证据**: `dhcp_client_service_impl.cpp:100`

---

### 问题3: 崩溃 - 空指针解引用

**现象**:
```
SIGSEGV: null pointer dereference
```

**原因**:
- 参数未充分校验
- 回调未注册

**定位路径**:
```bash
# 查看堆栈
hdc shell hilog | grep "stack"

# 定位代码位置
```

**解决方案**:
1. 确保所有回调已正确注册
2. 检查参数是否为 nullptr
3. 添加更多日志定位问题

**代码证据**: `services/dhcp_server/src/dhcp_option.cpp:38`

---

### 问题4: 内存泄漏

**现象**:
```
Leak: 1024 bytes allocated at dhcp_options.cpp:298
```

**原因**:
- `GetDhcpOptionString` 分配内存未释放

**定位路径**:
```bash
# 使用 ASan 检测
hb build --gn-args=is_asan=true dhcp

# 查看报告
```

**解决方案**:
1. 确保释放 `GetDhcpOptionString` 返回的内存
2. 使用智能指针（`std::unique_ptr`）管理内存

**代码证据**: `services/dhcp_client/src/dhcp_options.cpp:298`

---

## 功能问题

### 问题1: IP 获取失败

**现象**:
```
OnIpFailChanged: timeout getting IP
```

**原因**:
- DHCP 服务器未响应
- 网络问题
- 接口未 up

**定位路径**:
```bash
# 检查接口状态
hdc shell ifconfig wlan0

# 检查 DHCP 日志
hdc shell hilog | grep "DhcpClient"

# 抓包分析
hdc shell tcpdump -i wlan0 port 67 or port 68
```

**解决方案**:
1. 确保接口已 up
2. 检查网络连接
3. 确认 DHCP 服务器可用
4. 检查防火墙规则

**代码证据**: `dhcp_client_state_machine.cpp:200`

---

### 问题2: Server 启动失败

**现象**:
```
StartDhcpServer failed: address in use
```

**原因**:
- 端口已被占用
- 接口已绑定

**定位路径**:
```bash
# 检查端口占用
hdc shell netstat -tuln | grep :67

# 检查接口
hdc shell ifconfig
```

**解决方案**:
1. 停止其他 DHCP 服务
2. 使用不同接口
3. 检查配置是否重复

**代码证据**: `dhcp_s_server.cpp:300`

---

### 问题3: 租约表为空

**现象**:
```
GetDhcpClientInfos: 0 clients found
```

**原因**:
- 地址池未配置
- 无客户端连接

**定位路径**:
```bash
# 检查地址池
hdc shell hidumper -s 1127 -a -h

# 查看 Server 日志
hdc shell hilog | grep "DhcpServer"
```

**解决方案**:
1. 调用 `SetDhcpRange` 配置地址池
2. 检查网络连接
3. 确认客户端正在发送 DHCP 请求

**代码证据**: `dhcp_address_pool.cpp:100`

---

### 问题4: 权限拒绝

**现象**:
```
StartDhcpClient failed: permission denied (DHCP_PERMISSION_DENIED)
```

**原因**:
- 应用非原生进程
- 未持有 `NETWORK_DHCP` 权限

**定位路径**:
```bash
# 检查应用权限
hdc shell bm dump <package_name> | grep permission

# 查看权限日志
hdc shell hilog | grep "Permission"
```

**解决方案**:
1. 确保应用是系统应用
2. 在 `config.json` 中声明权限
3. 使用原生进程调用

**代码证据**: `dhcp_permission_utils.cpp:50`

---

## 性能问题

### 问题1: IP 获取延迟高

**现象**:
```
IP获取时间 > 10 秒
```

**原因**:
- 网络延迟
- DHCP 服务器响应慢
- 重试次数过多

**定位路径**:
```bash
# 测量时间戳
hdc shell hilog -T | grep "DhcpClient"

# 抓包分析 RTT
hdc shell tcpdump -i wlan0 port 67 or port 68
```

**解决方案**:
1. 优化网络连接
2. 减少 DHCP 重试次数
3. 启用 IP 缓存（WiFi 场景）

**代码证据**: `dhcp_socket.cpp:200`

---

### 问题2: 内存占用高

**现象**:
```
内存占用 > 10MB
```

**原因**:
- 内存泄漏
- 租约表过大
- 日志缓冲区溢出

**定位路径**:
```bash
# 查看内存
hdc shell ps -A | grep wifi_manager_service

# 使用 Valgrind 检测
valgrind --leak-check=full ...
```

**解决方案**:
1. 修复内存泄漏（参考问题4）
2. 清理过期租约
3. 控制日志级别

**代码证据**: `dhcp_binding.cpp:164`

---

## 调试工具

### 日志系统

```bash
# 查看 DHCP 日志
hdc shell hilog | grep DHCP

# 查看 Client 日志
hdc shell hilog | grep "DhcpClient"

# 查看 Server 日志
hdc shell hilog | grep "DhcpServer"

# 查看详细日志（包含时间戳）
hdc shell hilog -T | grep DHCP
```

### SA Dump

```bash
# 查看 Client SA 信息
hidumper -s 1126 -a -h

# 查看 Server SA 信息
hidumper -s 1127 -a -h

# 查看所有 SA
hidumper -ls
```

### 网络抓包

```bash
# 抓取 DHCPv4 包
hdc shell tcpdump -i wlan0 port 67 or port 68

# 抓取 DHCPv6 包
hdc shell tcpdump -i wlan0 port 546 or port 547

# 保存到文件
hdc shell tcpdump -i wlan0 port 67 -w dhcp.pcap
```

### 符号查看

```bash
# 查看导出符号
readelf -sW libdhcp_sdk.z.so | grep FUNC

# 查看依赖库
readelf -d libdhcp_client.z.so | grep NEEDED

# 查看段信息
readelf -l libdhcp_client.z.so
```

---

## 调试流程

### 问题定位流程

```
1. 收集信息
   ├── 日志 (hilog)
   ├── SA 状态 (hidumper)
   ├── 网络状态 (ifconfig, netstat)
   └── 抓包 (tcpdump)

2. 分析问题
   ├── 确定问题类型（构建/运行/功能）
   ├── 定位代码位置
   └── 查找相似问题

3. 解决问题
   ├── 查阅本文档
   ├── 查看代码注释
   └── 搜索 Issue

4. 验证修复
   ├── 重新编译
   ├── 部署测试
   └── 回归测试
```

---

## 相关链接

- [00_Overview](00_Overview.md) - 项目概览
- [03_Architecture](03_Architecture.md) - 架构说明
- [04_C_API_Reference](04_C_API_Reference.md) - API 文档
- [06_Build_System](06_Build_System.md) - 构建问题
- [08_Security_Review](08_Security_Review.md) - 安全问题
