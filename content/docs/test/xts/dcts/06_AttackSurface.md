# DCTS 攻击面分析

## 文档目的

本文档从安全研究员视角分析 DCTS 测试套件的攻击面，帮助理解外部输入入口、信任边界、敏感操作和潜在利用路径。

**受众**: 安全研究员、渗透测试工程师、安全审计人员

---

## 威胁模型概述

### 系统边界

```
┌─────────────────────────────────────────────────────┐
│              DCTS 测试环境 (Test Suite)           │
│  ┌───────────┐  ┌───────────┐  ┌───────────────┐  │
│  │ 测试用例   │  │ 测试工具   │  │ OpenHarmony  │  │
│  │ (受限)    │  │ (受限)    │  │ 系统 API     │  │
│  └───────────┘  └───────────┘  └───────────────┘  │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│              外部攻击面 (Attack Surface)           │
│  - 网络 (WiFi, Bluetooth, SoftBus)                │
│  - 文件系统 (分布式文件系统)                      │
│  - 用户输入 (测试参数、配置文件)                │
│  - 设备间通信 (RPC、IPC)                        │
└─────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 描述 | 信任级别 | 安全机制 |
|------|------|----------|----------|
| **测试用例** | 测试代码运行环境 | 中 | 测试框架隔离 |
| **测试工具** | 辅助测试组件 | 中 | 进程沙箱 |
| **OpenHarmony API** | 被测系统 API | 高 | 权限检查、Token 验证 |
| **分布式通信** | 设备间通信通道 | 中 | 设备认证、会话密钥 |
| **文件系统** | 测试数据存储 | 低 | 安全标签、权限控制 |

---

## 攻击面清单

### 1. 分布式通信攻击面

#### 1.1 RPC 远程调用

**证据**: `communication/dsoftbus_rpcets/rpcserver/entry/src/main/ets/serviceability/ServiceAbility.ts:40-70`

```typescript
class Stub extends rpc.RemoteObject {
    onRemoteMessageRequest(code: number, data: rpc.MessageSequence, reply: rpc.MessageSequence, option: rpc.MessageOption) {
        console.info(logTag + "onRemoteMessageRequest: " + code);
        switch(code) {
            case 1:
                let listener:any = data.readRemoteObject();
                let num:number = data.readInt();
                let str:string = data.readString();
                // 直接处理远程数据，无调用方身份验证
                listener.sendRequest(1, data2, reply2, option2)
        }
    }
}
```

**攻击面分析**:

| 入口 | 证据位置 | 数据类型 | 风险 |
|------|-----------|----------|------|
| **RPC 消息代码** | `ServiceAbility.ts:43` | `number` | 未验证调用方身份，可被伪造 |
| **RPC 数据参数** | `ServiceAbility.ts:46-48` | `RemoteObject`, `number`, `string` | 无输入验证，可注入恶意数据 |
| **RPC 回调** | `ServiceAbility.ts:51` | `RemoteObject` 回调 | 可能被劫持或伪造 |

**利用路径**:

```
攻击者设备                目标设备
    │                              │
    ├── writeInterfaceToken() ─────►┼── onRemoteMessageRequest()
    │       (token 可伪造)            │
    │                              │
    └── sendRequest() ─────────────┘
        (未验证调用方)
```

**风险等级**: **高**

#### 1.2 设备管理器通信

**证据**: `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:48-95`

```typescript
let dmInstance = distributedDeviceManager.createDeviceManager(TEST_BUNDLE_NAME);
expect(dmInstance !== null && dmInstance !== undefined).assertTrue();

const deviceId: String = dmInstance.getLocalDeviceId();
const dmNetworkId: String = dmInstance.getLocalDeviceNetworkId();
const deviceInfoList: Array<distributedDeviceManager.DeviceBasicInfo> = dmInstance.getAvailableDeviceListSync();
```

**攻击面分析**:

| 入口 | 证据位置 | 数据类型 | 风险 |
|------|-----------|----------|------|
| **设备 ID** | `DeviceManagerAPI.test.ets:88` | `String` | 敏感标识符，可在日志中泄露 |
| **网络 ID** | `DeviceManagerAPI.test.ets:89` | `String` | 网络拓扑信息 |
| **设备信息列表** | `DeviceManagerAPI.test.ets:91` | `DeviceBasicInfo[]` | 设备元数据泄露 |

**利用路径**:

```
1. 获取设备列表 → 2. 提取设备ID/网络ID → 3. 记录到日志 → 4. 泄露网络拓扑
```

**风险等级**: **中**

#### 1.3 能力连接管理

**证据**: `ability/dmsfwk/dmsfwkstagetest/entry/src/ohosTest/ets/test/DmsFwkStageTest.ets:2764-2837`

```typescript
let sessionId = 0;
let textEncoder = util.TextEncoder.create("utf-8");
let arrayBuffer = textEncoder.encodeInto("data send success");

