# 02. 目录结构

## 目的与适用范围

本文档描述 MMS 项目的目录组织和代码文件分布，帮助开发者快速定位代码。

**适用读者**: 所有开发者  
**阅读时间**: 约 10 分钟

---

## 顶层目录

```
/Volumes/lexar/code/d/work/oh/applications/standard/mms/
├── AppScope/                    # 应用级配置
│   └── resources/               # 应用级资源
├── doc/                         # 文档资料
│   └── image/                   # 图片资源
├── entry/                       # 主模块 (Entry Module)
│   ├── src/main/                # 源码主目录
│   └── ...                      # 配置文件
├── signs/                       # 签名文件
├── wiki/                        # 工程 Wiki (本文档)
├── build-profile.json5          # 构建配置
├── hvigorfile.js                # Hvigor 构建脚本
├── LICENSE                      # 许可证
├── OAT.xml                      # 开源合规
├── oh-package.json5             # 包配置
└── README.md                    # 项目说明
```

---

## 源代码目录 (entry/src/main/ets/)

### 总体结构

```
entry/src/main/ets/
├── Application/                 # AbilityStage
│   └── MyAbilityStage.ts        # 应用生命周期
├── MainAbility/                 # 主 Ability
│   └── MainAbility.ts           # UIAbility 实现
├── StaticSubscriber/            # 静态事件订阅
│   └── MmsStaticSubscriber.ts   # 短信接收处理
├── data/                        # 数据类型定义
│   ├── commonData.ets           # 常量定义
│   ├── tableData.ets            # 表结构定义
│   ├── LooseObject.ets          # 通用对象类型
│   └── ...
├── model/                       # 数据模型层
│   ├── BaseModel.ets            # 模型基类
│   ├── ConversationModel.ets    # 短信详情模型
│   ├── ConversationListModel.ets # 会话列表模型
│   ├── ContactsModel.ets        # 联系人模型
│   ├── CardModel.ets            # SIM 卡模型
│   └── ...
├── pages/                       # UI 页面
│   ├── index.ets                # 首页
│   ├── conversation/            # 会话详情页
│   ├── conversationlist/        # 会话列表组件
│   ├── infomsg/                 # 通知消息页
│   ├── settings/                # 设置页
│   ├── queryreport/             # 送达报告页
│   └── transmitmsg/             # 转发消息页
├── service/                     # 业务逻辑层
│   ├── SendMsgService.ets       # 短信发送服务
│   ├── ConversationService.ets  # 会话详情服务
│   ├── ConversationListService.ets # 会话列表服务
│   ├── NotificationService.ets  # 通知服务
│   ├── ContactsService.ets      # 联系人服务
│   └── ...
├── utils/                       # 工具类
│   ├── HiLog.ets                # 日志工具
│   ├── MmsPreferences.ets       # 偏好设置
│   ├── MmsDatabaseHelper.ets    # 数据库帮助类
│   ├── TelephoneUtil.ets        # 电话工具
│   └── ...
├── views/                       # 视图组件
│   ├── MmsListItem.ets          # 列表项组件
│   ├── MmsDialogs.ets           # 对话框组件
│   ├── receive/                 # 接收消息视图
│   └── ...
└── workers/                     # Worker 线程
    ├── DataWorkerWrapper.ets    # 数据 Worker 包装
    ├── WorkFactory.ets          # Worker 工厂
    └── base/                    # Worker 基类
```

---

## 按职责归类

### 1. 生命周期与入口

| 文件 | 职责 |
|------|------|
| `Application/MyAbilityStage.ts` | AbilityStage 生命周期，应用启动时初始化通知 |
| `MainAbility/MainAbility.ts` | 主 Ability 生命周期，页面加载，全局错误处理 |
| `StaticSubscriber/MmsStaticSubscriber.ts` | 静态订阅者，接收系统短信广播 |

### 2. 页面层 (UI)

| 目录/文件 | 职责 |
|-----------|------|
| `pages/index.ets` | 应用首页，包含会话列表 |
| `pages/indexController.ets` | 首页逻辑控制 |
| `pages/conversationlist/` | 会话列表组件和控制器 |
| `pages/conversation/` | 会话详情页（聊天界面）|
| `pages/infomsg/` | 通知消息列表页 |
| `pages/settings/` | 设置相关页面（3个子页面）|
| `pages/queryreport/` | 送达报告查询页 |
| `pages/transmitmsg/` | 消息转发页 |

### 3. 服务层 (业务逻辑)

| 文件 | 行数 | 职责 | 关键方法 |
|------|------|------|----------|
| `SendMsgService.ets` | 96 | 发送短信/MMS | sendMessage, sendMmsMessage |
| `ConversationService.ets` | 613 | 会话详情管理 | insertSmsMmsInfo, queryMessageDetail |
| `ConversationListService.ets` | ~400 | 会话列表管理 | insertSession, querySessionByCondition |
| `NotificationService.ets` | 174 | 通知管理 | sendNotify, cancelNotify |
| `ContactsService.ets` | ~200 | 联系人查询 | queryContactDataByCondition |
| `CallService.ets` | ~50 | 拨打电话 | makePhoneCall |
| `SimCardService.ets` | ~150 | SIM 卡状态 | init, deInit |
| `SettingService.ets` | ~100 | 设置管理 | 设置项读写 |
| `CommonService.ets` | ~50 | 通用服务 | getMmsContent |
| `ContractService.ets` | ~50 | 合约查询 | 运营商相关 |

