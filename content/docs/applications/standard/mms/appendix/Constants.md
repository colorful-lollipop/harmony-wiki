# 附录 B: 常量定义

## 目的与适用范围

本文档列出 MMS 应用中使用的关键常量定义，便于开发者查阅。

**适用读者**: 开发工程师  
**阅读时间**: 约 10 分钟

---

## 常量文件位置

主要常量定义在: `entry/src/main/ets/data/commonData.ets`

---

## 状态码

### 通用状态码

| 常量名 | 值 | 说明 |
|--------|-----|------|
| SUCCESS | 0 | 操作成功 |
| FAILURE | -1 | 操作失败 |

### 短信发送状态

| 常量名 | 值 | 说明 |
|--------|-----|------|
| SEND_MESSAGE_SUCCESS | 0 | 发送成功 |
| SEND_MESSAGE_SENDING | 1 | 发送中 |
| SEND_MESSAGE_FAILED | 2 | 发送失败 |
| SEND_DRAFT | 3 | 草稿 |

### 读取状态

| 常量名 | 值 | 说明 |
|--------|-----|------|
| is_read.UN_READ | 0 | 未读 |
| is_read.READ | 1 | 已读 |

### 锁定状态

| 常量名 | 值 | 说明 |
|--------|-----|------|
| is_lock.NO | 0 | 未锁定 |
| is_lock.YES | 1 | 已锁定 |

### 收藏状态

| 常量名 | 值 | 说明 |
|--------|-----|------|
| is_collect.NOT_FAVORITE | 0 | 未收藏 |
| is_collect.FAVORITE | 1 | 已收藏 |

### 发送报告状态

| 常量名 | 值 | 说明 |
|--------|-----|------|
| is_send_report.NO | 0 | 无送达报告 |
| is_send_report.YES | 1 | 有送达报告 |

### 发送方标识

| 常量名 | 值 | 说明 |
|--------|-----|------|
| is_sender.NO | 0 | 接收方 |
| is_sender.YES | 1 | 发送方 |

---

## 消息类型

### 消息展示类型

| 常量名 | 值 | 说明 |
|--------|-----|------|
| MESSAGE_SHOW_TYPE.NORMAL | 0 | 普通短信样式 |
| MESSAGE_SHOW_TYPE.THEME_NO_IMAGE | 1 | 无图主题样式 |
| MESSAGE_SHOW_TYPE.PPT_NO_IMAGE | 2 | 无图幻灯片样式 |
| MESSAGE_SHOW_TYPE.PPT_IMAGE | 3 | 带图幻灯片样式 |
| MESSAGE_SHOW_TYPE.THEME_IMAGE | 4 | 带图主题样式 |

### 消息内容类型

| 常量名 | 值 | 说明 |
|--------|-----|------|
| MSG_ITEM_TYPE.THEME | 0 | 主题 |
| MSG_ITEM_TYPE.IMAGE | 1 | 图片 |
| MSG_ITEM_TYPE.VIDEO | 2 | 视频 |
| MSG_ITEM_TYPE.AUDIO | 3 | 音频 |
| MSG_ITEM_TYPE.TEXT | 4 | 文本 |
| MSG_ITEM_TYPE.CARD | 5 | 名片 |

### SMS 类型

| 常量名 | 值 | 说明 |
|--------|-----|------|
| sms_type.COMMON | 0 | 普通短信 |
| sms_type.NOTICE | 1 | 通知短信 |

### 会话类型

| 常量名 | 值 | 说明 |
|--------|-----|------|
| session_type.COMMON | 0 | 普通会话 |
| session_type.BROADCAST | 1 | 广播 |
| session_type.GROUP_SEND | 2 | 群发 |

### 消息类型 (数据库)

| 常量名 | 值 | 说明 |
|--------|-----|------|
| MESSAGE_TYPE.NORMAL | 0 | 普通信息 |
| MESSAGE_TYPE.THEME | 1 | 主题彩信 |
| MESSAGE_TYPE.PPT | 2 | 幻灯片 |
| MESSAGE_TYPE.THEME_AND_PPT | 3 | 主题和幻灯片 |

---

## URI 定义

### 短信数据库

| 常量名 | 值 |
|--------|-----|
| URI_MESSAGE_LOG | `datashare:///com.ohos.smsmmsability` |
| URI_MESSAGE_INFO_TABLE | `/sms_mms/sms_mms_info` |
| URI_MESSAGE_SESSION_TABLE | `/sms_mms/session` |
| URI_MESSAGE_UNREAD_COUNT | `/sms_mms/sms_mms_info/unread_total` |
| URI_MESSAGE_MAX_GROUP | `/sms_mms/sms_mms_info/max_group` |
| URI_MESSAGE_MMS_PART | `/sms_mms/mms_part` |