abilityConnectionManager.sendData(sessionId, arrayBuffer.buffer).then(() => {
    console.log(TAG + 'abilityConnectionManager.sendData is success')
}).catch((err: BusinessError) => {
    console.log(TAG + 'abilityConnectionManager.sendData is failed' + err.code)
})
```

**攻击面分析**:

| 入口 | 证据位置 | 数据类型 | 风险 |
|------|-----------|----------|------|
| **会话 ID** | `DmsFwkStageTest.ets:2765` | `number` | 未验证会话归属 |
| **二进制数据** | `DmsFwkStageTest.ets:2767` | `ArrayBuffer` | 无加密，明文传输 |
| **错误码** | `DmsFwkStageTest.ets:2774` | `number` | 错误信息泄露 |

**风险等级**: **中**

---

### 2. 数据存储攻击面

#### 2.1 分布式文件系统

**证据**: `filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:53-77`

```javascript
async function getDistributedFilePath(testName) {
    let basePath;
    try {
        let context = featureAbility.getContext();
        basePath = await context.getOrCreateDistributedDir();
    } catch (e) {
        console.log("-------- getDistributedFilePath() failed for : " + e);
    }
    return basePath + "/" + testName;  // 路径拼接，未规范化
}
```

**攻击面分析**:

| 入口 | 证据位置 | 数据类型 | 风险 |
|------|-----------|----------|------|
| **文件路径** | `FileioJsUnit.test.js:61` | `String` | **路径遍历** - `basePath + "/" + testName` |
| **文件名** | `FileioJsUnit.test.js:61` | `String` | 来自测试参数，未验证 |
| **文件内容** | `FileioJsUnit.test.js:74` | `String` | 固定测试内容 |

**利用路径**:

```
攻击者输入: "../../../etc/passwd"
拼接结果: "/data/hmdfs/../../../etc/passwd"
        = "/etc/passwd"  ← 路径遍历成功！
```

**风险等级**: **高** (虽然 testName 通常是硬编码，但未来扩展存在风险)

#### 2.2 文件安全标签

**证据**: `filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:73, 3708, 3745, 3783, 3820, 3861`

```javascript
// 所有文件都使用 s0（公开）安全级别
securityLabel.setSecurityLabelSync(fpath, "s0");
```

**攻击面分析**:

| 安全级别 | 权限 | 文件位置 |
|-----------|------|----------|
| **s0** | 任何应用可访问 | `/mnt/hmdfs/...` |
| **s1** | 系统应用可访问 | 测试代码中未使用 |
| **s2** | 高安全级别 | 测试代码中未使用 |
| **s3** | 最高安全级别 | 测试代码中未使用 |
| **s4** | 设备专用 | 测试代码中未使用 |

**问题**: 所有测试文件使用最低安全级别 `s0`，任何有权限的应用都可访问分布式文件系统中的测试数据。

**风险等级**: **中**

#### 2.3 KV 存储与 RDB 数据库

**证据**: `distributeddatamgr/jstest/distributed_kv_store/client/hap/entry/src/ohosTest/js/test/KvStoreSecurityLevelS1Jsunit.test.js:172-177`

```javascript
async kvPut(key,value,valueType){
    let putValue = undefined;
    if(valueType == "String_MAX_VALUE_LENGTH"){
        let maxValueLength = 4194303;  // 4MB - 1，边界值
        console.info(logTag + 'maxValueLength = ' + maxValueLength);
        let maxValueLengthString = "v".repeat(maxValueLength);
        putValue = maxValueLengthString;
    }else if(valueType == "Number_Min"){
        putValue = Number.MIN_VALUE;  // JavaScript 最小负数
    }
}
```

**攻击面分析**:

| 入口 | 证据位置 | 数据类型 | 风险 |
|------|-----------|----------|------|
| **键名** | `KvStoreSecurityLevelS1Jsunit.test.js` | `String` | 未验证键名格式 |
| **值内容** | `KvStoreSecurityLevelS1Jsunit.test.js:178` | `String` | 未限制特殊字符 |
| **边界值** | `KvStoreSecurityLevelS1Jsunit.test.js:177` | `Number` | 最小值测试，可能触发异常 |

**风险等级**: **低** (测试代码生成边界值，但实际应用可能不处理)

---

### 3. 权限系统攻击面

#### 3.1 权限申请

**证据**: `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:48-55`

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import { Permissions, PermissionRequestResult } from 'permissions';

let atManager: abilityAccessCtrl.AtManager = abilityAccessCtrl.createAtManager();
let permissions: Array<Permissions> = ['ohos.permission.DISTRIBUTED_DATASYNC'];

atManager.requestPermissionsFromUser(gContext, permissions,
  (err: BusinessError<void> | null, data: PermissionRequestResult | undefined) => {
    hilog.info(domain, tag, "====>request success permissions" + JSON.stringify(data));
    hilog.info(domain, tag, "====>getPermissionRequestResult err" + JSON.stringify(err));
  });
```

