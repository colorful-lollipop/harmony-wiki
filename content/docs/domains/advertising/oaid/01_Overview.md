# OAID 项目概览

## 一句话定义

OAID (Open Anonymous Device Identifier) 是 OpenHarmony 提供的**开放匿名设备标识符系统服务**，为个性化广告投放提供非永久性设备标识，同时保护用户隐私数据。

---

## 能力边界

### ✅ 能做什么

| 功能 | 说明 |
|------|------|
| **生成匿名标识符** | 基于 UUID v4 算法生成 32 位类 UUID 格式标识符 |
| **支持个性化广告** | 为广告 SDK 提供设备级匿名标识 |
| **用户可控重置** | 用户可通过恢复出厂设置或系统应用重置 OAID |
| **未成年人保护** | 集成 Ads Service 检测未成年人模式 |
| **转化率归因** | 支持第三方追踪平台接入 |

### ❌ 不能做什么

| 限制 | 说明 |
|------|------|
| **非永久性标识** | OAID 可被重置，不能作为长期设备标识 |
| **非跨平台** | 仅在 OpenHarmony 系统内有效 |
| **非强制可用** | 用户关闭"跨应用关联访问权限"后返回全 0 OAID |
| **非应用级** | 同一设备上不同 App 获取的 OAID 相同 |
| **需用户授权** | 必须获得 `ohos.permission.APP_TRACKING_CONSENT` 权限 |

---

## 运行环境

### 系统要求

| 要求项 | 说明 |
|--------|------|
| **系统类型** | OpenHarmony Standard System |
| **API 版本** | API 10+ |
| **Subsystem** | advertising |
| **SA ID** | 6101 |

### 依赖的系统服务

```
oaid_service (SA 6101)
    ├── ability_runtime    # Ability 管理
    ├── access_token       # 权限管理
    ├── bundle_framework   # Bundle 管理
    ├── kv_store          # 分布式数据存储
    ├── ipc               # IPC 通信
    ├── safwk             # SA 框架
    ├── samgr             # SA 管理
    └── openssl           # 随机数生成
```

### 权限要求

| 权限 | 类型 | 授权方式 | 说明 |
|------|------|---------|------|
| `ohos.permission.APP_TRACKING_CONSENT` | user_grant | 动态申请 | 广告追踪权限 |

---

## 快速开始

### 1. 声明权限

在 `module.json5` 中声明权限：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.APP_TRACKING_CONSENT",
        "reason": "$string:tracking_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

### 2. 添加依赖

在 `oh-package.json5` 中添加：

```json
{
  "dependencies": {
    "@kit.AdsKit": "*",
    "@kit.AbilityKit": "*"
  }
}
```

### 3. 申请权限并获取 OAID

```typescript
import { identifier } from '@kit.AdsKit';
import { abilityAccessCtrl, common } from '@kit.AbilityKit';

async function getOAIDWithPermission(context: common.Context): Promise<string | null> {
  // 1. 申请权限
  const atManager = abilityAccessCtrl.createAtManager();
  const result = await atManager.requestPermissionsFromUser(
    context, 
    ['ohos.permission.APP_TRACKING_CONSENT']
  );
  
  if (result.authResults[0] !== 0) {
    console.warn('用户未授权广告追踪权限');
    return null;
  }
  
  // 2. 获取 OAID
  try {
    const oaid = await identifier.getOAID();
    console.info('OAID:', oaid);
    return oaid;
  } catch (err) {
    console.error('获取 OAID 失败:', err);
    return null;
  }
}
```

### 4. 回调方式获取

```typescript
import { identifier } from '@kit.AdsKit';

identifier.getOAID((err, data) => {
  if (err.code) {
    console.error('获取失败:', err.message);
  } else {
    console.info('OAID:', data);
  }
});
```

---

## 系统应用重置 OAID

> ⚠️ **注意**: `resetOAID()` 仅对系统应用开放

