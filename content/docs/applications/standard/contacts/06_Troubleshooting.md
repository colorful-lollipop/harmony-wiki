# 问题排查

## 概述

本文档收集 Contacts 应用开发和运行过程中的常见问题及解决方案。

## 构建问题

### 1. hvigor 构建失败

**问题描述**: 运行 `hvigor build` 失败

**可能原因**:
- Node.js 版本不兼容
- SDK 配置错误
- 模块依赖缺失

**排查步骤**:

```bash
# 1. 检查 Node.js 版本
node -v
# 要求版本: 16.x 或 18.x

# 2. 检查 hvigor 是否正确安装
npm list @ohos/hvigor
npm list @ohos/hvigor-ohos-plugin

# 3. 检查 SDK 配置
echo $OHOS_SDK_HOME
ls $OHOS_SDK_HOME

# 4. 清理并重新构建
hvigor clean
hvigor build --debug
```

**常见错误**:

| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| `Cannot find module '@ohos/hvigor'` | hvigor 未安装 | `npm install` |
| `SDK not found` | SDK 路径未配置 | 设置 `OHOS_SDK_HOME` |
| `Module not found` | 模块配置错误 | 检查 `build-profile.json5` |

---

### 2. 签名失败

**问题描述**: 构建成功但签名失败

**排查步骤**:

```bash
# 1. 检查签名文件是否存在
ls -la sign/

# 2. 验证签名配置
cat build-profile.json5 | grep -A 10 signingConfigs

# 3. 手动签名测试
hdc sign -p sign/contacts.p7b -k sign/OpenHarmony_stage.p12 entry.hap
```

**检查项**:
- `sign/IDE.cer` 是否存在
- `sign/OpenHarmony_stage.p12` 密码是否正确
- `sign/contacts.p7b` 是否有效

---

## 运行时问题

### 1. 权限被拒绝

**问题描述**: 应用运行时提示权限被拒绝

**症状**:
- 无法读取联系人
- 无法拨打电话
- 通话记录为空

**排查步骤**:

```typescript
// 1. 检查权限状态
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';

const AtManager = abilityAccessCtrl.createAtManager();
const status = AtManager.checkAccessToken(
    getCurrentProcessId(),
    'ohos.permission.READ_CONTACTS'
);
```

**解决方案**:

| 权限 | 检查方式 | 请求方式 |
|------|----------|----------|
| READ_CONTACTS | checkAccessToken | requestPermissionsFromUser |
| WRITE_CONTACTS | checkAccessToken | requestPermissionsFromUser |
| PLACE_CALL | checkAccessToken | 系统自动检查 |

**代码示例**:
```typescript
// 权限检查和请求
async ensurePermissions(): Promise<boolean> {
    const permissions = [
        'ohos.permission.READ_CONTACTS',
        'ohos.permission.WRITE_CONTACTS'
    ];
    
    const AtManager = abilityAccessCtrl.createAtManager();
    const results = await AtManager.requestPermissionsFromUser(
        globalThis.context,
        permissions
    );
    
    return results.authResults.every(r => r === 0);
}
```

---

### 2. DataAbility 访问失败

**问题描述**: 查询联系人数据时返回空或报错

**症状**:
- 联系人列表为空
- `DataShareHelper` 操作失败

**排查步骤**:

```typescript
// 1. 检查 URI 是否正确
const contactsUri = 'datashare:///com.ohos.contactsdataability';

// 2. 检查 DataShareHelper 创建
const helper = dataAbility.createDataAbilityHelper(context);

// 3. 测试基本查询
const resultSet = await helper.query(
    contactsUri + '/contacts/contact',
    [],
    ''
);
```

**常见原因**:
- DataAbility 服务未启动
- URI 格式错误
- 权限不足

---

### 3. 页面无法加载

**问题描述**: 应用启动后页面空白或报错

**排查步骤**:

```typescript
// 1. 检查页面路由配置
// 查看 module.json5 中的 pages 配置

// 2. 检查页面文件是否存在
ls -la entry/src/main/ets/pages/

// 3. 检查 MainAbility 日志
// 查看 HiLog 输出
```

