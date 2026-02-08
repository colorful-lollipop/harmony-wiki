# 常见问题与调试 (Troubleshooting)

## 概述

本文档收集 arkui_cangjie_wrapper 的常见构建、运行和调试问题及解决方案。

---

## 构建问题

### Q1: GN 构建失败，找不到模板

**问题描述**:
```
gn gen out/device
ERROR: Could not find template file //build/templates/cangjie/cjc.gni
```

**可能原因**:
- 缺少 `cangjie_ark_interop` 依赖仓库
- GN 路径配置错误

**解决方案**:

```bash
# 1. 确保所有依赖仓库已同步
repo sync --force-sync

# 2. 检查 OHOS 环境变量
echo $OHOS_HOME

# 3. 重新生成构建文件
gn gen out/device --check
```

**代码证据**: `kit/ArkUI/BUILD.gn:14`

```gn
import("//build/templates/cangjie/cjc.gni")
```

---

### Q2: Cangjie 编译错误 - 未找到模块

**问题描述**:
```
error: module 'ohos.arkui.component' not found
```

**可能原因**:
- 依赖的 `ace_engine` 未正确构建
- 模块路径配置错误

**解决方案**:

```bash
# 1. 先构建依赖
ninja -C out/device //foundation/arkui/arkui_ace_engine:*

# 2. 检查模块是否存在
ls -la out/device/libs/ | grep cj

# 3. 清理并重新构建
rm -rf out/device
gn gen out/device
ninja -C out/device arkui_cangjie_wrapper
```

---

### Q3: 编译产物版本不匹配

**问题描述**:
```
FAILED: ninja: product out/device required, but PROJECT.json is for mini
```

**可能原因**:
- 构建目标配置错误
- 产品类型不匹配

**解决方案**:

```bash
# 1. 检查产品配置
hb set

# 2. 选择正确的设备类型
# 选择 standard 设备而非 mini 设备

# 3. 重新生成
hb build arkui_cangjie_wrapper
```

---

## 运行时问题

### Q1: 组件渲染异常

**问题描述**: 组件显示不正确或闪烁

**可能原因**:
- 状态更新频率过高
- 组件 key 冲突
- 布局约束错误

**诊断方法**:

```bash
# 1. 开启调试日志
hilog -v D

# 2. 查看组件树
hdc shell aa dump -a > component_tree.txt
```

**解决方案**:

```cangjie
// 避免高频状态更新
func onInput(text: String) {
    // ❌ 错误：每次输入都更新
    this.text = text
    
    // ✅ 正确：使用防抖
    this.debouncedText = text
}
```

---

### Q2: 状态未触发重新渲染

**问题描述**: 状态变量改变但 UI 未更新

**可能原因**:
- 使用了普通变量而非 @State
- 深层对象属性未使用 @ObjectLink

**解决方案**:

```cangjie
// ❌ 错误：普通变量
var message: String = "Hello"

// ✅ 正确：@State 装饰
@State var message: String = "Hello"

// ❌ 错误：深层对象属性
class Model {
    var name: String = ""
}

// ❌ 可能不触发更新
ChildComponent(model: this.model)

// ✅ 正确：@ObjectLink
class Model {
    var name: String = ""
}

@Component
struct Parent {
    @State model: Model = Model()
    
    func build() {
        ChildComponent(model: this.model)
    }
}

@Component
struct Child {
    @ObjectLink model: Model
    
    func build() {
        Text(this.model.name)
    }
}
```

---

### Q3: 路由导航失败

**问题描述**: pushUrl 无效或页面未跳转

**可能原因**:
- 页面未在 `main_pages.json` 中注册
- 路由参数格式错误

**解决方案**:

```bash
# 1. 检查页面配置
cat resources/base/profile/main_pages.json

# 2. 检查路由参数
```

```cangjie
// main_pages.json
{
  "src": [
    "pages/Index",
    "pages/Detail"
  ]
}

// 路由调用
router.pushUrl({ url: "pages/Detail" })
```

---

## 调试方法

### 日志查看

#### Hilog 日志

```bash
# 查看 ArkUI 相关日志
hilog | grep -i "arkui"

# 查看 Cangjie 运行时日志
hilog | grep -i "cangjie"

# 查看错误日志
hilog --level ERROR
```

#### 组件树调试

```bash
# 导出组件树
hdc shell aa dump -a component

# 查看焦点状态
hdc shell aa dump -a focus
```

### 性能分析

```bash
# 帧率监控
hdc shell perfcmd --start
# ... 执行操作
hdc shell perfcmd --stop > perf.txt
```

---

## 常见错误码

### 基础错误码

| 错误码 | 常量 | 说明 | 解决方案 |
|--------|------|------|----------|
| 100001 | Internal error | 内部错误 | 检查日志，联系维护者 |

### 路由错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 1001 | 无效页面 | 检查 pages.json 注册 |
| 1002 | 页面栈满 | 减少页面层级 |
| 1003 | 页面不存在 | 检查页面路径 |

---

## 性能优化建议

### 状态更新优化

```cangjie
// ✅ 使用 @State 管理最小粒度状态
@Component
struct OptimizedComponent {
    @State private count: Int64 = 0
    // 只更新需要的部分
}

// ❌ 避免大对象作为单一 @State
@Component
struct BadComponent {
    @State private largeData: LargeData = LargeData()
    // 整个对象更新会导致整个组件重建
}
```

### 列表优化

```cangjie
// ✅ 使用 LazyForEach 进行懒加载
LazyForEach(this.dataSource) { item in
    ListItemComponent(item: item)
}

// ✅ 为列表项提供唯一 key
LazyForEach(this.dataSource, { item.id }) { item in
    ListItemComponent(item: item)
}
```

---

## 代码证据

| 问题类型 | 相关代码位置 |
|----------|--------------|
| 构建配置 | `kit/ArkUI/BUILD.gn` |
| 路由 | `ohos/arkui/ui_context/cj_router.cj` |
| 状态管理 | `ohos/arkui/state_management/local_storage.cj` |
| 组件 | `ohos/arkui/component/` |

---

## 相关文档

- [05_GN_Build.md](./05_GN_Build.md) - 构建系统
- [06_Security.md](./06_Security.md) - 安全评审
- [官方开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/arkui-cj/cj-ui-development-overview.md)
