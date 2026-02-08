# 项目概述

## 组件定位

`config_policy`（配置策略组件）是 OpenHarmony 系统中负责配置文件管理的核心框架模块，为各业务模块提供获取各配置层级的配置目录或配置文件路径的接口。

**一句话定义**：一个支持多层级优先级和运营商定制的配置文件解析框架。

**核心问题**：OpenHarmony 系统需要一种机制来管理不同优先级的配置文件（系统、芯片、生产），同时支持运营商定制的配置覆盖。

### 核心能力

| 能力 | 描述 | 代码证据 |
|------|------|----------|
| 多层级配置管理 | 支持 4 层配置目录优先级管理（system → chipset → sys_prod → chip_prod） | `config_policy_utils.h:25-26` |
| 配置文件定位 | 根据优先级查找配置文件路径，返回最高优先级或所有层级 | `config_policy_utils.c:474-477` |
| FollowX 机制 | 支持根据 SIM 卡/运营商条件动态选择配置路径 | `config_policy_utils.c:189-235` |
| 多语言绑定 | 提供 C++ 内部 API、JS N-API、ArkTS ANI、Cangjie FFI 接口 | `interfaces/kits/js/`, `interfaces/ets/ani/`, `interfaces/kits/cj/` |
| 跨平台支持 | 标准系统、小型系统、LiteOS、LiteOS-M | `frameworks/config_policy/BUILD.gn:26-65` |

### 能力边界

| 能做什么 | 不能做什么 |
|---------|-----------|
| ✅ 查询配置目录列表 | ❌ 写入或修改配置文件 |
| ✅ 获取单个配置文件路径（最高优先级） | ❌ 删除或创建配置文件 |
| ✅ 获取所有层级配置文件路径 | ❌ 直接访问任意文件系统路径（受限于配置目录） |
| ✅ 支持运营商定制 FollowX | ❌ 访问非配置目录的文件 |
| ✅ 同步和异步 API | ❌ 网络通信或进程间通信 |

### 系统能力

组件声明的系统能力（SystemCapability）：

```
SystemCapability.Customization.ConfigPolicy
SystemCapability.Customization.CustomConfig
```

**代码证据**: `bundle.json:15-18`

### 运行环境

| 环境 | 支持情况 | 差异 |
|------|----------|------|
| 标准系统 (standard) | ✅ 完整支持 | 全部功能，包括 N-API、ANI、FFI |
| 小型系统 (small) | ✅ 基础支持 | 仅核心 C 库，部分语言绑定 |

**代码证据**: `bundle.json:23`

---

## 版本与依赖

| 项目 | 值 |
|------|-----|
| 当前版本 | 3.2 |
| 编程语言 | C/C++（核心），C++（绑定层），TypeScript/ArkTS（ANI），Cangjie（FFI） |
| ROM 占用 | ~100KB |
| RAM 占用 | ~50KB |
| 适配系统 | 标准、小型系统 |

### 外部依赖

| 依赖组件 | 用途 |
|----------|------|
| ability_runtime | 能力运行时支持（用于 customConfig 模块） |
| c_utils | C 语言工具库 |
| hisysevent | HiSysEvent 日志和事件上报 |
| hilog | 日志框架 |
| napi | Node.js API 绑定 |
| init | 系统初始化，系统参数读取 |
| ipc | 进程间通信（传递依赖） |
| runtime_core | 运行时核心（ANI 支持） |
| bounds_checking_function | 边界检查函数（安全字符串函数） |

**代码证据**: `bundle.json:29-42`

---

## 快速开始

### C/C++ 内部 API 使用示例

```cpp
#include "config_policy_utils.h"

// 获取配置目录列表
CfgDir *cfgDir = GetCfgDirList();
if (cfgDir != NULL) {
    // 遍历所有配置目录
    for (int i = 0; i < MAX_CFG_POLICY_DIRS_CNT && cfgDir->paths[i] != NULL; i++) {
        printf("Config dir %d: %s\n", i, cfgDir->paths[i]);
    }
    FreeCfgDirList(cfgDir);  // 重要：释放内存
}

// 获取指定配置文件的所有层级路径（从低到高）
const char *cfgPath = "etc/xml/config.xml";
CfgFiles *cfgFiles = GetCfgFiles(cfgPath);
if (cfgFiles != NULL) {
    for (int i = 0; i < MAX_CFG_POLICY_DIRS_CNT && cfgFiles->paths[i] != NULL; i++) {
        printf("Config file %d: %s\n", i, cfgFiles->paths[i]);
    }
    FreeCfgFiles(cfgFiles);  // 重要：释放内存
}

// 获取最高优先级的配置文件路径
const char *userPath = "etc/xml/user.xml";
char buf[MAX_PATH_LEN] = {0};
char *filePath = GetOneCfgFile(userPath, buf, MAX_PATH_LEN);
if (filePath != NULL) {
    printf("Highest priority config file: %s\n", filePath);
}

// 使用 FollowX 模式获取配置文件（运营商定制）
char buf2[MAX_PATH_LEN] = {0};
// 根据默认 SIM 卡的运营商配置查找
filePath = GetOneCfgFileEx(userPath, buf2, MAX_PATH_LEN, FOLLOWX_MODE_SIM_DEFAULT, NULL);
// 或者根据 SIM 1 的运营商配置查找
filePath = GetOneCfgFileEx(userPath, buf2, MAX_PATH_LEN, FOLLOWX_MODE_SIM_1, NULL);
```

