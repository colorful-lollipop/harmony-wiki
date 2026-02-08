# OpenHarmony Print Scan Framework - 常见问题与解决方案

**目的**: 提供构建、运行和调试过程中常见问题的诊断和解决路径

**适用范围**: OpenHarmony Print Scan Framework 3.1

---

## 目录

- [构建问题](#构建问题)
- [运行时问题](#运行时问题)
- [调试技巧](#调试技巧)
- [权限相关](#权限相关)
- [设备发现问题](#设备发现问题)

---

## 构建问题

### 问题 1: 编译错误 - 未找到头文件

**现象**:
```
error: 'napi_inner_print.h' file not found
error: 'native_engine/native_engine.h' file not found
```

**原因**: LSP（Language Server Protocol）无法找到某些头文件

**解决方案**:
1. 检查头文件路径是否正确
2. 确认依赖模块已正确编译
3. 如果是 OpenHarmony 特有头文件，确保在正确的环境中编译

---

## 运行时问题

### 问题 2: 打印服务无法启动

**现象**: 打印服务无法启动，应用调用打印 API 失败

**可能原因**:
1. PrintServiceAbility 未成功注册为 SA
2. 权限配置不正确
3. 依赖服务（CUPS, Bundle）未就绪

**定位路径**:
```bash
# 检查服务注册状态
hilog -T PrintServiceAbility | grep "RegisterSystemAbility"

# 检查服务启动状态
hilog -T PrintServiceAbility | grep "OnStart\|OnStop"

# 检查 IPC 服务可用性
hilog -T PrintServiceAbility | grep "GetSystemAbility"
```

**解决方案**:
1. 确认 `printservice.rc` 配置正确
2. 检查 `print_sa_profiles.xml` SA Profile 配置
3. 验证依赖服务（CUPS, Bundle）是否正常运行

---

### 问题 3: 扫描服务无法启动

**现象**: 扫描服务无法启动，`scan.init()` 返回失败

**可能原因**:
1. ScanServiceAbility 未成功注册为 SA
2. SANE 服务未启动
3. 权限不足

**定位路径**:
```bash
# 检查扫描服务注册状态
hilog -T ScanServiceAbility | grep "RegisterSystemAbility"

# 检查 SANE 服务状态
hilog -T SaneServerManager | grep "OnStart"

# 检查权限错误
hilog -T ScanServiceAbility | grep "E_SCAN_NO_PERMISSION"
```

**解决方案**:
1. 确认 `scanservice.rc` 配置正确
2. 检查应用是否已申请 `ohos.permission.PRINT` 权限
3. 验证 `scan_sa_profiles.xml` SA Profile 配置

---

### 问题 4: 打印任务卡住

**现象**: 打印任务提交后一直处于 `PRINT_JOB_QUEUED` 或 `PRINT_JOB_RUNNING` 状态，无法完成

**可能原因**:
1. CUPS 打印服务器未响应
2. 打印机连接断开
3. 打印任务配置错误

**定位路径**:
```bash
# 检查打印任务状态
hilog -T PrintServiceAbility | grep "PrintJob.*state"

# 检查 CUPS 连接状态
hilog -T PrintCUPSClient | grep -i "cups.*connect\|cups.*error"

# 检查打印队列状态
lpstat -a
```

**解决方案**:
1. 检查 CUPS 服务是否运行：`systemctl status cups` 或 `ps aux | grep cupsd`
2. 重启 CUPS 服务：`systemctl restart cups`
3. 检查打印机网络连接
4. 取消卡住的打印任务，重新提交

---

### 问题 5: 扫描器无法发现

**现象**: USB 或网络扫描器无法被发现

**可能原因**:
1. USB 权限不足（`MANAGE_USB_CONFIG`）
2. 网络扫描器未启动（mDNS）
3. 设备驱动未加载
4. SANE 服务不可用

**定位路径**:
```bash
# 检查 USB 设备发现
hilog -T ScanUSBManager | grep "DiscoverUSB"

# 检查 mDNS 服务
hilog -T ScanServiceAbility | grep "mDNS\|mdns"

# 检查 SANE 服务连接
hilog -T SaneManagerClient | grep "SANE"
```

**解决方案**:
1. 确认应用已申请 `ohos.permission.MANAGE_USB_CONFIG` 权限
2. 检查 USB 设备是否正确连接
3. 重启扫描服务
4. 手动调用 `stopScannerDiscovery()` 后重新开始

---

### 问题 6: 内存占用过高

**现象**: 打印/扫描服务占用内存持续增长，最终被系统杀死

**可能原因**:
1. 大量打印任务或扫描任务未正确释放
2. 图片数据缓存未清理
3. IPC 回调对象泄漏

**定位路径**:
```bash
# 检查进程内存使用
pidof print_service
cat /proc/<pid>/status | grep -i "vmrss"

# 检查内存增长趋势
hilog -T PrintServiceAbility | grep "memory\|leak"

# 启用详细内存统计
export OHOS_HIVIEW_MALLOC_TRACE=1
export OHOS_HIVIEW_ENABLE_LEAK_CHECK=1
```

**解决方案**:
1. 检查并修复内存泄漏（智能指针、RAII）
2. 确保异步回调完成后释放资源
3. 限制同时进行的任务数量
4. 实现资源清理机制

---

## 权限相关

### 问题 7: 权限不足错误

**现象**: 应用调用 API 时返回 `E_PRINT_NO_PERMISSION` (201) 或 `E_SCAN_NO_PERMISSION` (201)

**错误位置**:
- 打印：`interfaces/kits/napi/print_napi/src/print_module.cpp`
- 扫描：`interfaces/kits/napi/scan_napi/src/scan_module.cpp`
- 服务：`services/print_service/include/print_service_ability.h`
- 服务：`services/scan_service/include/scan_service_ability.h`

**解决方案**:
1. 在应用 `module.json5` 中添加权限声明：
```json
{
  "requestPermissions": [
    {
      "name": "ohos.permission.PRINT",
      "reason": "$string:print_permission_reason"
    }
  ]
}
```

2. 管理员授予权限：通过系统设置 → 权限管理 → 应用 → 权限列表
3. 确认应用是否为系统应用（某些权限仅系统应用可用）

---

### 问题 8: 权限检查不一致

**现象**: 某些 N-API 方法检查权限，某些不检查，导致权限绕过风险

**受影响方法**:
- `print()` - 可能未检查
- `startPrintJob()` - 应该检查 `MANAGE_PRINT_JOB`
- 系统扩展方法（`addPrinters()`, `updatePrintJobState()`）- 未检查

**解决方案**:
1. **统一权限检查**: 在所有服务方法入口添加统一的权限检查函数
2. **最小权限原则**: 每个方法只检查最小必要权限
3. **权限审计**: 定期审查权限检查点的完整性

---

## 设备发现问题

### 问题 9: 打印机连接失败

**现象**: 打印机列表中显示，但连接时返回错误

**可能原因**:
1. 打印机网络 IP 变化
2. 打印机休眠或断电
3. 驱动不兼容
4. 认证凭据错误（特别是 SMB 打印机）

**定位路径**:
```bash
# 检查连接日志
hilog -T PrintServiceAbility | grep -i "connect.*fail\|unable.*connect"

# 检查打印机状态
hilog -T PrintServiceAbility | grep "PRINTER_DISCONNECTED"

# 测试网络连接
ping <printer-ip>
telnet <printer-ip> 23
```

**解决方案**:
1. 刷新打印机列表：`stopDiscoverPrinter()` → `startDiscoverPrinter()`
2. 检查打印机网络连接（电源、网线）
3. 重新配置打印机（如果支持）
4. 对于 SMB 打印机，检查用户名和密码

---

### 问题 10: 扫描设备状态异常

**现象**: 扫描器显示为"忙碌"（SCANNER_STATUS_BUSY），但实际无扫描任务

**可能原因**:
1. 上次扫描任务异常终止，设备状态未正确重置
2. USB 通信错误导致设备异常
3. SANE 后端进程崩溃

**定位路径**:
```bash
# 检查扫描器状态
hilog -T ScanServiceAbility | grep -i "SCANNER_BUSY\|status"

# 检查 SANE 服务
hilog -T SaneServerManager | grep -i "crash\|error"

# 重置扫描器
# 关闭并重新打开扫描器
closeScanner()
openScanner()
```

**解决方案**:
1. 重启扫描服务
2. 完全关闭并重新打开扫描器（调用 `closeScanner()` 后再 `openScanner()`）
3. 检查 SANE 服务是否需要重启

---

## 调试技巧

### 技巧 1: 启用详细日志

```bash
# 设置日志级别为 DEBUG
export OHOS_HIVIEW_LOG_LEVEL = DEBUG

# 启用所有模块日志
export OHOS_HIVIEW_LOG_ON = true

# 查看 Print 模块日志
hilog -T PrintServiceAbility

# 查看扫描模块日志
hilog -T ScanServiceAbility

# 查看 SANE 服务日志
hilog -T SaneServerManager
```

### 技巧 2: 使用 HiDumper

```bash
# 导出 PrintServiceAbility 的内存和状态
hidumper -s PrintServiceAbility

# 导出当前打印任务状态
hidumper -a PrintServiceAbility printService

# 导出扫描服务状态
hidumper -s ScanServiceAbility scanServiceAbility
```

### 技巧 3: 网络抓包

对于网络打印机和扫描器的问题：

```bash
# 安装 tcpdump（需要 root 权限）
opkg install tcpdump

# 抓包分析 IPP 通信
tcpdump -i any -s 631 -w capture.pcap

# 抓包分析 SMB 通信
tcpdump -i any -s 445 -w capture.pcap
```

---

## 日志分析

### 日志级别

| 日志级别 | 用途 | 日志示例 |
|---------|------|---------|
| **DEBUG** | 详细调试信息 | 调用栈、变量值 |
| **INFO** | 关键流程信息 | 服务启动/停止、任务状态变化 |
| **WARN** | 警告信息 | 重试操作、非预期状态 |
| **ERROR** | 错误信息 | 操作失败、异常 |

### 关键日志关键字

| 关键字 | 说明 | 搜索命令 |
|---------|------|---------|
| `print_job` | 打印任务 | `hilog -T PrintServiceAbility | grep print_job` |
| `printer` | 打印机 | `hilog -T PrintServiceAbility | grep printer` |
| `scan` | 扫描 | `hilog -T ScanServiceAbility | grep scan` |
| `permission` | 权限 | `hilog -T PrintServiceAbility | grep permission` |
| `cups` | CUPS 通信 | `hilog -T PrintServiceAbility | grep cups` |
| `sane` | SANE 服务 | `hilog -T SaneServerManager | grep sane` |

---

## 性能优化

### 打印服务性能

| 优化方向 | 说明 | 实施建议 |
|---------|------|----------|
| **异步处理** | 避免阻塞主线程 | 使用 EventHandler 处理事件 |
| **任务队列** | 控制并发任务数 | 使用 OperationQueue |
| **内存缓存** | 减少重复解析 | 缓存打印机能力信息 |
| **连接池复用** | 重用 CUPS 连接 | 避免频繁连接 |

### 扫描服务性能

| 优化方向 | 说明 | 实施建议 |
|---------|------|----------|
| **批量扫描** | 减少扫描次数 | 支持批量模式扫描 |
| **图片压缩** | 减少数据传输量 | 使用合适的压缩格式 |
| **懒加载** | 按需加载 SANE 后端 | 延迟非必要后端加载 |

---

## 故障排除清单

### 环境检查

- [ ] 系统版本是否满足要求
- [ ] 磁盘空间是否充足（至少 10MB）
- [ ] 依赖服务（CUPS, Bundle）是否运行
- [ ] USB 设备权限是否授予
- [ ] 网络连接是否正常

### 服务状态检查

- [ ] PrintServiceAbility 是否正常运行（SA 已注册）
- [ ] ScanServiceAbility 是否正常运行（SA 已注册）
- [ ] SaneServerManager 是否正常运行
- [ ] CUPS 服务是否可访问

### 配置检查

- [ ] `print_sa_profiles.xml` 配置是否正确
- [ ] `cupsd.conf` 和 `cups-files.conf` 是否正确
- [ ] 权限 DAC 配置是否正确

### 日志收集

- [ ] 收集服务启动日志
- [ ] 收集错误日志和堆栈信息
- [ ] 收集系统事件日志（HiSysEvent）
- [ ] 记录复现步骤

---

**相关链接**:
- [项目概览](00_Overview.md)
- [对外 API](04_External_API.md)
- [内部架构](05_Internal_API.md)
- [安全风险评审](08_Security_Review.md)
