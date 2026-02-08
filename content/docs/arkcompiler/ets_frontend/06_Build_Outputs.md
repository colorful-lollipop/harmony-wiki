# 编译产物与运行时加载

## 产物清单

### 可执行文件

| 文件名 | 描述 | 位置 |
|--------|------|------|
| `es2panda` | JS/TS 编译器 | `out/<product>/<toolchain>/arkcompiler/ets_frontend/` |
| `ets2panda` | ETS 编译器 | `out/<product>/<toolchain>/arkcompiler/ets_frontend/ets2panda/` |
| `merge_abc` | ABC 合并工具 | `out/<product>/<toolchain>/arkcompiler/ets_frontend/` |

### 动态库

| 文件名 | 描述 | 依赖 |
|--------|------|------|
| `libes2panda.so` | 公共编译器库 | runtime_core, protobuf, icu |
| `libarkguard.so` | 代码保护库 | - |

### 字节码文件

| 文件类型 | 描述 | 用途 |
|----------|------|------|
| `.abc` | ARK 字节码 | 运行时加载执行 |
| `.asm` | 字节码汇编 (调试用) | 调试输出 |

## 安装路径

### 系统安装

```
/system/bin/
├── es2panda
└── ets2panda

/system/lib/
├── libes2panda.so
└── libarkguard.so

/hap/
└── <app>/
    └── bundle_name/
        └── ...
            ├── entry/
            │   └── ets/
            │       └── ...
            └── default/
                └── ...
```

### SDK 安装

```
/sdk/
├── arkcompiler/
│   └── ets_frontend/
│       ├── es2panda
│       ├── es2panda_public/
│       │   └── include/
│       │       └── *.h
│       └── lib/
│           └── libes2panda.so
```

## 运行时加载关系

### 编译时依赖链

```
┌─────────────────────────────────────────────────────────┐
│                    编译时依赖                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ets_frontend                                            │
│      │                                                   │
│      ├── es2panda/ets2panda                              │
│      │       │                                           │
│      │       ├── runtime_core (libpanda_core.so)         │
│      │       ├── protobuf                                │
│      │       ├── icu                                     │
│      │       └── typescript                              │
│      │                                                   │
│      ├── merge_abc                                       │
│      │       └── protobuf                                │
│      │                                                   │
│      └── arkguard                                       │
│              └── (独立，无外部依赖)                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 运行时加载

```
应用程序 (.hap)
        │
        ▼
┌─────────────────────────────────────────┐
│  Bundle Installation                     │
│  - 解析 bundle.json                       │
│  - 验证签名                              │
│  - 提取资源                              │
└─────────────────┬─────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│  ArkTS Compiler (ets2panda)              │
│  - 编译 .ets → .abc                       │
│  - 运行在编译时                          │
└─────────────────┬─────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│  ARK Runtime (arkcompiler_ets_runtime)   │
│  - 加载 .abc 字节码                       │
│  - 解释/JIT 执行                          │
│  - 运行时                                │
└─────────────────────────────────────────┘
```

## 产物使用场景

### 场景 1: 应用开发

```bash
# 开发时编译
es2abc --output app.abc src/index.js

# 或使用构建系统
hdc shell aa compile -b bundleName -p entryPath
```

### 场景 2: 批量构建

```bash
# 独立编译
ark_standalone_build=true gn gen out/clang_x64
ninja -C out/clang_x64 arkcompiler/ets_frontend:es2panda_build
```

### 场景 3: 代码保护

```bash
# 使用 arkguard 混淆
arkguard --input app.abc --output protected.abc
```

## 版本兼容性

### 产物版本

| 产物 | 当前版本 | 兼容性 |
|------|----------|--------|
| ets_frontend | 3.1 | HarmonyOS NEXT |
| 字节码格式 | 3.0+ | ARK Runtime 3.0+ |
| ABC | 3.0+ | ARK Runtime 3.0+ |

### 升级注意事项

- 字节码版本升级可能需要重新编译
- 旧版运行时可能无法加载新版字节码

## 相关文档

- [命令行接口](02_CLI_Reference.md)
- [GN 构建系统](04_Build_System.md)
