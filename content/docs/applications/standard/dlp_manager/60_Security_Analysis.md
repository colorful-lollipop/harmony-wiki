# 安全风险评审

## 目的

本文档对DLP Manager进行安全风险评审，识别攻击面、信任边界和可被利用点，并提供修复建议。

## 适用范围

- 进行安全评审的审计人员
- 需要了解安全机制的开发人员
- 负责安全加固的工程师

---

## 威胁模型

### 资产定义

| 资产 | 说明 | 敏感级别 |
|------|------|----------|
| DLP文件内容 | 受保护的文件数据 | 高 |
| 用户权限信息 | 授权用户列表、权限级别 | 高 |
| 域账号信息 | 企业用户身份信息 | 高 |
| 加密密钥 | HUKs管理的密钥 | 极高 |
| 沙箱环境 | 隔离的应用运行环境 | 中 |

### 攻击者模型

| 攻击者类型 | 能力 | 目标 |
|------------|------|------|
| 普通应用 | 可调用系统API | 获取DLP文件内容 |
| 恶意用户 | 可操作用户界面 | 绕过权限检查 |
| 系统应用 | 较高系统权限 | 获取加密密钥 |
| 物理攻击者 | 设备访问权限 | 提取DLP文件 |

---

## 攻击面分析

### 1. 外部输入攻击面

```
┌─────────────────────────────────────────────────────────────┐
│                      外部输入攻击面                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  文件URI    ────────┐                                       │
│                     │                                       │
│  Want参数   ────────┼──────>  DLP Manager  ──────>  处理    │
│                     │          入口点                       │
│  用户输入   ────────┘                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 1.1 文件URI输入

**风险点**: URI路径遍历、非法URI格式

**验证点**: `entry/src/main/ets/Ability/MainAbilityEx.ets:223-244`

```typescript
let uri = String(want.uri);
if (!isValidPath(uri)) {
  HiLog.error(TAG, `invalid uri in want.uri`);
  return false;
}
// 检查FUSE路径
if (uri.indexOf(Constants.FUSE_PATH) !== -1) {
  HiLog.error(TAG, `invalid uri in want.uri`);
  return false;
}
```

**检查项**:
- ✅ 验证URI以`file://`开头（`entry/src/main/ets/common/FileUtils/utils.ets:507`）
- ✅ 检查是否包含`/mnt/data/fuse/`路径
- ✅ 检查URI是否已被打开

**建议**: 
- 考虑使用更严格的URI白名单
- 规范化URI路径（resolve . 和 ..）

#### 1.2 Want参数输入

**风险点**: 参数缺失、参数类型错误、参数内容注入

**验证点**: `entry/src/main/ets/Ability/MainAbilityEx.ets:197-246`

```typescript
async checkValidWant(want: Want): Promise<boolean> {
  let parameters = want.parameters;
  if (parameters === undefined) {
    HiLog.error(TAG, `need parameters in want`);
    return false;
  }
  if (parameters.fileName === undefined) {
    HiLog.error(TAG, `need fileName in want.parameters`);
    return false;
  }
  // ... 更多检查
}
```

**检查项**:
- ✅ 检查parameters是否存在
- ✅ 检查fileName.name是否存在
- ✅ 检查uri是否存在
- ✅ 检查callerToken和callerBundleName是否存在
- ❌ **缺失**: 参数类型检查
- ❌ **缺失**: 参数长度检查
- ❌ **缺失**: 参数内容转义/过滤

**风险评级**: 🟡 中风险

### 2. 权限检查攻击面

#### 2.1 权限检查点

**检查点1**: 入口参数权限（`MainAbilityEx.checkValidWant:215-220`）

```typescript
this.callerToken = parameters[Constants.PARAMS_CALLER_TOKEN] as number;
let callerBundleName: string = parameters[Constants.PARAMS_CALLER_BUNDLE_NAME] as string;
if (this.callerToken === undefined || callerBundleName === undefined) {
  HiLog.error(TAG, `need caller info in want.parameters`);
  return false;
}
```

