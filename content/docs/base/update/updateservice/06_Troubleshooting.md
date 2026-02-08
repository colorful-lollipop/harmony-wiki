# 06_故障排查指南

> 常见问题定位与解决方案。

## 1. 构建问题

### 1.1 GN 构建错误

#### 问题: 缺少 IDL 生成文件

**错误信息**:
```
error: 'update_service_stub.h' file not found
```

**原因**: IDL 文件未正确生成

**解决方案**:
```bash
# 重新生成 IDL
cd //base/update/updateservice
gn gen out/default
ninja -C out/default //base/update/updateservice/services/engine:update_service_interface
```

**检查点**:
- `interfaces/inner_api/engine/IUpdateService.idl` 是否存在
- `updateengine_idl_path` 路径配置是否正确

---

#### 问题: 依赖找不到

**错误信息**:
```
ninja: error: dependency '//base/update/updateservice/foundations:update_foundations' not found
```

**原因**: 依赖模块未构建

**解决方案**:
```bash
# 先构建依赖模块
ninja -C out/default //base/update/updateservice/foundations:update_foundations

# 或者全量构建
hb build -f
```

---

### 1.2 编译错误

#### 问题: 头文件路径错误

**错误信息**:
```
fatal error: 'update_service.h' file not found
```

**解决方案**:
```bash
# 检查 include_dirs 配置
gn args out/default --list | grep include

# 确保以下路径在 include_dirs 中
# - interfaces/inner_api/include
# - services/engine/include
# - foundations/ability/define
```

---

## 2. 运行时问题

### 2.1 SA 启动失败

#### 问题: SA 无法启动

**日志**:
```
E/SAMGR: SystemAbilityManagerImpl::LoadSystemAbility: fail to get library
E/UPDATE_SERVICE: dlopen libupdateservice.z.so failed
```

**排查步骤**:

1. 检查库文件是否存在:
```bash
ls -la /system/lib/libupdateservice.z.so
ls -la /system/lib/module/libupdate.z.so
```

2. 检查 SELinux 权限:
```bash
# 查看 SELinux 上下文
ls -Z /system/lib/libupdateservice.z.so

# 检查是否有关闭 SELinux
getenforce
```

3. 检查配置文件:
```bash
# 验证 SA 配置
cat /system/sa/3006.json

# 验证 RC 配置
cat /system/etc/init/updater_sa.rc
```

**常见原因**:
- 库文件未安装
- SELinux 策略缺失
- 配置文件错误

---

#### 问题: SA 启动后立即停止

**日志**:
```
I/UPDATE_SERVICE: UpdateService OnStart publish success
I/SAMGR: UnloadSystemAbility: id 3006, reason idle
```

**排查步骤**:

1. 检查是否配置了按需启动:
```json
// 3006.json
{
  "run-on-create": false,
  "auto-restart": true
}
```

2. 检查是否有调用者:
```bash
# 等待调用请求
hilog | grep "UpdateService"
```

3. 检查空闲超时配置:
```json
// 3006.json
"timedevent": {
  "value": "14400"  // 4 小时空闲后卸载
}
```

---

### 2.2 API 调用失败

#### 问题: API 返回权限错误

**错误码**:
```
BusinessError { code: 202, message: "..." }
```

**原因**: 非系统应用调用

**解决方案**:
```javascript
// 检查应用是否为系统应用
// 在 config.json 中配置
"app": {
  "signature": "...",
  "bundleName": "com.example.systemapp"
}
```

**排查步骤**:
```bash
# 查看调用者权限
hidumper -sa 3006

# 检查 token 类型
```

---

#### 问题: API 返回参数错误

**错误码**:
```
BusinessError { code: 401, message: "..." }
```

**原因**: 参数格式错误

**解决方案**:
```javascript
// 检查参数类型
// 错误示例
updater.download({
    allowNetwork: 'INVALID_NETWORK'  // 应该是枚举值
})

// 正确示例
updater.download({
    allowNetwork: 'WIFI'  // 或 'CELLULAR'
})
```

---

### 2.3 网络问题

#### 问题: 下载失败

**日志**:
```
E/UPDATE_SERVICE: download failed: network error
```

**排查步骤**:

1. 检查网络权限:
```json
// config.json
"requestPermissions": [
  {
    "name": "ohos.permission.GET_NETWORK_INFO"
  }
]
```

2. 检查网络连接:
```bash
# 测试网络连通性
ping update-server.example.com
```

3. 检查证书配置:
```bash
# 检查证书存储
ls /etc/security/certs/
```

---

### 2.4 文件问题

#### 问题: 升级包验证失败

**日志**:
```
E/UPDATE_SERVICE: verify upgrade package failed
```

**排查步骤**:

1. 检查文件路径:
```javascript
// 确保路径正确
localUpdater.verifyUpgradePackage({
    packagePath: '/data/update/ota_package/update.zip'
})
```

