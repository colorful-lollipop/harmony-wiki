# 问题排查

> **目的**: 记录常见构建、运行、调试问题及解决方案  
> **适用范围**: 开发调试、问题定位

## 构建问题

### 问题 1：GN 工具链未找到

**错误信息**:
```
ERROR: GN not found in PATH
```

**解决方案**:
```bash
# 1. 确保已安装 Python 3.x
python3 --version

# 2. 安装 GN
# 从 https://chrome-infra-packages.appspot.com/p/gn 下载
# 或使用包管理器:
# macOS:
brew install gn

# Linux:
sudo apt-get install gn

# 3. 添加到 PATH
export PATH=$PATH:/path/to/gn
```

### 问题 2：Ninja 构建失败

**错误信息**:
```
ninja: error: loadbuild.ninja: No such file or directory
```

**解决方案**:
```bash
# 1. 先运行 GN 生成
gn gen out/default --args='...'

# 2. 然后运行 Ninja
ninja -C out/default

# 3. 如果 GN 失败，检查参数
gn gen out/default --args='...' --debug
```

### 问题 3：头文件找不到

**错误信息**:
```
fatal error: 'bluetooth_xxx.h' file not found
```

**解决方案**:
```bash
# 1. 检查 include_dirs 配置
# 2. 确保 build.gn 中正确配置 include_dirs:
include_dirs = [
  "//foundation/communication/bluetooth/interfaces/inner_api/include",
  "//foundation/communication/bluetooth/frameworks/inner/ipc/interface",
]
```

### 问题 4：模块依赖缺失

**错误信息**:
```
Dependency 'xxx' not found
```

**解决方案**:
```bash
# 1. 检查 bundle.json 中的 deps 配置
# 2. 确保依赖组件已拉取
# 3. 清理构建缓存后重试
rm -rf out/
gn gen out/default --args='...'
```

## 运行时问题

### 问题 1：蓝牙服务未启动

**错误信息**:
```
Bluetooth service not available
```

**排查步骤**:
```bash
# 1. 检查 SA 1130 是否运行
hdc shell sa.ps | grep 1130

# 2. 检查 SAMGR 日志
hdc shell hilog | grep SAMGR

# 3. 手动启动蓝牙服务
hdc shell start bluetoothservice
```

### 问题 2：N-API 模块加载失败

**错误信息**:
```
Module 'bluetooth' not found
```

**排查步骤**:
```bash
# 1. 检查 so 是否在正确位置
hdc shell ls -la /system/lib/module/libbluetooth_napi.z.so

# 2. 检查依赖
hdc shell ldd /system/lib/module/libbluetooth_napi.z.so

# 3. 检查权限
hdc shell chmod 644 /system/lib/module/libbluetooth_napi.z.so
```

### 问题 3：权限被拒

**错误信息**:
```
Permission denied
```

**解决方案**:
```json
// 1. 在 config.json 中添加权限声明
"reqPermissions": [
  {
    "name": "ohos.permission.USE_BLUETOOTH"
  },
  {
    "name": "ohos.permission.ACCESS_BLUETOOTH"
  }
]
```

### 问题 4：连接超时

**错误信息**:
```
Connection timeout
```

**排查步骤**:
```bash
# 1. 检查目标设备是否在附近
# 2. 检查设备是否已配对
# 3. 检查蓝牙是否开启
# 4. 查看详细日志
hdc shell hilog | grep bt
```

## 调试方法

### 日志查看

| 日志级别 | 过滤命令 |
|----------|----------|
| Debug | `hilog | grep "bt_"` |
| Info | `hilog | grep "Bluetooth"` |
| Error | `hilog | grep -E "ERROR|bt_.*error"` |

### 关键日志标签

| 标签 | 组件 |
|------|------|
| `bt_napi_native_module` | N-API 主模块 |
| `bt_fwk_host` | 主机框架 |
| `bt_fwk_ble` | BLE 框架 |
| `bt_fwk_a2dp` | A2DP 框架 |
| `bt_fwk_gatt` | GATT 框架 |

### 调试命令

```bash
# 1. 查看蓝牙状态
hdc shell btcli status

# 2. 查看已配对设备
hdc shell btcli paired

# 3. 查看连接状态
hdc shell btcli connections

# 4. 启用详细日志
hdc shell setparam debug.bt.verbose true
```

### IPC 调试

```bash
# 1. 查看 IPC 调用
hdc shell ipcspy

# 2. 跟踪 SAMGR
hdc shell hilog | grep SAMGR

# 3. 检查 SA 注册状态
hdc shell sa.list | grep 1130
```

## 常见错误码

### N-API 错误码

| 错误码 | 说明 | 排查方向 |
|--------|------|----------|
| 401 | 参数错误 | 检查 API 参数 |
| 10001 | 未初始化 | 调用 init |
| 10002 | 未开启 | 调用 enable |
| 10006 | 设备未找到 | 检查地址 |
| 10007 | 连接失败 | 检查目标设备 |
| 10008 | 认证被拒 | 重新配对 |

### 系统错误码

| 错误码 | 说明 | 排查方向 |
|--------|------|----------|
| -1 | 通用失败 | 查看详细日志 |
| EPERM | 权限不足 | 检查权限 |
| ENOENT | 文件不存在 | 检查依赖 |
| ETIMEDOUT | 超时 | 网络/设备问题 |

## 性能问题

### 扫描性能

**问题**: BLE 扫描功耗高

**优化建议**:
```typescript
// 1. 使用主动扫描而非被动扫描
// 2. 设置合适的扫描间隔
// 3. 扫描后及时停止
bluetoothBLE.startBLEScan({
  interval: 100,
  window: 50,
  dutyMode: 'balanced'
});

// 及时停止
bluetoothBLE.stopBLEScan();
```

### 连接性能

**问题**: GATT 连接慢

**优化建议**:
```typescript
// 1. 使用已配对设备
// 2. 缓存 GATT 连接
// 3. 使用批量操作而非多次读写
```

## 崩溃排查

### 获取崩溃日志

```bash
# 1. 查看崩溃日志
hdc shell crashlist

# 2. 获取 tombstone
hdc shell ls /data/tombstones/

# 3. 查看 JS 堆栈
hdc shell hiappevent list
```

### 常见崩溃原因

| 原因 | 解决方案 |
|------|----------|
| 空指针访问 | 检查返回值 |
| 内存越界 | 使用安全函数 |
| 线程安全问题 | 使用同步原语 |
| 资源未释放 | 使用 RAII |

## 提交问题报告

### 问题报告模板

```markdown
## 问题描述
[简述问题]

## 复现步骤
1. [步骤 1]
2. [步骤 2]

## 预期行为
[期望的结果]

## 实际行为
[实际的结果]

## 环境信息
- 设备型号:
- OpenHarmony 版本:
- Bluetooth 模块版本:

## 日志
[相关日志]

## 复现概率
- [ ] 100%
- [ ] 50%
- [ ] 偶发
```

---

[返回 SUMMARY](SUMMARY.md)