**检查点2**: 文件权限检查（`entry/src/main/ets/common/FileUtils/utils.ets:173-189`）

```typescript
function getAuthPerm(accountName: string, dlpProperty: dlpPermission.DLPProperty): dlpPermission.DLPFileAccess {
  let perm: dlpPermission.DLPFileAccess = dlpPermission.DLPFileAccess.NO_PERMISSION;
  if (accountName === dlpProperty.ownerAccount) {
    return dlpPermission.DLPFileAccess.FULL_CONTROL;
  }
  // 检查everyoneAccessList
  if ((dlpProperty.everyoneAccessList !== undefined) && (dlpProperty.everyoneAccessList.length > 0)) {
    perm = Math.max(...dlpProperty.everyoneAccessList);
  }
  // 检查authUserList
  let authUserList = dlpProperty.authUserList ?? [];
  for (let i = 0; i < authUserList.length; ++i) {
    let authUser = authUserList[i];
    if (authUser.authAccount === accountName) {
      return authUser.dlpFileAccess;
    }
  }
  return perm;
}
```

#### 2.2 可被利用点 #1: 权限验证绕过

**风险描述**: 权限检查逻辑依赖于`accountName`的准确性，如果账号信息可被篡改，可能导致权限绕过。

**证据**: `entry/src/main/ets/common/FileUtils/utils.ets:173-189`

**攻击路径**:
1. 攻击者获取目标账号名
2. 通过某种方式修改或伪造账号信息
3. 获得对DLP文件的访问权限

**影响**: 高 - 可能导致未授权访问DLP文件

**修复建议**:
- 增加账号信息完整性校验（签名验证）
- 账号信息应从可信源（系统账号服务）获取，不信任应用传入
- 增加访问日志记录，便于审计

**风险评级**: 🔴 高风险

### 3. 文件操作攻击面

#### 3.1 文件解析

**风险点**: DLP文件头解析可能存在缓冲区溢出或格式错误

**代码位置**: `entry/src/main/ets/common/FileUtils/utils.ets:511-615`

```typescript
function getAccountTypeAndRealFileType(context, fd: number): Promise<DLPGeneralInfo> {
  let z = new ArrayBuffer(Constants.HEAD_LENGTH_IN_BYTE);  // 80字节
  let option: ChangeOption = { offset: 0, length: Constants.HEAD_LENGTH_IN_BYTE };
  fs.readSync(fd, z, option);
  let buf = new Uint32Array(z, 0, Constants.HEAD_LENGTH_IN_U32);  // 20个u32
  // 根据magic判断格式
  if (buf && buf[0] === Constants.DLP_ZIP_MAGIC) {
    return handleZipFile(context, fd);
  } else {
    return handleNonZipFile(fd, buf);
  }
}
```

#### 3.2 可被利用点 #2: 文件解析越界读取

**风险描述**: `handleNonZipFile`函数中，`certOffset`计算后可能超出文件实际大小。

**证据**: `entry/src/main/ets/common/FileUtils/utils.ets:594-615`

```typescript
async function handleNonZipFile(fd: number, buf: Uint32Array): Promise<Result<DLPGeneralInfo>> {
  let cert = new ArrayBuffer(buf[Constants.CERT_SIZE]);
  let certOffset = Constants.CERT_OFFSET_4GB * buf[Constants.CERT_OFFSET + 1] + buf[Constants.CERT_OFFSET];
  const option: ChangeOption = { offset: certOffset, length: buf[Constants.CERT_SIZE] };
  try {
    fs.readSync(fd, cert, option);
    // ...
  }
}
```

**攻击路径**:
1. 构造恶意DLP文件
2. 设置`buf[CERT_OFFSET]`和`buf[CERT_SIZE]`为异常值
3. 导致越界读取或大量内存分配

**影响**: 中 - 可能导致拒绝服务或信息泄露

**修复建议**:
- 读取前验证offset和length是否在文件大小范围内
- 限制单次读取的最大长度
- 使用try-catch捕获异常

**风险评级**: 🟡 中风险

#### 3.3 ZIP文件解压

