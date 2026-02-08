# 安全风险评审

## 概述

本文档对 samgr_lite 进行安全风险评审，分析攻击面、信任边界和潜在安全风险，并提供修复建议。

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│  │   Service    │    │   Feature    │    │  Application │     │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘     │
│         │                   │                   │              │
│         └───────────────────┼───────────────────┘              │
│                             │                                  │
│                    ┌────────┴────────┐                        │
│                    │   Samgr Core    │                        │
│                    │  (可信区域)      │                        │
│                    └────────┬────────┘                        │
│                             │                                  │
├─────────────────────────────┼──────────────────────────────────┤
│                             │                                  │
│  ┌──────────────┐    ┌──────┴──────┐    ┌──────────────┐     │
│  │ samgr_server │    │IPC (Binder) │    │samgr_client  │     │
│  │  (IPC服务端)  │◄──►│             │◄──►│  (IPC客户端)  │     │
│  └──────────────┘    └─────────────┘    └──────────────┘     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 攻击面分析

### 1. N-API 接口攻击面

| 接口 | 攻击面 | 风险等级 |
|------|--------|----------|
| `SAMGR_GetInstance()` | 无（仅获取单例） | 低 |
| `SAMGR_RegisterFactory()` | 参数校验不足 | 中 |
| `SAMGR_SendRequest()` | 消息队列溢出 | 中 |
| `SAMGR_SendSharedRequest()` | 引用计数管理 | 中 |
| Message Handler | 任意代码执行 | 高 |

### 2. IPC 攻击面

| 接口 | 攻击面 | 风险等级 |
|------|--------|----------|
| `IClientProxy::Invoke()` | 序列化数据解析 | 高 |
| `IServerProxy::Invoke()` | 函数 ID 路由 | 中 |
| `SAMGR_GetRemoteIdentity()` | 远程身份泄露 | 中 |

### 3. 资源攻击面

| 资源 | 攻击面 | 风险等级 |
|------|--------|----------|
| 消息队列 | 队列溢出/DoS | 中 |
| 任务池 | 线程耗尽 | 中 |
| 内存分配 | 内存耗尽 | 中 |

## 安全风险清单

### 风险 1：服务名/Feature 名长度无校验

**风险等级**: 中

**证据位置**:
- `README.md` 约束章节：服务名和 Feature 名必须 < 16 字节
- `samgr_lite.h:65-70`：定义了 `MAX_SYSCAP_NAME_LEN` 但未对服务名强制校验

**问题描述**:
虽然文档约束服务名必须 < 16 字节，但代码中未发现对服务名长度进行强制校验的代码。恶意服务可能传入超长服务名导致缓冲区溢出。

**触发条件**:
```c
// 恶意服务注册
char longName[32] = "This_is_a_very_long_service_name_32";
SAMGR_GetInstance()->RegisterService((Service *)longName);  // 未校验长度
```

**影响**:
- 潜在的缓冲区溢出
- 服务发现混乱

**修复建议**:
```c
// 在 RegisterService 中添加长度校验
BOOL (*RegisterService)(Service *service) {
    const char *name = service->GetName(service);
    if (name == NULL || strlen(name) >= MAX_SERVICE_NAME_LEN) {
        return FALSE;  // 拒绝注册
    }
    // ...
}
```

---

### 风险 2：Message Handler 中的输入验证不足

**风险等级**: 高

**证据位置**:
- `samgr_lite/source/message.c`：消息处理入口
- `message.h:95-104`：Request 结构中 `data` 和 `len` 字段

**问题描述**:
Request 中的 `data` 和 `len` 由调用者控制，Message Handler 如果直接使用这些数据而不进行验证，可能导致：
- 缓冲区溢出
- 越界访问
- 类型混淆攻击

**触发条件**:
```c
// 恶意客户端发送恶意请求
Request request = {
    .msgId = MSG_PROCESS,
    .data = maliciousBuffer,  // 恶意数据
    .len = 0xFFFF,            // 伪造长度
};
SAMGR_SendRequest(&identity, &request, NULL);
```

**影响**:
- 内存破坏
- 远程代码执行（如果恶意数据被当作代码执行）