2. 检查文件权限:
```bash
ls -la /data/update/ota_package/
# 应该为 0770 或更宽松
```

3. 检查签名:
```bash
# 验证签名
openssl dgst -sha256 -verify public.pem -signature sig update.zip
```

---

## 3. 调试方法

### 3.1 日志查看

#### hilog 日志

```bash
# 查看所有更新服务日志
hilog | grep -E "UPDATE_SERVICE|UpdateService"

# 实时查看
hilog | grep -E "UpdateService"

# 查看最近 100 条
hilog -T 100 | grep UpdateService
```

#### Dfx 日志

```bash
# 查看 HiSysEvent
hidumper -hie

# 查看特定事件
hisysevent_query -n UPDATE_SERVICE
```

---

### 3.2 状态查看

#### SA 状态

```bash
# 查看 SA 列表
hidumper -sa

# 查看特定 SA 详情
hidumper -sa 3006

# 查看 SA 依赖
hidumper -sa 3006 --dump
```

#### Dump 信息

```bash
# 获取 SA dump
hidumper -sa 3006 -dump

# 获取升级任务信息
hidumper -sa 3006 -a
```

---

### 3.3 压力测试

#### 频繁调用

```javascript
// 模拟频繁调用
async function stressTest() {
    for (let i = 0; i < 100; i++) {
        try {
            await updater.checkNewVersion()
        } catch (e) {
            console.error(`Call ${i} failed:`, e)
        }
    }
}
```

#### 并发调用

```javascript
// 并发测试
async function concurrentTest() {
    const promises = []
    for (let i = 0; i < 10; i++) {
        promises.push(updater.getTaskInfo())
    }
    const results = await Promise.all(promises)
}
```

---

## 4. 性能问题

### 4.1 响应时间过长

#### 检查方法

```bash
# 使用 systrace
python3 $ANDROID_HOME/platform-tools/systrace/systrace.py \
    -a com.ohos.update \
    -b 16384 \
    -o trace.html \
    sched gfx view wm am
```

#### 优化建议

| 操作 | 预期时间 | 优化建议 |
|------|----------|----------|
| checkNewVersion | < 1s | 使用缓存 |
| download | N/A | 后台线程 |
| upgrade | N/A | 分阶段处理 |

---

### 4.2 内存问题

#### 内存泄漏检查

```bash
# 使用 valgrind 或 ASan
gn gen out/default --args="use_clang_debug=true"
ninja -C out/default //base/update/updateservice:updateservice

# 运行测试
```

---

## 5. 常见错误码

### 5.1 业务错误码

| 错误码 | 名称 | 描述 | 解决方案 |
|--------|------|------|----------|
| 0 | SUCCESS | 成功 | - |
| 100 | FAIL | 通用失败 | 查看详细日志 |
| 103 | FORBIDDEN | 操作禁止 | 检查权限 |
| 201 | APP_NOT_GRANTED | 无权限 | 申请权限 |
| 202 | NOT_SYSTEM_APP | 非系统应用 | 签名系统应用 |
| 401 | PARAM_ERR | 参数错误 | 检查参数 |
| 801 | UN_SUPPORT | 不支持 | 检查能力 |

### 5.2 系统错误码

| 错误码 | 描述 |
|--------|------|
| -1 | 未知错误 |
| -2 | 内存不足 |
| -3 | 文件不存在 |
| -4 | 权限不足 |
| -5 | 网络错误 |

---

## 6. 调试工具

### 6.1 命令行工具

| 工具 | 用途 |
|------|------|
| `hidumper` | 系统状态转储 |
| `hilog` | 日志查看 |
| `bm` | Bundle Manager |
| `aa` | Ability Manager |

### 6.2 日志过滤

```bash
# 过滤更新服务日志
hilog | grep -E "UPDATE|UpdateService|UpdateClient"

过滤错误# 日志
hilog | grep -E "ERROR|Error|error"

# 过滤特定模块
hilog --tag UpdateService -v Debug
```

---

## 7. 回退与恢复

### 7.1 回退升级包

```bash
# 手动触发回退
reboot bootloader

# 或使用系统回滚
hdc shell
bm dump -n <package_name>
```

### 7.2 恢复出厂设置

```javascript
// 强制恢复出厂设置 (需要 FORCE_FACTORY_RESET 权限)
const restorer = client.getRestorer()
restorer.forceFactoryReset()
```

---

## 8. 联系方式

### 8.1 提单渠道

- **Gitee Issue**: https://gitee.com/openharmony/update_updateservice/issues

### 8.2 文档反馈

- **Wiki**: https://gitee.com/openharmony/update_updateservice/wikis

---

## 9. 下一步

- **架构设计**: [01_Architecture.md](./01_Architecture.md)
- **N-API 参考**: [02_N-API.md](./02_N-API.md)
- **安全评审**: [05_Security.md](./05_Security.md)