**代码位置**: `entry/src/main/ets/common/FileUtils/utils.ets:546-592`

```typescript
async function handleZipFile(context, fd: number): Promise<Result<DLPGeneralInfo>> {
  let random = String(Math.random()).substring(Constants.RAND_START, Constants.RAND_END);
  let filePath = context.filesDir + '/saveAs' + random;
  let dirPath = context.filesDir + '/saveAsUnzip' + random;
  // ...
  await zlib.decompressFile(filePath, dirPath);
}
```

#### 3.4 可被利用点 #3: ZIP炸弹攻击

**风险描述**: 解压ZIP文件没有大小限制检查，可能遭受ZIP炸弹攻击。

**攻击路径**:
1. 构造高压缩比的恶意ZIP文件
2. 系统解压时耗尽磁盘空间或内存

**影响**: 中 - 拒绝服务

**修复建议**:
- 解压前检查压缩包大小
- 设置解压后最大文件大小限制
- 监控解压过程中的磁盘空间

**风险评级**: 🟡 中风险

### 4. RPC通信攻击面

#### 4.1 RPC接口身份验证

**代码位置**: `entry/src/main/ets/rpc/DlpPermissionAbilityServiceStub.ets:47-59`

```typescript
private checkCallerIdentity(data: rpc.MessageSequence): boolean {
  try {
    let token = data.readInterfaceToken();
    if (token !== Constant.SA_INTERFACE_TOKEN) {
      HiLog.error(TAG, `Interface token is invalid`);
      return false;
    }
    return true;
  } catch (error) {
    HiLog.wrapError(TAG, error, 'check caller identity failed');
  }
  return false;
}
```

#### 4.2 可被利用点 #4: RPC接口令牌硬编码

**风险描述**: 接口令牌硬编码在常量中，可能被提取后伪造请求。

**证据**: `entry/src/main/ets/common/constant.ets:124`

```typescript
public static SA_INTERFACE_TOKEN = 'OHOS.Security.DlpCredentialAbility';
```

**攻击路径**:
1. 攻击者从代码中提取令牌字符串
2. 构造RPC请求，设置相同令牌
3. 绕过身份验证

**影响**: 中 - 可能调用敏感RPC接口

**修复建议**:
- 使用动态令牌或挑战-响应机制
- 增加调用者UID/BID验证
- 敏感操作增加权限校验

**风险评级**: 🟡 中风险

### 5. 加密与密钥管理攻击面

#### 5.1 HUKs使用

**代码位置**: `entry/src/main/ets/common/huks/HuksCipherUtil.ets:22`

```typescript
export default class HuksCipherUtils {
  public static async generateKey(keyAlias: string, options: huks.HuksOptions): Promise<boolean> {
    if (isInvalidStr(keyAlias)) {
      HiLog.error(TAG, 'keyAlias is invalid.');
      return false;
    }
    // ...
    await huks.generateKeyItem(keyAlias, options);
  }
}
```

#### 5.2 可被利用点 #5: 密钥别名冲突

**风险描述**: 密钥别名如果可预测或重复，可能导致密钥覆盖或误用。

**证据**: `entry/src/main/ets/common/constant.ets:324`

```typescript
public static readonly ASSOCIATION_KEY_ALIAS: string = 'ASSOCIATION_KEY_ALIAS';
```

**攻击路径**:
1. 攻击者知道密钥别名格式
2. 构造相同的别名请求密钥操作
3. 可能覆盖或获取其他密钥

**影响**: 高 - 密钥泄露或被篡改

**修复建议**:
- 使用包含随机数或哈希的唯一别名
- 密钥别名与应用ID/用户ID绑定
- 定期轮换密钥

**风险评级**: 🔴 高风险

### 6. 日志与信息泄露攻击面

#### 6.1 日志输出

**代码位置**: 多处使用HiLog打印日志

```typescript
HiLog.info(TAG, `open filename: ${FileUtils.getFileNameByUri(uri)}, as: ${file.fd}`);
```

#### 6.2 可被利用点 #6: 敏感信息泄露到日志

**风险描述**: 日志中可能输出敏感信息（文件路径、账号、密钥别名等）。

