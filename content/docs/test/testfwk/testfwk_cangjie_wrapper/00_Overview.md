# 项目概览

**文档目的**: 帮助新人快速理解 testfwk_cangjie_wrapper 项目定位、边界和核心能力  
**目标读者**: 新加入的开发者、架构师、测试工程师  
**阅读时间**: 约 10 分钟

---

## 1. 项目定位

### 1.1 一句话定义

`testfwk_testfwk_cangjie_wrapper` 是 OpenHarmony 为 **Cangjie（仓颉）语言开发者**提供的 **UI 自动化测试框架包装器**，基于底层 `arkxtest` 的 UI 测试能力封装而成。

### 1.2 解决的问题

| 问题 | 解决方案 |
|------|----------|
| Cangjie 开发者无法使用现有 ArkTS UI 测试框架 | 提供 Cangjie 语言的 UiTest API |
| 测试代码与业务代码语言不一致 | 统一使用 Cangjie 编写测试 |
| 底层 UI 测试能力复用 | 包装 arkxtest，复用成熟实现 |

### 1.3 项目边界

**包含**:
- Cangjie 语言的 UI 测试 API（Driver, On, Component, UiWindow）
- 与 arkxtest 原生层的 FFI 绑定
- 系统参数访问封装
- 跨平台 mock 实现（Windows/Mac）

**不包含**:
- UI 测试底层实现（在 `arkxtest` 项目中）
- ArkTS/JS API（在 `arkxtest` 中提供）
- 单元测试框架（使用 Cangjie 自带的 `std.unittest`）

**证据**: `README.md:5-6`

```
The testfwk_testfwk_cangjie_wrapper is a UI test framework (UiTest) provided 
for developers using the Cangjie language for application development on OpenHarmony.
```

---

## 2. 核心能力

### 2.1 功能清单

| 能力类别 | 具体功能 |
|----------|----------|
| **组件定位** | 通过文本、ID、类型、描述定位组件；支持相对位置定位 |
| **组件操作** | 点击、长按、双击、输入文本、滚动、拖拽、捏合手势 |
| **窗口操作** | 查找窗口、移动、调整大小、最大化/最小化/关闭 |
| **系统操作** | 按键触发、屏幕旋转、截图、显示唤醒 |
| **鼠标操作** | 点击、移动、拖拽、滚轮滚动 |
| **多指操作** | PointerMatrix 定义多指手势 |
| **事件监听** | Toast 和对话框显示事件监听 |

### 2.2 技术特性

| 特性 | 说明 |
|------|------|
| **语言** | Cangjie（华为自研编程语言） |
| **API 级别** | API Level 22+ (`@!APILevel[syscap: "SystemCapability.Test.UiTest"]` 证据: `ui_test_api.cj:38-41`) |
| **同步/异步** | 大部分 API 为同步调用，支持在 Worker 线程执行 (`workerthread: true`) |
| **异常处理** | 使用 BusinessException 统一异常抛出 |
| **平台支持** | 仅支持 Standard 系统（`adapted_system_type: ["standard"]` 证据: `bundle.json:18`） |

---

## 3. 运行环境

### 3.1 系统要求

| 要求项 | 规格 |
|--------|------|
| **操作系统** | OpenHarmony Standard System |
| **设备类型** | 标准设备（手机、平板等） |
| **ROM** | 400KB |
| **RAM** | 356KB |

**证据**: `bundle.json:17-20`

```json
{
  "adapted_system_type": [ "standard" ],
  "rom": "400KB",
  "ram": "356KB"
}
```

### 3.2 依赖组件

```
testfwk_cangjie_wrapper
├── arkxtest                    # UI 测试底层能力 (C++ 实现)
├── ability_cangjie_wrapper     # Ability 框架绑定
├── hiviewdfx_cangjie_wrapper   # HiLog 日志
├── cangjie_ark_interop         # Cangjie-ArkTS 互操作 (FFI, Exception, JSON)
└── init                        # 系统参数查询
```

**证据**: `bundle.json:24-29`, `README.md:29-34`

### 3.3 启用测试模式

UiTest 需要系统参数启用测试模式才能工作：

```bash
# 启用测试模式
hdc shell param set persist.ace.testmode.enabled 1

# 检查是否启用
hdc shell param get persist.ace.testmode.enabled
```

**证据**: `ui_test_api.cj:55-58`, `README.md:30`

```cangjie
let testEnable = Systemparameter.get(TESTMODE_ENABLE, def: "0")
if (testEnable != "1") {
    TEST_LOG.warn("UiTestKit_exporter: systemParameter \"${TESTMODE_ENABLE}\" is not set!")
}
```

---

## 4. 目录结构

```
test/testfwk/testfwk_cangjie_wrapper/
├── figures/                     # 架构图
│   └── testfwk_cangjie_wrapper_architecture_en.png
├── kit/                         # Test Kit 模块
│   └── TestKit/
│       ├── BUILD.gn            # kit.TestKit 构建目标
│       └── index.cj            # 公共 API 导出 (23 行)
├── mock/                        # 跨平台 Mock 实现
│   └── ohos.ui_test.cj         # Windows/Mac 平台空实现
├── ohos/                        # UI 测试核心实现
│   └── ui_test/
│       ├── BUILD.gn            # ohos.ui_test 构建目标
│       ├── cj_process.cj       # 进程管理 FFI (51 行)
│       ├── const.cj            # API 常量定义 (124 行)
│       ├── systemparameter.cj  # 系统参数访问 (101 行)
│       ├── ui_test_api.cj      # 主要 API 实现 (2000+ 行)
│       ├── ui_test_common.cj   # 公共类型定义 (709 行)
│       └── ui_test_ffi.cj      # FFI 绑定定义 (82 行)
└── test/                        # 测试用例 (Wiki 不引用)
    └── uitest/
```

