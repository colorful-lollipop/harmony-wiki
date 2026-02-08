# 常见问题

---

## 目的

本文档提供 DBMS 服务的常见构建、运行和调试问题，以及问题定位路径。

---

## 适用范围

- ✅ 构建相关问题
- ✅ 运行时问题
- ✅ 调试和日志方法
- ✅ 常见错误及解决方案

---

## 构建相关问题

### Q1: 编译失败 "undefined reference to IDistributedBms"

**症状**: 编译时提示找不到 IDistributedBms 的定义

**原因**: 头文件包含顺序错误

**解决方案**:
```cpp
// 确保正确的包含顺序
#include "distributed_bms_interface.h"  // 先包含接口定义
#include "distributed_bms_proxy.h"  // 再包含 Proxy
```

**证据**: `interfaces/inner_api/include/distributed_bms_proxy.h:21`

### Q2: 链接错误 "undefined reference to VerifyCallingPermission"

**症状**: 链接时提示权限验证函数未定义

**原因**: 未链接到正确的库

**解决方案**:
```gn
// 在 BUILD.gn 中确保正确链接
deps = [
    "${dbms_inner_api_path}:dbms_fwk",
    "//foundation/bundlemanager/bundle_framework:appexecfwk_core"
]
```

**证据**: `services/dbms/BUILD.gn:61`

### Q3: SA 配置文件生成失败

**症状**: `distributedbms.cfg` 或 `402.json` 未生成到输出目录

**原因**: GN 配置错误或路径问题

**解决方案**:
```bash
# 检查 GN 配置
gn gen --root=/path/to/ohos --args="distributed_bundle_framework_enable=true"

# 检查输出目录
ls out/default/system/profile/
ls out/default/system/etc/init/
```

**证据**: `services/dbms/sa_profile/BUILD.gn`

---

## 运行时问题

### Q4: DBMS 服务启动失败

**症状**: 日志显示 "DistributedBms: OnStart failed"

**可能原因**:

1. **Bundle Manager SA 未运行**
   - 日志: "GetBundleMgr GetSystemAbility is null"
   - 解决: 确保 bundle_framework 正确编译和安装

2. **Device Manager 未初始化**
   - 日志: "Init device manager failed"
   - 解决: 检查 Device Manager 服务状态

3. **SA 注册失败**
   - 日志: "DistributedBms: OnStart failed"
   - 解决: 检查 SA 配置文件是否正确

**日志位置**:
```bash
# 查看 DBMS 服务日志
hilog -x | grep DistributedBundleMgrService

# 查看系统服务启动日志
hilog -x | grep d-bms
```

**证据**: `services/dbms/src/distributed_bms.cpp:125-165`

### Q5: 跨设备查询失败

**症状**: JS API 返回错误码 17700027（DISTRIBUTED_SERVICE_NOT_RUNNING）

**可能原因**:

1. **远程设备 DBMS 服务未启动**
   - 解决: 检查远程设备状态

2. **网络连接问题**
   - 解决: 检查分布式软总线状态

3. **设备未在线**
   - 解决: 等待设备上线

**验证方法**:
```bash
# 检查设备列表
hdc shell bm dump -a

# 检查 DBMS SA 状态
hdc shell service check 402
```

**证据**: `README_zh.md:33-40`

### Q6: 权限拒绝错误

**症状**: JS API 返回错误码 17700002（PERMISSION_DENIED）

**可能原因**:

1. **应用未声明所需权限**
   - 解决: 在 module.json5 中添加权限声明

2. **用户未授权**
   - 解决: 引导用户授权

3. **Token 类型错误**
   - 解决: 确保应用使用正确的 Token 类型

**权限声明示例**:
```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
        "reason": "$string:permission_reason"
      }
    ]
  }
}
```

**证据**: `services/dbms/src/distributed_bms.cpp:638-651`

### Q7: 图片资源加载失败

**症状**: RemoteAbilityInfo 的 icon 字段为空或加载失败

**可能原因**:

1. **图片压缩配置禁用**
   - 解决: 确保 `distributed_bundle_image_framework_enable = true`

2. **图标文件不存在**
   - 解决: 检查 Bundle 资源配置

3. **内存不足**
   - 解决: 增加设备内存

4. **Base64 编码失败**
   - 解决: 检查 EncodeBase64 函数

**验证方法**:
```bash
# 检查 feature flags
hdc shell bm dump --distributed_bundle_framework

# 检查日志中的编码错误
hilog -x | grep "EncodeBase64"
```

**证据**: `services/dbms/src/distributed_bms.cpp:502-508`

