# 常见问题

> 本文档收集了 app_domain_verify 部件的常见构建、运行和调试问题及解决方案。

## 1. 构建问题

### Q1: 编译失败 - 找不到头文件

**错误信息**:
```
fatal error: 'app_domain_verify_mgr_client.h' file not found
```

**原因**:
缺少 `app_domain_verify` 部件的依赖配置。

**解决方案**:

1. **在 bundle.json 中添加依赖**:
```json
{
  "deps": {
    "components": [
      "app_domain_verify"
    ]
  }
}
```

2. **在模块的 BUILD.gn 中添加依赖**:
```gn
external_deps = [
  "app_domain_verify:app_domain_verify_mgr_client",
  "app_domain_verify:app_domain_verify_common"
]
```

3. **在代码中包含正确头文件**:
```cpp
#include "app_domain_verify_mgr_client.h"
```

---

### Q2: 编译失败 - 符号未定义

**错误信息**:
```
undefined reference to `AppDomainVerifyMgrClient::GetInstance()'
```

**原因**:
链接了错误的库或者链接顺序问题。

**解决方案**:

1. 确保链接了正确的库:
```gn
external_deps = [
  "app_domain_verify:app_domain_verify_mgr_client",
]
```

2. 检查版本脚本是否正确导出符号:
```gn
# 在对应的 BUILD.gn 中
configs += [ ":symbol_export_config" ]
```

---

### Q3: 编译产物大小异常

**问题描述**:
编译产物体积过大或过小。

**排查步骤**:

1. **检查优化标志**:
```bash
# 查看编译选项
grep -r "is_debug\|O[0-3]" out/compile_commands.json
```

2. **检查链接的库**:
```bash
# 查看依赖的共享库
ldd your_app | grep app_domain_verify
```

3. **检查是否有未清理的编译产物**:
```bash
# 清理后重新编译
rm -rf out/
./build.sh --product-name <product> --build-target app_domain_verify
```

---

## 2. 运行问题

### Q4: Manager Service 无法启动

**问题描述**:
SA 6200 (Manager Service) 启动失败。

**排查步骤**:

1. **检查 SA 配置**:
```bash
# 查看 SA 配置文件
cat system/profile/6200.json
```

2. **检查服务日志**:
```bash
# 查看日志
hilog | grep -i "app_domain_verify"
```

3. **检查依赖库**:
```bash
# 检查库文件是否存在
ls -la system/lib/libapp_domain_verify_*.so
```

**常见原因**:
- 库文件权限不正确
- 依赖的库缺失
- 配置文件语法错误

---

### Q5: Agent Service 按需未启动

**问题描述**:
发起域名校验时 Agent Service (SA 6201) 未启动。

**排查步骤**:

1. **检查开机事件是否触发**:
```bash
# 查看服务状态
hilog | grep -i "BOOT_COMPLETED"
```

2. **检查定时任务配置**:
```bash
# 查看 SA 配置文件
cat system/profile/6201.json
# 确认 "start-on-demand" 配置正确
```

3. **手动触发启动**:
```bash
# 通过 bm 命令触发
bm dump -a <ability_name>
```

---

### Q6: 域名校验持续失败

**问题描述**:
应用安装后域名校验一直失败。

**排查步骤**:

1. **检查网络连通性**:
```bash
# 测试域名解析
ping www.example.com

# 测试 HTTPS 连接
curl -v https://www.example.com/.well-known/assetlinks.json
```

2. **检查 assetlinks.json 格式**:
```json
{
  "applinking": {
    "apps": [
      {
        "bundleName": "com.example.app",
        "appIdentifier": "ABCDEFGHIJKLMN",
        "fingerprint": [
          {
            "algorithm": "SHA256",
            "certDigest": "base64编码的证书指纹"
          }
        ]
      }
    ]
  }
}
```

3. **检查应用签名**:
```bash
# 获取应用签名
bm dump -b <bundle_name> | grep fingerprint
```

4. **检查日志**:
```bash
# 查看详细校验日志
hilog | grep -i "verify\|domain\|assetlink"
```

---

### Q7: FilterAbilities 返回空结果

**问题描述**:
调用 `FilterAbilities()` 返回空的 ability 列表。

**排查步骤**:

1. **检查 Want 参数**:
```bash
# 确认 Want 中的 URL 格式正确
hilog | grep -i "want\|uri"
```

2. **检查域名校验状态**:
```bash
# 查询校验状态
hilog | grep -i "verify_status\|filter"
```

3. **检查ability过滤逻辑**:
```bash
# 查看过滤详情
hilog | grep -i "ability_filter\|origin\|filtered"
```

**常见原因**:
- URL 域名未通过校验
- ability 的 Skill 配置不匹配
- Want 中的 URL 格式错误

---

### Q8: N-API 调用返回错误码

**问题描述**:
调用 `queryAssociatedDomains()` 返回错误。

**错误码说明**:

| 错误码 | 描述 | 解决方案 |
|-------|------|---------|
| 401 | 参数错误 | 检查 bundleName 格式 |
| 16300001 | 内部错误 | 查看日志定位问题 |
| 16300002 | 权限不足 | 需要系统应用权限 |
| 16300003 | 服务不可用 | 检查 Manager Service 状态 |

**排查步骤**:

```typescript
import bundle from '@ohos.bundle.appDomainVerify'

