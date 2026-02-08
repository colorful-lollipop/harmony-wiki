# 目录结构

## 顶层目录

```
arkweb_cangjie_wrapper/
├── figures/                          # 架构图和说明图片
├── kit/                              # Cangjie ArkWeb Kit 层代码
│   └── ArkWeb/                       # Kit 模块入口
├── ohos/                             # Cangjie ArkWeb 核心代码
│   └── web/                          # Web 模块
│       └── webview/                  # WebView 实现
├── mock/                             # 跨平台 Mock 实现
├── test/                             # 测试代码（文档不覆盖）
├── wiki/                             # 工程文档
├── BUILD.gn                          # 根构建配置
├── bundle.json                       # 组件配置
├── README.md                         # 项目说明
└── LICENSE                           # 许可证
```

## 目录职责说明

### figures/ — 架构资源目录

存放项目相关的架构图和说明图片。

| 文件 | 说明 |
|------|------|
| `arkweb_cangjie_wrapper_architecture_en.png` | 项目整体架构图（英文版） |

**证据来源**：`README.md` 第 9-11 行

### kit/ArkWeb/ — Kit 层入口

提供面向开发者的 API 导出层。

| 文件 | 类型 | 职责 |
|------|------|------|
| `index.cj` | Cangjie 源文件 | 重新导出 `ohos.web.webview` 的所有公共类型 |
| `BUILD.gn` | GN 构建文件 | 定义 `kit.ArkWeb` 共享库目标 |

**证据来源**：
- `kit/ArkWeb/index.cj` 第 18-20 行：`package kit.ArkWeb` + `public import ohos.web.webview.*`
- `kit/ArkWeb/BUILD.gn` 第 19-28 行：定义 `ohos_cangjie_shared_library("kit.ArkWeb")`

### ohos/web/ — 模块层

定义 Web 模块的基础结构和跨平台入口。

| 文件 | 类型 | 职责 |
|------|------|------|
| `web.cj` | Cangjie 源文件 | 平台相关实现（Linux/OpenHarmony） |
| `BUILD.gn` | GN 构建文件 | 定义 `ohos.web` 共享库目标 |

**证据来源**：
- `ohos/web/BUILD.gn` 第 19-29 行：定义 `ohos_cangjie_shared_library("ohos.web")`
- 平台判断逻辑：`if (is_mingw || is_mac)` 使用 mock 实现

### ohos/web/webview/ — 实现层

核心 API 实现目录，包含所有 WebView 相关的 Cangjie 封装。

| 文件 | 职责 |
|------|------|
| `back_forward_list.cj` | BackForwardList 类实现 |
| `web_cookie_manager.cj` | WebCookieManager 类实现 |
| `webview_controller.cj` | WebviewController 类实现 |
| `webview_ffi.cj` | FFI 绑定声明（foreign 函数） |
| `webview_common.cj` | 公共类型定义（枚举、类） |
| `webview_utils.cj` | 工具函数和错误处理 |
| `BUILD.gn` | 构建配置 |

**证据来源**：`ohos/web/webview/BUILD.gn` 第 24-31 行定义 sources 列表

### mock/ — 跨平台 Mock

提供 Windows 和 macOS 平台的 Mock 实现，用于本地开发和测试。

| 文件 | 对应实现 |
|------|----------|
| `ohos.web.cj` | `ohos/web/web.cj` 的 Mock |
| `ohos.web.webview.cj` | `ohos/web/webview/*.cj` 的 Mock |

**证据来源**：
- `ohos/web/BUILD.gn` 第 21-22 行：`if (is_mingw || is_mac)` 时使用 mock
- `ohos/web/webview/BUILD.gn` 第 21-22 行：同上

### test/ — 测试目录

存放测试代码，按照项目规范，测试内容不在本文档覆盖范围内。

**证据来源**：任务约束明确说明忽略测试相关内容

## 模块依赖关系

```
kit.ArkWeb (kit/)
    └── ohos.web.webview (ohos/web/webview/)
            ├── cangjie_ark_interop:ohos.business_exception
            ├── cangjie_ark_interop:ohos.ffi
            ├── cangjie_ark_interop:ohos.labels
            ├── hiviewdfx_cangjie_wrapper:ohos.hilog
            ├── multimedia_cangjie_wrapper:ohos.multimedia.image
            ├── arkui_cangjie_wrapper:ohos.base
            ├── arkui_cangjie_wrapper:ohos.arkui.component.util
            └── webview:cj_webview_ffi (native)

ohos.web (ohos/web/)
    └── 平台相关（web.cj 或 mock）

ohos.web.webview (ohos/web/webview/)
    └── 核心实现（6 个 cj 文件）
```

**证据来源**：`ohos/web/webview/BUILD.gn` 第 34-47 行

## 平台适配策略

项目采用条件编译实现跨平台支持：

| 平台 | 行为 |
|------|------|
| OpenHarmony / Linux | 使用完整实现（`ohos/web/web/*.cj`） |
| Windows (mingw) | 使用 Mock 实现（`mock/*.cj`） |
| macOS | 使用 Mock 实现（`mock/*.cj`） |

**证据来源**：`ohos/web/BUILD.gn` 第 21-25 行和 `ohos/web/webview/BUILD.gn` 第 21-32 行

## 代码规模统计

| 目录 | 文件数 | 主要内容 |
|------|--------|----------|
| kit/ArkWeb/ | 2 | Kit 层入口和构建配置 |
| ohos/web/ | 2 | 模块层入口和构建配置 |
| ohos/web/webview/ | 7 | 核心实现（6 cj + 1 gn） |
| mock/ | 2 | 跨平台 Mock |
| 总计 | 13 | — |

**关键文件行数统计**：

| 文件 | 行数 | 主要内容 |
|------|------|----------|
| `webview_controller.cj` | 850 | WebviewController 完整实现 |
| `webview_ffi.cj` | 716 | FFI 函数声明 |
| `web_cookie_manager.cj` | 208 | Cookie 管理 |
| `webview_common.cj` | 293 | 公共类型定义 |
| `webview_utils.cj` | 96 | 工具函数 |
| `back_forward_list.cj` | 96 | 历史记录管理 |