### 联系人数据库

| 常量名 | 值 |
|--------|-----|
| URI_ROW_CONTACTS | `datashare:///com.ohos.contactsdataability` |
| CONTACT_DATA_URI | `/contacts/contact_data` |
| PROFILE_DATA_URI | `/profile/raw_contact` |
| CONTACT_SEARCHE | `/contacts/search_contact` |
| CONTACT_URI | `/contacts/contact` |

---

## 配置键值

### 偏好设置键

| 常量名 | 用途 |
|--------|------|
| KEY_OF_INTEGRATION_SWITCH | 通知信息整合开关 |
| KEY_OF_MALICIOUS_WEB_SWITCH | 恶意网站识别开关 |
| KEY_OF_SHOW_CONTACT_SWITCH | 显示联系人头像开关 |
| KEY_OF_DELIVERY_REPORT_SWITCH | 送达报告开关 |
| KEY_OF_AUTO_RETRIEVE_SWITCH | 自动下载 MMS 开关 |
| KEY_OF_RECALL_MESSAGE_SWITCH | 取消发送开关 |
| KEY_OF_AUTO_DELETE_INFO_SWITCH | 自动删除通知开关 |
| KEY_OF_SIM_COUNT | SIM 卡数量 |
| KEY_OF_SIM_0_SPN | SIM 1 运营商名 |
| KEY_OF_SIM_1_SPN | SIM 2 运营商名 |
| KEY_OF_SIM_0_EXIST_FLAG | SIM 1 存在标志 |
| KEY_OF_SIM_1_EXIST_FLAG | SIM 2 存在标志 |
| KEY_OF_SIM_0_NUMBER | SIM 1 号码 |
| KEY_OF_SIM_1_NUMBER | SIM 2 号码 |
| KEY_OF_DEFAULT_SLOT | 默认卡槽 |
| KEY_OF_HAVE_MULTI_SIM_CARD_READY | 多卡就绪标志 |
| KEY_OF_HAVE_SIM_CARD_READY | 单卡就绪标志 |
| KEY_OF_NEW_SIM_0_SMSC | SIM 1 短信中心号 |
| KEY_OF_NEW_SIM_1_SMSC | SIM 2 短信中心号 |

---

## 事件定义

### 公共事件

| 常量名 | 值 |
|--------|-----|
| SUBSCRIBER_EVENT | `usual.event.SMS_RECEIVE_COMPLETED` |
| RECEIVE_TRANSMIT_EVENT | `usual.event.RECEIVE_COMPLETED_TRANSMIT` |
| MMS_SUBSCRIBER_EVENT | `usual.event.MMS_RECEIVE_COMPLETED` |

### Emitter 事件 ID

| 常量名 | 值 | 说明 |
|--------|-----|------|
| EVENT_SIM_STATE_CHANGE | 1 | SIM 状态变化 |
| EVENT_SLOTID_CHANGE | 2 | 卡槽变化 |

---

## 应用信息

| 常量名 | 值 |
|--------|-----|
| BUNDLE_NAME | `com.ohos.mms` |
| ABILITY_NAME | `com.ohos.mms.MainAbility` |
| CONTACT_BUNDLE_NAME | `com.ohos.contacts` |
| CONTACT_ABILITY_NAME | `com.ohos.contacts.MainAbility` |

---

## 其他常量

### 数值常量

| 常量名 | 值 | 说明 |
|--------|-----|------|
| SIM_COUNT | 2 | SIM 卡数量 |
| SIM_ONE | 0 | SIM 卡 1 |
| SIM_TWO | 1 | SIM 卡 2 |
| FULL_SCREEN_SEND_LENGTH | 38 | 全屏发送字数 |
| CANCEL_TIME_COUNT | 6 | 取消发送倒计时 |

### 字符串常量

| 常量名 | 值 |
|--------|-----|
| EMPTY_STR | `""` |
| COMMA | `,` |

### MMS URL

| 常量名 | 值 |
|--------|-----|
| MMS_URL | `http://mmsc.monternet.com` |

---

## 相关链接

- [目录结构](../02_DirectoryStructure.md) - 代码组织
- [数据流](../04_DataFlow.md) - 数据流转

---

*常量定义基于: entry/src/main/ets/data/commonData.ets*
