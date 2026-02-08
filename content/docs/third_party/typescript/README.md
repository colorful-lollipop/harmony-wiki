# TypeScript（OpenHarmony 适配版）

## 库基本信息

| 项目 | 信息 |
|------|------|
| **库名称** | TypeScript |
| **OH 组件名** | @ohos/typescript |
| **上游版本** | 4.9.5 |
| **OH 版本** | 3.1 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/Microsoft/TypeScript.git |
| **所属子系统** | thirdparty |

## OpenHarmony 定位

TypeScript 在 OpenHarmony 中扮演着 **ETS（Extensible TypeScript）语言核心编译器** 的角色。它不仅是标准的 JavaScript 超集编译器，更是 OpenHarmony ArkTS/eTS 应用开发技术栈的基础组件。

### 核心职责

1. **语言编译**：将 eTS/TypeScript 源码编译为可执行的 JavaScript/TypeScript 中间产物
2. **类型检查**：通过静态类型检查发现代码中的潜在错误
3. **IDE 支持**：提供代码补全、跳转定义、类型提示等语言服务
4. **开发工具**：作为 arkcompiler、ace_ets2bundle 等工具链的底层依赖

### 特殊适配

与上游 Microsoft/TypeScript 不同，OH 适配版增加了以下特性：

- **Struct 组件语法**：支持声明自定义 UI 组件
- **装饰器支持**：@Builder、@BuilderParam、@Styles、@Extend 等
- **状态样式**：stateStyles 状态样式机制
- **IDE 增强**：针对 eTS 的代码导航和补全优化

## 文档导航

### 快速入门

如果是首次了解此库，建议阅读顺序为：

1. **01_Overview.md** - 了解库的基本信息和 OH 定位
2. **04_Usage_in_OH.md** - 查看在 OH 中的使用场景
3. **02_Patches.md** - 深入理解 OH 的适配修改

### 按需查阅

| 需求 | 推荐文档 |
|------|---------|
| 了解库的基本功能 | 01_Overview.md |
| 查看 OH 做了哪些修改 | 02_Patches.md |
| 了解构建配置 | 03_Build_Integration.md |
| 查看谁在使用此库 | 04_Usage_in_OH.md |
| 了解新增的 API | 05_API_Differences.md |
| 查看安全注意事项 | 06_Security.md |

## 依赖关系概览

```
                    ┌─────────────────────────────┐
                    │     OpenHarmony 应用层       │
                    └─────────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────┐
                    │   ace_ets2bundle (打包工具)   │
                    │     开发工具子系统            │
                    └─────────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────┐
                    │   @ohos/typescript (编译器)   │
                    │     thirdparty 子系统         │
                    └─────────────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
    ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
    │  ets2panda      │   │    IDE 工具链    │   │    构建工具链    │
    │  (代码检查)      │   │   (语言服务)      │   │   (静态编译)     │
    └─────────────────┘   └─────────────────┘   └─────────────────┘
```

## 关键特点

### 无独立 Patch 文件

与 curl、openssl 等使用 .patch 文件的库不同，TypeScript 的 OH 适配采用**直接源码集成**方式。所有 eTS 特性修改直接嵌入 TypeScript 源码树中，这使得：

- 升级上游版本时需要更谨慎的冲突处理
- 修改分散在多个源文件中
- 需要通过源码分析识别 OH 特定修改

### 预构建产物

OH 使用预编译的 tgz 包分发 TypeScript：

- **产物路径**：`out/ohos-typescript-4.9.5-r4.tgz`
- **构建脚本**：`compile_typescript.py`
- **安装位置**：`ets/build-tools/ets-loader/node_modules/typescript.txt`

### 许可证管理

自动收集 Apache-2.0 许可证信息，分发到 SDK 的 ets-loader 目录。

## 版本历史

| OH 版本 | TypeScript 版本 | 主要变更 |
|--------|----------------|---------|
| 3.1 | 4.9.5 | eTS 完整支持（从上游 4.9.5 适配） |

## 相关资源

- **上游仓库**：https://github.com/Microsoft/TypeScript
- **TypeScript 官网**：https://www.typescriptlang.org/
- **OpenHarmony 开发文档**：https://developer.harmonyos.com
- **ETS 开发指南**：https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/arkts-utils/README.md