**攻击面分析**:

| 权限 | 危险级别 | 影响 |
|--------|---------|------|
| `DISTRIBUTED_DATASYNC` | **高** | 跨设备数据同步，可窃取敏感数据 |
| `DISTRIBUTED_FILE` | 中 | 跨设备文件访问 |
| `GET_NETWORK_INFO` | 中 | 网络信息泄露 |

**利用路径**:

```
1. 恶意应用申请 DISTRIBUTED_DATASYNC
2. 用户授权
3. 应用监听分布式数据变化
4. 窃取其他设备的敏感数据
```

**风险等级**: **高** (依赖用户授权)

#### 3.2 Token 管理

**证据**: `distributedhardware/distributedscreentest/test.cpp:267-320`

```cpp
#include "accesstoken_kit.h"
#include "nativetoken_kit.h"

using namespace OHOS::Security::AccessToken;

int QueryRemoteDeviceInfo(int mode) {
    uint64_t tokenId;
    const char *perms[2];
    perms[0] = OHOS_PERMISSION_DISTRIBUTED_SOFTBUS_CENTER;
    perms[1] = OHOS_PERMISSION_DISTRIBUTED_DATASYNC;
    
    NativeTokenInfoParams infoInstance = {
        .dcapsNum = 0,
        .permsNum = 2,
        .aclsNum = 0,
        .dcaps = NULL,
        .perms = perms,
        .acls = NULL,
        .processName = "dscreen_test_demo",
        .aplStr = "system_core",
    };
    
    tokenId = GetAccessTokenId(&infoInstance);
    SetSelfTokenID(tokenId);
    OHOS::Security::AccessToken::AccessTokenKit::ReloadNativeTokenInfo();
}
```

**攻击面分析**:

| 入口 | 证据位置 | 数据类型 | 风险 |
|------|-----------|----------|------|
| **Token ID** | `distributedscreentest/test.cpp:296` | `uint64_t` | 可能被截获或重放 |
| **权限列表** | `distributedscreentest/test.cpp:288-289` | `char*[]` | 硬编码权限 |
| **进程名称** | `distributedscreentest/test.cpp:303` | `char*` | 可伪造进程身份 |

**风险等级**: **中**

---

### 4. 输入验证攻击面

#### 4.1 RPC 边界值处理

**证据**: `communication/dsoftbus_rpcets/rpcclient/entry/src/ohosTest/ets/test/RpcRequestEtsUnit.test.ets:2117-2200`

```typescript
/*
 * @tc.name    : test Writebyte interface, boundary value verification
 */
it("SUB_DSoftbus_RPC_API_NEW_MessageSequence_0520", async (done: () => void) => {
  let data: rpc.MessageSequence = rpc.MessageSequence.create();
  try{
    data.writeByte(128);  // 超过有符号 byte 最大值 127
    data.writeByte(0);
    data.writeByte(1);
    data.writeByte(2);
    data.writeByte(127);  // 最大边界值
    expect(reply.readByte()).assertEqual(-128);  // 溢出回绕
    expect(reply.readByte()).assertEqual(0);
  }
});
```

**攻击面分析**:

| 测试 | 正常范围 | 边界值 | 测试的溢出行为 |
|------|---------|--------|--------------|
| **writeByte** | -128 到 127 | 128, -129 | 回绕到 127, -128 |
| **writeShort** | -32768 到 32767 | 32768, -32769 | 回绕到 32767, -32768 |

**利用路径**:

```
正常情况: 0-127
写入 128 → 溢出 → -128 (回绕)
写入 129 → 溢出 → 127 (回绕)

如果应用未正确验证边界值：
- 可能导致整数溢出
- 可能触发未定义行为
- 可能绕过安全检查
```

**风险等级**: **中** (这是测试用例，验证被测系统的边界处理)