**证据示例**: 
- `entry/src/main/ets/common/FileUtils/utils.ets:140`: 打印文件描述符
- `entry/src/main/ets/common/FileUtils/utils.ets:551`: 打印文件大小

**攻击路径**:
1. 攻击者获取日志访问权限
2. 从日志中提取敏感信息
3. 利用信息进行进一步攻击

**影响**: 低-中 - 信息泄露

**修复建议**:
- 对敏感信息进行脱敏处理（如URI只显示文件名）
- 使用日志分级，敏感信息使用debug级别
- 生产环境关闭debug日志

**风险评级**: 🟢 低风险

### 7. 沙箱逃逸攻击面

#### 7.1 沙箱机制

DLP Manager通过`installDlpSandbox`创建沙箱环境，但沙箱应用可能尝试逃逸。

#### 7.2 可被利用点 #7: 沙箱权限配置不当

**风险描述**: 如果沙箱应用权限配置过于宽松，可能逃逸沙箱限制。

**影响**: 高 - 绕过DLP保护机制

**修复建议**:
- 最小权限原则配置沙箱应用权限
- 定期审计沙箱权限配置
- 监控沙箱应用的异常行为

**风险评级**: 🟡 中风险（依赖于系统沙箱实现）

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                          外部不可信区                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │ 第三方应用 │  │ 用户输入  │  │  网络请求  │                      │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                      │
└───────┼─────────────┼─────────────┼─────────────────────────────┘
        │             │             │
        ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      输入验证边界                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  checkValidWant() / isValidPath() / 参数校验            │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DLP Manager 应用区                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │  Ability  │  │  Manager  │  │  Handler  │                      │
│  └──────────┘  └──────────┘  └──────────┘                      │
└─────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│                         系统服务边界                             │
│  ┌─────────────────┐  ┌─────────────────┐                      │
│  │ DLP Permission  │  │    Account      │                      │
│  │     Service     │  │     Service     │                      │
│  └─────────────────┘  └─────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 风险汇总

| 序号 | 风险点 | 影响 | 评级 | 修复优先级 |
|------|--------|------|------|------------|
| 1 | 权限验证绕过 | 未授权访问 | 🔴 高 | P0 |
| 5 | 密钥别名冲突 | 密钥泄露 | 🔴 高 | P0 |
| 2 | 文件解析越界 | DoS/信息泄露 | 🟡 中 | P1 |
| 3 | ZIP炸弹攻击 | DoS | 🟡 中 | P1 |
| 4 | RPC令牌硬编码 | 接口滥用 | 🟡 中 | P1 |
| 7 | 沙箱权限配置 | 绕过保护 | 🟡 中 | P1 |
| 6 | 日志信息泄露 | 信息泄露 | 🟢 低 | P2 |

---

## 修复建议汇总

### 立即修复（P0）

1. **加强账号信息验证**: 从系统账号服务获取账号信息，不信任应用传入
2. **密钥别名唯一性**: 使用包含随机数或应用ID的密钥别名

### 短期修复（P1）

3. **文件解析边界检查**: 读取前验证offset和length
4. **ZIP解压限制**: 设置解压大小上限
5. **RPC令牌动态化**: 使用挑战-响应机制或调用者身份验证
6. **沙箱权限审计**: 定期审计沙箱应用权限配置

### 长期优化（P2）

7. **日志脱敏**: 敏感信息脱敏后输出
8. **安全监控**: 增加异常行为监控和告警
9. **安全测试**: 引入Fuzzing测试和渗透测试

---

## 检查局限性

1. **未审计测试代码**: 根据约束排除测试目录
2. **未审计系统服务**: DLP Permission Service实现未审计
3. **未进行动态分析**: 仅基于静态代码分析
4. **未验证实际利用**: 部分风险点为理论分析，未实际验证

---

## 相关链接

- [架构设计](10_Architecture.md) - 了解架构安全设计
- [对外接口](30_Public_API.md) - 查看接口安全分析
- [内部接口](40_Inner_API.md) - 查看内部接口安全
