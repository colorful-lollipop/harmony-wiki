# 安全风险评审

> **目的**: 基于代码证据分析PermissionManager的安全风险、攻击面和修复建议  
> **适用范围**: 安全研究人员、安全审计人员  
> **最后更新**: 2026-02-05

---

## 威胁模型

### 系统边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                      不信任区域 (调用方应用)                          │
│  - 可能传递恶意构造的Want参数                                        │
│  - 可能尝试权限提升                                                  │
│  - 可能尝试绕过权限检查                                              │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼ 通过系统服务验证
┌─────────────────────────────────────────────────────────────────────┐
│                        信任边界 (AT权限服务)                         │
│  - 验证调用者身份                                                    │
│  - 校验权限请求合法性                                                │
│  - 启动PermissionManager时传递可信参数                              │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        PermissionManager (本应用)                    │
│  - 接收经过系统验证的参数                                            │
│  - 但仍需防御性编程                                                  │
│  - 处理IPC通信、窗口管理、权限操作                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### 数据流分析

```
外部输入
    │
    ├── Want参数 (bundleName, tokenId, permissions, callback)
    │       └── 解析位置: GrantDialogModel.getCallerAppInfo()
    │
    ├── IPC消息 (rpc.RemoteObject)
    │       └── 处理位置: GrantDialogModel.terminateWithResult()
    │
    └── 系统事件 (bundleMonitor回调)
            └── 处理位置: MainAbility.onWindowStageCreate()
    │
    ▼
敏感操作
    │
    ├── 权限授予/撤销 (atManager.grantUserGrantedPermission())
    │       └── 调用位置: GrantDialogModel.grantPermissionWithResult()
    │
    ├── 应用信息查询 (bundleManager.getBundleInfo())
    │       └── 调用位置: MainAbility.getAllApplications()
    │
    ├── 窗口创建/销毁 (window.createWindow()/destroyWindow())
    │       └── 调用位置: GrantDialogModel.createWindow()
    │
    └── IPC回调 (proxy.sendMessageRequest())
            └── 调用位置: GrantDialogModel.terminateWithResult()
```

---

## 攻击面清单

### 1. 输入验证攻击面

| 攻击面 | 位置 | 风险等级 | 证据 |
|--------|------|----------|------|
| Want参数解析 | GrantDialogModel.ets:56-88 | 中 | 从want.parameters解析多个参数 |
| Intent参数传递 | ServiceExtAbility.ets:38-59 | 中 | onRequest接收want参数 |
| 剪贴板数据读取 | GrantDialogModel.ets:293-304 | 低 | getPasteBoardInfo()读取剪贴板 |

### 2. IPC通信攻击面

| 攻击面 | 位置 | 风险等级 | 证据 |
|--------|------|----------|------|
| RPC消息发送 | GrantDialogModel.ets:496-522 | 中 | sendMessageRequest可能失败 |
| IPC回调对象 | GrantDialogModel.ets:66-67 | 高 | 从want解析callback对象 |
| 窗口绑定Token | SecurityExtAbility.ets:88-89 | 中 | bindDialogTarget使用外部token |

### 3. 权限操作攻击面

| 攻击面 | 位置 | 风险等级 | 证据 |
|--------|------|----------|------|
| 权限授予 | GrantDialogModel.ets:426-443 | 高 | grantUserGrantedPermission |
| 权限撤销 | GrantDialogModel.ets:453-470 | 高 | revokeUserGrantedPermission |
| 权限查询 | MainAbility.ets:141-157 | 中 | verifyAccessTokenSync |

### 4. 窗口管理攻击面

| 攻击面 | 位置 | 风险等级 | 证据 |
|--------|------|----------|------|
| 窗口创建 | GrantDialogModel.ets:225-254 | 中 | createWindow可能失败 |
| 窗口重复检查 | SecurityExtAbility.ets:71-83 | 中 | dialogSet检查重复窗口 |
| 窗口销毁 | GrantDialogModel.ets:496-522 | 中 | destroyWindow资源释放 |

### 5. 数据存储攻击面