**证据**: `README.md:35-46`, `bundle.json:32-35`

### 4.1 源代码统计

| 文件 | 行数 | 说明 |
|------|------|------|
| `ui_test_api.cj` | ~2000 | Driver, On, Component, UiWindow 等核心类 |
| `ui_test_common.cj` | 709 | 枚举、Point、Rect、WindowFilter 等 |
| `ui_test_ffi.cj` | 82 | FFI 函数声明，ApiCallParams 结构体 |
| `const.cj` | 124 | API 操作字符串常量 |
| `systemparameter.cj` | 101 | 系统参数 get/set |
| `cj_process.cj` | 51 | 进程 uid/pid/tid 获取 |
| `index.cj` | 23 | TestKit 模块导出 |

---

## 5. 关键概念

### 5.1 核心类说明

#### Driver（驱动器）
- **定位**: UI 测试的核心入口
- **职责**: 
  - 协调管理整个测试流程
  - 提供全局 UI 操作能力
  - 负责组件查找、断言、等待
- **使用**: 通过 `Driver.create()` 创建实例

**证据**: `README.md:18`, `ui_test_api.cj:115`

#### On（选择器）
- **定位**: 组件特征描述器
- **职责**:
  - 通过多属性（文本、ID、类型）精准定位组件
  - 支持多属性组合匹配
  - 支持相对定位（前/后/内/窗口内）
- **使用**: 链式调用构建选择条件

**证据**: `README.md:19`, `ui_test_api.cj:1187`

#### Component（组件）
- **定位**: UI 组件对象封装
- **职责**:
  - 单个组件的细粒度操作
  - 属性获取（文本、ID、类型、边界）
  - 手势操作（点击、拖拽、捏合）
- **获取**: 通过 `Driver.findComponent(on: On)` 获得

**证据**: `README.md:20`, `ui_test_api.cj:1580`

#### UiWindow（窗口）
- **定位**: 窗口对象封装
- **职责**:
  - 窗口属性查询（标题、包名、边界）
  - 窗口操作（移动、调整大小、关闭）
- **获取**: 通过 `Driver.findWindow(filter: WindowFilter)` 获得

**证据**: `README.md:21`, `ui_test_api.cj:908`

### 5.2 架构分层

```
┌─────────────────────────────────────────────────────────────────┐
│                      接口层 (Interface Layer)                    │
│  ┌──────────┐ ┌──────┐ ┌───────────┐ ┌──────────┐              │
│  │  Driver  │ │  On  │ │ Component │ │ UiWindow │              │
│  └────┬─────┘ └──┬───┘ └─────┬─────┘ └────┬─────┘              │
└───────┼──────────┼───────────┼────────────┼────────────────────┘
        │          │           │            │
        └──────────┴───────────┴────────────┘
                           │
┌──────────────────────────▼─────────────────────────────────────┐
│                  框架层 (Framework Layer)                        │
│                   UiTest Wrapper (Cangjie)                       │
│                     ohos.ui_test 模块                            │
└──────────────────────────┬─────────────────────────────────────┘
                           │ FFI
┌──────────────────────────▼─────────────────────────────────────┐
│                  依赖层 (Dependency Layer)                       │
│  ┌──────────┐ ┌──────┐ ┌──────────────────┐ ┌──────────────┐  │
│  │ arkxtest │ │ init │ │ ability_cangjie_ │ │ hiviewdfx_   │  │
│  │(UI测试)  │ │(系统)│ │   wrapper        │ │ cangjie_     │  │
│  └──────────┘ └──────┘ └──────────────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**证据**: `README.md:23-34`, 架构图 `figures/testfwk_cangjie_wrapper_architecture_en.png`

### 5.3 FFI 调用模式

所有 UI 测试操作通过统一的 FFI 接口调用底层：

```cangjie
// 1. 初始化连接
CJ_InitConnection(token)

// 2. 构建 API 调用参数
let params = ApiCallParams(apiId, callerRef, jsonParams)

// 3. 调用底层
let result = CJ_ApiCall(params)

// 4. 解析结果
if (result.code != 0) {
    throw BusinessException(result.code, errorMessage)
}
```

**证据**: `ui_test_ffi.cj:23-27`, `ui_test_api.cj:75-84`

---

## 6. 版本信息

| 属性 | 值 |
|------|-----|
| **组件名** | @ohos/testfwk_cangjie_wrapper |
| **版本** | 6.1 |
| **许可证** | Apache License 2.0 |
| **子系统** | testfwk |
| **发布形式** | code-segment |
| **API 级别** | since: "22" |

**证据**: `bundle.json:2-14`

---

## 7. 外部文档链接

- **API 参考**: [cj-apis-ui_test.md](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/TestKit/cj-apis-ui_test.md)
- **开发指南**: [cj-arkxtest-guidelines.md](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/application-test/cj-arkxtest-guidelines.md)
- **代码贡献**: [Code Contribution](https://gitcode.com/openharmony/docs/blob/master/en/contribute/code-contribution.md)

---

## 8. 下一步阅读

- **[架构设计](10_Architecture.md)** - 深入理解模块职责和数据流
- **[API 参考](20_API_Reference.md)** - 完整 API 清单和使用方法
- **[构建系统](30_Build_System.md)** - 了解如何编译和集成

---

*本文档基于代码仓库静态分析生成*  
*证据位置: README.md, bundle.json, ohos/ui_test/*.cj*
