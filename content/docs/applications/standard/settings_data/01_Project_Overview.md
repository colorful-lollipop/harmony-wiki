# 项目概览

## 项目定位

**SettingsData** 是 OpenHarmony 标准系统预置的系统应用，定位为系统设置数据的**统一存储与管理中心**。它负责：

1. **数据持久化** - 将系统设置数据持久化存储到关系型数据库
2. **数据共享** - 通过 DataAbility 对外提供标准化的数据访问接口
3. **多用户支持** - 管理系统用户相关的数据表（创建/删除用户时自动管理）
4. **权限管控** - 保护敏感设置项的访问安全

### 核心价值

| 价值点 | 说明 |
|--------|------|
| 统一存储 | 所有系统设置数据集中存储，避免分散管理 |
| 安全可控 | 敏感设置项需要系统级权限才能修改 |
| 跨应用访问 | 通过 DataShare 协议支持其他应用读取设置 |
| 多用户隔离 | 不同用户的数据相互隔离，保障隐私 |

---

## 核心能力详解

### 1. 数据持久化能力

**实现方式**: 使用 `@ohos.data.relationalStore` 关系型数据库

**证据**: `entry/src/main/ets/Utils/SettingsDBHelper.ets:162-174`

```typescript
public async initRdbStore() {
  Log.info('call initRdbStore start');
  let rdbStore = await relationalStore.getRdbStore(this.context as Context, {
    name: SettingsDataConfig.DB_NAME,
    securityLevel: 1
  });
  // ...
}
```

**数据库配置**:
- 数据库名称: `settingsdata.db`
- 安全等级: 1 (S1)
- 数据表:
  - `SETTINGSDATA` - 公共设置数据
  - `USER_SETTINGSDATA_<userId>` - 用户设置数据
  - `USER_SETTINGSDATA_SECURE_<userId>` - 用户安全设置数据

**证据**: `entry/src/main/ets/Utils/SettingsDataConfig.ets:25-33`

### 2. DataShare 数据共享能力

**实现方式**: 继承 `DataShareExtensionAbility`

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:55`

```typescript
export default class DataExtAbility extends DataShareExtensionAbility {
  // insert, update, query, delete 实现
}
```

**暴露的 URI**:
- `datashare:///com.ohos.settingsdata/entry/settingsdata/SETTINGSDATA`
- `datashare:///com.ohos.settingsdata/entry/settingsdata/USER_SETTINGSDATA`
- `datashare:///com.ohos.settingsdata/entry/settingsdata/USER_SETTINGSDATA_SECURE`

**证据**: `entry/src/main/resources/base/profile/data_share_config.json:2-19`

### 3. 多用户管理能力

**实现方式**: 监听用户变更公共事件

**证据**: `entry/src/main/ets/StaticSubscriber/UserChangeStaticSubscriber.ets:27-79`

| 事件 | 处理动作 |
|------|----------|
| `COMMON_EVENT_USER_ADDED` | 创建用户对应的数据表并加载默认数据 |
| `COMMON_EVENT_USER_REMOVED` | 删除用户对应的数据表 |

### 4. 权限管控能力

**实现方式**: 权限校验 + 受信任列表

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:271-298`

```typescript
private verifyPermission(value: relationalStore.ValuesBucket, callBack: (GrantStatus: boolean) => void) {
  // 1. 检查是否为受信任列表中的 key
  if (this.isTrustList(keyWord) || process.uid == rpc.IPCSkeleton.getCallingUid()) {
    callBack(true);
    return;
  }
  // 2. 检查是否拥有 MANAGE_SECURE_SETTINGS 权限
  let grantStatus = abilityAccessCtrl.createAtManager().verifyAccessToken(
    tokenID, 'ohos.permission.MANAGE_SECURE_SETTINGS');
}
```

**受信任列表** (无需权限可修改):
- `settings.display.SCREEN_BRIGHTNESS_STATUS`
- `settings.display.AUTO_SCREEN_BRIGHTNESS`
- `settings.display.SCREEN_OFF_TIMEOUT`

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:47-51`

---

## 运行环境要求

### SDK 版本

| 配置项 | 值 | 说明 |
|--------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |
| targetSdkVersion | 23 | 目标 SDK 版本 |

**证据**: `build-profile.json5:9-11`

### 设备支持

| 设备类型 | 支持情况 |
|----------|----------|
| default | ✅ 支持 |
| tablet | ✅ 支持 |

**证据**: `entry/src/main/module.json5:8-11`

### 安全区域

| 安全区域 | 说明 | 触发条件 |
|----------|------|----------|
| EL1 | 设备级安全区域 | 数据库文件大小 ≤ 48 字节 |
| EL2 | 更高级别安全区域 | 数据库文件大小 > 48 字节 |

**证据**: `entry/src/main/ets/Utils/SettingsDBHelper.ets:97-113`

```typescript
public getArea() {
  const dbFile = EL2_DB_PATH;
  if (this.area === undefined) {
    try {
      let stat = fs.statSync(dbFile);
      if (stat.size > VALID_DB_LENGTH) {  // VALID_DB_LENGTH = 48
        this.area = contextConstant.AreaMode.EL2;
      } else {
        this.area = contextConstant.AreaMode.EL1;
      }
    } catch {
      this.area = contextConstant.AreaMode.EL1;
    }
  }
}
```

---

## 关键概念

### DataAbility

DataAbility 是 OpenHarmony 中用于数据共享的组件。SettingsData 通过 DataAbility 向其他应用提供设置数据的访问接口。

**相关文档**: [架构设计](./03_Architecture.md)

### RDbStore

RdbStore 是 OpenHarmony 中关系型数据库的抽象，提供 SQLite 级别的数据库操作能力。

**证据**: `entry/src/main/ets/Utils/SettingsDBHelper.ets`

### DataSharePredicates

DataSharePredicates 是用于 DataShare 数据查询的谓词类，支持构建复杂的查询条件。

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:37`

### 权限模型

SettingsData 采用分级权限模型：
- **普通设置**: 无需特殊权限
- **敏感设置**: 需要 `ohos.permission.MANAGE_SECURE_SETTINGS` 权限
- **受信任列表**: 系统级 key 无需权限也可修改

---

## 技术选型理由

| 技术/框架 | 选型理由 |
|-----------|----------|
| ArkTS | OpenHarmony 推荐开发语言，提供类型安全 |
| DataShareExtensionAbility | 标准的数据共享框架 |
| relationalStore | 官方关系型数据库解决方案 |
| commonEventManager | 系统级事件订阅机制 |

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| API 参考 | [04_API_Reference.md](./04_API_Reference.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |
