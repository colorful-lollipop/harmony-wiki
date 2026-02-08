# 目录结构

## 整体结构

```
/base/hiviewdfx/hichecker/
├── frameworks/              # 框架实现代码
│   └── native/              # Native 层核心实现
├── interfaces/              # 对外接口
│   ├── native/innerkits/    # C++ 内部接口（InnerKit）
│   │   ├── include/         # 头文件目录
│   │   └── BUILD.gn         # 构建配置
│   ├── js/kits/napi/        # N-API 接口（JS/TS）
│   │   ├── include/         # N-API 头文件
│   │   ├── src/            # N-API 实现
│   │   ├── js_leak_watcher/ # JsLeakWatcher 模块
│   │   └── BUILD.gn
│   └── ets/ani/            # ETS/ANI 接口（ArkTS）
│       └── hichecker/
│           ├── include/
│           ├── src/
│           └── BUILD.gn
├── test/                    # 测试用例（本文档不涉及）
├── figures/                 # 架构图等资源
├── wiki/                    # 本文档目录
├── bundle.json              # 组件配置
├── BUILD.gn                 # 根构建入口
├── hichecker.gni            # 模块配置
└── README_zh.md             # 项目说明
```

## 模块职责

### frameworks/native

**职责**: Native 层核心逻辑实现

| 文件 | 职责 |
|------|------|
| [hichecker.cpp](../frameworks/native/hichecker.cpp) | 核心检测逻辑、规则管理、告警处理 |
| [caution.cpp](../frameworks/native/caution.cpp) | Caution 数据结构实现 |
| [hichecker_wrapper.cpp](../frameworks/native/hichecker_wrapper.cpp) | 包装层实现 |
| [BUILD.gn](../frameworks/native/BUILD.gn) | 构建配置 |

**依赖**: hilog, faultloggerd(backtrace), init(beget)

### interfaces/native/innerkits

**职责**: 对内部子系统提供的 C++ 接口

| 文件 | 职责 |
|------|------|
| [include/hiccheckER.h](../interfaces/native/innerkits/include/hichecker.h) | HiChecker 公共接口 |
| [include/caution.h](../interfaces/native/innerkits/include/caution.h) | Caution 告警接口 |
| [include/hiccheckER_wrapper.h](../interfaces/native/innerkits/include/hichecker_wrapper.h) | 包装接口 |
| [BUILD.gn](interfaces/native/innerkits/BUILD.gn) | 构建配置 |

**产出**: `libhichecker.so`

### interfaces/js/kits/napi

**职责**: 对应用层提供的 JS/TS 接口（N-API）

| 文件 | 职责 |
|------|------|
| [include/napi_hichecker.h](interfaces/js/kits/napi/include/napi_hichecker.h) | N-API 头文件 |
| [src/napi_hichecker.cpp](interfaces/js/kits/napi/src/napi_hichecker.cpp) | HiChecker N-API 实现 |
| [js_leak_watcher/](interfaces/js/kits/napi/js_leak_watcher/) | 内存泄漏检测模块 |
| [BUILD.gn](interfaces/js/kits/napi/BUILD.gn) | 构建配置 |

**产出**: `libhichecker.so` (JS 模块)

### interfaces/ets/ani

**职责**: 对 ArkTS 应用提供的 ANI 接口

| 文件 | 职责 |
|------|------|
| [include/ani_hiccheckER.h](interfaces/ets/ani/hichecker/include/ani_hichecker.h) | ANI 头文件 |
| [src/ani_hichecker.cpp](interfaces/ets/ani/hichecker/src/ani_hichecker.cpp) | ANI 接口实现 |
| [BUILD.gn](interfaces/ets/ani/hichecker/BUILD.gn) | 构建配置 |

**产出**: `libani_hichecker.so` 或对应 ANI 包

## 忽略的目录

以下目录不在本文档分析范围内（测试相关内容）：

- `test/` - 单元测试
- `test/unittest/` - 单元测试用例
- `test/*fuzz*` - 模糊测试
