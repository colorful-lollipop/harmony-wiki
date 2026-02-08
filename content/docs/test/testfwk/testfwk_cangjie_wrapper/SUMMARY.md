# SUMMARY - 文档导航

**项目**: testfwk_testfwk_cangjie_wrapper  
**用途**: 新人快速导航与全文索引

---

## 推荐阅读顺序

### 第一阶段：建立认知（15 分钟）

1. **[项目概览](00_Overview.md)**
   - 项目定位与边界
   - 核心能力与运行环境
   - 目录结构说明
   - 关键概念解释

2. **[架构设计](10_Architecture.md)**
   - 分层架构总览
   - 数据流向图
   - 依赖关系图
   - 关键时序说明

### 第二阶段：深入理解（30 分钟）

3. **[API 参考手册](20_API_Reference.md)**
   - Driver 类 - 核心入口
   - On 类 - 组件选择器
   - Component 类 - 组件操作
   - UiWindow 类 - 窗口操作
   - FFI 调用链
   - 错误码参考

4. **[构建系统](30_Build_System.md)**
   - GN 目标清单
   - 依赖关系
   - 编译产物
   - 安装路径

### 第三阶段：安全与运维（20 分钟）

5. **[安全分析](40_Security.md)**
   - 攻击面分析
   - 信任边界
   - 风险点清单
   - 修复建议

6. **[问题排查](50_Troubleshooting.md)**
   - 常见问题
   - 错误码速查
   - 调试方法
   - 日志分析

---

## 快速索引

### 按主题索引

| 主题 | 相关文档 | 关键章节 |
|------|----------|----------|
| **入门上手** | [概览](00_Overview.md), [架构](10_Architecture.md) | 项目定位、架构图 |
| **API 使用** | [API 参考](20_API_Reference.md) | Driver, On, Component |
| **集成构建** | [构建系统](30_Build_System.md) | BUILD.gn, 产物路径 |
| **安全审计** | [安全分析](40_Security.md) | 攻击面、风险点 |
| **问题定位** | [问题排查](50_Troubleshooting.md) | 错误码、调试 |
| **调用链追踪** | [附录](appendix/Callgraphs.md) | 关键调用链 |

### 按文件索引

| 文件 | 说明 | 跳转 |
|------|------|------|
| `ohos/ui_test/ui_test_api.cj` | 主要 API 实现 | [API 参考](20_API_Reference.md) |
| `ohos/ui_test/ui_test_ffi.cj` | FFI 绑定 | [架构](10_Architecture.md#ffi-层) |
| `ohos/ui_test/const.cj` | 常量定义 | [API 参考](20_API_Reference.md#常量定义) |
| `BUILD.gn` | 构建配置 | [构建系统](30_Build_System.md) |
| `bundle.json` | 项目配置 | [概览](00_Overview.md#项目配置) |

---

## API 快速索引

### Driver 类方法

| 类别 | 方法 | 说明 |
|------|------|------|
| **创建** | `create()` | 创建 Driver 实例 |
| **查找** | `findComponent()`, `findComponents()`, `findWindow()`, `waitForComponent()` | 查找 UI 元素 |
| **断言** | `assertComponentExist()` | 断言组件存在 |
| **按键** | `pressBack()`, `pressHome()`, `triggerKey()`, `triggerCombineKeys()` | 按键操作 |
| **手势** | `click()`, `doubleClick()`, `longClick()`, `swipe()`, `drag()`, `fling()` | 触屏手势 |
| **鼠标** | `mouseClick()`, `mouseMoveTo()`, `mouseDrag()`, `mouseScroll()` | 鼠标操作 |
| **显示** | `screenCap()`, `setDisplayRotation()`, `getDisplaySize()`, `wakeUpDisplay()` | 显示操作 |
| **输入** | `inputText()` | 文本输入 |
| **事件** | `createUiEventObserver()` | 创建事件观察者 |

### On 类方法

| 类别 | 方法 | 说明 |
|------|------|------|
| **属性** | `text()`, `id()`, `onType()`, `description()` | 属性匹配 |
| **状态** | `enabled()`, `focused()`, `clickable()`, `scrollable()` | 状态过滤 |
| **位置** | `isBefore()`, `isAfter()`, `within()`, `inWindow()` | 相对定位 |

### Component 类方法

| 类别 | 方法 | 说明 |
|------|------|------|
| **操作** | `click()`, `doubleClick()`, `longClick()`, `inputText()`, `clearText()` | 组件操作 |
| **查询** | `getText()`, `getId()`, `getType()`, `getBounds()` | 属性获取 |
| **滚动** | `scrollToTop()`, `scrollToBottom()`, `scrollSearch()` | 滚动操作 |
| **手势** | `dragTo()`, `pinchOut()`, `pinchIn()` | 手势操作 |

### UiWindow 类方法

| 类别 | 方法 | 说明 |
|------|------|------|
| **查询** | `getBundleName()`, `getTitle()`, `getBounds()`, `getWindowMode()` | 窗口属性 |
| **操作** | `focus()`, `moveTo()`, `resize()`, `maximize()`, `minimize()`, `close()` | 窗口操作 |

---

## 错误码速查

| 错误码 | 说明 | 位置 |
|--------|------|------|
| 17000001 | Initialization failed | Driver.create |
| 17000003 | Assertion failed | assertComponentExist |
| 17000004 | obj create return null reference | OBJ_LOST |
| 14700101 | System parameter can not be found | Systemparameter |
| 14700102 | System parameter value is invalid | Systemparameter |
| 14700103 | System permission operation permission denied | Systemparameter |
| 14700104 | System internal error (OOM, deadlock) | Systemparameter |

---

## 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| Cangjie | 仓颉 | 华为自研编程语言 |
| UiTest | UI Test | UI 自动化测试框架 |
| Driver | 驱动器 | UI 测试核心入口类 |
| Component | 组件 | UI 界面元素封装 |
| FFI | Foreign Function Interface | 外部函数接口 |
| GN | Generate Ninja | 元构建系统 |
| Syscap | System Capability | 系统能力标识 |

---

*最后更新: 2026-02-06*