### JavaScript/ArkTS N-API 使用示例

```typescript
import configPolicy from '@ohos.configPolicy';

// 1. 异步获取最高优先级配置文件（推荐）
configPolicy.getOneCfgFile('etc/xml/config.xml').then((filePath: string) => {
    console.log('Config file path:', filePath);
}).catch((error: Error) => {
    console.error('Failed:', error.message);
});

// 2. 同步获取最高优先级配置文件
const filePath = configPolicy.getOneCfgFileSync('etc/xml/config.xml');
console.log('Config file path:', filePath);

// 3. 使用 FollowX 模式获取配置文件
const filePathSim1 = configPolicy.getOneCfgFileSync(
    'etc/xml/config.xml',
    configPolicy.FollowXMode.SIM_1  // 使用 SIM 1 的运营商配置
);

// 4. 使用用户自定义 FollowX 规则
const filePathCustom = configPolicy.getOneCfgFileSync(
    'etc/xml/config.xml',
    configPolicy.FollowXMode.USER_DEFINED,
    'etc/carrier/custom'  // 自定义 extra 路径
);

// 5. 异步获取所有层级的配置文件
configPolicy.getCfgFiles('etc/xml/config.xml').then((files: string[]) => {
    console.log('All config files:', files);
});

// 6. 同步获取所有层级的配置文件
const allFiles = configPolicy.getCfgFilesSync('etc/xml/config.xml');
console.log('All config files:', allFiles);

// 7. 获取配置目录列表
configPolicy.getCfgDirList().then((dirs: string[]) => {
    console.log('Config directories:', dirs);
});

// 8. 同步获取配置目录列表
const dirs = configPolicy.getCfgDirListSync();
console.log('Config directories:', dirs);
```

### ArkTS ANI 使用示例

```typescript
import configPolicy from '@ohos.configPolicy';

// ANI 接口仅提供同步函数
const filePath = configPolicy.getOneCfgFileSync('etc/xml/config.xml');
const allFiles = configPolicy.getCfgFilesSync('etc/xml/config.xml');
const dirs = configPolicy.getCfgDirListSync();
```

### Cangjie FFI 使用示例

```cangjie
// Cangjie FFI 提供的函数（需要在 SDK 中使用）
import libconfig_policy_ffi

let cfgDir = libconfig_policy_ffi.CJ_GetCfgDirList()
let cfgFiles = libconfig_policy_ffi.CJ_GetCfgFiles("etc/xml/config.xml")
let oneFile = libconfig_policy_ffi.CJ_GetOneCfgFile("etc/xml/config.xml")
```

---

## 语言绑定与 API 差异

config_policy 组件提供 4 种语言绑定，功能对比如下：

| 语言 | 绑定层 | 接口文件 | 同步支持 | 异步支持 | FollowX 扩展 |
|------|---------|----------|---------|---------|-------------|
| C/C++ | 内部 API | `config_policy_utils.h` | ✅ | ❌ | ✅ (`GetOneCfgFileEx`, `GetCfgFilesEx`) |
| JavaScript | N-API | `configPolicy_napi.cpp` | ✅ | ✅ | ✅ (通过 followMode, extra 参数) |
| ArkTS | ANI | `config_policy_ani.cpp` | ✅ | ❌ | ✅ (通过 followMode, extra 参数) |
| Cangjie | FFI | `config_policy_ffi.cpp` | ✅ | ❌ | ✅ (通过 followMode, extra 参数) |

**代码证据**: `bg_54442dc6` 接口探索结果

---

## 跨平台支持差异

| 平台 | 支持的功能 | 构建产物 |
|------|-----------|---------|
| 标准系统 | 全部功能（N-API、ANI、FFI） | `ohos_shared_library` (.so) |
| LiteOS | 核心库 + 部分 N-API | `shared_library` (.so) |
| LiteOS-M | 核心库（静态） | `static_library` (.a) |
| SDK 预览 | Mock 实现 | `ohos_shared_library` (无真实功能) |

**代码证据**: `frameworks/config_policy/BUILD.gn:26-65`

---

## 性能与资源使用

| 资源 | 占用 | 说明 |
|-------|------|------|
| ROM | ~100KB | 核心库 + 所有语言绑定 |
| RAM | ~50KB | 运行时内存占用（不含动态分配） |
| 最大配置层级数 | 32 | `MAX_CFG_POLICY_DIRS_CNT` |
| 最大路径长度 | 256 字节 | `MAX_PATH_LEN` |

**代码证据**: `interfaces/inner_api/include/config_policy_utils.h:25-26`, `bundle.json:27-28`

---

## 相关文档

- [目录结构与模块职责](./02_Directory_Structure.md) - 代码组织方式和文件定位
- [架构设计](./03_Architecture.md) - 组件架构、数据流、FollowX 机制
- [N-API 接口文档](./04_N-API_Reference.md) - JavaScript N-API 接口详细说明
- [C++ 内部 API](./05_Cpp_Inner_API.md) - C/C++ 内部 API 详细说明
- [GN 构建配置](./06_GN_Build.md) - 构建目标、依赖、编译产物
- [安全风险评审](./07_Security_Review.md) - 攻击面分析和风险评估
- [工作笔记](./_work/NOTES.md) - 代码证据汇总和关键发现
