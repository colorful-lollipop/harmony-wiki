# 07. 编译产物

> 目的: 详细说明ScreenLock的编译产物结构和安装部署  
> 适用范围: 构建工程师、部署人员

---

## 1. 产物概述

### 1.1 产物类型

| 类型 | 扩展名 | 数量 | 说明 |
|------|--------|------|------|
| HAP | `.hap` | 3个 | Harmony Ability Package |
| HAR | `.har` | 10个 | Harmony Archive |

### 1.2 产物清单

| 产物名称 | 模块 | 类型 | 说明 |
|----------|------|------|------|
| `entry-default.hap` | entry | HAP | 应用入口包 |
| `phone-default.hap` | phone | HAP | 手机端锁屏 |
| `pc-default.hap` | pc | HAP | PC端锁屏 |
| `common.har` | common | HAR | 公共库 |
| `screenlock.har` | screenlock | HAR | 锁屏核心库 |
| `noticeitem.har` | noticeitem | HAR | 通知库 |
| `datetimecomponent.har` | datetimecomponent | HAR | 日期时间库 |
| `wallpapercomponent.har` | wallpapercomponent | HAR | 壁纸库 |
| `shortcutcomponent.har` | shortcutcomponent | HAR | 快捷操作库 |
| `batterycomponent.har` | batterycomponent | HAR | 电池库 |
| `signalcomponent.har` | signalcomponent | HAR | 信号库 |
| `wificomponent.har` | wificomponent | HAR | WiFi库 |
| `clockcomponent.har` | clockcomponent | HAR | 时钟库 |

---

## 2. HAP包结构

### 2.1 HAP包内部结构

```
entry-default.hap (ZIP格式)
├── ets/                     # ArkTS/ArkUI编译产物
│   ├── MainAbility/         # MainAbility编译代码
│   ├── pages/               # 页面编译代码
│   └── default/             # 默认路由
├── resources/               # 资源文件
│   ├── base/
│   │   ├── element/         # 字符串、颜色、尺寸
│   │   ├── media/           # 图片资源
│   │   └── profile/         # 配置文件
│   └── rawfile/             # 原始文件
├── config.json / module.json5  # 模块配置
├── pack.info                # 包信息
└── signature/               # 签名信息
    └── systemui.p7b
```

### 2.2 HAP包对比

| 特性 | entry.hap | phone.hap | pc.hap |
|------|-----------|-----------|--------|
| **模块类型** | entry | feature | feature |
| **主Ability** | MainAbility (UIAbility) | ServiceExtAbility | ServiceExtAbility |
| **设备类型** | phone, tablet | phone | - |
| **权限数量** | 0 | 21 | 22 |
| **包含页面** | 1个 | 7个 | 7个 |
| **尺寸** | 最小 | 中等 | 中等 |

---

## 3. HAR包结构

### 3.1 HAR包内部结构

```
common.har (ZIP格式)
├── ets/                     # TypeScript声明和编译代码
│   ├── components/          # 组件定义
│   └── default/             # 源码编译产物
│       ├── abilitymanager/
│       ├── commonEvent/
│       ├── event/
│       └── ...
├── resources/               # 公共资源
├── index.ets / index.d.ts   # 入口文件
├── module.json5             # 模块配置
└── oh-package.json5         # 包配置
```

### 3.2 HAR包用途

