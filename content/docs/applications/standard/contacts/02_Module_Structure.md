# 模块结构

## 模块概览

Contacts 应用由以下模块组成：

| 模块名 | 类型 | 路径 | 主要职责 |
|--------|------|------|----------|
| entry | entry | `entry/` | 主入口、UI 页面、组件 |
| common | feature | `common/` | 公共工具、权限管理 |
| contact | feature | `feature/contact/` | 联系人核心逻辑 |
| call | feature | `feature/call/` | 通话记录管理 |
| dialpad | feature | `feature/dialpad/` | 拨号盘功能 |
| phonenumber | feature | `feature/phonenumber/` | 电话号码处理 |
| account | feature | `feature/account/` | 账号管理 |

## entry 模块

**路径**: `entry/src/main/ets/`

### 目录结构

```
entry/src/main/ets/
├── Application/
│   └── MyAbilityStage.ts      # 应用 Stage 入口
├── MainAbility/
│   └── MainAbility.ts         # 主 Ability
├── StaticSubscriber/
│   └── StaticSubscriber.ts   # 静态事件订阅者
├── workers/
│   ├── base/
│   │   ├── Worker.ts         # Worker 基类
│   │   ├── WorkerTask.ts     # 任务基类
│   │   └── WorkerWrapper.ts  # Worker 包装器
│   ├── factory/
│   │   └── WorkFactory.ts    # Worker 工厂
│   ├── DataWorker.ts         # 数据 Worker
│   └── ...
├── model/
│   ├── contact/
│   │   ├── BatchSelectContactSource.ts
│   │   ├── SearchContactsSource.ts
│   │   └── ContactListDataSource.ts
│   ├── favorite/
│   │   ├── FavoriteDataSource.ts
│   │   └── ...
│   └── ...
├── pages/
│   ├── index/               # 主页面
│   ├── contact/             # 联系人相关页面
│   ├── dialer/              # 拨号页面
│   ├── favorite/           # 收藏页面
│   └── ...
├── presenter/
│   ├── PresenterManager.ts  # Presenter 管理器
│   ├── contact/
│   │   ├── ContactListPresenter.ts
│   │   ├── DetailPresenter.ts
│   │   ├── BatchSelectContactsPresenter.ts
│   │   └── ...
│   ├── dialer/
│   │   ├── DialerPresenter.ts
│   │   └── CallRecordPresenter.ts
│   └── ...
├── component/
│   ├── contact/
│   │   ├── ContactListItemView.ts
│   │   ├── BatchSelectContactItemView.ts
│   │   └── ...
│   ├── contactdetail/
│   │   ├── DetailInfoList.ts
│   │   ├── DetailCalllog.ts
│   │   └── ...
│   └── ...
└── util/
    ├── StringFormatUtil.ts
    ├── CalendarUtil.ts
    └── ...
```

### 关键组件

| 组件 | 职责 | 证据路径 |
|------|------|----------|
| `MainAbility.ts` | 应用入口，处理生命周期 | line 29 |
| `PresenterManager.ts` | Presenter 工厂和管理 | line 23 |
| `MyAbilityStage.ts` | Stage 模型入口 | - |
| `StaticSubscriber.ts` | 系统事件订阅 | - |

## feature/contact 模块

**路径**: `feature/contact/src/main/ets/`

### 核心子目录

```
feature/contact/src/main/ets/
├── contract/               # 数据契约层
│   ├── Contacts.ets        # 联系人 URI 定义
│   ├── ContactsColumns.ets # 联系人表字段
│   ├── Data.ets
│   ├── DataColumns.ets
│   ├── RawContacts.ets
│   ├── Email.ets
│   ├── Phone.ets
│   ├── Nickname.ets
│   ├── Birthday.ets
│   ├── Im.ets
│   ├── Organization.ets
│   ├── StructuredPostal.ets
│   └── ... (20+ 契约文件)
│
├── entity/                # 领域实体层
│   ├── Contact.ets        # 联系人实体
│   ├── ContactBuilder.ets # 联系人构建器
│   ├── RawContact.ets     # 原始联系人
│   ├── DataItem.ets       # 数据项
│   ├── SearchContact.ets  # 搜索联系人
│   ├── PhoneDataItem.ets
│   ├── EmailDataItem.ets
│   └── ... (10+ 实体文件)
│
└── repo/                  # 数据仓库层
    ├── ContactRepository.ets    # 联系人数据仓库
    ├── IContactRepository.ets   # 仓库接口
    ├── ContactList.ets
    ├── ContactDelta.ets
    ├── ContactUsuallyListItem.ets
    ├── FavoriteListItem.ets
    └── ... (10+ 仓库文件)
```

### 数据契约 (Contract)

| 契约文件 | 对应表/数据类型 | 说明 |
|----------|----------------|------|
| `Contacts.ets` | contacts | 联系人主表 URI 定义 |
| `ContactsColumns.ets` | contacts | 联系人表字段 |
| `RawContacts.ets` | raw_contacts | 原始联系人 |
| `RawContactsColumns.ets` | raw_contacts | 原始联系人字段 |
| `Data.ets` | data | 数据项通用 |
| `DataColumns.ets` | data | 数据项字段 |
| `Phone.ets` | phone | 电话号码 |
| `Email.ets` | email | 邮箱地址 |
| `Nickname.ets` | nickname | 昵称 |
| `Birthday.ets` | birthday | 生日 |
| `Im.ets` | im | 即时通讯 |
| `Organization.ets` | organization | 组织信息 |
| `StructuredPostal.ets` | postal | 结构化地址 |
| `Event.ets` | event | 事件 |
| `Relation.ets` | relation | 关系 |
| `Note.ets` | note | 备注 |
| `Website.ets` | website | 网站 |
| `Aim.ets` | aim | AIM |
| `House.ets` | house | 房屋 |
| `SearchContacts.ets` | - | 搜索结果 |

