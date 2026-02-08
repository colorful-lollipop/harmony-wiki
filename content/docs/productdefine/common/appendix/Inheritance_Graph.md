# 继承关系图

## 目的

本文档以可视化方式展示 `productdefine/common` 仓库中各配置文件的**继承关系**，帮助理解配置层级和依赖。

---

## 整体继承架构

```mermaid
graph TB
    subgraph "基础层"
        Mini[base/mini_system.json<br/>轻量系统基础<br/>~15部件]
        Small[base/small_system.json<br/>小型系统基础<br/>~20部件]
        Std[base/standard_system.json<br/>标准系统基础<br/>~25部件]
    end
    
    subgraph "模版层"
        Rich[inherit/rich.json<br/>全量功能模版<br/>~350部件]
        Headless[inherit/headless.json<br/>无头系统<br/>~80部件]
        
        Watch[inherit/watch.json<br/>运动表<br/>~60部件]
        Wearable[inherit/wearable.json<br/>可穿戴<br/>~80部件]
        LiteWearable[inherit/liteWearable.json<br/>轻量可穿戴<br/>~50部件]
        
        IPCamera[inherit/ipcamera.json<br/>IP摄像头<br/>~120部件]
        
        Phone[inherit/phone.json<br/>手机<br/>~200部件]
        Tablet[inherit/tablet.json<br/>平板<br/>~250部件]
        PC[inherit/pc.json<br/>PC<br/>TODO]
        TV[inherit/tv.json<br/>TV<br/>~200部件]
        TwoIn1[inherit/2in1.json<br/>2合1设备<br/>~280部件]
        
        Chipset[inherit/chipset_common.json<br/>芯片组件<br/>~40部件]
    end
    
    subgraph "产品层"
        Arm64[products/system-arm64-default.json<br/>64位系统]
        Arm[products/system-arm-default.json<br/>32位系统]
        SDK[products/ohos-sdk.json<br/>SDK]
    end
    
    Mini --> Watch
    Mini --> Wearable
    Mini --> LiteWearable
    
    Small --> IPCamera
    
    Std --> Rich
    Std --> Headless
    
    Rich --> Phone
    Rich --> Tablet
    Rich --> PC
    Rich --> TV
    Rich --> TwoIn1
    
    Phone --> Arm64
    Phone --> Arm
    
    style Mini fill:#e1f5e1
    style Small fill:#e1f5e1
    style Std fill:#e1f5e1
    style Rich fill:#fff4e1
    style Phone fill:#e1f0ff
    style Tablet fill:#e1f0ff
    style Arm64 fill:#ffe1e1
    style Arm fill:#ffe1e1
```

---

## 标准系统继承链

### 标准系统产品（system-arm64-default）

```mermaid
graph LR
    Std[base/standard_system.json<br/>~25部件<br/>核心基础]
    -->
    Rich[inherit/rich.json<br/>+325部件<br/>全量功能]
    -->
    Phone[inherit/phone.json<br/>-150部件<br/>+25部件<br/>手机裁剪]
    -->
    Product[products/system-arm64-default.json<br/>+3部件<br/>产品定制]
    
    style Std fill:#e1f5e1
    style Rich fill:#fff4e1
    style Phone fill:#e1f0ff
    style Product fill:#ffe1e1
```

**继承详情**:

| 层级 | 文件 | 部件数 | 主要作用 |
|------|------|--------|----------|
| 1 | `base/standard_system.json` | ~25 | 最小基础：启动、通信、安全基础 |
| 2 | `inherit/rich.json` | ~350 | 全量功能：所有标准部件 |
| 3 | `inherit/phone.json` | ~200 | 手机裁剪：移除NFC、打印等 |
| 4 | `products/system-arm64-default.json` | ~203 | 产品定制：添加selinux等 |

---

## 轻量系统继承链

### 运动表产品（watch）

```mermaid
graph LR
    Mini[base/mini_system.json<br/>~15部件<br/>极简基础]
    -->
    Watch[inherit/watch.json<br/>+45部件<br/>手表功能]
    
    style Mini fill:#e1f5e1
    style Watch fill:#e1f0ff
```

**继承详情**:

| 层级 | 文件 | 部件数 | 主要作用 |
|------|------|--------|----------|
| 1 | `base/mini_system.json` | ~15 | 最轻量基础 |
| 2 | `inherit/watch.json` | ~60 | 手表功能：传感器、蓝牙、轻量UI |

---

## 小型系统继承链

### IP摄像头产品（ipcamera）

```mermaid
graph LR
    Small[base/small_system.json<br/>~20部件<br/>小型基础]
    -->
    IPCamera[inherit/ipcamera.json<br/>+100部件<br/>摄像头功能]
    
    style Small fill:#e1f5e1
    style IPCamera fill:#e1f0ff
```