**修复建议**:
```c
// 在 MessageHandle 中验证输入
BOOL MessageHandle(Service *service, Request *request)
{
    // 验证 len 与实际数据大小匹配
    if (request->len > MAX_DATA_SIZE) {
        return FALSE;
    }
    
    // 验证数据指针有效
    if (request->data != NULL && !IsMemoryReadable(request->data, request->len)) {
        return FALSE;
    }
    
    // 安全的内存复制
    char buffer[MAX_BUFFER_SIZE];
    memcpy_s(buffer, sizeof(buffer), request->data, request->len);
    // ...
}
```

---

### 风险 3：Token Bucket 限流可能失效

**风险等级**: 中

**证据位置**:
- `samgr_endpoint/source/token_bucket.c`：令牌桶实现
- `samgr_endpoint/BUILD.gn`：包含限流逻辑

**问题描述**:
令牌桶用于 IPC 消息限流，但如果配置不当或存在竞争条件，可能导致限流失效。

**触发条件**:
```c
// 并发请求超过限流阈值
for (int i = 0; i < 1000; i++) {
    thread_create(SendIPCRequest, ...);  // 并发发送
}
```

**影响**:
- 拒绝服务攻击（DoS）
- 系统资源耗尽

**修复建议**:
1. 在令牌桶实现中添加原子操作
2. 增大令牌桶容量和补充速率
3. 添加突发流量检测

---

### 风险 4：IPC 消息序列化边界检查不足

**风险等级**: 高

**证据位置**:
- `samgr_endpoint/source/endpoint_rpc.c`：IPC 消息处理
- `iproxy_client.h:91-114`：IClientProxy Invoke 接口

**问题描述**:
从 IpcIo 中读取数据时，如果读取大小超过实际数据，可能导致越界读取或解析错误。

**触发条件**:
```c
// 恶意 IPC 消息
IpcIo req;
// ... 构造短于预期的消息
int32 value = IpcIoPopInt32(req);  // 可能读取无效数据
```

**影响**:
- 信息泄露
- 解析错误导致崩溃

**修复建议**:
```c
static int32 Invoke(IServerProxy *iProxy, int funcId, void *origin, 
                    IpcIo *req, IpcIo *reply)
{
    // 验证数据可用性
    if (IpcIoAvailable(req) < sizeof(int32)) {
        IpcIoPushBool(reply, FALSE);
        return EC_FAILURE;
    }
    
    int32 value = IpcIoPopInt32(req);
    // ...
}
```

---

### 风险 5：samgr_server 权限控制不完整

**风险等级**: 中

**证据位置**:
- `samgr_server/BUILD.gn:27-29`：包含 permission_lite 依赖
- `samgr_server/source/samgr_server_rpc.c`：权限检查逻辑

**问题描述**:
虽然 samgr_server 依赖 permission_lite 进行权限控制，但需要验证：
- 所有敏感操作是否都有权限检查
- 权限验证失败时的错误处理是否安全

**触发条件**:
```c
// 无权限调用
IClientProxy *proxy = GetUnpermittedService();
proxy->Invoke(proxy, SENSITIVE_FUNC_ID, &req, NULL, NULL);  // 可能绕过权限
```

**影响**:
- 未授权访问敏感服务
- 权限提升

**修复建议**:
1. 确保所有 IPC 调用入口都有权限验证
2. 使用白名单而非黑名单机制
3. 记录权限拒绝日志

---

### 风险 6：客户端代理工厂验证不足

**风险等级**: 中

**证据位置**:
- `samgr_client/source/remote_register_rpc.c`：客户端注册
- `registry.h:69`：Creator 函数类型

**问题描述**:
SAMGR_RegisterFactory 接受任意的 creator/destroyer 函数指针，如果传入恶意函数指针，可能导致：
- 任意代码执行
- 内存破坏

**触发条件**:
```c
// 恶意工厂注册
SAMGR_RegisterFactory("malicious", "feature", 
                      MaliciousCreator,  // 恶意函数
                      MaliciousDestroyer);
```

**影响**:
- 代码执行
- 内存破坏

