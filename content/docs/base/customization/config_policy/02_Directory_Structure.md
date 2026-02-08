# 目录结构与模块职责

## 源代码目录结构

```
/base/customization/config_policy/
├── common/                          # 公共配置
│   └── config/                      # 配置文件目录
├── frameworks/                      # 核心实现代码
│   ├── config_policy/               # 配置策略核心模块
│   │   ├── src/                    # C/C++ 实现代码
│   │   │   └── config_policy_utils.c  # 核心配置策略实现
│   │   └── etc/                    # 系统配置文件
│   │       ├── BUILD.gn
│   │       └── customization.para.dac
│   └── dfx/                        # DFX 相关代码
│       ├── hisysevent_adapter/      # HiSysEvent 适配器
│       │   ├── hisysevent_adapter.h
│       │   └── hisysevent_adapter.cpp
│       └── hisysevent.yaml          # HiSysEvent 配置
├── interfaces/                     # API 接口层
│   ├── inner_api/                  # C++ 内部 API
│   │   └── include/
│   │       ├── config_policy_utils.h   # 内部 API 头文件
│   │       └── config_policy_impl.h    # 实现头文件
│   ├── kits/                       # JS/N-API 接口
│   │   ├── js/                     # JavaScript API
│   │   │   ├── include/
│   │   │   │   ├── config_policy_napi.h    # N-API 头文件
│   │   │   │   └── custom_config_napi.h   # Custom Config N-API
│   │   │   ├── src/
│   │   │   │   ├── config_policy_napi.cpp  # N-API 实现
│   │   │   │   └── custom_config_napi.cpp  # Custom Config 实现
│   │   │   └── BUILD.gn
│   │   └── cj/                     # CJ/FFI 接口
│   │       ├── include/
│   │       ├── src/
│   │       │   ├── config_policy_ffi.cpp
│   │       │   ├── config_policy_ffi.h
│   │       │   ├── config_policy_log.h
│   │       │   └── config_policy_mock.cpp
│   │       └── BUILD.gn
│   └── ets/                        # ETS 接口
│       └── ani/                    # ANI (ArkTS Native Interface)
│           ├── include/
│           │   ├── config_policy_ani.h
│           │   ├── custom_config_ani.h
│           │   └── ani_utils.h
│           ├── src/
│           │   ├── config_policy_ani.cpp
│           │   ├── custom_config_ani.cpp
│           │   └── ani_utils.cpp
│           ├── ets/
│           │   ├── @ohos.configPolicy.ets
│           │   └── @ohos.customization.customConfig.ets
│           └── BUILD.gn
├── BUILD.gn                        # 根构建入口
├── config_policy.gni               # GN 配置参数
├── bundle.json                     # 组件配置清单
└── LICENSE                         # Apache 2.0 许可证
```

---

## 模块职责

### 1. frameworks/config_policy/src/

**职责**: 实现配置策略的核心逻辑

| 文件 | 职责 | 关键函数 |
|------|------|----------|
| `config_policy_utils.c` | 配置层级查询、FollowX 机制实现 | `GetCfgDirList()`, `GetCfgFiles()`, `GetOneCfgFile()` |

**代码证据**: `frameworks/config_policy/src/config_policy_utils.c`

### 2. interfaces/inner_api/

**职责**: 为其他子系统提供 C++ 内部接口

| 文件 | 职责 | 导出符号 |
|------|------|----------|
| `config_policy_utils.h` | 公共头文件，定义数据结构和 API | `GetCfgDirList`, `FreeCfgDirList`, `GetCfgFiles`, `FreeCfgFiles`, `GetOneCfgFile`, `GetOneCfgFileEx`, `GetCfgFilesEx` |

**代码证据**: `interfaces/inner_api/include/config_policy_utils.h`

### 3. interfaces/kits/js/

**职责**: 提供 JavaScript/ArkTS N-API 接口

| 文件 | 职责 |
|------|------|
| `config_policy_napi.cpp` | N-API 绑定实现，导出 JS 接口 |
| `custom_config_napi.cpp` | Custom Config N-API 实现 |

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp`

### 4. interfaces/ets/ani/

**职责**: 提供 ArkTS Native Interface (ANI) 接口

| 文件 | 职责 |
|------|------|
| `config_policy_ani.cpp` | ETS 接口绑定实现 |
| `custom_config_ani.cpp` | Custom Config ETS 实现 |

**代码证据**: `interfaces/ets/ani/src/config_policy_ani.cpp`

### 5. frameworks/dfx/

**职责**: 提供 DFX（可观测性）支持

| 文件 | 职责 |
|------|------|
| `hisysevent_adapter.cpp` | HiSysEvent 日志适配器 |

**代码证据**: `frameworks/dfx/hisysevent_adapter/hisysevent_adapter.cpp`

---

## 模块依赖关系

```
interfaces/kits/js/  ──────┐
interfaces/kits/cj/  ──────┼──►  frameworks/config_policy/src/  ──►  frameworks/dfx/
interfaces/ets/ani/  ──────┘         │
                                        │
interfaces/inner_api/  ◄──────────────┘
        │
        ▼
    (外部子系统)
```

---

## 排除的目录

以下目录在文档分析中已被排除：

| 目录 | 排除原因 |
|------|----------|
| `test/` | 测试代码，不属于业务逻辑 |
| `test/unittest/` | 单元测试 |
| `test/fuzztest/` | 模糊测试 |
| `.git/` | 版本控制 |
| `.gitee/` | Gitee 平台配置 |
| `figures/` | 图片资源 |
