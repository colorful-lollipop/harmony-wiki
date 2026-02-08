# 编译产物

## 产物清单

### 运行时产物

| 产物类型 | 路径模式 | 说明 |
|---------|---------|------|
| `.node` | `libes2panda.node` | Native Addon (macOS/Linux) |
| `.dll` | `libes2panda.dll` | Native 动态库 (Windows) |
| `.abc` | `**/*.abc` | Ark 字节码文件 |
| `.so` | `lib*.so` | 共享库 (Linux) |
| `.dylib` | `lib*.dylib` | 动态库 (macOS) |

### 编译器产物

| 产物类型 | 路径 | 说明 |
|---------|------|------|
| JS 库 | `ets_loader_lib_dir/` | 转译后的 JS 文件 |
| 声明文件 | `ets_loader_declarations_dir/` | TypeScript 声明 |
| 组件配置 | `component_config.json` | 组件元数据 |
| Kit 配置 | `kit_configs/` | 系统 Kit 配置 |

### 配置文件

| 产物 | 路径 | 说明 |
|------|------|------|
| 构建配置 | `build_config.json` | 构建配置信息 |
| 组件配置 | `component_config.json` | 组件定义 |
| 表单配置 | `form_config.json` | 卡片组件配置 |

## 输出目录结构

### 标准系统产物

```
out/xxx/
└── developtools/
    └── ace_ets2bundle/
        ├── ets_loader/                    # ETS 加载器
        │   ├── lib/                       # JS 库文件
        │   ├── declarations/             # 类型声明
        │   ├── component_config.json     # 组件配置
        │   ├── form_config.json          # 表单配置
        │   ├── build_config.json          # 构建配置
        │   ├── kit_configs/               # Kit 配置
        │   ├── components/                # 组件定义
        │   ├── form_components/          # 卡片组件
        │   ├── server/                    # 服务器
        │   ├── codegen/                   # 代码生成
        │   ├── insight_intents/          # 意图
        │   └── sysResource.js            # 系统资源
        ├── ets_loader_ark/               # Ark 运行时加载器
        │   ├── lib/
        │   ├── declarations/
        │   ├── components/
        │   ├── form_components/
        │   ├── insight_intentials/
        │   ├── server/
        │   └── codegen/
        └── ets_loader_ark_hap/           # HAP 相关
            └── OAT.xml                   # 权限配置

api/
└── ohos_declaration/
    └── ohos_declaration_ets/             # ETS 声明
        └── api/                          # 声明文件
```

### SDK 产物

```
sdk/
└── ets/
    ├── index.d.ts                        # 主声明文件
    ├── lib/                              # 编译后的 JS
    ├── component_config.json            # 组件配置
    └── ...
```

## 运行时加载关系

### 应用启动加载链

```
HAP 包加载
    │
    ├──► 加载 entry.hap
    │       │
    │       └──► 加载 abc/ 目录下的 .abc 文件
    │               │
    │               └──► Ark Runtime 解析 ABC
    │                       │
    │                       ├──► 加载系统模块
    │                       │       └──► @kit/* 的 .abc
    │                       │
    │                       └──► 执行应用代码
    │
    └──► 加载 native/ 目录下的 .so/.node
            │
            └──► N-API 调用原生功能
```

### JS 运行时依赖

```
应用 JS/ETS 代码
    │
    ├──► require('@kit.XXX')
    │       │
    │       └──► 加载 ets_loader/lib/*.js
    │               │
    │               └──► 加载系统 API 定义
    │
    └──► requireNapi('xxx')
            │
            └──► 通过 N-API 调用 native/.so
                    │
                    └──► Koala Native Addon
                            │
                            └──► libes2panda_public.so
```

## 产物安装路径

### 系统安装路径

| 产物 | 安装路径 |
|------|---------|
| ETS 库 | `/system/lib/ace/ets/` |
| 组件 | `/system/lib/ace/ets/components/` |
| 声明 | `/system/developtools/ets/declarations/` |
| Native 库 | `/system/lib/` |

### SDK 安装路径

| 产物 | 安装路径 |
|------|---------|
| ETS SDK | `/sdk/ets/` |
| 声明文件 | `/sdk/ets/declarations/` |
| 工具链 | `/sdk/ets/tools/` |

## 字节码文件

### ABC 文件结构

```
header (8 bytes)
    ├── magic: 0xABC12345
    ├── version: major.minor.patch
    └── flags

constant pool
    ├── strings
    ├── numbers
    ├── functions
    └── types

code section
    ├── functions
    ├── classes
    └── methods

debug info (optional)
    ├── line number tables
    └── local variable tables
```

### 字节码生成流程

```
ETS 源码 (.ets)
    │
    ├──► TypeScript Parser
    │       └──► AST
    │
    ├──► UI 语法转换
    │       └──► 转换后 AST
    │
    ├──► 类型检查
    │       └──► 验证通过
    │
    └──► es2abc 工具
        ├──► 生成中间表示
        ├──► 优化
        └──► 输出 .abc
```

## 产物验证

### 验证清单

| 验证项 | 方法 | 说明 |
|-------|------|------|
| 文件完整性 | `ls -la` | 检查文件存在 |
| 字节码有效性 | `file *.abc` | 验证文件格式 |
| 声明一致性 | `tsc --declaration` | 验证类型声明 |
| 运行时加载 | `ldd *.so` | 检查动态链接 |

## 相关文档

- [架构说明](02_Architecture.md)
- [GN 构建目标](07_GN_Targets.md)
- [安全风险评审](09_Security_Review.md)
