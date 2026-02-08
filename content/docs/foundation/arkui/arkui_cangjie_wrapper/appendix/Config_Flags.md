# 关键配置与宏 (Config Flags)

## 概述

本文档描述 arkui_cangjie_wrapper 中使用的关键配置、宏定义和 feature flags。

---

## API Level 宏

### @APILevel 宏

用于标注 API 的版本信息和系统能力要求。

#### 宏定义

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.ArkUI.ArkUI.Full"
]
```

#### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `since` | String | API 起始版本号 |
| `syscap` | String | 系统能力要求 |

#### 使用示例

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.ArkUI.ArkUI.Full"
]
public class LocalStorage {
    // API 实现
}
```

#### 代码证据

| 文件 | 行号 | 说明 |
|------|------|------|
| `ohos/base/length.cj` | 27-37 | Length 接口 |
| `ohos/arkui/state_management/local_storage.cj` | 34-37 | LocalStorage 类 |

---

## @Hide 宏

用于标记内部 API，不对外暴露。

#### 使用示例

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.ArkUI.ArkUI.Full"
]
@!Hide
public func aboutToBeDeleted(): Bool {
    return this.clear()
}
```

#### 代码证据

| 文件 | 行号 | 说明 |
|------|------|------|
| `ohos/base/length.cj` | 67-70 | aboutToBeDeleted 方法 |

---

## 状态管理宏

### @State

组件内状态装饰器。

#### 使用示例

```cangjie
@Component
struct Counter {
    @State private count: Int64 = 0
    
    func build() {
        Button("Count: \(${this.count})")
            .onClick(() => {
                this.count++
            })
    }
}
```

### @Prop

父子组件单向同步。

#### 使用示例

```cangjie
@Component
struct Child {
    @Prop count: Int64
    
    func build() {
        Text("Count: \(${this.count})")
    }
}

@Component
struct Parent {
    @State parentCount: Int64 = 0
    
    func build() {
        Child(count: this.parentCount)
    }
}
```

### @Link

父子组件双向绑定。

#### 使用示例

```cangjie
@Component
struct Child {
    @Link count: Int64
    
    func build() {
        Button("Increment")
            .onClick(() => {
                this.count++
            })
    }
}

@Component
struct Parent {
    @State parentCount: Int64 = 0
    
    func build() {
        Child(count: this.parentCount)
    }
}
```

### @Provide

跨层级提供状态。

#### 使用示例

```cangjie
@Component
struct Parent {
    @Provide message: String = "Hello"
    
    func build() {
        Middle {
            Child()  // 可访问 message
        }
    }
}
```

### @Consume

跨层级消费状态。

#### 使用示例

```cangjie
@Component
struct Child {
    @Consume message: String
    
    func build() {
        Text(this.message)
    }
}
```

---

## GN 构建配置

### Cangjie 共享库模板

```gn
ohos_cangjie_shared_library("module.name") {
  sources = [ "file.cj" ]
  
  cj_deps = [
    "dep1:module.dep1",
    "dep2:module.dep2",
  ]
  
  cj_external_deps = [
    "external:capability.name",
  ]
  
  external_deps = [ "engine:module.name" ]
  
  subsystem_name = "arkui"
  part_name = "arkui_cangjie_wrapper"
}
```

#### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `sources` | List | Cangjie 源文件 |
| `cj_deps` | List | 内部 Cangjie 依赖 |
| `cj_external_deps` | List | 外部 Cangjie 依赖 |
| `external_deps` | List | C/C++ 外部依赖 |
| `subsystem_name` | String | 子系统名 |
| `part_name` | String | 部件名 |

### 代码证据

| 文件 | 说明 |
|------|------|
| `kit/ArkUI/BUILD.gn` | Kit 构建配置 |
| `ohos/base/BUILD.gn` | 基础模块构建 |
| `ohos/arkui/component/BUILD.gn` | 组件构建 |

---

## Bundle 配置

### bundle.json 结构

```json
{
  "name": "@ohos/arkui_cangjie_wrapper",
  "version": "6.1",
  "component": {
    "name": "arkui_cangjie_wrapper",
    "subsystem": "arkui",
    "features": [],
    "adapted_system_type": ["standard"],
    "rom": "8930KB",
    "ram": "8100KB"
  }
}
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `name` | 模块名 |
| `version` | 版本 |
| `subsystem` | 所属子系统 |
| `adapted_system_type` | 适配系统类型 |
| `rom` | ROM 占用 |
| `ram` | RAM 占用 |

### 代码证据

| 文件 | 行号 | 说明 |
|------|------|------|
| `bundle.json` | 1-64 | 完整配置 |

---

## 系统能力 (System Capability)

### ArkUI 能力

| 能力名 | 说明 |
|--------|------|
| `SystemCapability.ArkUI.ArkUI.Full` | 完整 ArkUI 能力 |

### 使用示例

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.ArkUI.ArkUI.Full"
]
public class Component {
    // 需要完整 ArkUI 能力的实现
}
```

---

## 相关文档

- [03_N-API.md](../03_N-API.md) - API 参考
- [05_GN_Build.md](../05_GN_Build.md) - 构建系统