**修复建议**:
```c
int SAMGR_RegisterFactory(const char *service, const char *feature, 
                          Creator creator, Destroyer destroyer)
{
    // 验证函数指针不为 NULL
    if (creator == NULL || destroyer == NULL) {
        return EC_INVALID_PARAM;
    }
    
    // 验证函数指针在有效地址范围内
    if (!IsAddressValid(creator) || !IsAddressValid(destroyer)) {
        return EC_INVALID_PARAM;
    }
    
    // 验证服务/Feature 名称
    if (!IsValidServiceName(service) || !IsValidFeatureName(feature)) {
        return EC_INVALID_PARAM;
    }
    // ...
}
```

---

### 风险 7：跨进程空指针解引用

**风险等级**: 中

**证据位置**:
- `samgr_endpoint/source/default_client_rpc.c`：客户端 IPC 实现
- `samgr_endpoint/source/endpoint_rpc.c`：端点处理

**问题描述**:
跨进程调用时，如果远程服务不可用，可能导致空指针解引用。

**触发条件**:
```c
// 远程服务已崩溃
IUnknown *iUnknown = SAMGR_GetFeatureApi(REMOTE_SERVICE, REMOTE_FEATURE);
if (iUnknown == NULL) {
    return;  // 正确处理
}
// 但某些路径可能未检查
```

**影响**:
- 系统崩溃
- 拒绝服务

**修复建议**:
```c
IUnknown *GetRemoteApi(const char *service, const char *feature)
{
    IUnknown *iUnknown = SAMGR_GetFeatureApi(service, feature);
    if (iUnknown == NULL) {
        HILOG_WARN(HILOG_MODULE_SAMGR, "Remote service %s not available", service);
        return NULL;
    }
    // ... 进一步验证
}
```

---

### 风险 8：向量容器溢出

**风险等级**: 中

**证据位置**:
- `common.h:105-126`：Vector 结构定义
- `samgr_lite_inner.h:26-27`：`MAX_SERVICE_NUM = 0x7FF0`

**问题描述**:
Vector 使用 `int16` 作为索引和计数，当添加元素超出 `INT16_MAX` 时可能溢出。

**触发条件**:
```c
// 大量服务注册
for (int i = 0; i < 50000; i++) {
    RegisterService(&serviceArray[i]);  // 可能溢出
}
```

**影响**:
- 索引越界
- 内存破坏

**修复建议**:
```c
int16 VECTOR_Add(Vector *vector, void *element)
{
    if (vector->top >= vector->max) {
        // 拒绝添加或尝试扩容
        return INVALID_INDEX;
    }
    // ...
}
```

## 安全最佳实践

### 输入验证

1. **所有外部输入必须验证**
   - 服务名长度
   - Feature 名长度
   - Request data/length 匹配
   - IpcIo 序列化数据

2. **验证时机**
   - 在入口点验证
   - 在处理前验证
   - 不要依赖调用者验证

### 内存安全

1. **使用安全内存函数**
   ```c
   // 避免
   strcpy(dst, src);
   
   // 使用
   strcpy_s(dst, dstSize, src);
   memcpy_s(dst, dstSize, src, srcSize);
   ```

2. **验证指针有效性**
   ```c
   if (ptr != NULL && IsMemoryReadable(ptr, size)) {
       // 安全使用
   }
   ```

### 并发安全

1. **关键操作使用原子操作**
2. **避免死锁**
3. **正确使用锁**

### 权限控制

1. **默认拒绝**
2. **最小权限原则**
3. **记录审计日志**

## 安全检查清单

| 检查项 | 状态 | 备注 |
|--------|------|------|
| 服务名长度校验 | 待完善 | 需在 RegisterService 中添加 |
| Request 输入验证 | 待完善 | 需在 Handler 中加强 |
| IpcIo 边界检查 | 待完善 | 需验证读取大小 |
| Token Bucket 原子性 | 待评估 | 需审查实现 |
| 权限控制完整性 | 待评估 | 需审查 samgr_server |
| 工厂函数指针验证 | 待完善 | 需添加验证 |
| 空指针检查 | 待评估 | 需审查跨进程路径 |
| Vector 溢出保护 | 待评估 | 需审查边界 |

## 相关安全文档

- OpenHarmony 安全指南
- IPC 安全最佳实践
- 内存安全编码规范

## 下一章

- [最佳实践](./13_Best_Practices.md) - 开发规范与注意事项
