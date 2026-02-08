# 编译产物

> 本章档说明本项目的编译产物清单、安装路径和运行时加载关系。

## 产物清单

### 最终产物

| 产物名称 | 类型 | 模块 | 设备类型 |
|----------|------|------|----------|
| `phone-wallpaper.hap` | Hap 包 | phone-wallpaper | phone |
| `pad-wallpaper.hap` | Hap 包 | pad-wallpaper | pad |

### 产物结构

```
phone-wallpaper.hap
├── module.json5              # 模块配置
├── resources/                # 资源文件
│   ├── base/
│   │   ├── element/         # 字符串资源
│   │   ├── media/           # 图片资源
│   │   └── profile/         # 页面配置
│   └── default/             # 默认资源
├── ets/                      # 字节码文件
│   └── ...                  # 编译后的 ArkTS 字节码
└── signature/               # 签名信息（如有）
```

## 安装路径

### 系统预置路径

| 产物 | 预置路径 | 说明 |
|------|----------|------|
| phone-wallpaper.hap | `/system/app/phone_wallpaper/` | 系统预置应用目录 |
| pad-wallpaper.hap | `/system/app/pad_wallpaper/` | 系统预置应用目录 |

### 沙箱路径

| 路径 | 用途 |
|------|------|
| `/data/app/<bundleName>/<userId>/` | 用户数据目录 |
| `/cache/<bundleName>/` | 缓存目录 |

## 运行时加载关系

### 应用启动加载流程

```mermaid
graph TD
    subgraph 系统启动
        BM[BundleManager] --> |加载模块| System
        PM[PackageManager] --> |验证签名| System
    end

    subgraph 应用启动
        AS[AbilityStage] --> |onCreate| App
        MA[MainAbility] --> |onCreate| App
        WE[WallpaperExtAbility] --> |onCreated| App
    end

    subgraph 资源加载
        Res[resources/base/] --> |加载| App
        Res2[resources/default/] --> |加载| App
    end

    subgraph 窗口创建
        WM[WindowManager] --> |创建窗口| WE
        Page[pages/index.ets] --> |渲染| WM
    end
```

### 模块加载时序

```mermaid
sequenceDiagram
    participant System as 系统
    participant BM as BundleManager
    participant AS as AbilityStage
    participant WE as WallpaperExtAbility
    participant UI as pages/index

    System->>BM: 安装 phone-wallpaper.hap
    BM->>System: 解析模块配置

    Note over AS: 用户触发壁纸服务
    System->>AS: onCreate()
    AS->>AS: 应用初始化

    System->>WE: onCreated(want)
    WE->>System: windowManager.create()
    System->>WE: 窗口创建成功
    WE->>System: loadContent("pages/index")
    System->>UI: 加载页面

    UI->>System: @StorageLink 订阅数据
    WE->>System: wallPaper.getPixelMap()
    System-->>WE: PixelMap 数据
    WE->>System: AppStorage.SetOrCreate()
    System->>UI: 更新壁纸显示
```

## 依赖加载顺序

### 静态依赖

| 依赖项 | 加载时机 | 说明 |
|--------|----------|------|
| **@ohos.wallpaper** | 运行时按需加载 | 系统服务代理 |
| **@ohos.window** | 运行时按需加载 | 系统窗口管理 |
| **@ohos.WallpaperExtension** | 编译时链接 | 基类定义 |
| **@ohos.application.Ability** | 编译时链接 | 基类定义 |
| **@ohos.application.AbilityStage** | 编译时链接 | 基类定义 |

### 动态依赖

| 依赖项 | 加载时机 | 说明 |
|--------|----------|------|
| **WindowManager** | 调用 create() 时 | 创建窗口实例 |
| **wallPaper** | 调用 getPixelMap() 时 | 获取壁纸数据 |

## 资源加载

### 资源类型

| 资源类型 | 路径 | 加载方式 |
|----------|------|----------|
| **字符串** | `resources/base/element/string.json` | $string:xxx |
| **图片** | `resources/base/media/*` | $media:xxx |
| **页面配置** | `resources/base/profile/main_pages.json` | $profile:xxx |

### 资源打包

- 所有资源编译到 `.hap` 包内
- 资源按设备类型分拣（phone/pad）
- 签名信息嵌入 `.hap` 包

## 版本信息

| 属性 | 值 | 位置 |
|------|-----|------|
| **Version Code** | `1000000` | AppScope/app.json5 |
| **Version Name** | `1.0.0` | AppScope/app.json5 |
| **Bundle Name** | `com.ohos.wallpaper` | AppScope/app.json5 |
| **Module Type** | feature | module.json5 |

## 相关文档

- [构建配置](06_Build.md)
- [架构说明](03_Architecture.md)
- [安全评审](08_Security.md)
