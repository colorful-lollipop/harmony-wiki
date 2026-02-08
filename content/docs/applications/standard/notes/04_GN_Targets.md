# GN 目标梳理

## 1. 构建配置概览

本项目使用 **hvigor** 作为构建工具（OpenHarmony 的 Gradle 替代品），配置文件为 `build-profile.json5`。

**证据位置**: `build-profile.json5:1-56`

```json5
{
  "app": {
    "products": [
      {
        "name": "default",
        "compileSdkVersion": 23,
        "compatibleSdkVersion": 23,
        "runtimeOS": "OpenHarmony"
      }
    ],
    "modules": [
      { "name": "default", "srcPath": "./product/default" },
      { "name": "utils", "srcPath": "./common/utils" },
      { "name": "resources", "srcPath": "./common/resources" },
      { "name": "component", "srcPath": "./features" }
    ]
  }
}
```

## 2. 模块配置

### 2.1 模块列表

| 模块名 | 路径 | 类型 | 描述 |
|--------|------|------|------|
| `default` | `product/default` | feature | 主应用模块 |
| `utils` | `common/utils` | har | 工具类库 |
| `resources` | `common/resources` | resource | 资源模块 |
| `component` | `features` | har | 功能组件库 |

### 2.2 模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                    模块依赖关系                              │
│                                                             │
│  default (feature)                                          │
│     ├── utils (har)                                         │
│     │    └── access/ (MediaLibraryAccess)                   │
│     └── component (har)                                      │
│          └── features/src/main/ets/components/              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 3. 构建文件配置

### 3.1 product/default 构建配置

**路径**: `product/default/build-profile.json5`

**类型**: Feature Module (可独立安装)

**配置**:
```json5
{
  "name": "default",
  "srcPath": "./product/default",
  "targets": [
    {
      "name": "default",
      "applyToProducts": [ "default" ]
    }
  ]
}
```

### 3.2 common/utils 构建配置

**路径**: `common/utils/build-profile.json5`

**类型**: HAR (Static Library)

**导出内容**:
- `baseUtil/` - 工具类
- `model/` - 数据模型
- `access/` - 访问控制

### 3.3 features 构建配置

**路径**: `features/build-profile.json5`

**类型**: HAR (Static Library)

**配置**:
```json5
{
  "module": {
    "name": "component",
    "type": "har",
    "deviceTypes": [ "default", "tablet" ]
  }
}
```

## 4. 编译参数

### 4.1 SDK 版本

| 参数 | 值 | 说明 |
|------|-----|------|
| compileSdkVersion | 23 | 编译 SDK 版本 |
| compatibleSdkVersion | 23 | 兼容 SDK 版本 |
| runtimeOS | OpenHarmony | 运行时系统 |

**证据位置**: `build-profile.json5:23-26`

### 4.2 设备类型

| 设备类型 | 支持 | 说明 |
|----------|------|------|
| default | ✅ | 默认设备 |
| tablet | ✅ | 平板设备 |

**证据位置**: `features/src/main/module.json5:21-23`

## 5. 构建产物

### 5.1 默认模块产物

| 产物类型 | 文件名 | 路径 |
|----------|--------|------|
| HAP | `default.hap` | `build/outputs/hap/default/` |

### 5.2 HAR 模块产物

| 模块 | 产物类型 | 文件名 |
|------|----------|--------|
| utils | HAR | `utils.har` |
| component | HAR | `component.har` |
| resources | HAR | `resources.har` |

**证据位置**: `build-profile.json5:30-55`

## 6. 构建命令

### 6.1 常用命令

| 命令 | 描述 |
|------|------|
| `hvigor assembleHap` | 构建 HAP 安装包 |
| `hvigor buildHap` | 编译 HAP |
| `hvigor clean` | 清理构建产物 |
| `hvigor installHap` | 安装 HAP 到设备 |

### 6.2 构建输出

| 输出目录 | 描述 |
|----------|------|
| `build/` | 构建中间产物 |
| `build/outputs/hap/` | HAP 安装包 |
| `build/libs/` | HAR 库文件 |

## 7. 相关跳转

| 目标 | 链接 |
|------|------|
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 编译产物 | [05_Build_Artifacts.md](05_Build_Artifacts.md) |
| 安全评估 | [06_Security_Review.md](06_Security_Review.md) |
