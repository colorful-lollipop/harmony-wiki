# 07. 问题排查

## 目的与适用范围

本文档提供 MMS 应用的常见问题排查方法和调试技巧。

**适用读者**: 开发工程师、测试工程师、运维工程师  
**阅读时间**: 约 20 分钟

---

## 常见问题

### 1. 短信发送失败

#### 症状
- 点击发送后显示发送失败图标
- 短信状态为 "发送失败"

#### 排查步骤

```
1. 检查 SIM 卡状态
   ↓
2. 查看日志中的错误码
   ↓
3. 检查 telephony.sms 调用结果
   ↓
4. 验证数据库写入
```

**关键日志标签**: `SendMsgService`

```
# 搜索发送相关日志
hdc hilog | grep SendMsgService
```

**常见错误码**:

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| 0 | SEND_SMS_SUCCESS | 正常 |
| 1 | SEND_SMS_SENDING | 发送中 |
| 2 | SEND_SMS_FAIL | 发送失败，检查网络和 SIM 卡 |

#### 代码定位

```typescript
// SendMsgService.ets:31-58
sendMessage(params, callback) {
    sms.sendMessage({
        // ...
        sendCallback: (err, value) => {
            if (err) {
                // ⚠️ 发送失败，查看 err 详情
                HiLog.w(TAG, 'sendMessage failed: ' + JSON.stringify(err));
                sendStatus = common.int.SEND_MESSAGE_FAILED;
            }
        }
    });
}
```

---

### 2. 短信接收不到

#### 症状
- 其他设备发送短信，本机无提示
- 通知栏没有新消息通知

#### 排查步骤

```
1. 检查 StaticSubscriber 是否注册
   ↓
2. 查看系统广播是否到达
   ↓
3. 检查 PDU 解析是否成功
   ↓
4. 验证数据库是否写入
   ↓
5. 检查通知发送
```

**关键日志标签**: `MmsStaticSubscriber`

```bash
# 查看接收处理日志
hdc hilog | grep MmsStaticSubscriber
```

#### 代码定位

```typescript
// MmsStaticSubscriber.ts:35
onReceiveEvent(data): void {
    HiLog.i(TAG, 'onReceiveEvent, event:' );
    // ⚠️ 检查事件是否触发
    if (data.event === common.string.SUBSCRIBER_EVENT) {
        this.dealSmsReceiveData(data, this.context);
    }
}
```

**检查点**:
1. `module.json5` 中 StaticSubscriber 配置是否正确
2. 权限 `ohos.permission.RECEIVE_SMS` 是否授予
3. PDU 数据格式是否正常

---

### 3. 通知不显示

#### 症状
- 收到短信但无通知提示
- 应用角标不更新

#### 排查步骤

```
1. 检查通知权限
   ↓
2. 查看通知发布日志
   ↓
3. 检查 WantAgent 创建
   ↓
4. 验证角标设置
```

**关键日志标签**: `NotificationService`

```bash
# 查看通知日志
hdc hilog | grep NotificationService
```

#### 代码定位

```typescript
// NotificationService.ets:46
sendNotify(actionData) {
    let notificationRequest = this.buildNotificationRequest(actionData);
    Notification.publish(notificationRequest);
    // ⚠️ 检查 publish 是否成功
}
```

**权限检查**:
```bash
# 查看权限授予情况
hdc shell aa dump -a com.ohos.mms | grep permission
```

---

### 4. 数据库操作失败

#### 症状
- 短信保存失败
- 会话列表不更新
- 查询返回空结果

#### 排查步骤

```
1. 检查 DataShareHelper 创建
   ↓
2. 查看 SQL 执行错误
   ↓
3. 验证 URI 是否正确
   ↓
4. 检查 Worker 线程通信
```

**关键日志标签**: `ConversationModel`, `ConversationListModel`

```bash
# 查看数据库操作日志
hdc hilog | grep -E "ConversationModel|ConversationListModel"
```

#### 代码定位

```typescript
// ConversationModel.ets:29-44
async insertSmsMmsInfo(valueBucket, callback, context) {
    let dataHelper = await dataShare.createDataShareHelper(
        context, common.string.URI_MESSAGE_LOG);
    // ⚠️ 检查 context 和 URI 是否有效
    dataHelper.insert(managerUri, valueBucket)
        .then(res => { /* 成功 */ })
        .catch(error => {
            // ⚠️ 查看错误详情
            HiLog.e(TAG, 'insertSmsMmsInfo fail: ' + JSON.stringify(error));
        });
}
```

**常见错误**:

| 错误 | 原因 | 解决 |
|------|------|------|
| URI not found | 数据提供方未启动 | 检查短信服务是否运行 |
| Permission denied | 权限未授予 | 检查权限配置 |
| Context null | 上下文失效 | 检查生命周期 |

---

### 5. Worker 线程异常

#### 症状
- 数据库操作无响应
- 页面卡顿
- 日志中出现 Worker 错误

#### 排查步骤

```
1. 检查 Worker 初始化
   ↓
2. 查看 Worker 通信日志
   ↓
3. 检查消息格式
   ↓
4. 验证回调函数
```

**关键日志标签**: `DataWorkerWrapper`, `DataWorkerTask`

```bash
# 查看 Worker 日志
hdc hilog | grep -E "DataWorkerWrapper|DataWorkerTask"
```

#### 代码定位

