# 构建工具链

## 概述

`build-tools/` 目录包含 30+ 个工具，用于处理、解析、检查 API 声明文件。

## 工具分类

```
build-tools/
├── API 解析工具
│   ├── dts_parser/              # d.ts 解析核心工具
│   └── collect_api/             # API 信息收集
├── API 检查工具
│   ├── api_check_plugin/        # API 规范检查
│   ├── jsdoc_format_plugin/     # JSDoc 格式检查
│   └── api_label_detection/     # API 标签检测
├── API 收集/比较工具
│   ├── collect_application_api/  # 应用 API 收集
│   ├── api_collector/           # API 收集器
│   ├── api_diff/                # API 差异比较 (旧)
│   └── diff_api/                # API 差异比较 (新)
├── API 处理工具
│   ├── handleApiFiles.js        # API 文件处理
│   ├── delete_systemapi_plugin/ # 系统 API 删除
│   └── process_label_noninterop.py # 互操作标签处理
├── ArkUI 工具
│   ├── arkui_transformer/       # ArkUI 转换
│   └── arkui/                   # ArkUI 相关
├── 权限工具
│   └── permissions_converter/   # 权限转换
├── SDK 升级工具
│   └── sdk_upgrade_assistant_plugin/ # 升级辅助
└── 其他工具
    ├── declgen.js               # 声明生成
    ├── checkApiVersion.js       # 版本检查
    └── ...
```

## 核心工具详解

### 1. dts_parser - d.ts 解析工具

**用途**: 解析 `.d.ts` 声明文件，提供多种分析功能

**目录结构**:

```
dts_parser/
├── src/
│   ├── bin/                    # 命令行配置
│   ├── coreImpl/
│   │   ├── parser/            # 基础解析
│   │   ├── checker/           # 格式检查
│   │   ├── diff/              # 差异比较
│   │   └── statistics/        # 统计
│   ├── typedef/                # 类型定义
│   └── utils/                  # 工具函数
├── test/                       # 测试用例
└── README_zh.md
```

**功能模块**:

| 模块 | 功能 | 关键接口 |
|------|------|---------|
| parser | 基础解析 | `parseDir()`, `parseFile()` |
| checker | 规范检查 | `checkEntryLocal()` |
| diff | 差异比较 | `diffSDK()` |
| statistics | API 统计 | `getApiStatisticsInfos()` |

**使用方式**:

```bash
# 解析目录
node ./dts_parser/src/main.ts -N collect -C ./api --output ./

# 比较差异
node ./dts_parser/src/main.ts -N diff \
  --old ./old_sdk/api \
  --new ./new_sdk/api \
  --output ./diff_report

# API 统计
node ./dts_parser/src/main.ts -N statistics \
  -C ./api --output ./stats.json
```

**证据**: `build-tools/dts_parser/README_zh.md:1-247`

### 2. api_check_plugin - API 规范检查工具

**用途**: 校验 d.ts 文件中的 JSDoc 规范

**目录结构**:

```
api_check_plugin/
├── config/                    # 权限配置和检查规则
├── plugin/                    # 词库
├── src/                       # 源码
├── test/                      # 测试
└── README_zh.md
```

**功能**:

| 功能 | 描述 |
|------|------|
| JSDoc 规范检查 | 检查标签使用、顺序、内容 |
| 权限配置检查 | 校验 permission 声明 |
| 兼容性检查 | API 变更兼容性分析 |

**使用方式**:

```bash
# 线下检查
cd build-tools/api_check_plugin
npm run test

# 线上检查
node ./src/main.ts -N checkOnline \
  --path ./api \
  --checker ./config/rules.json \
  --output ./report
```

**证据**: `build-tools/api_check_plugin/README_zh.md:1-42`

### 3. collect_application_api - 应用 API 收集工具

**用途**: 解析应用代码，收集使用的 API

**目录结构**:

```
collect_application_api/
├── src/                       # 源码
├── deps/                      # 依赖
└── README.md
```

**功能**:
- 扫描应用代码
- 提取 API 调用
- 生成 API 使用报告

### 4. api_diff / diff_api - API 差异比较工具

**用途**: 比较两个版本 SDK 的 API 差异

| 工具 | 类型 | 说明 |
|------|------|------|
| api_diff | 旧版 | 基础差异比较 |
| diff_api | 新版 | 增强差异比较 |

**使用方式**:

```bash
# 差异比较
node ./diff_api/src/main.ts -N diff \
  --old ./sdk_v1/api \
  --new ./sdk_v2/api \
  --output ./diff.xlsx
```

### 5. process_label_noninterop.py - 互操作标签处理

**用途**: 处理 ArkUI 的 `@noninterop` 标签

**证据**: `BUILD.gn:731-748`

**功能**:
- 解析互操作标签
- 生成互操作适配代码
- 处理动态/静态 SDK 差异

### 6. arkui_transformer.py - ArkUI 转换工具

**用途**: 转换 ArkUI 组件声明

**证据**: `BUILD.gn:676-694`

**使用方式**:

```python
python arkui_transformer.py \
  --input ./input/api \
  --output ./output/api \
  --source_root_dir ./
```

### 7. permissions_converter - 权限转换工具

**用途**: 从配置提取权限信息

**证据**: `README_zh.md:46`

**功能**:
- 解析权限配置
- 生成权限声明
- 供 DevEco Studio 使用

### 8. sdk_upgrade_assistant_plugin - SDK 升级辅助工具

**用途**: 帮助开发者解决 SDK 升级问题

**功能**:
- 检测 API 变更
- 提供迁移建议
- 自动修复常见问题

## 工具调用链

### 构建时工具调用

```
Source (.d.ts)
    │
    ▼
┌─────────────────┐
│ ohos_base_split │ ◄── parse_interface_sdk.py
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ ohos_declaration_template │ ◄── remove_internal.py
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ ohos_handle_declaration_template │ ◄── delete_arkui_label.py
└────────┬────────────────┘
         │
         ▼
    Output (SDK)
```

**证据**: `BUILD.gn:24-67`, `BUILD.gn:132-166`, `BUILD.gn:169-207`

### 工具依赖关系

| 工具 | 依赖 | 被依赖 |
|------|------|--------|
| dts_parser | TypeScript, Node.js | api_check_plugin |
| process_label_noninterop.py | Node.js | BUILD.gn |
| arkui_transformer.py | Node.js, npm | BUILD.gn |

## 工具开发指南

### 环境要求

| 要求 | 版本 |
|------|------|
| Node.js | 14.x |
| npm | 6.x+ |
| Python | 3.x |
| TypeScript | 4.x+ |

### 本地开发

```bash
# 1. 安装依赖
cd build-tools/
npm install

cd dts_parser/
npm install

# 2. 运行测试
npm test

# 3. 打包
npm run build
```

### 新增工具步骤

1. 在 `build-tools/` 创建工具目录
2. 添加 `README_zh.md` 文档
3. 实现核心功能
4. 添加单元测试
5. 更新本文档

## 常见问题

| 问题 | 解决方案 |
|------|---------|
| 工具运行报错 | 检查 Node.js 版本 (需 14.x) |
| 解析结果为空 | 检查输入路径是否正确 |
| 权限检查失败 | 更新 config/config.json |

## 相关文档

- [API 声明文件](01_API_Declarations.md)
- [GN 配置](03_Build_Configuration.md)
