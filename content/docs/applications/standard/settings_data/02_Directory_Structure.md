# 目录结构与模块职责

## 整体目录树

```
settingsdata/
│
├── .codecheck/                    # 代码检查配置
│
├── .git/                          # Git 版本控制
│
├── AppScope/                      # 应用级作用域配置
│   └── resources/
│       └── base/                  # 基础资源文件
│
├── entry/                         # 主模块
│   └── src/main/
│       ├── ets/                   # ArkTS 源代码
│       │   ├── Application/       # 应用入口
│       │   ├── DataAbility/       # DataAbility 实现
│       │   ├── StaticSubscriber/  # 静态事件订阅
│       │   ├── Utils/             # 工具类
│       │   └── common/            # 公共类型定义
│       │
│       ├── resources/             # 资源文件
│       │   ├── base/
│       │   │   ├── element/       # 元素资源（颜色、字符串等）
│       │   │   ├── media/         # 媒体资源（图标等）
│       │   │   └── profile/       # 配置文件
│       │   │       ├── data_share_config.json
│       │   │       ├── main_pages.json
│       │   │       └── static_subscriber_config.json
│       │   │
│       │   └── rawfile/           # 原始文件
│       │       └── default_settings.json
│       │
│       └── module.json5           # 模块配置文件
│
├── hvigor/                        # hvigor 构建工具
│
├── signature/                     # 签名文件目录
│
├── build-profile.json5            # 构建配置
│
├── hvigorfile.js                  # hvigor 入口脚本
│
├── hvigorw                        # hvigor 脚本（Linux/Mac）
│
├── hvigorw.bat                    # hvigor 脚本（Windows）
│
├── oh-package.json5               # 包管理配置
│
├── OAT.xml                        # 权限配置检查
│
└── LICENSE                        # Apache 2.0 许可证
```

---

## 模块职责详解

### entry/src/main/ets/Application

**职责**: 应用入口模块，提供 AbilityStage 生命周期管理

| 文件 | 职责 | 关键代码 |
|------|------|----------|
| `DataAbilityStage.ts` | DataAbility 的 Stage 入口，在应用启动时初始化上下文 | `onCreate()` 设置 `globalThis.abilityContext` |

**证据**: `entry/src/main/ets/Application/DataAbilityStage.ts:19-23`

```typescript
export default class DataAbilityStage extends AbilityStage {
  onCreate() :void {
    Log.info('DataAbilityStage onCreate');
    globalThis.abilityContext = this.context;
  }
}
```

---

### entry/src/main/ets/DataAbility

**职责**: DataAbility 核心实现，提供数据 CRUD 操作

| 文件 | 职责 | 关键函数 |
|------|------|----------|
| `DataExtAbility.ets` | 继承 DataShareExtensionAbility，实现 insert/update/query/delete | `onCreate()`, `insert()`, `update()`, `query()`, `delete()`, `verifyPermission()` |

**证据**: `entry/src/main/ets/DataAbility/DataExtAbility.ets:55-303`

**核心功能**:

1. **insert(uri, value, callback)** - 插入数据
   - 权限校验 → 执行系统设置 → 写入数据库

2. **update(uri, predicates, value, callback)** - 更新数据
   - 权限校验 → 执行系统设置 → 更新数据库

3. **query(uri, predicates, columns, callback)** - 查询数据
   - 直接查询数据库返回 ResultSet

4. **verifyPermission()** - 权限校验
   - 检查受信任列表或 `MANAGE_SECURE_SETTINGS` 权限

---

### entry/src/main/ets/StaticSubscriber

**职责**: 静态事件订阅，监听用户变更事件

| 文件 | 职责 | 关键事件 |
|------|------|----------|
| `UserChangeStaticSubscriber.ets` | 监听用户添加/删除事件，管理用户数据表 | `COMMON_EVENT_USER_ADDED`, `COMMON_EVENT_USER_REMOVED` |

**证据**: `entry/src/main/ets/StaticSubscriber/UserChangeStaticSubscriber.ets:27-79`

**事件处理**:

| 事件 | 处理逻辑 |
|------|----------|
| USER_ADDED | 创建用户数据表，加载默认数据 |
| USER_REMOVED | 删除用户数据表 |

---

### entry/src/main/ets/Utils

**职责**: 提供工具类支持

| 文件 | 职责 | 关键功能 |
|------|------|----------|
| `SettingsDBHelper.ets` | RDB 数据库初始化与管理 | `initRdbStore()`, `getRdbStore()`, `loadDefaultSettingsData()` |
| `GlobalContext.ets` | 全局上下文管理 | 单例模式存储全局对象 |
| `SettingsDataConfig.ets` | 配置常量定义 | 数据库名、表名、字段名 |
| `Log.ts` | 日志工具 | 基于 hilog 的日志输出 |
| `SettingsDataConfig.ets` | (重复) 配置常量 |