| HAR包 | 主要导出 | 被谁依赖 |
|-------|----------|----------|
| common | EventManager, WindowManager, Log等 | entry, phone, pc, features/* |
| screenlock | ScreenLockService, ScreenLockModel等 | phone, pc |
| noticeitem | NotificationManager等 | phone, pc |
| 其他features | 各自组件 | phone, pc |

---

## 4. 安装路径

### 4.1 系统应用安装路径

| 路径 | 说明 |
|------|------|
| `/system/app/com.ohos.systemui/` | 系统应用安装目录 |
| `/system/app/com.ohos.systemui/entry.hap` | Entry HAP |
| `/system/app/com.ohos.systemui/phone.hap` | Phone HAP |
| `/system/app/com.ohos.systemui/pc.hap` | PC HAP |

### 4.2 应用数据目录

| 路径 | 说明 |
|------|------|
| `/data/app/el1/100/base/com.ohos.systemui/` | 应用数据目录 |
| `/data/app/el1/100/base/com.ohos.systemui/files/` | 文件存储 |
| `/data/app/el1/100/base/com.ohos.systemui/preferences/` | 偏好设置 |

---

## 5. 运行时加载关系

### 5.1 启动加载链

```
系统启动
    │
    ▼
系统服务加载
    │
    ▼
AMS (Ability Manager Service)
    │
    ▼
启动 com.ohos.systemui ServiceExtAbility
    │
    ├── 加载 phone.hap
    │       ├── 加载依赖的 HAR
    │       │       ├── common.har
    │       │       ├── screenlock.har
    │       │       ├── noticeitem.har
    │       │       └── ...
    │       │
    │       └── 执行 ServiceExtAbility.onCreate()
    │               ├── 创建锁屏窗口
    │               └── 加载 pages/index
    │
    └── （可选）加载 entry.hap
```

### 5.2 模块依赖加载

```
phone.hap
    ├── common.har
    │   └── @ohos/hilog
    ├── screenlock.har
    │   ├── common.har
    │   └── @ohos.screenLock
    ├── noticeitem.har
    │   ├── common.har
    │   └── @ohos.notification
    ├── datetimecomponent.har
    │   └── common.har
    ├── wallpapercomponent.har
    │   ├── common.har
    │   └── @ohos.wallpaper
    ├── shortcutcomponent.har
    │   ├── common.har
    │   └── @ohos.power
    ├── batterycomponent.har
    │   ├── common.har
    │   └── @ohos.batteryInfo
    ├── signalcomponent.har
    │   ├── common.har
    │   └── @ohos.telephony.*
    ├── wificomponent.har
    │   ├── common.har
    │   └── @ohos.wifi
    └── clockcomponent.har
        └── common.har
```

---

## 6. 产物大小分析

### 6.1 预估大小

| 产物 | 预估大小 | 说明 |
|------|----------|------|
| entry.hap | ~500KB | 仅入口Ability |
| phone.hap | ~2MB | 包含所有锁屏页面和逻辑 |
| pc.hap | ~2MB | 类似phone |
| common.har | ~200KB | 通用工具 |
| screenlock.har | ~500KB | 锁屏核心 |
| 其他features.har | ~100-200KB | 各功能组件 |

### 6.2 总大小估算

| 场景 | 大小 |
|------|------|
| 仅Entry | ~500KB |
| Phone完整安装 | ~5MB |
| PC完整安装 | ~5MB |
| 全部安装 | ~10MB |

---

## 7. 部署方式

### 7.1 系统预置部署

ScreenLock作为系统预置应用，随系统镜像一起发布：

```bash
# 系统镜像构建时
/system/app/com.ohos.systemui/
    ├── entry.hap
    ├── phone.hap
    └── pc.hap
```

### 7.2 开发调试部署

```bash
# 使用hdc工具安装
hdc install entry.hap
hdc install phone.hap
hdc install pc.hap

# 或推送到系统目录（需root）
hdc shell mount -o remount,rw /
hdc file send phone.hap /system/app/com.ohos.systemui/
```

---

## 8. 版本管理

### 8.1 版本号定义

**文件**: `AppScope/app.json5`

```json5
{
  "app": {
    "versionCode": 1000000,
    "versionName": "1.0.0"
  }
}
```

### 8.2 版本号规则

| 字段 | 格式 | 示例 |
|------|------|------|
| versionCode | 整数 | 1000000 |
| versionName | 主.次.修订 | 1.0.0 |

---

## 9. 产物验证

### 9.1 签名验证

```bash
# 验证HAP签名
hdc shell bm dump -n com.ohos.systemui
```

### 9.2 完整性检查

| 检查项 | 方法 |
|--------|------|
| HAP完整性 | ZIP解压检查 |
| 签名有效性 | 使用签名工具验证 |
| 配置正确性 | 解析module.json5 |

---

*关键结论: ScreenLock编译产物包括3个HAP包（entry、phone、pc）和10个HAR库包，总大小约10MB。作为系统预置应用安装在/system/app/com.ohos.systemui/目录。*