---

## 调试方法

### 日志查看

#### 查看 DBMS 服务日志

```bash
# 实时查看 DBMS 日志
hilog -T DistributedBundleMgrService -v

# 查看所有相关日志
hilog -x | grep -E "DistributedBundle|d-bms|DBMS"

# 保存日志到文件
hilog -x > dbms_log.txt
```

**日志域**: `LOG_DOMAIN = 0xD0011E0`

**证据**: `services/dbms/BUILD.gn:54-57`

#### 查看 JS API 日志

```bash
# 查看 N-API 绑定日志
hilog -T DistributedBundleMgrService -v

# 查看参数解析日志
hilog -x | grep "ParseElementName"
```

**证据**: `interfaces/kits/js/distributedBundle/distributed_bundle.cpp:40-61`

### 调试模式

#### 启用详细日志

```bash
# 编译时添加详细日志宏
gn gen --args="hilog_enable=true"

# 或在代码中添加调试日志
APP_LOGD("Debug: %{public}s", debugInfo.c_str());
```

#### 使用 dump 命令

```bash
# dump DBMS SA 信息
hdc shell service dump 402

# dump Bundle Manager 信息
hdc shell bm dump -a

# dump 设备信息
hdc shell dump ohos.account
```

**证据**: `services/dbms/include/distributed_bms.h:139-140`

### GDB 调试

#### 附加到 DBMS 进程

```bash
# 查找 DBMS 进程 PID
hdc shell ps -A | grep d-bms

# 附加 GDB
hdc shell gdbserver --attach <pid> :1234

# 从主机连接
gdb :1234
```

### HAP 调试

#### 安装测试 HAP

```bash
# 构建并安装 HAP
hdc install -r test.hap

# 查看安装结果
hdc shell bm dump -n test.hap
```

---

## 常见错误码

### DBMS 服务错误码

| 错误码 | 描述 | 可能原因 | 解决方案 |
|---------|------|----------|----------|
| ERR_BUNDLE_MANAGER_PERMISSION_DENIED | 权限被拒绝 | 检查应用权限声明 |
| ERR_BUNDLE_MANAGER_SYSTEM_API_DENIED | 系统 API 拒绝 | 确保调用者是系统应用 |
| ERR_BUNDLE_MANAGER_DEVICE_ID_NOT_EXIST | 设备 ID 不存在 | 检查设备 ID 正确性 |
| ERR_BUNDLE_MANAGER_PARAM_ERROR | 参数错误 | 检查参数格式 |
| ERR_BUNDLE_MANAGER_INVALID_USER_ID | 用户 ID 无效 | 检查用户 ID |
| ERR_APPEXECFWK_SERVICE_INTERNAL_ERROR | 服务内部错误 | 查看服务日志 |

**证据**: `services/dbms/src/distributed_bms.cpp:255-346`

### JS API 错误码

| 错误码 | 描述 | 解决方案 |
|---------|------|----------|
| 17700001 | Bundle name not found | 检查 bundleName |
| 17700003 | Ability name not found | 检查 abilityName |
| 17700007 | Device ID not found | 检查 deviceId |
| 17700027 | Distributed service not running | 检查服务启动状态 |
| 401 | Parameter error | 检查参数类型和数量 |

**证据**: `README_zh.md:35-39`

---

## 性能问题

### Q8: 跨设备查询慢

**症状**: JS API 响应时间过长

**可能原因**:

1. **网络延迟**
   - 解决: 检查网络质量

2. **远程 DBMS 响应慢**
   - 解决: 检查远程设备性能

3. **多次 IPC 调用**
   - 解决: 使用批量查询接口

**优化建议**:
```javascript
// 使用批量查询替代多次单条查询
const results = await distributedBundle.getRemoteAbilityInfo([
    elementName1, elementName2, elementName3
]);
```

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:407-492`

### Q9: 内存占用过高

**症状**: DBMS 进程占用内存 > 6577KB

**可能原因**:

1. **缓存过多数据**
   - 解决: 清理缓存

2. **大量并发请求**
   - 解决: 限流

3. **图片资源未释放**
   - 解决: 检查资源管理

**内存监控**:
```bash
# 查看进程内存
hdc shell ps -A -o pid,rss,cmd | grep d-bms

# 使用 smem 工具
hdc shell smem | grep d-bms
```

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| SA 启动流程 | services/dbms/src/distributed_bms.cpp:125-165 |
| 日志域定义 | services/dbms/BUILD.gn:54-57 |
| 错误码定义 | README_zh.md:35-39 |
| Feature flags | dbms.gni:28-59 |
