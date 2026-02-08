# 常见问题 (FAQ)

## 构建问题

### Q1: 编译错误 - 未找到头文件

**问题**:
```
fatal error: samgr_lite.h: No such file or directory
```

**解决**:
确保在 BUILD.gn 中正确配置了 include_dirs：
```gn
include_dirs = [
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr",
    "//foundation/systemabilitymgr/samgr_lite/interfaces/kits/registry",
]
```

### Q2: 链接错误 - 未定义的引用

**问题**:
```
undefined reference to `SAMGR_GetInstance'
```

**解决**:
确保链接了正确的库：
```gn
public_deps = [
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
]
```

### Q3: A-core 动态库编译失败

**问题**:
```
error: '-fPIC' is not allowed for non-shared libraries
```

**解决**:
A-core 需要构建为动态库，确保配置正确：
```gn
if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  shared_library("samgr") {
    cflags = ["-fPIC"]
  }
}
```

## 运行问题

### Q4: 服务无法注册

**问题**:
`RegisterService()` 返回 FALSE

**可能原因**:
1. 服务名超过 16 字节
2. Service 结构未正确初始化
3. 初始化时机过早（在 Samgr 启动前）

**解决**:
```c
// 检查服务名长度
if (strlen(SERVICE_NAME) >= 16) {
    // 缩短服务名
}

// 确保使用正确的初始化宏
SYSEX_SERVICE_INIT(Init);
```

### Q5: 消息无法送达

**问题**:
`SAMGR_SendRequest()` 返回成功但处理函数未收到消息

**可能原因**:
1. Identity 无效
2. 目标服务消息队列已满
3. 目标服务已停止

**解决**:
```c
// 检查 Identity 有效性
if (identity->queueId == NULL) {
    HILOG_ERROR(HILOG_MODULE_APP, "Invalid queue ID");
    return;
}

// 增加消息队列深度
TaskConfig config = {...};
config.queueSize = 50;  // 增大队列
```

### Q6: 跨进程调用失败

**问题**:
IPC 调用返回错误或无响应

**可能原因**:
1. 不在 A-core 平台
2. 远程服务未注册
3. 权限不足

**解决**:
```c
// 检查平台
#if defined(__liteos_a__) || defined(__linux__)
// IPC 调用代码
#else
#error "IPC only supported on A-core"
#endif

// 检查远程服务可用性
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(serviceName, featureName);
if (iUnknown == NULL) {
    HILOG_WARN(HILOG_MODULE_APP, "Remote service not available");
    return;
}
```

## API 使用问题

### Q7: 如何获取默认 Feature API

**问题**:
不确定应该使用 `GetDefaultFeatureApi` 还是 `GetFeatureApi`

**解决**:
```c
// 获取默认 Feature API（没有指定 Feature 名时使用）
IUnknown *iUnknown = SAMGR_GetInstance()->GetDefaultFeatureApi(SERVICE_NAME);

// 获取指定 Feature API
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(SERVICE_NAME, FEATURE_NAME);
```

### Q8: 消息处理的最佳实践

**问题**:
不确定如何在 MessageHandle 中正确处理消息

**解决**:
```c
BOOL MessageHandle(Service *service, Request *request)
{
    switch (request->msgId) {
        case MSG_TYPE_A:
            // 处理类型 A 的消息
            return TRUE;
        case MSG_TYPE_B:
            // 处理类型 B 的消息
            return TRUE;
        default:
            // 未知消息类型
            return FALSE;
    }
}
```

### Q9: 如何正确释放 IUnknown

**问题**:
不确定何时调用 `Release`

**解决**:
```c
// 1. 获取接口
IUnknown *iUnknown = SAMGR_GetInstance()->GetDefaultFeatureApi(SERVICE_NAME);

// 2. 查询具体接口
ApiType *api = NULL;
iUnknown->QueryInterface(iUnknown, VERSION, (void **)&api);

// 3. 使用完毕后释放
api->Release((IUnknown *)api);

// 注意：不要释放 iUnknown，因为不是你自己创建的
```

### Q10: 广播服务如何使用

**解决**:
```c
// 获取广播服务接口
PubSubInterface *pubSubApi = NULL;
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(
    BROADCAST_SERVICE, PUB_SUB_FEATURE);
iUnknown->QueryInterface(iUnknown, DEFAULT_VERSION, (void **)&pubSubApi);

// 订阅主题
Topic topic = MY_TOPIC;
pubSubApi->subscriber.AddTopic(iUnknown, &topic);
pubSubApi->subscriber.Subscribe(iUnknown, &topic, &consumer);

// 发布主题
pubSubApi->provider.Publish(iUnknown, &topic, data, len);
```

## 调试问题

### Q11: 如何打印所有已注册服务

**解决**:
```c
// 在服务代码中调用
SAMGR_PrintServices();  // 打印服务列表
SAMGR_PrintOperations(); // 打印操作列表
```

### Q12: 如何调试消息路由

**解决**:
使用 HILOG 日志：
```c
HILOG_INFO(HILOG_MODULE_APP, "Sending request, msgId=%d", request->msgId);
HILOG_INFO(HILOG_MODULE_APP, "Target identity: sid=%d, fid=%d",
           identity->serviceId, identity->featureId);
```

### Q13: 如何查看任务状态

**解决**:
检查 `task_manager.c` 中的调试接口（如果启用）：
```c
// 打印任务池状态
SAMGR_PrintTaskPoolStatus();
```

## 性能问题

### Q14: 消息队列满导致阻塞

**解决**:
```c
// 增大消息队列
TaskConfig config = {...};
config.queueSize = 100;  // 根据需要调整

// 或使用异步处理，避免队列堆积
```

### Q15: 内存占用过高

**解决**:
1. 减少共享任务数
2. 减小任务栈大小
3. 使用消息池复用内存

### Q16: 跨进程调用延迟高

**解决**:
1. 减少跨进程调用频率
2. 使用批量操作替代多次调用
3. 考虑使用共享内存进行大数据传输

## 相关文档

- [SamgrLite API](./04_SamgrLite_API.md)
- [消息通信 API](./05_Message_API.md)
- [IPC 跨进程接口](./06_IPC_API.md)
- [安全风险评审](./12_Security_Review.md)