| 攻击面 | 位置 | 风险等级 | 证据 |
|--------|------|----------|------|
| 全局状态存储 | globalContext.ets | 低 | GlobalContext存储共享状态 |
| LocalStorage | 多处 | 低 | 页面间状态传递 |

---

## 可利用点详细分析

### 可利用点 1: IPC回调对象注入

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:66-67`

```typescript
let callback: Property = want.parameters['ohos.ability.params.callback'] as Property;
let proxy: rpc.RemoteObject = callback.value as rpc.RemoteObject;
```

**触发路径**:
1. 恶意应用尝试直接启动GrantAbility
2. 构造包含恶意callback的want参数
3. PermissionManager使用此callback发送IPC消息

**影响**:
- 可能导致IPC消息发送到错误的目标
- 可能泄露授权结果给恶意应用

**修复建议**:
```typescript
// 当前代码 (有风险)
let proxy: rpc.RemoteObject = callback.value as rpc.RemoteObject;

// 建议增加验证
let proxy: rpc.RemoteObject = callback.value as rpc.RemoteObject;
if (!proxy || !(proxy instanceof rpc.RemoteObject)) {
    Log.error('Invalid callback object');
    return DEFAULT_CALLER_APP_INFO;
}
```

**风险等级**: 🔴 高

---

### 可利用点 2: 权限操作结果未验证

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:426-443`

```typescript
private async grantPermissionWithResult(...): Promise<optionAndState> {
    try {
        await atManager.grantUserGrantedPermission(tokenId, permission, flag);
        Log.info(`grant permission success, permission: ${permission}.`);
        return {
            operationResult: Constants.RESULT_SUCCESS,
            permissionState: abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
        }
    } catch (error) {
        Log.error(`grant permission faild...`);
        return {
            operationResult: Constants.RESULT_FAILURE,
            permissionState: abilityAccessCtrl.GrantStatus.PERMISSION_DENIED
        }
    }
}
```

**触发路径**:
1. 用户点击"允许"按钮
2. clickHandle()调用grantPermissionWithResult()
3. 权限服务可能拒绝授权（如策略禁止）
4. 但UI层可能未正确处理失败情况

**影响**:
- UI显示与实际权限状态不一致
- 用户误以为已授权成功

**修复建议**:
```typescript
// 建议在UI层明确显示操作结果
const result = await this.grantPermissionWithResult(...);
if (result.operationResult !== Constants.RESULT_SUCCESS) {
    // 显示错误提示给用户
    this.showErrorDialog('授权失败，请重试');
}
```

**风险等级**: 🟡 中

---

### 可利用点 3: 窗口重复创建竞争条件

**证据**: `permissionmanager/src/main/ets/SecurityExtAbility/SecurityExtAbility.ets:71-83`

```typescript
let dialogSet: Set<String> = GlobalContext.load('dialogSet');
if (!dialogSet) {
    dialogSet = new Set<String>();
    GlobalContext.store('dialogSet', dialogSet);
}
let token: String = String(callerToken) + '_' + String(windId);
if (dialogSet.has(token)) {
    Log.info('window already exists.');
    return;
}
```

**触发路径**:
1. 两个并发请求同时到达onRequest()
2. 同时检查dialogSet，都发现不存在
3. 同时创建窗口，导致重复对话框

**影响**:
- 用户体验问题（重复弹窗）
- 可能导致状态混乱

**修复建议**:
```typescript
// 使用原子操作或锁机制
// 或使用系统级的窗口唯一性检查
async createWindow(name: string, ...): Promise<void> {
    // 尝试创建窗口，如果失败说明已存在
    try {
        const win = await window.createWindow({ ctx: this.context, name, windowType });
    } catch (err) {
        if (err.code === Constants.CREATE_WINDOW_REPEATED) {
            Log.info('window already exists');
            return;
        }
        throw err;
    }
}
```

**风险等级**: 🟡 中

---

