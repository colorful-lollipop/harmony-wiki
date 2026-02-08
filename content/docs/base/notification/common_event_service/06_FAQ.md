# 06_常见问题

## 构建问题

### Q1: 构建失败，提示找不到依赖

**问题描述**:
```
error: cannot find dependency: bundle_framework:appexecfwk_base
```

**解决方案**:
1. 确保已初始化子模块：
   ```bash
   repo sync --no-tags -c
   ```

2. 确保构建系统完整：
   ```bash
   ./build.sh --prebuilts
   ```

3. 检查依赖组件是否已构建：
   ```bash
   hb set -p xxx  # 设置产品
   hb build -p bundle_framework  # 先构建依赖
   ```

**相关文件**:
- `bundle.json` - 组件依赖声明
- `event.gni` - 构建变量

---

### Q2: 编译产物未生成

**问题描述**:
构建成功但 out 目录中没有 .so 文件

**解决方案**:
1. 检查构建配置：
   ```bash
   hb build -p common_event_service --build-target cesfwk_services
   ```

2. 检查输出路径：
   ```bash
   ls -la out/standard/xxx/system/lib64/ | grep ces
   ```

3. 检查 GN 配置：
   ```bash
   gn desc out/standard/xxx/ //base/notification/common_event_service/services:cesfwk_services
   ```

---

### Q3: Ninja 构建失败

**问题描述**:
```
ninja: fatal: execopt: No such file or directory
```

**解决方案**:
1. 确保 Python 和 Ninja 版本正确：
   ```bash
   python3 --version
   ninja --version
   ```

2. 清理并重新构建：
   ```bash
   rm -rf out/standard/xxx/*
   hb build -p common_event_service
   ```

---

## 运行问题

### Q4: SA 服务启动失败

**问题描述**:
系统启动后 CES 服务不可用

**排查步骤**:
1. 检查 SA 日志：
   ```bash
   hidumper -sa 3299
   ```

2. 检查服务状态：
   ```bash
   sa_list | grep 3299
   ```

3. 检查系统日志：
   ```bash
   hilog | grep -i ces
   ```

**常见原因**:
- 依赖服务未启动
- 权限配置错误
- 配置文件损坏

---

### Q5: 事件订阅无响应

**问题描述**:
应用订阅了事件但从未收到回调

**排查步骤**:
1. 验证订阅是否成功：
   ```javascript
   CommonEvent.createSubscriber(info, (err, subscriber) => {
       if (err) {
           console.error(`Error: ${err.code}`)
       }
   })
   ```

2. 检查事件名称是否正确：
   ```javascript
   // 确认事件名称大小写
   CommonEvent.subscribe(subscriber, (err, data) => {
       console.info(`Event: ${data.event}`)
   })
   ```

3. 检查权限：
   ```javascript
   const subscribeInfo = {
       events: ["usual.event.BOOT_COMPLETED"],
       publisherPermission: "ohos.permission.RECEIVER_STARTUP_COMPLETED"
   }
   ```

---

### Q6: 有序事件处理异常

**问题描述**:
有序事件未按预期顺序处理

**解决方案**:
1. 设置正确的优先级：
   ```javascript
   const subscribeInfo = {
       events: ["my_ordered_event"],
       priority: 100  // 范围 -100~1000
   }
   ```

2. 正确调用 finishCommonEvent：
   ```javascript
   subscriberCallback(err, data) {
       // 处理事件
       data.setCode(0)
       data.setData("result")
       data.finishCommonEvent()
   }
   ```

3. 检查超时配置：
   ```
   默认为 5 秒，超时触发 ORDERED_EVENT_PROC_TIMEOUT
   ```

---

## 调试技巧

### Q7: 如何调试 CES 服务

**方法 1: 使用 hilog**
```bash
# 查看 CES 相关日志
hilog | grep -E "(CES|cesfwk|commonevent)"

# 实时跟踪
hilog -T CES -T Ces
```

**方法 2: 使用 hidumper**
```bash
# 查看 SA 状态
hidumper -sa 3299

# 查看CES服务信息
hidumper -sa 3299 -a
```

**方法 3: GDB 调试**
```bash
# 附加到进程
gdb attach $(pidof foundation)

# 加载符号
symbol-file out/standard/xxx/system/lib64/libcesfwk_services.z.so
```

---

### Q8: 如何添加自定义事件

**步骤 1: 定义事件名称**
```javascript
const MY_CUSTOM_EVENT = "my_app.custom_event"
```

**步骤 2: 发布事件**
```javascript
const publishData = {
    bundleName: "com.example.myapp",
    code: 100,
    data: "custom data"
}
CommonEvent.publish(MY_CUSTOM_EVENT, publishData, (err) => {
    if (!err) {
        console.info("Event published")
    }
})
```

**步骤 3: 订阅事件**
```javascript
const subscribeInfo = {
    events: [MY_CUSTOM_EVENT]
}
CommonEvent.createSubscriber(subscribeInfo, (err, subscriber) => {
    CommonEvent.subscribe(subscriber, (err, data) => {
        console.info(`Received: ${data.event}`)
    })
})
```

---

### Q9: 如何处理订阅者死亡

**问题描述**:
订阅者进程崩溃后需要清理

**解决方案**:
1. 设置死亡回调：
   ```javascript
   // N-API 层自动处理
   ```

2. 使用死亡接收者（Native）：
   ```cpp
   // CommonEventDeathRecipient
   class MyDeathRecipient : public IRemoteObject::DeathRecipient {
       void OnRemoteDied(const wptr<IRemoteObject> &remote) {
           // 清理订阅
       }
   }
   ```

3. 检查订阅者有效性：
   ```cpp
   if (subscriber->IsValid()) {
       // 使用订阅者
   }
   ```

---

### Q10: 粘性事件不工作

**问题描述**:
粘性事件发布后，新订阅者未收到

**排查步骤**:
1. 检查发布选项：
   ```cpp
   CommonEventPublishInfo publishInfo;
   publishInfo.SetSticky(true);  // 必须设置粘性
   ```

2. 验证事件是否存在：
   ```bash
   # 查看粘性事件
   hidumper -sa 3299 -a
   ```

3. 检查权限：
   ```cpp
   // 发布粘性事件可能需要特殊权限
   ```

---

## 性能问题

### Q11: 事件延迟过高

**问题描述**:
事件从发布到到达订阅者耗时过长

**优化建议**:
1. 减少订阅者数量
2. 优化事件回调处理时间
3. 避免在回调中执行耗时操作
4. 使用有序事件时确保快速完成

---

### Q12: 内存占用过高

**问题描述**:
CES 服务内存占用持续增长

**排查步骤**:
1. 检查泄漏：
   ```bash
   # 使用 heaptrack 或 perf
   ```

2. 减少粘性事件数量
3. 及时取消订阅
4. 限制订阅者优先级范围

---

## 相关文档

- [概览](00_Overview.md)
- [架构](01_Architecture.md)
- [N-API 接口](02_N-API.md)
- [内部 API](03_Inner_API.md)
- [构建编译](04_Build.md)
- [安全评审](05_Security.md)
