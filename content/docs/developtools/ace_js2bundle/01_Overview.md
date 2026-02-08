# 01 - 项目概览

## 项目定位

### 一句话定义

**ace_js2bundle** 是 OpenHarmony 应用开发工具链中的前端编译构建组件，负责将开发者编写的类 Web 范式源码（HML/CSS/JS）转换为 ArkUI 框架可执行的 JavaScript Bundle 或 Ark 字节码。

### 在 OpenHarmony 中的位置

```
┌────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 应用开发工具链                   │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   IDE/Dev   │───→│ ace_js2bundle│───→│    Ark Compiler    │ │
│  │   Studio    │    │  (本项目)    │    │   (ets2abc/abc2o)   │ │
│  └─────────────┘    └─────────────┘    └─────────────────────┘ │
│         │                  │                      │            │
│         ▼                  ▼                      ▼            │
│    开发者编写          HML/CSS/JS              Ark 字节码       │
│    HML/CSS/JS          → JS Bundle            → 可执行文件      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 项目边界

| 属于本项目 | 不属于本项目 |
|------------|--------------|
| HML/CSS/JS 语法解析与转换 | Ark 运行时（ace_engine） |
| Webpack 插件/loader 开发 | Ark 编译器（ts2abc/es2abc） |
| 构建配置与流程控制 | IDE 集成 |
| 资源处理与优化 | 应用包管理（HAP 打包） |

## 核心能力

### 1. 语法编译转换

将声明式 UI 代码转换为命令式渲染函数：

```
输入 (HML):
<div class="container">
  <text class="title">{{title}}</text>
</div>

输出 (JS):
function(vm) {
  return _c('div', { attrs: { class: 'container' } }, [
    _c('text', { attrs: { class: 'title' } }, [_v(_s(vm.title))])
  ])
}
```

**代码证据**: `ace-loader/src/lite/lite-transform-template.js:138-147`

### 2. 语法验证

- 标签合法性检查
- 属性类型校验
- 事件绑定验证
- 自定义组件检查

**代码证据**: `ace-loader/src/card-loader.js:60-91`

### 3. 多设备支持

| 设备类型 | 配置 | 输出格式 |
|----------|------|----------|
| 富设备 (Rich) | `webpack.rich.config.js` | JS Bundle + ABC |
| 瘦设备 (Lite) | `webpack.lite.config.js` | JS Bundle + BIN |
| 卡片 (Card) | `webpack.rich.config.js` + card 模式 | JSON + ABC |

**代码证据**: `ace-loader/webpack.rich.config.js:351-356`

### 4. 代码优化

- **代码分割**: commons.js + vendors.js
- **Tree Shaking**: 未使用代码消除
- **压缩**: TerserPlugin 压缩混淆
- **缓存**: 文件系统缓存加速增量构建

**代码证据**: `ace-loader/webpack.rich.config.js:358-417`

### 5. Ark 字节码生成

调用 Ark 编译器将 JS 转换为 ABC：

```
JS Bundle → ts2abc/es2abc → ABC (Ark Bytecode)
```

**代码证据**: `ace-loader/src/genAbc-plugin.js:71-130`

## 运行环境

### 构建时环境

| 依赖 | 版本 | 用途 |
|------|------|------|
| Node.js | >= 12.18.3 | 运行时 |
| npm | >= 6.14.8 | 包管理 |
| webpack | 5.72.1 | 构建框架 |
| babel | 7.x | 语法转换 |

**代码证据**: `ace-loader/package.json:27-58`

### 运行时环境

| 目标平台 | 说明 |
|----------|------|
| ArkUI 框架 | OpenHarmony 官方 UI 框架 |
| QuickJS | 瘦设备 JS 引擎 |
| Ark Runtime | 富设备 Ark 运行时 |

## 关键概念

### HML (HarmonyOS Markup Language)

类 HTML 的声明式 UI 语言：

```hml
<div class="container">
  <text if="{{showTitle}}" class="title">{{title}}</text>
  <list for="{{items}}">
    <list-item>{{$item.name}}</list-item>
  </list>
</div>
```

### 指令 (Directives)

| 指令 | 说明 | 示例 |
|------|------|------|
| `if` | 条件渲染 | `<div if="{{condition}}">` |
| `for` | 列表渲染 | `<div for="{{list}}">` |
| `show` | 显示控制 | `<div show="{{visible}}">` |

### 设备级别

| 级别 | 设备 | 能力 |
|------|------|------|
| rich | 手机/平板 | 完整 JS API |
| lite | 手表/IoT | 精简 JS API |
| card | 服务卡片 | 静态展示，无 JS 逻辑 |

## 项目元数据

| 属性 | 值 |
|------|-----|
| 组件名 | @ohos/ace_js2bundle |
| 版本 | 3.1 |
| 许可证 | Apache License 2.0 |
| 子系统 | developtools |
| 适配系统 | standard |

**代码证据**: `bundle.json:1-35`

## 相关文档

- [目录结构](./02_Directory_Structure.md)
- [架构说明](./03_Architecture.md)
- [构建系统](./06_Build_System.md)