```typescript
// DataWorkerWrapper.ets:68
runInWorker(request: string, callBack, param) {
    HiLog.i(TAG, `runInWorker ${request}`);
    switch (request) {
        case common.RUN_IN_WORKER_METHOD.insertSmsMmsInfo:
            this.mConversationModel.insertSmsMmsInfo(...);
            break;
        // ⚠️ 检查 request 是否匹配
        default:
            HiLog.w(TAG, `${request} not allow!!!`);
    }
}
```

---

### 6. SIM 卡状态异常

#### 症状
- 显示无 SIM 卡
- 无法选择发送卡槽
- 运营商信息显示错误

#### 排查步骤

```
1. 检查 SIM 卡物理状态
   ↓
2. 查看 SIM 状态监听日志
   ↓
3. 检查 telephony.sim API 调用
   ↓
4. 验证偏好设置存储
```

**关键日志标签**: `CardModel`

```bash
# 查看 SIM 卡日志
hdc hilog | grep CardModel
```

#### 代码定位

```typescript
// CardModel.ets:77-86
private getSimState(): void {
    for (let i = 0; i < telephonySim.getMaxSimCount(); i++) {
        telephonySim.getSimState(i, (err, value) => {
            if (err) {
                // ⚠️ 获取状态失败
                HiLog.e(TAG, 'getSimState error: ' + JSON.stringify(err));
            } else {
                this.notifySimStateChange(i, value);
            }
        });
    }
}
```

---

## 调试方法

### 日志系统

#### HiLog 封装

```typescript
// utils/HiLog.ets
import Log from '@ohos.hilog';

export default class HiLog {
    static i(tag: string, ...args: any[]) {
        Log.info(0x01, tag, args.join(' '));
    }
    static w(tag: string, ...args: any[]) {
        Log.warn(0x01, tag, args.join(' '));
    }
    static e(tag: string, ...args: any[]) {
        Log.error(0x01, tag, args.join(' '));
    }
}
```

#### 日志标签清单

| 标签 | 所属模块 | 用途 |
|------|----------|------|
| SendMsgService | 短信发送 | 发送流程日志 |
| ConversationService | 会话详情 | 数据操作日志 |
| NotificationService | 通知 | 通知流程日志 |
| MmsStaticSubscriber | 短信接收 | 接收处理日志 |
| CardModel | SIM 卡 | SIM 状态日志 |
| DataWorkerWrapper | Worker | 线程通信日志 |
| MainAbility | 生命周期 | Ability 日志 |
| app | 应用 | 全局日志 |

### 日志查看命令

```bash
# 实时查看所有日志
hdc hilog

# 过滤特定标签
hdc hilog | grep SendMsgService

# 过滤多个标签
hdc hilog | grep -E "SendMsgService|MmsStaticSubscriber"

# 查看错误级别日志
hdc hilog | grep E/

# 保存日志到文件
hdc hilog > mms.log
```

### 调试技巧

#### 1. 数据库查询调试

```typescript
// 在 Model 中添加调试日志
async querySmsMmsInfoByCondition(actionData, callback, context) {
    // 添加输入参数日志
    HiLog.i(TAG, 'Query params: ' + JSON.stringify(actionData));
    
    let condition = this.buildQuerySmsMmsInfoCondition(actionData);
    // 添加生成的条件日志
    HiLog.i(TAG, 'Query condition built');
    
    dataHelper.query(managerUri, condition, columns)
        .then(resultSet => {
            HiLog.i(TAG, 'Query result count: ' + resultSet.rowCount);
            // ...
        });
}
```

#### 2. 网络请求调试

```typescript
// HTTP 请求添加详细日志
sendMmsMessage(params, callback) {
    let httpRequest = http.createHttp();
    HiLog.i(TAG, 'HTTP request to: ' + common.string.MMS_URL);
    HiLog.i(TAG, 'Request params: ' + JSON.stringify(params));
    
    httpRequest.request(url, options, (err, data) => {
        if (err) {
            HiLog.e(TAG, 'HTTP error: ' + JSON.stringify(err));
        } else {
            HiLog.i(TAG, 'HTTP response: ' + JSON.stringify(data));
        }
    });
}
```

#### 3. 生命周期调试

```typescript
// MainAbility.ts 中添加生命周期日志
export default class MainAbility extends Ability {
    onCreate(want, launchParam): void {
        HiLog.i(TAG, 'Ability onCreate');
        // ...
    }
    
    onForeground(): void {
        HiLog.i(TAG, 'Ability onForeground');
        // ...
    }
    
    onBackground(): void {
        HiLog.i(TAG, 'Ability onBackground');
        // ...
    }
}
```

---

## 性能优化

### 数据库操作优化

```typescript
// 使用 Worker 线程避免阻塞 UI
if (globalThis.DataWorker != null) {
    // 异步执行
    globalThis.DataWorker.sendRequest(method, params, callback);
} else {
    // 同步执行（不推荐）
    this.model.method(params, callback, context);
}
```

### 列表渲染优化

```typescript
// 使用 LazyForEach 实现虚拟列表
LazyForEach(this.dataSource, (item, index) => {
    ListItem() {
        MessageItem({ item: item })
    }
}, (item, index) => JSON.stringify(item))
```

---

## 相关链接

- [目录结构](02_DirectoryStructure.md) - 源代码位置
- [系统 API](03_SystemAPIs.md) - API 使用说明
- [数据流](04_DataFlow.md) - 数据流转流程

---

*调试指南基于代码: entry/src/main/ets/*