try {
    let domains = bundle.queryAssociatedDomains("com.example.app");
    console.log(`Domains: ${domains}`);
} catch (error) {
    console.error(`Error: ${error.code} - ${error.message}`);
    // 查看 hilog 获取详细错误
}
```

---

## 3. 调试问题

### Q9: 如何启用详细日志

**解决方案**:

1. **设置日志级别**:
```bash
# 设置hiloglevel为DEBUG
hilog -v D
```

2. **过滤特定模块日志**:
```bash
# 只显示 app_domain_verify 相关日志
hilog | grep -E "APP_DOMAIN_VERIFY|DomainVerify"
```

3. **添加临时日志**:
```cpp
// 在代码中添加
APP_DOMAIN_VERIFY_HILOGD("Debug info: %{public}s", debugInfo.c_str());
```

---

### Q10: 如何调试 IPC 通信

**解决方案**:

1. **启用 IPC 调试日志**:
```bash
# 查找 Binder 调试日志
hilog | grep -i "binder\|ipc"
```

2. **检查 IPC 参数**:
```cpp
// 在 stub 中添加日志
int AppDomainVerifyMgrServiceStub::OnRemoteRequest(
    uint32_t code, MessageParcel& data, MessageParcel& reply, MessageOption& option) {
    APP_DOMAIN_VERIFY_HILOGD("IPC code: %{public}u", code);
    // 打印参数
    APP_DOMAIN_VERIFY_HILOGD("Data size: %{public}zu", data.GetDataSize());
    // ...
}
```

3. **使用 dump 命令**:
```bash
# 获取服务状态
bm dump -a <service_name>
```

---

### Q11: 如何分析内存问题

**解决方案**:

1. **查看内存使用**:
```bash
# 查看进程内存
cat /proc/<pid>/status | grep -E "VmSize|VmRSS"
```

2. **使用内存分析工具**:
```bash
# 启动内存跟踪
hdc shell " perf func --pid <pid> --mm"
```

3. **检查内存泄漏**:
```cpp
// 确认智能指针使用正确
std::shared_ptr<VerifyTask> task = std::make_shared<VerifyTask>();
// 确保任务完成后自动释放
```

---

### Q12: 如何验证数据库状态

**解决方案**:

1. **查看数据库文件**:
```bash
# 定位数据库
ls -la /data/service/el1/public/app_domain_verify_agent_service/
```

2. **使用 RDB 工具查询**:
```cpp
// 导出数据验证
auto dataMgr = AppDomainVerifyRdbDataManager::GetInstance();
std::vector<VerifyResultInfo> results;
dataMgr.QueryAllVerifyStatus(results);
```

3. **检查表结构**:
```sql
-- 连接数据库后执行
.sqlite3 app_domain_verify.db
.schema
SELECT * FROM app_verify_status LIMIT 10;
```

---

## 4. 性能问题

### Q13: 校验耗时过长

**问题描述**:
单个域名校验超过预期时间。

**排查步骤**:

1. **分解耗时**:
   - DNS 解析时间
   - TCP 连接时间
   - TLS 握手时间
   - 请求发送时间
   - 响应接收时间
   - JSON 解析时间
   - 签名验证时间

2. **检查网络**:
```bash
# 测试网络延迟
curl -w "DNS: %{time_namelookup}s, TCP: %{time_connect}s" https://domain/.well-known/assetlinks.json
```

3. **检查资源竞争**:
```bash
# 查看 CPU 使用
top -H -p <pid>
```

**优化建议**:
- 使用连接池复用 TCP 连接
- 添加超时限制
- 并行校验多个域名

---

### Q14: 高并发场景性能下降

**问题描述**:
大量应用同时安装时性能下降。

**排查步骤**:

1. **检查 FFRT 线程池配置**:
```cpp
// 查看线程数
constexpr int THREAD_POOL_SIZE = 4;  // frameworks/common/src/httpsession/app_domain_verify_task_mgr.cpp
```

2. **检查任务队列**:
```cpp
// 查看队列积压
auto taskCount = taskMgr_->GetTaskCount();
if (taskCount > MAX_QUEUE_SIZE) {
    // 处理队列溢出
}
```

3. **检查数据库锁**:
```cpp
// 避免长时间事务
auto guard = MakeScopeGuard([&]() { /* 提交 */ });
// 快速完成操作
```

---

## 5. 崩溃问题

### Q15: 服务崩溃 - Segmentation Fault

**问题描述**:
Manager Service 或 Agent Service 崩溃。

**排查步骤**:

1. **获取崩溃堆栈**:
```bash
# 查看崩溃日志
hilog | grep -i "signal\|SIGSEGV\|backtrace"
```

2. **使用 addr2line**:
```bash
# 定位崩溃位置
addr2line -e libapp_domain_verify_mgr_service.so <address>
```

3. **检查空指针**:
```cpp
// 常见空指针场景
if (ptr == nullptr) {
    return;
}
```

---

### Q16: 服务崩溃 - Out of Memory

**问题描述**:
服务因内存不足崩溃。

**排查步骤**:

1. **查看内存日志**:
```bash
hilog | grep -i "OOM\|memory\|VmSize"
```

2. **分析内存使用**:
```bash
# 持续监控
watch -n 1 'cat /proc/<pid>/status | grep Vm'
```

3. **检查内存泄漏**:
```cpp
// 使用智能指针
std::unique_ptr<Data> data = std::make_unique<Data>();
```

---

## 6. 相关文档

| 文档 | 链接 |
|-----|------|
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| Inner API | [02_Inner_API.md](./02_Inner_API.md) |
| N-API | [03_N_API.md](./03_N_API.md) |
| 安全风险评审 | [06_Security.md](./06_Security.md) |