> **证据来源**: `feature/contact/src/main/ets/contract/` 目录 (24 个文件)

### 领域实体 (Entity)

| 实体类 | 职责 | 证据路径 |
|--------|------|----------|
| `Contact.ets` | 联系人聚合实体 | - |
| `ContactBuilder.ets` | 构建联系人对象 | - |
| `RawContact.ets` | 原始联系人数据 | - |
| `DataItem.ets` | 数据项基类 | - |
| `SearchContact.ets` | 搜索结果实体 | - |

> **证据来源**: `feature/contact/src/main/ets/entity/` 目录 (14 个文件)

### 数据仓库 (Repository)

| 仓库类 | 职责 | 证据路径 |
|--------|------|----------|
| `ContactRepository.ets` | 联系人 CRUD 操作 | `repo/ContactRepository.ets` |
| `IContactRepository.ets` | 仓库接口定义 | `repo/IContactRepository.ets` |
| `DAOperation.ets` | DataAbility 操作封装 | `repo/DAOperation.ets` |

## feature 模块

### call 模块

**路径**: `feature/call/src/main/ets/`

**职责**: 通话记录管理

| 子模块 | 职责 |
|--------|------|
| missedcall/ | 未接来电管理 |
| callrecord/ | 通话记录 |

### dialpad 模块

**路径**: `feature/dialpad/src/main/ets/`

**职责**: 拨号盘 UI 和逻辑

| 子模块 | 职责 |
|--------|------|
| dialer/ | 拨号器 |

### phonenumber 模块

**路径**: `feature/phonenumber/src/main/ets/`

**职责**: 电话号码处理工具

### account 模块

**路径**: `feature/account/src/main/ets/`

**职责**: SIM 卡账号管理

## common 模块

**路径**: `common/src/main/ets/`

### 目录结构

```
common/src/main/ets/
├── Constants.ets           # 常量定义
├── permission/
│   └── PermissionManager.ts  # 权限管理
└── util/
    ├── ArrayUtil.ts        # 数组工具
    ├── HiLog.ts            # 日志工具
    ├── StringUtil.ts       # 字符串工具
    ├── ObjectUtil.ts       # 对象工具
    └── SharedPreferencesUtils.ts  # 首选项工具
```

### 关键公共组件

| 组件 | 职责 | 证据路径 |
|------|------|----------|
| `PermissionManager.ts` | 权限请求和校验 | `permission/PermissionManager.ets` |
| `HiLog.ts` | 日志输出 | `util/HiLog.ets` |
| `Constants.ets` | 全局常量 | `Constants.ets` |

### PermissionManager

```typescript
// 证据来源: common/src/main/ets/permission/PermissionManager.ets
export class PermissionManager {
  // 单例模式
  static getInstance(): PermissionManager
  
  // 权限检查
  isAllPermissionsGranted(): boolean
  
  // 权限请求
  async initPermissions(): void
}
```

### Constants

```typescript
// 证据来源: common/src/main/ets/Constants.ets
class EventClass {
  NEW_WANT: number          // 新 Intent 事件
}

class StorageClass {
  mainTabsIndex: string     // 主 Tab 索引
  teleNumber: string        // 电话号码
  targetPage: string        // 目标页面
}

class ConfigClass {
  useDataWorker: boolean    // 是否使用 Worker
  needCache: boolean       // 是否需要缓存
}
```

## 模块依赖关系

### 依赖图

```
                    ┌─────────────┐
                    │   entry     │  (入口层)
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │   common    │  │  feature/*  │  │   workers   │
   │  (工具层)    │  │  (功能层)    │  │  (线程层)   │
   └─────────────┘  └──────┬──────┘  └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   Native    │  (@ohos.* APIs)
                    │    APIs     │
                    └─────────────┘
```

### 依赖规则

| 依赖方向 | 规则 | 例子 |
|----------|------|------|
| entry → feature | ✅ 允许 | pages 依赖 presenters |
| feature → entry | ❌ 禁止 | 功能模块不应依赖 UI |
| entry → common | ✅ 允许 | MainAbility 使用 HiLog |
| common → entry | ❌ 禁止 | 公共模块不应依赖具体 UI |
| feature → feature | ⚠️ 谨慎 | 仅必要时通过接口依赖 |

### 稳定接口

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `IContactRepository` | 稳定 | 数据访问接口 |
| `Contact` 实体 | 稳定 | 领域模型 |
| `Constants` | 稳定 | 全局常量 |
| `PermissionManager` | 稳定 | 权限管理 |

### 可替换点

| 可替换组件 | 替换方式 | 影响范围 |
|------------|----------|----------|
| `ContactRepository` | 实现 `IContactRepository` 接口 | 联系人数据访问 |
| `Presenter` | 实现 Presenter 接口 | 业务逻辑 |
| `Worker` | 替换 WorkFactory | 线程模型 |