**常见原因**:
- 页面路径配置错误
- 资源文件缺失
- UI 组件渲染异常

---

### 4. Worker 线程异常

**问题描述**: DataWorker 任务执行失败或卡顿

**排查步骤**:

```typescript
// 1. 检查 Worker 消息格式
worker.postMessage({
    type: 'QUERY_CONTACTS',
    data: { limit: 100 }
});

// 2. 检查 Worker 错误处理
worker.onerror = (err) => {
    HiLog.e(TAG, `Worker error: ${err.message}`);
};

// 3. 检查 Worker 是否存活
if (worker.terminate) {
    worker.terminate();
}
```

---

## 数据问题

### 1. 联系人同步失败

**问题描述**: 联系人数据未正确同步或显示

**排查步骤**:

```typescript
// 1. 检查联系人查询
const contacts = await repository.findAll({
    page: 1,
    count: 50
});

// 2. 检查数据完整性
for (const contact of contacts) {
    if (!contact.hasDisplayName) {
        HiLog.w(TAG, `Contact ${contact.id} missing display name`);
    }
}

// 3. 检查账号关联
const accounts = await getAccounts();
```

---

### 2. 数据库版本不兼容

**问题描述**: 升级后联系人数据丢失或损坏

**排查步骤**:

```typescript
// 1. 检查数据库版本
const version = await rdbStore.getVersion();

// 2. 执行数据迁移
if (version < CURRENT_VERSION) {
    await migrateData(version, CURRENT_VERSION);
}

// 3. 备份重要数据
```

---

## 性能问题

### 1. 联系人加载慢

**问题描述**: 联系人列表加载时间过长

**优化建议**:

| 优化项 | 实施方式 |
|--------|----------|
| 分页加载 | 使用 `findAll({ page, count })` |
| 懒加载 | 详情页按需加载 |
| 索引优化 | 确保 `QUICK_SEARCH_KEY` 索引存在 |
| Worker 处理 | 大数据量操作放入 Worker |

**代码示例**:
```typescript
// 分页查询
async loadContacts(page: number = 1, count: number = 50): Promise<void> {
    const result = await contactRepository.findAll({
        page,
        count: page <= 2 ? 50 : 500  // 前两页50条，后续500条
    });
    this.updateContactList(result);
}
```

---

### 2. 搜索响应慢

**问题描述**: 联系人搜索响应时间过长

**优化建议**:
- 使用 `searchContact` 替代全表扫描
- 确保 `search_contacts` 表索引存在
- 限制搜索结果数量

---

## 日志查看

### 日志标签

| 模块 | 标签 | 说明 |
|------|------|------|
| MainAbility | `MainAbility` | 主 Ability 生命周期 |
| PermissionManager | `PermissionManager` | 权限管理 |
| ContactRepository | `ContactRepository` | 数据访问 |
| Worker | `Worker` | 线程操作 |

### 查看日志

```bash
# 查看所有日志
hilog | grep -E "(MainAbility|PermissionManager|ContactRepository)"

# 查看特定标签
hilog | grep "MainAbility"

# 实时查看
hilog -T MainAbility
```

---

## 调试技巧

### 1. 使用真机调试

```bash
# 连接设备
hdc list targets

# 安装应用
hdc install entry/default/outputs/default/entry.hap

# 启动应用
hdc shell aa start -b com.ohos.contacts -a com.ohos.contacts.MainAbility
```

### 2. 使用模拟器调试

```bash
# 启动模拟器
emulator -avd <avd_name>

# 安装应用
hdc install entry/default/outputs/default/entry.hap
```

### 3. 断点调试

DevEco Studio 支持：
- TypeScript/ArkTS 断点
- 条件断点
- 日志断点
- 远程调试

---

## 常见错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 0 | 成功 | - |
| -1 | 通用失败 | 检查日志 |
| 401 | 参数错误 | 检查 API 参数 |
| 201 | 权限被拒 | 请求权限 |
| 202 | 签名错误 | 检查签名配置 |
| 1001 | SDK 未初始化 | 初始化 SDK |
| 1002 | SDK 版本不兼容 | 升级 SDK |