#### 4.2 Null/Undefined 检查

**证据**: `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:82-88`

```typescript
try {
  let dmInstance = distributedDeviceManager.createDeviceManager(TEST_BUNDLE_NAME);
  expect(dmInstance !== null && dmInstance !== undefined).assertTrue();
  const deviceId: String = dmInstance.getLocalDeviceId();
  if (deviceId === null) {
    console.log("getLocalDeviceId fail");
  }
  expect(deviceId !== null).assertTrue();
}
```

**攻击面分析**:

| 检查类型 | 频率 | 风险 |
|---------|------|------|
| **null 检查** | 高频 | 空指针解引用 |
| **undefined 检查** | 高频 | 属性访问异常 |
| **双重检查** | 中频 | null && undefined 重复检查 |

**问题**: 测试代码中有大量 null/undefined 检查，但被测系统的验证逻辑不可见。

**风险等级**: **低** (这是测试代码的防御性编程)

---

### 5. 敏感信息泄露攻击面

#### 5.1 日志泄露

**证据**: `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:95`

```typescript
hilog.info(domain, tag, `Local Device Id: ${deviceId}`);
hilog.info(domain, tag, 'getLocalDeviceNetworkId: %{public}s', dmNetworkId);
hilog.info(domain, tag, 'Device Name: %{public}s', deviceName);
```

**攻击面分析**:

| 泄露信息 | 证据位置 | 敏感度 |
|-----------|-----------|--------|
| **设备 ID** | `DeviceManagerAPI.test.ets:95` | **高** |
| **网络 ID** | `DeviceManagerAPI.test.ets:96` | **高** |
| **设备名称** | `DeviceManagerAPI.test.ets:97` | **中** |
| **设备列表 JSON** | 多处 | **高** |

**利用路径**:

```
1. 读取测试日志 (hilog, console.log)
2. 提取设备ID/网络ID
3. 映射网络拓扑
4. 选择目标设备进行后续攻击
```

**风险等级**: **高** (测试环境通常日志级别较高)

#### 5.2 错误详情泄露

**证据**: `testtools/disetsTest/client/testService.ets:88-93`

```typescript
catch (err) {
    console.info(logTag + 'get deviceManager is failed' + JSON.stringify(error))
    const e = err as BusinessError;
    hilog.info(domain, tag, `***** connectServiceExtensionAbility failed. err code is ${code}, message is ${message}`);
}
```

**攻击面分析**:

| 泄露内容 | 证据位置 | 敏感度 |
|-----------|-----------|--------|
| **错误对象** | `testService.ets:89` | **中** |
| **错误码** | `testService.ets:92` | **高** |
| **错误消息** | `testService.ets:92` | **中** |
| **堆栈信息** | 可能 | **高** |

**风险等级**: **中**

---

## 攻击面总结

### 高风险点 (需优先关注)

| 编号 | 攻击面 | 风险等级 | 关键证据 |
|------|--------|---------|-----------|
| **AS1** | RPC 调用方身份验证缺失 | 高 | `ServiceAbility.ts:43-70` |
| **AS2** | 文件路径拼接未规范化 | 高 | `FileioJsUnit.test.js:61` |
| **AS3** | 日志中的设备信息泄露 | 高 | `DeviceManagerAPI.test.ets:95-97` |
| **AS4** | DISTRIBUTED_DATASYNC 权限滥用 | 高 | `DeviceManagerAPI.test.ets:48-55` |
| **AS5** | AbilityConnection 数据未加密 | 中 | `DmsFwkStageTest.ets:2764-2774` |

### 中风险点 (需关注)

| 编号 | 攻击面 | 风险等级 | 关键证据 |
|------|--------|---------|-----------|
| **AM1** | 文件安全级别过低 (s0) | 中 | `FileioJsUnit.test.js:73,3708,...` |
| **AM2** | Token 可能被截获 | 中 | `distributedscreentest/test.cpp:296` |
| **AM3** | KV/RDB 边界值处理 | 中 | `KvStoreSecurityLevelS1Jsunit.test.js:172-177` |
| **AM4** | 错误详情泄露 | 中 | `testService.ets:88-93` |
| **AM5** | 设备信息泄露 | 中 | `DeviceManagerAPI.test.ets:82-95` |

### 低风险点 (可选优化)

| 编号 | 攻击面 | 风险等级 | 关键证据 |
|------|--------|---------|-----------|
| **AL1** | Null/Undefined 检查覆盖不全 | 低 | `DeviceManagerAPI.test.ets:82-88` |
| **AL2** | RPC 边界值测试用例 | 低 | `RpcRequestEtsUnit.test.ets:2117-2200` |
| **AL3** | 设备绑定流程未展示 | 低 | `DeviceManager.test.ets:...` |