### 前置条件

1. 应用必须为系统应用（通过 `CheckSystemApp()` 验证）
2. 应用 BundleName 必须在白名单配置中

### 配置白名单

编辑 `/etc/advertising/oaid/oaid_service_config.json`：

```json
{
  "resetOAIDBundleName": [
    "com.example.system.app1",
    "com.example.system.app2"
  ],
  "providerBundleName": "com.example.provider",
  "providerAbilityName": "ExampleAbility",
  "providerTokenName": "example_token"
}
```

### 调用重置接口

```typescript
import { identifier } from '@kit.AdsKit';

try {
  identifier.resetOAID();
  console.info('OAID 重置成功');
} catch (err) {
  // 错误码 202: 非系统应用
  // 错误码 17300002: 不在白名单
  console.error('重置失败:', err.code, err.message);
}
```

---

## 数据存储说明

### 存储位置

| 数据类型 | 存储路径 | 加密 |
|----------|----------|------|
| OAID 值 | `/data/service/el1/public/database/oaid_service_manager/oaidservice` | ✅ 加密 |
| 未成年信息 | `/data/service/el1/public/database/oaid_service_manager/underAgeInfo` | ✅ 加密 |
| 更新检查 | `/data/service/el1/public/database/oaid_service_manager/update_check.json` | ❌ 明文 |

### KVStore 配置

```cpp
// 数据库配置选项 (services/oaid_manager/src/oaid_service.cpp:314-325)
options.createIfMissing = true;
options.encrypt = true;              // 启用加密
options.autoSync = false;
options.kvStoreType = DistributedKv::KvStoreType::SINGLE_VERSION;
options.area = DistributedKv::EL1;   // 加密安全区
options.securityLevel = DistributedKv::SecurityLevel::S1;
```

---

## 错误处理

### 常见错误码

| 错误码 | 常量 | 说明 | 处理建议 |
|--------|------|------|----------|
| 0 | ERR_OK | 成功 | - |
| 202 | OAID_ERROR_CODE_NOT_SYSTEM_APP | 非系统应用调用系统 API | 仅系统应用可调用 |
| 17300002 | OAID_ERROR_NOT_IN_TRUST_LIST | 不在白名单中 | 配置白名单 |
| 17300001 | ERR_SYSYTEM_ERROR | 系统内部错误 | 重试或检查服务状态 |

### 全 0 OAID 处理

当用户关闭"跨应用关联访问权限"开关时，返回 `00000000-0000-0000-0000-000000000000`：

```typescript
const oaid = await identifier.getOAID();
if (oaid === '00000000-0000-0000-0000-000000000000') {
  // 提示用户开启开关
  console.warn('用户已禁用广告追踪');
}
```

---

## 相关链接

### 官方文档
- [OpenHarmony OAID 服务文档](https://gitee.com/openharmony/docs)
- [广告标识服务 API 参考](https://gitee.com/openharmony/docs/blob/master/en/application-dev/reference/apis/js-apis-advertising.md)

### 参考实现
- [华为 HMS 广告示例](https://github.com/HMS-Core/hms-ads-demo-harmonyos)
- [华为开发者联盟 Ads Kit](https://developer.huawei.com/consumer/cn/hms/huawei-adskit/)

### 内部文档
- [架构分析](02_Architecture.md)
- [接口文档](04_Interface.md)
- [安全风险评估](06_SecurityReview.md)

---

## 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| OAID | Open Anonymous Device Identifier | 开放匿名设备标识符 |
| SA | System Ability | 系统能力/系统服务 |
| N-API | Native API | JavaScript 与 C++ 互操作接口 |
| IPC | Inter-Process Communication | 进程间通信 |
| KVStore | Key-Value Store | 键值存储 |
| HAP | HarmonyOS Ability Package | HarmonyOS 应用包 |
| SA ID | System Ability ID | 系统能力标识符 |
