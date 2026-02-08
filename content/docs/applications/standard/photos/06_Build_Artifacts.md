# 编译产物

## 1. 产物概述

### 1.1 编译产物类型

| 产物类型 | 说明 | 生成模块 |
|----------|------|----------|
| **HAP** | Harmony Ability Package，应用安装包 | phone_photos (entry) |
| **HAR** | Harmony Archive，静态共享库 | common, browser, editor, formAbility, thirdselect, timeline |

---

## 2. HAP 产物

### 2.1 文件信息

| 属性 | 值 |
|------|-----|
| **产物名** | `photos.hap` |
| **类型** | Harmony Ability Package |
| **入口模块** | phone_photos |
| **模块类型** | entry |

### 2.2 输出路径

```
# Debug 产物
./build/outputs/hap/debug/phone_photos/default/

# Release 产物
./build/outputs/hap/release/phone_photos/default/
```

### 2.3 HAP 结构

```
photos.hap/
├── META-INF/
│   ├── MANIFEST.MF          # 清单文件
│   └── CERT.SF              # 签名信息
├── libs/                     # 依赖库
├── module.json5              # 模块配置
├── resources.index           # 资源索引
├── resources/                # 编译后资源
│   ├── base/
│   │   ├── element/         # 元素资源
│   │   ├── layout/          # 布局资源
│   │   └── media/           # 媒体资源
│   └── rawfile/             # 原始文件
└── ets/                      # ArkTS 字节码
    ├── Application/         # 应用入口
    ├── MainAbility/         # 主能力
    ├── pages/               # 页面
    └── ...
```

---

## 3. HAR 产物

### 3.1 HAR 清单

| 模块名 | 类型 | 产物名 |
|--------|------|--------|
| photos_common | HAR | `photos_common.har` |
| photos_browser | HAR | `photos_browser.har` |
| photos_editor | HAR | `photos_editor.har` |
| photos_formAbility | HAR | `photos_formAbility.har` |
| photos_thirdselect | HAR | `photos_thirdselect.har` |
| photos_timeline | HAR | `photos_timeline.har` |

### 3.2 输出路径

```
./build/default/cache/default/default/
```

### 3.3 HAR 结构

```
photos_common.har/
├── index.d.ts              # 类型声明
├── index.ets                # 模块入口
├── module.json5             # 模块配置
└── libs/                    # 依赖库
    └── ...
```

---

## 4. 运行时加载关系

### 4.1 加载顺序

```
系统启动
    │
    ▼
加载 HAP
    │
    ├── 解析 module.json5
    │
    ├── 加载 @ohos/common (photos_common.har)
    │
    ├── 加载 @ohos/browser (photos_browser.har)
    │
    ├── 加载 @ohos/editor (photos_editor.har)
    │
    ├── 加载 @ohos/formAbility (photos_formAbility.har)
    │
    ├── 加载 @ohos/thirdselect (photos_thirdselect.har)
    │
    └── 加载 @ohos/timeline (photos_timeline.har)
    │
    ▼
初始化 AbilityStage
    │
    ▼
创建 MainAbility
    │
    ▼
加载页面 (引用 HAR 中的组件)
```

### 4.2 模块依赖图

```
phone_photos (HAP)
    │
    ├── photos_common (HAR) ───┐
    ├── photos_browser (HAR) ───┼── 编译时依赖
    ├── photos_editor (HAR) ───┤
    ├── photos_formAbility (HAR)
    ├── photos_thirdselect (HAR)
    └── photos_timeline (HAR)
```

---

## 5. 安装路径

### 5.1 系统安装

| 路径 | 说明 |
|------|------|
| `/system/app/com.ohos.photos/` | 系统应用目录 |
| `/system/app/com.ohos.photos/Photos.hap` | 安装的 HAP |

### 5.2 安装命令

```bash
# Debug 安装
hdc install ./build/outputs/hap/debug/phone_photos/default/photos.hap

# Release 安装
hdc install ./build/outputs/hap/release/phone_photos/default/photos.hap
```

---

## 6. 产物验证

### 6.1 HAP 验证

```bash
# 查看 HAP 信息
hdc appiamond

# 查看安装的应用
hdc shell bm dump -a
```

### 6.2 签名验证

```bash
# 验证签名
hdc shell
> java -jar signcenter_tool.jar -verify -infile Photos.hap
```

---

## 7. 产物大小

### 7.1 大小估算

| 产物 | 估算大小 | 说明 |
|------|----------|------|
| photos.hap | ~10-20MB | 含所有 HAR 依赖 |
| photos_common.har | ~2-5MB | 共享基础模块 |
| photos_browser.har | ~3-6MB | 浏览模块 |
| photos_editor.har | ~2-4MB | 编辑模块 |
| photos_thirdselect.har | ~1-2MB | 选择器模块 |
| photos_timeline.har | ~1-2MB | 日视图模块 |

> **注意**: 实际大小取决于编译优化和资源文件。

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [05_Build_System.md](05_Build_System.md) | 构建系统 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 问题排查 |