---

## 利用路径示例

### 示例 1: RPC 身份伪造攻击

**场景**: 攻击者伪造设备调用 RPC 服务

```
步骤 1: 伪造设备身份
  - 获取目标设备的 bundleName
  - 伪造 InterfaceToken (如果 token 验证机制薄弱)

步骤 2: 发起恶意 RPC 调用
  - 构造恶意 MessageSequence
  - 注入边界值或恶意数据

步骤 3: 绕过安全检查
  - 如果未验证调用方 UID/PID
  - 如果 token 验证仅基于字符串匹配

步骤 4: 执行恶意操作
  - 读取敏感数据
  - 修改关键配置
  - 触发拒绝服务
```

**关键代码位置**: `communication/dsoftbus_rpcets/rpcserver/entry/src/main/ets/serviceability/ServiceAbility.ts:40-70`

### 示例 2: 路径遍历攻击

**场景**: 通过文件路径拼接访问任意文件

```
步骤 1: 获取分布式目录基路径
  - context.getOrCreateDistributedDir()
  - 返回: /mnt/hmdfs/100/account/merge_view/data/com.acts.fileio.test.server

步骤 2: 构造恶意文件名
  - testName = "../../../../../../../../etc/passwd"

步骤 3: 路径拼接 (无规范化)
  - result = basePath + "/" + testName
  - = /mnt/hmdfs/100/account/merge_view/data/com.acts.fileio.test.server/../../../../../../../../etc/passwd
  - = /etc/passwd  ← 成功遍历！

步骤 4: 读取或修改敏感文件
  - fs.readSync(path)
  - fs.writeSync(path, malicious_content)
```

**关键代码位置**: `filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:53-61`

### 示例 3: 日志信息泄露

**场景**: 从测试日志中提取设备拓扑信息

```
步骤 1: 获取测试设备或模拟器日志
  - hilog 读取
  - console.log 输出

步骤 2: 提取设备信息
  - deviceId: "OH123456789012345"
  - networkId: "OH123456789012345"
  - deviceName: "Test Device"

步骤 3: 映射网络拓扑
  - 识别所有设备及其关系
  - 识别信任链路

步骤 4: 选择攻击目标
  - 选择特权设备
  - 选择包含敏感数据的设备
```

**关键代码位置**: `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:95-97`

---

## 防护建议

### 短期修复建议 (高优先级)

1. **RPC 身份验证增强**
   - 在 `onRemoteMessageRequest` 中验证调用方 UID/PID
   - 使用加密的 token 而非纯字符串
   - 限制调用频率和来源

2. **路径规范化**
   - 在 `getDistributedFilePath` 中使用 `path.normalize()` 或类似函数
   - 验证最终路径是否在允许的目录内
   - 禁止 `../` 和绝对路径

3. **日志脱敏**
   - 移除设备 ID、网络 ID 等敏感信息
   - 对错误对象使用脱敏函数
   - 限制日志级别，生产环境使用 ERROR 级别

4. **权限最小化**
   - 评估 `DISTRIBUTED_DATASYNC` 权限的必要性
   - 考虑使用细粒度权限
   - 添加权限使用审计日志

### 中期改进建议 (中优先级)

5. **文件安全级别提升**
   - 根据数据敏感度使用 s1-s4 级别
   - 在测试中使用与生产一致的安全级别
   - 添加安全级别转换机制

6. **输入验证增强**
   - 在所有外部输入处添加验证
   - 使用类型安全的序列化/反序列化
   - 添加长度和格式限制

7. **错误处理标准化**
   - 建立统一的错误处理策略
   - 避免静默吞没错误
   - 确保资源正确释放

### 长期架构建议 (低优先级)

8. **安全测试框架**
   - 添加安全测试用例
   - 集成模糊测试
   - 添加渗透测试脚本

9. **安全监控**
   - 建立异常行为监控
   - 添加安全事件日志
   - 集成入侵检测系统

10. **安全审计**
    - 定期代码安全审计
    - 依赖漏洞扫描
    - 配置安全审查

---

## 相关文档

- [安全风险评估](06_Security_Review.md) - 详细风险分析和修复建议
- [项目概览](01_Overview.md) - 项目架构和运行环境
- [模块详解](04_Modules.md) - 各模块的 API 和功能
- [测试框架](appendix/Test_Frameworks.md) - 测试框架和安全测试方法