### 可利用点 4: Want参数空值未充分处理

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:56-88`

```typescript
public async getCallerAppInfo(want: Want): Promise<CallerAppInfo> {
    if (!want.parameters) {
        Log.error(`want.parameters is undefined!`);
        return DEFAULT_CALLER_APP_INFO;
    }
    let bundleName: string = want.parameters['ohos.aafwk.param.callerBundleName'] as string ?? '';
    let tokenId: number = want.parameters['ohos.aafwk.param.callerToken'] as number ?? -1;
    // ...
}
```

**触发路径**:
1. want.parameters存在但缺少某些字段
2. 使用空值合并运算符提供默认值
3. 但默认值可能导致后续逻辑异常

**影响**:
- tokenId为-1可能导致权限操作失败
- bundleName为空可能导致应用信息显示异常

**修复建议**:
```typescript
public async getCallerAppInfo(want: Want): Promise<CallerAppInfo> {
    if (!want.parameters) {
        Log.error(`want.parameters is undefined!`);
        return DEFAULT_CALLER_APP_INFO;
    }
    
    let bundleName: string = want.parameters['ohos.aafwk.param.callerBundleName'] as string ?? '';
    let tokenId: number = want.parameters['ohos.aafwk.param.callerToken'] as number ?? -1;
    
    // 增加参数有效性验证
    if (!bundleName || bundleName === '') {
        Log.error('Invalid bundleName');
        return DEFAULT_CALLER_APP_INFO;
    }
    if (tokenId === -1) {
        Log.error('Invalid tokenId');
        return DEFAULT_CALLER_APP_INFO;
    }
    // ...
}
```

**风险等级**: 🟡 中

---

### 可利用点 5: 剪贴板数据泄露

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:293-304`

```typescript
public getPasteBoardInfo(): string {
    let systemPasteboardDataSource: string = '';
    try {
        let systemPasteboard: pasteboard.SystemPasteboard = pasteboard.getSystemPasteboard();
        let data = systemPasteboard.getDataSource();
        systemPasteboardDataSource = data || '';
    } catch (error) {
        Log.error(`getSystemPasteboard faild...`);
    }
    Log.info(`systemPasteboard dataSource: ${systemPasteboardDataSource}.`);
    return systemPasteboardDataSource;
}
```

**触发路径**:
1. 剪贴板中包含敏感信息
2. PermissionManager读取并记录到日志
3. 日志可能被未授权访问

**影响**:
- 敏感信息泄露（密码、个人信息等）

**修复建议**:
```typescript
public getPasteBoardInfo(): string {
    try {
        let systemPasteboard: pasteboard.SystemPasteboard = pasteboard.getSystemPasteboard();
        let data = systemPasteboard.getDataSource();
        // 不记录敏感内容到日志
        Log.info('Got pasteboard data source');  // 不打印具体内容
        return data || '';
    } catch (error) {
        Log.error(`getSystemPasteboard failed...`);
        return '';
    }
}
```

**风险等级**: 🟡 中

---

### 可利用点 6: IPC资源未释放

**证据**: `permissionmanager/src/main/ets/ServiceExtAbility/GrantDialogModel.ets:496-522`

```typescript
public async terminateWithResult(...): Promise<void> {
    let option = new rpc.MessageOption();
    let data = new rpc.MessageSequence();
    let reply = new rpc.MessageSequence();
    let setDialogData = new rpc.MessageSequence();
    try {
        data.writeInterfaceToken(Constants.ACCESS_TOKEN);
        // ...
        callerAppInfo.proxy.sendMessageRequest(Constants.RESULT_CODE, data, reply, option);
    } catch (error) {
        Log.error(`terminateWithResult faild...`);
    } finally {
        data.reclaim();
        reply.reclaim();
        setDialogData.reclaim();
        // ...
    }
}
```

**触发路径**:
1. sendMessageRequest抛出异常
2. 进入catch块
3. finally块释放资源
4. **但setDialogData可能未初始化就被释放**

**影响**:
- 可能导致资源泄漏或崩溃

