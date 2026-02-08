# DCTS 安全风险评估

## 评审范围

本评审覆盖 `test/xts/dcts` 仓库中的所有测试模块：

| 模块 | 路径 | 评审状态 | 关键风险 |
|------|------|----------|----------|
| ability | `ability/` | 已评审 | RPC 身份验证、权限滥用 |
| communication | `communication/` | 已评审 | RPC 边界值、会话安全 |
| distributeddatamgr | `distributeddatamgr/` | 已评审 | KV/RDB 安全级别、数据同步 |
| distributedhardware | `distributedhardware/` | 已评审 | 设备认证、Token 管理 |
| filemanagement | `filemanagement/` | 已评审 | 路径遍历、文件安全标签 |
| multimedia | `multimedia/` | 已评审 | 会话管理、媒体控制 |
| testtools | `testtools/` | 已评审 | 错误处理、日志泄露 |
| common | `common/` | 已评审 | 共享内存安全 |

**评审日期**: 2026-02-07
**评审方法**: 代码审计 + 证据收集 + 风险分析

---

## 威胁模型概述

### 系统边界

```
┌─────────────────────────────────────────────────────┐
│              DCTS 测试环境                          │
│  ┌───────────┐  ┌───────────┐  ┌─────────────────────┐    │
│  │ 测试用例   │  │ 测试工具   │  │ OpenHarmony 系统    │    │
│  │ (受限)    │  │ (受限)    │  │ (被测)              │    │
│  └───────────┘  └───────────┘  └─────────────────────┘    │
└─────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│                    外部攻击面                                │
│  - 网络 (WiFi, Bluetooth, SoftBus)                                   │
│  - 设备间通信 (软总线、RPC)                                       │
│  - 文件系统 (分布式文件系统)                                          │
│  - 用户输入 (测试参数、配置文件)                                      │
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

## 安全风险分析

### 风险 1: RPC 调用方身份验证缺失

**位置**: `communication/dsoftbus_rpcets/rpcserver/entry/src/main/ets/serviceability/ServiceAbility.ts:40-70`

**证据**:
```typescript
class Stub extends rpc.RemoteObject {
    constructor(descriptor) {
        super(descriptor);
    }
    onRemoteMessageRequest(code: number, data: rpc.MessageSequence, reply: rpc.MessageSequence, option: rpc.MessageOption) {
        try{
            console.info(logTag + "onRemoteMessageRequest: " + code);
            switch(code) {
                case 1:
                    {
                        console.info(logTag + "case 1 start");
                        let listener:any = data.readRemoteObject();
                        let num:number = data.readInt();
                        let str:string = data.readString();
                        console.info(logTag + "case 1 num is " + num);
                        console.info(logTag + "case 1 str is " + str);
                        // 问题：未验证调用方身份
                        let data2:any = rpc.MessageParcel.create();
                        let reply2:any = rpc.MessageParcel.create();
                        let option2:any = new rpc.MessageOption();
                        data2.writeInt(num);
                        data2.writeString(str);
                        console.info(logTag + "case 1 start sendRequest");
                        // 问题：未验证 token 或调用方 UID/PID
                        listener.sendRequest(1, data2, reply2, option2)
```

**触发路径**:
```
攻击者设备                RPC 服务端
    │                              │
    ├── writeInterfaceToken() ─────►┼── onRemoteMessageRequest()
    │       (token 可伪造)            │
    │                              │
    │       读取数据               │
    │       处理请求               │
    │       返回结果               │
    └── readInterfaceToken() ◄───┘
        (未验证调用方)
```

**影响评估**:
- **可利用性**: 中（需要伪造 InterfaceToken 或绕过验证）
- **权限提升可能**: 中（如果服务端有特权操作）
- **影响范围**: 所有 RPC 服务端测试用例

**修复建议**:
```typescript
// 建议修复：在 onRemoteMessageRequest 中添加调用方验证
onRemoteMessageRequest(code: number, data: rpc.MessageSequence, reply: rpc.MessageSequence, option: rpc.MessageOption) {
    // 1. 验证 InterfaceToken
    let descriptor: string = data.readInterfaceToken();
    if (descriptor !== "TestAbilityStub") {
        console.error(logTag + "invalid descriptor: " + descriptor);
        return false;
    }

    // 2. 验证调用方 UID/PID（如果系统支持）
    let callerUid: number = getCallingUid(); // 系统API
    let allowedUids: number[] = [12345, 67890]; // 白名单
    if (!allowedUids.includes(callerUid)) {
        console.error(logTag + "unauthorized caller: " + callerUid);
        return false;
    }

    // 3. 处理请求
    // ...
}
```

---

### 风险 2: 文件路径拼接未规范化

**位置**: `filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:53-77`

**证据**:
```javascript
async function getDistributedFilePath(testName) {
    let basePath;
    try {
        let context = featureAbility.getContext();
        basePath = await context.getOrCreateDistributedDir();
    } catch (e) {
        console.log("-------- getDistributedFilePath() failed for : " + e);
    }
    // 问题：直接拼接，未规范化路径
    return basePath + "/" + testName;
}
```

**触发路径**:
```
用户输入
    │
    ├── testName = "test_file"
    │
    └── getDistributedFilePath()
           │
           ▼
    basePath + "/" + testName
           │
           └── 路径遍历风险！
           如果 testName = "../../../../../../etc/passwd"
```

**影响评估**:
- **可利用性**: 中（需要控制 testName 参数）
- **权限提升可能**: 高（可读取任意文件）
- **影响范围**: 所有分布式文件操作测试用例

**修复建议**:
```javascript
// 建议修复：使用路径规范化
const path = require('path');

async function getDistributedFilePath(testName) {
    let basePath;
    try {
        let context = featureAbility.getContext();
        basePath = await context.getOrCreateDistributedDir();
    } catch (e) {
        console.log("-------- getDistributedFilePath() failed for : " + e);
    }

    // 1. 规范化输入
    let safeName = testName.replace(/\.\./g, '').replace(/\.\.\g, '');

    // 2. 规范化路径
    let fullPath = path.join(basePath, safeName);

    // 3. 解析规范化路径（消除 ../）
    let normalizedPath = path.normalize(fullPath);

    // 4. 验证路径是否在允许目录内
    if (!normalizedPath.startsWith(basePath)) {
        throw new Error("Path traversal attempt detected");
    }

    return normalizedPath;
}
```

---

### 风险 3: 文件安全级别过低

**位置**: `filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:73, 3708, 3745, 3783, 3820, 3861`

**证据**:
```javascript
async function prepareFile(fpath) {
    try {
        let file = fs.openSync(fpath, fs.OpenMode.CREATE | fs.OpenMode.READ_WRITE);
        fs.truncateSync(file.fd);
        // 问题：所有测试文件使用 s0（公开）安全级别
        securityLabel.setSecurityLabelSync(fpath, "s0");
        fs.writeSync(file.fd, DISTRIBUTED_FILE_CONTENT);
        fs.fsyncSync(file.fd);
        fs.closeSync(file);
        return true;
    } catch (e) {
        console.log('Failed to prepareFile for ' + e)
        return false
    }
}
```

**触发路径**:
```
测试应用
    │
    ├── 创建测试文件
    │      │
    │      └── setSecurityLabelSync(fpath, "s0")
    │              (公开级别)
    │
    └── 任何有权限的应用都可读取
           │
           └── 数据泄露风险
```

**影响评估**:
- **可利用性**: 低（需要其他应用有权限）
- **权限提升可能**: 低（已有权限，只是数据泄露）
- **影响范围**: 所有分布式文件测试数据

**安全级别说明**:
| 级别 | 名称 | 权限 | 适用场景 |
|------|------|------|----------|
| S0 | 公开 | 所有应用 | 公开数据、临时文件 |
| S1 | 系统应用 | 系统应用 | 系统配置 |
| S2 | 加密 | 授权应用 | 加密敏感数据 |
| S3 | 高敏感 | 高敏感应用 | 个人隐私数据 |
| S4 | 设备专用 | 设备专属应用 | 设备配置 |

**修复建议**:
```javascript
// 建议修复：根据数据敏感度使用适当的安全级别
async function prepareSecureFile(fpath, data, sensitivity) {
    try {
        let file = fs.openSync(fpath, fs.OpenMode.CREATE | fs.OpenMode.READ_WRITE);
        fs.truncateSync(file.fd);

        // 根据敏感度选择安全级别
        let securityLevel;
        if (sensitivity === "public") {
            securityLevel = "s0";
        } else if (sensitivity === "system") {
            securityLevel = "s1";
        } else if (sensitivity === "private") {
            securityLevel = "s3"; // 使用高敏感级别
        } else {
            securityLevel = "s4"; // 设备专用
        }

        securityLabel.setSecurityLabelSync(fpath, securityLevel);
        fs.writeSync(file.fd, data);
        fs.fsyncSync(file.fd);
        fs.closeSync(file);
        return true;
    } catch (e) {
        console.log('Failed to prepareSecureFile for ' + e)
        return false
    }
}
```

---

### 风险 4: 设备信息日志泄露

**位置**: `distributedhardware/devicemanagerteststatic/entry/src/main/src/test/DeviceManagerAPI.test.ets:95-97`

**证据**:
```typescript
it('SUB_DH_DeviceManager_Dcts_0100',
  TestType.FUNCTION | Size.MEDIUMTEST | Level.LEVEL0,
  async (): Promise<void> => {
    hilog.info(domain, tag, '-----------------SUB_DH_DeviceManager_Dcts_0100 START------------------------');
    try {
      let dmInstance = distributedDeviceManager.createDeviceManager(TEST_BUNDLE_NAME);
      expect(dmInstance !== null && dmInstance !== undefined).assertTrue();
      const deviceId: String = dmInstance.getLocalDeviceId();
      expect(deviceId.length > 0).assertTrue();
      // 问题：设备 ID 被记录到日志
      hilog.info(domain, tag, `Local Device Id: ${deviceId}`);

      const dmNetworkId: String = dmInstance.getLocalDeviceNetworkId();
      // 问题：网络 ID 被记录到日志
      hilog.info(domain, tag, 'getLocalDeviceNetworkId: %{public}s', dmNetworkId);

      const deviceType = dmInstance.getDeviceType(dmNetworkId);
      const deviceName = dmInstance.getDeviceName(dmNetworkId);
      // 问题：设备名称被记录到日志
      hilog.info(domain, tag, 'Device Name: %{public}s', deviceName);
    } catch (err) {
      hilog.info(domain, tag, '%{public}s', 'SUB_DH_DeviceManager_Dcts_0100 failed for ' + err);
      const e = err as BusinessError;
      hilog.info(domain, tag, `check permission failed err.code: ${e.code}`);
      expect(e.code == 201).assertTrue();
    }
});
```

**触发路径**:
```
测试执行
    │
    ├── hilog.info(`Local Device Id: ${deviceId}`)
    │      └── 设备 ID 泄露
    │
    ├── hilog.info(`getLocalDeviceNetworkId: ${dmNetworkId}`)
    │      └── 网络 ID 泄露
    │
    ├── hilog.info(`Device Name: ${deviceName}`)
    │      └── 设备名称泄露
    │
    └── 测试日志
           │
           └── 攻击者可读取并映射网络拓扑
```

**影响评估**:
- **可利用性**: 高（日志文件可被读取）
- **权限提升可能**: 低（信息泄露，非直接提权）
- **影响范围**: 所有设备管理测试用例

**日志脱敏建议**:
```typescript
// 建议修复：脱敏敏感信息
it('SUB_DH_DeviceManager_Dcts_0100', ..., async (): Promise<void> => {
    hilog.info(domain, tag, '-----------------SUB_DH_DeviceManager_Dcts_0100 START------------------------');
    try {
      let dmInstance = distributedDeviceManager.createDeviceManager(TEST_BUNDLE_NAME);
      expect(dmInstance !== null && dmInstance !== undefined).assertTrue();
      const deviceId: String = dmInstance.getLocalDeviceId();
      expect(deviceId.length > 0).assertTrue();

      // 脱敏：仅记录前 4 位
      hilog.info(domain, tag, `Device Id: ${deviceId.substring(0, 4)}***`);

      const dmNetworkId: String = dmInstance.getLocalDeviceNetworkId();
      hilog.info(domain, tag, `Network Id: ${dmNetworkId.substring(0, 4)}***`);

      const deviceName = dmInstance.getDeviceName(dmNetworkId);
      hilog.info(domain, tag, 'Device Type: ${deviceType}`);

      // 或者完全移除敏感信息
      // hilog.info(domain, tag, 'Device info obtained successfully');

    } catch (err) {
      // 错误码脱敏
      hilog.info(domain, tag, 'Operation failed with error');
      const e = err as BusinessError;
      expect(e.code == 201).assertTrue();
    }
});
```

---

### 风险 5: RPC 边界值溢出行为

**位置**: `communication/dsoftbus_rpcets/rpcclient/entry/src/ohosTest/ets/test/RpcRequestEtsUnit.test.ets:2117-2200`

**证据**:
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
    data.writeByte(127);  // 最大边界值
    // 验证回绕行为
    expect(reply.readByte()).assertEqual(-128);  // 溢出回绕
    expect(reply.readByte()).assertEqual(0);
    expect(reply.readByte()).assertEqual(1);
    expect(reply.readByte()).assertEqual(127);
  }
});
```

**触发路径**:
```
正常范围: [-128, 127]
    │
    ├── 写入 128 (溢出)
    │      └── -128 (回绕)
    │
    ├── 写入 129 (溢出)
    │      └── 127 (回绕)
    │
    └── 溢出/回绕行为
           │
           └── 可能绕过边界检查
```

**影响评估**:
- **可利用性**: 中（需要构造特定输入）
- **权限提升可能**: 中（如果边界检查用于安全判断）
- **影响范围**: 所有 RPC 序列化数据操作

**修复建议**:
```typescript
// 建议修复：在 RPC 实现中添加边界检查
class MessageSequence {
    writeByte(value: number): void {
        // 1. 添加边界检查
        if (value < -128 || value > 127) {
            throw new Error(`Byte value out of range: ${value}`);
        }

        // 2. 记录警告并拒绝非法值
        console.warn(`Value ${value} exceeds byte range [-128, 127]`);

        // 3. 执行写入（内部实现）
        this._writeByteInternal(value);
    }
}
```

---

### 风险 6: 错误处理不当导致资源泄漏

**位置**: `testtools/disetsTest/client/testService.ets:67-106`

**证据**:
```typescript
try {
    dmInstance = deviceManager.createDeviceManager(bundleNameKv);
    console.info(logTag + 'get deviceManager is success')
} catch (error) {
    console.info(logTag + 'get deviceManager is failed' + JSON.stringify(error))
    // 问题：错误仅记录，未进行资源清理
}

// ... 连接服务扩展能力 ...
try {
    connection = context.connectServiceExtensionAbility(want, connect);
    console.info(logTag + " ***** connectServiceExtensionAbility success. got id: " + connection);
} catch (err) {
    console.info(logTag + " ***** connectServiceExtensionAbility failed. got id: " + connection);
    // 问题：连接失败时，connection 可能未正确关闭
    let code = (err as BusinessError).code;
    let message = (err as BusinessError).message;
    console.error(logTag + ` ***** connectServiceExtensionAbility failed. err code is ${code}, message is ${message}`);
}
// 问题：没有 finally 块确保资源清理
```

**触发路径**:
```
尝试操作
    │
    ├── 成功 ─────────────► 资源正常使用
    │
    └── 失败 (异常)
           │
           ├── 记录日志
           │
           └── 返回（未清理资源）
                  │
                  └── 资源泄漏！
```

**影响评估**:
- **可利用性**: 低（需要多次失败操作）
- **权限提升可能**: 低（拒绝服务）
- **影响范围**: 所有使用 try-catch 的测试用例

**修复建议**:
```typescript
// 建议修复：使用 finally 块确保资源清理
try {
    dmInstance = deviceManager.createDeviceManager(bundleNameKv);
    console.info(logTag + 'get deviceManager is success')
} catch (error) {
    console.info(logTag + 'get deviceManager is failed' + JSON.stringify(error))
} finally {
    // 确保资源清理
    if (dmInstance !== null && dmInstance !== undefined) {
        dmInstance = null;
    }
    if (connection !== null && connection !== undefined) {
        context.disconnectServiceExtensionAbility(connection);
        connection = null;
    }
}
```

---

## 未覆盖安全测试

| 缺失测试 | 风险等级 | 建议测试方法 |
|----------|---------|--------------|
| 网络中间人攻击模拟 | 高 | 添加 MITM 测试，模拟篡改 RPC 数据包 |
| 证书撤销检查 | 高 | 测试过期证书、撤销列表的处理 |
| 会话劫持防护 | 中 | 添加会话 ID 篡改测试 |
| 拒绝服务压力测试 | 中 | 添加并发压力测试 |
| 并发竞态条件测试 | 中 | 测试多设备同时操作的场景 |
| 模糊测试 | 中 | 对输入接口进行模糊测试 |
| 侧信道攻击测试 | 低 | 测试时间侧信道（计时攻击） |

---

## 缓解措施

### 测试环境隔离

```
建议措施:
1. 测试网络与生产网络隔离
   - 使用专用测试 Wi-Fi
   - 配置防火墙规则
2. 使用专用测试设备池
   - 物理隔离测试设备
   - 定期重置设备状态
3. 测试数据定期清理
   - 每次测试后清理分布式文件
   - 清理 KV/RDB 测试数据
   - 重置设备信任关系
4. 敏感测试数据加密存储
   - 使用高安全级别存储测试凭证
   - 避免在代码中硬编码敏感信息
```

### 测试代码安全

```
建议措施:
1. 禁止硬编码敏感信息
   - 设备 ID、网络 ID、认证令牌
   - 使用配置文件管理测试参数
2. 使用配置管理敏感参数
   - 环境变量或配置文件
   - 避免在提交代码中包含生产凭证
3. 添加静态代码安全扫描
   - 集成 ESLint 安全插件
   - 使用 SAST 工具扫描依赖
4. 定期代码安全审查
   - 每次代码审查包含安全检查项
   - 建立安全编码规范
```

### 依赖管理

```
建议措施:
1. 定期更新测试框架依赖
   - OpenHarmony SDK 更新
   - 测试工具更新
2. 扫描已知安全漏洞
   - 使用漏洞数据库
   - 订阅安全公告
3. 锁定依赖版本
   - 在 package.json 中锁定版本
   - 使用 package-lock.json
4. 使用可信镜像源
   - 官方镜像仓库
   - 验证镜像签名
```

---

## 审计建议

### 代码审查清单

- [ ] 敏感信息检测（硬编码密码、密钥、设备 ID）
- [ ] 输入验证覆盖（边界值、异常值、类型检查）
- [ ] 权限检查测试覆盖（白名单、最小权限）
- [ ] 错误处理完整性（finally 块、资源清理、错误传播）
- [ ] 日志安全性（避免敏感数据泄露、日志级别控制）
- [ ] RPC 身份验证（调用方 UID/PID、Token 机制）
- [ ] 路径规范化（使用 path.normalize、消除 ../）
- [ ] 安全级别使用（根据敏感度选择适当级别）
- [ ] 资源管理（文件句柄、网络连接、会话）

### 自动化测试建议

- [ ] 集成 SAST 工具扫描测试代码
- [ ] 添加敏感信息检测 CI 检查
- [ ] 自动化安全测试用例执行
- [ ] 定期依赖安全扫描
- [ ] 添加模糊测试 CI 任务
- [ ] 集成日志分析工具

### 持续监控建议

- [ ] 建立异常行为监控
- [ ] 添加安全事件日志
- [ ] 集成入侵检测系统
- [ ] 定期安全审计报告
- [ ] 建立漏洞修复跟踪机制

---

## 结论

### 总体评估

| 评估项 | 评级 | 说明 |
|--------|------|------|
| **代码质量** | 中 | 测试代码结构清晰，但安全防御不足 |
| **安全覆盖** | 中 | 有基础安全测试，但深度不够 |
| **风险控制** | 中 | 已识别主要风险，但缺乏缓解措施 |
| **改进空间** | 高 | 需要加强输入验证、身份验证和日志安全 |

### 关键建议（按优先级）

**高优先级（立即修复）**:
1. **RPC 身份验证增强** - 在所有 RPC 服务端添加调用方验证
2. **路径规范化** - 在所有文件操作中使用路径规范化
3. **日志脱敏** - 移除日志中的设备 ID、网络 ID 等敏感信息

**中优先级（近期修复）**:
4. **文件安全级别提升** - 根据数据敏感度使用适当的安全级别
5. **错误处理改进** - 使用 finally 块确保资源清理
6. **权限最小化** - 评估并减少不必要的高危权限使用

**低优先级（长期改进）**:
7. **边界值验证** - 在所有输入处添加边界检查
8. **模糊测试** - 对输入接口进行模糊测试
9. **安全审计** - 建立定期的安全审计流程
10. **监控告警** - 建立异常行为监控和告警机制

---

## 相关文档

- [攻击面分析](06_AttackSurface.md) - 详细的攻击面清单和利用路径
- [项目概览](01_Overview.md) - 项目定位和运行环境
- [目录结构](02_Directory_Structure.md) - 模块职责和代码地图
- [架构设计](03_Architecture.md) - 组件图和数据流
- [模块详解](04_Modules.md) - 各模块的 API 和功能