### 4. 模型层 (数据访问)

| 文件 | 职责 | 数据库 URI |
|------|------|------------|
| `ConversationModel.ets` | 短信详情 CRUD | sms_mms_info 表 |
| `ConversationListModel.ets` | 会话列表 CRUD | session 表 |
| `ContactsModel.ets` | 联系人查询 | contacts 表 |
| `CardModel.ets` | SIM 卡信息管理 | preferences |
| `SettingsModel.ets` | 设置数据读写 | telephony.sms |
| `BaseModel.ets` | 模型基类 | - |

### 5. 数据定义

| 文件 | 内容 |
|------|------|
| `commonData.ets` | 常量定义（状态码、URI、配置键）|
| `tableData.ets` | 数据库表字段名常量 |
| `LooseObject.ets` | 通用对象类型定义 |
| `MmsBoolean.ets` | 布尔值包装类 |
| `Pasteboard.ets` | 剪贴板数据类型 |

### 6. 工具类

| 文件 | 职责 |
|------|------|
| `HiLog.ets` | 日志封装，基于 @ohos.hilog |
| `MmsPreferences.ets` | SharedPreferences 封装 |
| `MmsDatabaseHelper.ets` | 数据库创建和升级 |
| `TelephoneUtil.ets` | 电话号码格式化、验证 |
| `DateUtil.ets` | 日期时间格式化 |
| `StringUtil.ets` | 字符串处理 |
| `DeviceUtil.ets` | 设备信息获取 |
| `WantUtil.ets` | Want 参数解析 |

### 7. 视图组件

| 文件 | 职责 |
|------|------|
| `MmsListItem.ets` | 消息列表项组件 |
| `MmsDialogs.ets` | 对话框组件封装 |
| `MmsMenu.ets` | 菜单组件 |
| `MultiSimCardMenu.ets` | 双卡选择菜单 |
| `SettingItem.ets` | 设置项组件 |
| `receive/receive.ets` | 接收消息气泡视图 |

### 8. Worker 线程

| 文件 | 职责 |
|------|------|
| `DataWorkerWrapper.ets` | 数据操作 Worker 封装 |
| `WorkFactory.ets` | Worker 实例工厂 |
| `base/Worker.ts` | Worker 基类 |
| `base/WorkerTask.ts` | 任务基类 |
| `base/WorkerWrapper.ts` | Worker 包装基类 |

---

## 资源文件 (entry/src/main/resources/)

```
resources/
├── base/                        # 基础资源
│   ├── element/                 # 元素资源
│   │   ├── color.json           # 颜色定义
│   │   ├── float.json           # 尺寸定义
│   │   └── string.json          # 字符串
│   ├── media/                   # 媒体资源
│   │   └── icon/                # 图标
│   ├── profile/                 # 配置文件
│   │   ├── main_pages.json      # 页面路由
│   │   └── static_subscriber_config.json # 静态订阅者配置
│   └── theme.json               # 主题配置
├── en_US/                       # 英文资源
├── zh_CN/                       # 中文资源
└── rawfile/                     # 原始文件
    └── icon/                    # 图标资源
```

---

## 配置文件

### 模块配置 (entry/src/main/module.json5)

```json
{
  "module": {
    "name": "entry",
    "type": "entry",
    "srcEntry": "./ets/Application/MyAbilityStage.ts",
    "mainElement": "com.ohos.mms.MainAbility",
    "pages": "$profile:main_pages",
    "abilities": [...],
    "extensionAbilities": [...],
    "requestPermissions": [...]
  }
}
```

**关键配置项**:
- `mainElement`: 主 Ability 名称
- `pages`: 页面配置文件引用
- `requestPermissions`: 权限声明（10 项系统权限）

### 页面路由 (resources/base/profile/main_pages.json)

```json
{
  "src": [
    "pages/index",
    "pages/conversation/conversation",
    "pages/infomsg/InfoMsg",
    "pages/settings/settings",
    "pages/settings/advancedSettings/advancedSettings",
    "pages/settings/ringtoneSettings/ringtoneSettings",
    "pages/queryreport/queryReport",
    "pages/transmitmsg/transmitMsg"
  ]
}
```

---

## 构建配置

### 项目级配置

| 文件 | 用途 |
|------|------|
| `build-profile.json5` | 编译 SDK 版本、签名配置 |
| `hvigorfile.js` | Hvigor 构建脚本 |
| `oh-package.json5` | 包依赖声明 |

### Entry 模块配置

| 文件 | 用途 |
|------|------|
| `entry/build-profile.json5` | 模块构建设置 |
| `entry/hvigorfile.js` | 模块构建脚本 |
| `entry/oh-package.json5` | 模块包配置 |

---

## 相关链接

- [架构设计](01_Architecture.md) - 了解模块间关系
- [系统 API](03_SystemAPIs.md) - 查看外部依赖
- [附录 A: 调用链](appendix/Callgraphs.md) - 查看调用关系

---

*文件统计: 52 个 ets/ts 源文件*