**修复建议**:
```typescript
public async terminateWithResult(...): Promise<void> {
    let option = new rpc.MessageOption();
    let data: rpc.MessageSequence | null = null;
    let reply: rpc.MessageSequence | null = null;
    let setDialogData: rpc.MessageSequence | null = null;
    
    try {
        data = new rpc.MessageSequence();
        reply = new rpc.MessageSequence();
        setDialogData = new rpc.MessageSequence();
        
        data.writeInterfaceToken(Constants.ACCESS_TOKEN);
        // ...
        callerAppInfo.proxy.sendMessageRequest(Constants.RESULT_CODE, data, reply, option);
    } catch (error) {
        Log.error(`terminateWithResult faild...`);
    } finally {
        // 安全释放
        data?.reclaim();
        reply?.reclaim();
        setDialogData?.reclaim();
        // ...
    }
}
```

**风险等级**: 🟢 低

---

## 安全检查清单

### 已检查项

- [x] Want参数解析和验证
- [x] IPC通信安全
- [x] 权限操作安全性
- [x] 窗口管理安全性
- [x] 资源释放检查
- [x] 日志敏感信息泄露

### 未涉及项

- [ ] 网络通信（本项目无网络功能）
- [ ] 文件系统访问（除系统API外）
- [ ] 数据库操作（无数据库访问）
- [ ] 加密算法使用

### 局限性

1. **代码范围**: 仅分析主模块代码（permissionmanager/src/main/ets/），不包括测试代码
2. **动态分析**: 本评审基于静态代码分析，未进行动态测试
3. **依赖库**: 未深入分析系统服务层的安全实现
4. **攻击场景**: 主要考虑已知攻击向量，可能存在未覆盖的攻击方式

---

## 修复建议汇总

| 优先级 | 问题 | 位置 | 建议 |
|--------|------|------|------|
| 🔴 高 | IPC回调对象验证缺失 | GrantDialogModel.ets:66-67 | 增加类型检查 |
| 🟡 中 | 权限操作结果处理 | dialogPlus.ets | UI层显示操作结果 |
| 🟡 中 | 窗口竞争条件 | SecurityExtAbility.ets:71-83 | 使用原子操作 |
| 🟡 中 | Want参数验证 | GrantDialogModel.ets:56-88 | 增加有效性检查 |
| 🟡 中 | 剪贴板日志泄露 | GrantDialogModel.ets:293-304 | 移除敏感日志 |
| 🟢 低 | IPC资源释放 | GrantDialogModel.ets:496-522 | 使用可选链释放 |

---

## 信任边界验证

```
┌─────────────────────────────────────────────────────────────────────┐
│                     信任边界验证点                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. AT服务验证调用者身份                                             │
│     ✓ 由系统服务处理，PermissionManager信任传入参数                  │
│                                                                     │
│  2. PermissionManager内部验证                                        │
│     △ Want参数存在性检查 (部分完善)                                  │
│     ✗ IPC回调对象类型验证 (缺失)                                     │
│     ✓ 窗口重复创建检查                                               │
│                                                                     │
│  3. 权限操作前的最终检查                                             │
│     ✓ 由AT服务再次验证                                               │
│     ✓ PermissionManager传递正确tokenId                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 总结

### 整体安全评估

| 评估项 | 评级 | 说明 |
|--------|------|------|
| 输入验证 | 🟡 中 | 基本验证存在，但部分参数验证不完善 |
| IPC安全 | 🟡 中 | 有异常处理，但对象验证不足 |
| 权限操作 | 🟢 低 | 依赖底层服务，本身逻辑较安全 |
| 资源管理 | 🟡 中 | 基本良好，有少量改进空间 |
| 信息泄露 | 🟡 中 | 剪贴板信息可能泄露到日志 |

### 主要风险

1. **IPC回调对象注入** - 高风险，需要立即修复
2. **参数验证不完善** - 中风险，建议修复
3. **竞争条件** - 中风险，特定场景下可能触发

### 安全建议

1. 增加IPC对象的类型验证
2. 完善Want参数的有效性检查
3. 修复日志敏感信息泄露
4. 考虑增加窗口创建的原子性保护
5. 定期进行安全审计和渗透测试

---

*上一篇: [构建系统](05_Build_System.md) | 返回 [README](README.md) | 下一篇: [常见问题排查](07_Troubleshooting.md)*