**继承详情**:

| 层级 | 文件 | 部件数 | 主要作用 |
|------|------|--------|----------|
| 1 | `base/small_system.json` | ~20 | 小型系统基础 |
| 2 | `inherit/ipcamera.json` | ~120 | 摄像头功能：相机、编解码、网络 |

---

## 特殊系统继承链

### 无头系统（headless）

```mermaid
graph LR
    Std[base/standard_system.json<br/>~25部件]
    -->
    Headless[inherit/headless.json<br/>+55部件<br/>-图形/多媒体]
    
    style Std fill:#e1f5e1
    style Headless fill:#e1f0ff
```

**继承详情**:

| 层级 | 文件 | 部件数 | 主要作用 |
|------|------|--------|----------|
| 1 | `base/standard_system.json` | ~25 | 标准基础 |
| 2 | `inherit/headless.json` | ~80 | 无头裁剪：移除UI、图形、多媒体 |

### SDK（ohos-sdk）

```mermaid
graph LR
    SDK[products/ohos-sdk.json<br/>独立配置<br/>不继承]
    
    style SDK fill:#ffe1e1
```

**说明**: `ohos-sdk.json` 不继承任何模版，独立定义 SDK 所需部件。

---

## 设备类型对比

### 从 rich.json 派生的设备类型

```mermaid
graph TD
    Rich[inherit/rich.json<br/>~350部件<br/>全量功能]
    
    Rich --> Phone[inherit/phone.json<br/>~200部件<br/>-150部件]
    Rich --> Tablet[inherit/tablet.json<br/>~250部件<br/>-100部件]
    Rich --> PC[inherit/pc.json<br/>TODO]
    Rich --> TV[inherit/tv.json<br/>~200部件<br/>-150部件]
    Rich --> TwoIn1[inherit/2in1.json<br/>~280部件<br/>-70部件]
    
    Phone --> PhoneCut[裁剪: NFC,打印,MSDP,<br/>人脸/指纹认证]
    Tablet --> TabletCut[裁剪: 电话子系统,<br/>位置部分功能]
    TV --> TVCut[裁剪: 电话,相机,<br/>多模输入等]
    
    style Rich fill:#fff4e1
    style Phone fill:#e1f0ff
    style Tablet fill:#e1f0ff
    style TV fill:#e1f0ff
    style TwoIn1 fill:#e1f0ff
```

### 功能裁剪对比表

| 功能 | rich | phone | tablet | headless | ipcamera | watch |
|------|------|-------|--------|----------|----------|-------|
| 电话 | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| NFC | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| 打印 | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| 图形/显示 | ✅ | ✅ | ✅ | ❌ | ✅ | ⚠️ |
| 相机 | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ |
| 多媒体 | ✅ | ✅ | ✅ | ❌ | ✅ | ⚠️ |
| 人脸/指纹 | ✅ | ❌ | ⚠️ | ❌ | ❌ | ❌ |
| MSDP | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## 芯片组件模版

### chipset_common.json

```mermaid
graph LR
    Chipset[inherit/chipset_common.json<br/>~40部件<br/>独立模版]
    -->
    Products[各类产品<br/>芯片组件]
    
    style Chipset fill:#fff4e1
```

**说明**: `chipset_common.json` 定义芯片组件依赖的最小部件集合，通常被 vendor 目录下的芯片配置继承。

---

## 继承深度统计

### 各产品继承深度

| 产品/模版 | 继承深度 | 继承链 |
|-----------|----------|--------|
| `system-arm64-default` | 3 | base/standard → inherit/rich → inherit/phone → product |
| `system-arm-default` | 3 | base/standard → inherit/rich → inherit/phone → product |
| `inherit/phone` | 2 | base/standard → inherit/rich → inherit/phone |
| `inherit/tablet` | 2 | base/standard → inherit/rich → inherit/tablet |
| `inherit/headless` | 2 | base/standard → inherit/headless |
| `inherit/ipcamera` | 2 | base/small → inherit/ipcamera |
| `inherit/watch` | 2 | base/mini → inherit/watch |
| `ohos-sdk` | 1 | 独立配置 |

### 继承深度建议

```mermaid
graph LR
    A[基础配置] --> B[中间模版]
    B --> C[产品配置]
    
    style A fill:#e1f5e1
    style B fill:#fff4e1
    style C fill:#ffe1e1
```

**最佳实践**:
- 推荐深度：2-3 层
- 最大深度：不超过 4 层
- 原因：过深的继承链增加理解和维护难度

---

## 相关跳转

- [项目概览](01_Overview.md) - 理解项目定位
- [目录结构](02_Structure.md) - 了解目录组织
- [配置继承](03_Inheritance.md) - 学习继承机制
- [产品配置](04_Products.md) - 查看具体配置