**SettingsDBHelper 核心功能** (`entry/src/main/ets/Utils/SettingsDBHelper.ets`):

1. **initRdbStore()** - 初始化 RDB 数据库
2. **getRdbStore()** - 获取数据库实例
3. **firstStartupConfig()** - 首次启动配置（创建表、加载默认数据）
4. **loadDefaultSettingsData()** - 从 JSON 文件加载默认设置
5. **getArea()** - 确定安全区域（EL1/EL2）

---

### entry/src/main/ets/common

**职责**: 公共类型定义

| 文件 | 职责 | 定义内容 |
|------|------|----------|
| `Common.ts` | 接口和枚举定义 | `IContent`, `TableType` |

**证据**: `entry/src/main/ets/common/Common.ts:16-26`

```typescript
export interface IContent {
  settings: Array<Map<string, string>>;
  user: Array<Map<string, string>>;
  userSecure: Array<Map<string, string>>;
}

export enum TableType {
  SETTINGS,
  USER,
  USER_SECURE
}
```

---

### entry/src/main/resources

**职责**: 资源文件目录

#### profile/ 配置文件

| 文件 | 职责 | 证据 |
|------|------|------|
| `data_share_config.json` | DataShare URI 配置与跨用户模式 | `entry/src/main/resources/base/profile/data_share_config.json` |
| `main_pages.json` | 页面配置 | `entry/src/main/resources/base/profile/main_pages.json` |
| `static_subscriber_config.json` | 静态订阅者配置 | `entry/src/main/resources/base/profile/static_subscriber_config.json` |

#### rawfile/ 原始文件

| 文件 | 职责 | 证据 |
|------|------|------|
| `default_settings.json` | 默认设置值配置 | `entry/src/main/resources/rawfile/default_settings.json` |

**default_settings.json 内容** (`entry/src/main/resources/rawfile/default_settings.json:2-31`):

```json
{
  "settings": [
    { "name": "settings.audio.ringtone", "value": "5" },
    { "name": "settings.audio.media", "value": "5" },
    { "name": "settings.audio.voicecall", "value": "5" },
    { "name": "settings.display.navigationbar_status", "value": "1" },
    { "name": "cellular_data_enable", "value": "1" },
    { "name": "settings.display.auto_screen_brightness", "value": "1" },
    { "name": "ota_disable_automatic_update", "value": "1" }
  ]
}
```

---

### entry/src/main/module.json5

**职责**: 模块配置文件，定义 ExtensionAbility

**证据**: `entry/src/main/module.json5:1-48`

**ExtensionAbility 配置**:

| 类型 | 名称 | URI | 权限 |
|------|------|-----|------|
| dataShare | DataExtAbility | `datashare://com.ohos.settingsdata.DataAbility` | `ohos.permission.MANAGE_SECURE_SETTINGS` |
| staticSubscriber | UserChangeStaticSubscriber | - | - |

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                    DataAbilityStage                          │
│                   (应用入口初始化)                            │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    DataExtAbility                            │
│              (DataShare 数据提供)                            │
│           ┌────────────────────────────────┐                 │
│           │ 依赖                            │                 │
│           ├────────────────────────────────┤                 │
│           │ • SettingsDBHelper (数据库)    │                 │
│           │ • SettingsDataConfig (配置)    │                 │
│           │ • Audio (系统设置)             │                 │
│           │ • abilityAccessCtrl (权限)     │                 │
│           └────────────────────────────────┘                 │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   SettingsDBHelper                           │
│                (RDB 数据库管理)                              │
│           ┌────────────────────────────────┐                 │
│           │ 依赖                            │                 │
│           ├────────────────────────────────┤                 │
│           │ • relationalStore (数据库)     │                 │
│           │ • preferences (首选项)         │                 │
│           │ • GlobalContext (上下文)       │                 │
│           │ • default_settings.json (默认) │                 │
│           └────────────────────────────────┘                 │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                UserChangeStaticSubscriber                    │
│              (用户变更事件监听)                               │
│           ┌────────────────────────────────┐                 │
│           │ 依赖                            │                 │
│           ├────────────────────────────────┤                 │
│           │ • commonEventManager (事件)    │                 │
│           │ • SettingsDBHelper (数据库)    │                 │
│           └────────────────────────────────┘                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 不含测试的说明

根据 Wiki 生成规范，本文档未包含以下测试相关目录：

- `test/` - 单元测试
- `tests/` - 集成测试
- `unittest/` - 单元测试
- `*_test.*` - 测试文件
- `*_fuzzer.*` - 模糊测试

---

## 相关文档

| 文档 | 链接 |
|------|------|
| 项目概览 | [01_Project_Overview.md](./01_Project_Overview.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| API 参考 | [04_API_Reference.md](./04_API_Reference.md) |
