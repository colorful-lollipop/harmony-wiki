# 编译产物 - display_manager

> 本文档说明 display_manager 模块的编译产物、安装路径和运行时加载关系

---

## 文档目的

本文档提供：
- 完整的编译产物清单
- 产物安装路径
- 运行时加载关系
- 产物间的依赖关系

## 适用范围

- **适用对象**：系统集成人员、ROM 开发者、调试工程师
- **前置知识**：熟悉 OpenHarmony 系统镜像结构、动态库加载机制

---

## 产物概览

display_manager 模块编译生成以下主要产物：

| 产物 | 类型 | 安装路径 | 说明 |
|------|------|----------|------|
| libdisplaymgrservice.z.so | 共享库 | system/lib/ | 主服务（System Ability） |
| libdisplaymgr.so | 共享库 | system/lib/platformsdk/ | 内部 API 客户端库 |
| libbrightness.so | 共享库 | system/lib/module/ | N-API JS 模块 |
| libbrightness_manager.a | 静态库 | （不安装） | 亮度管理静态库 |
| libdisplay_manager_brightness_taihe_native.so | 共享库 | system/lib/ | ETS/ANI 模块 |

---

## 产物详情

### 1. libdisplaymgrservice.z.so（主服务）

**来源目标**：`state_manager/service:displaymgrservice`

**产物属性**：
| 属性 | 值 |
|------|------|
| **文件名** | libdisplaymgrservice.z.so |
| **类型** | ohos_shared_library |
| **库类型** | sa（System Ability） |
| **安装路径** | system/lib/ |
| **大小** | ~200-300KB（估算） |

**功能说明**：
- System Ability 主服务，SA ID 3308
- 包含 DisplayPowerMgrService 实现
- 包含 ScreenController、GradualAnimator 等核心类
- 链接 brightness_manager 静态库

**加载方式**：
- 由 System Ability Manager 在系统启动时加载
- 配置来源：`state_manager/sa_profile/3308.json`
- 进程名：powermgr

**依赖库**：
```
libbrightness_manager.a（静态链接）
libipc_core.z.so
libffrt.so
libhilog.so
libhicollie.so
libeventhandler.so
...（见 GN 配置的 external_deps）
```

**安全配置**：
- CFI（Control Flow Integrity）启用
- PAC-RET（Pointer Authentication）启用

---

### 2. libdisplaymgr.so（客户端库）

**来源目标**：`state_manager/interfaces/inner_api:displaymgr`

**产物属性**：
| 属性 | 值 |
|------|------|
| **文件名** | libdisplaymgr.so |
| **类型** | ohos_shared_library |
| **标签** | platformsdk |
| **安装路径** | system/lib/platformsdk/ |
| **大小** | ~50-100KB（估算） |

**功能说明**：
- 提供 DisplayPowerMgrClient API
- 供 Native 应用和框架层使用
- 封装 IPC 调用细节
- 自动处理服务断开重连

**使用方式**：
```cpp
#include "display_power_mgr_client.h"
// 链接: -ldisplaymgr
```

**依赖库**：
```
libipc_core.z.so
libhilog.so
libpowermgr_client.z.so
```

---

### 3. libbrightness.so（N-API 模块）

**来源目标**：`state_manager/frameworks/napi:brightness`

**产物属性**：
| 属性 | 值 |
|------|------|
| **文件名** | libbrightness.so |
| **类型** | ohos_shared_library |
| **模块名** | brightness |
| **安装路径** | system/lib/module/ |
| **大小** | ~30-50KB（估算） |

**功能说明**：
- 传统 N-API JS 模块
- 导出 @ohos.display.brightness 接口
- 提供 5 个 JS API：getValue, setValue, getMode, setMode, setKeepScreenOn

**使用方式**：
```javascript
import brightness from '@ohos.display.brightness';
brightness.setValue(128);
```

**依赖库**：
```
libdisplaymgr.so
libace_napi.z.so
libark_jsruntime.so
```

---

### 4. libdisplay_manager_brightness_taihe_native.so（ETS 模块）

**来源目标**：`state_manager/frameworks/ets/taihe:display_manager_taihe`

**产物属性**：
| 属性 | 值 |
|------|------|
| **文件名** | libdisplay_manager_brightness_taihe_native.so |
| **类型** | ohos_shared_library |
| **命名空间** | @ohos.brightness |
| **安装路径** | system/lib/ |
| **大小** | ~20-30KB（估算） |

**功能说明**：
- 新版 ETS/ANI 模块
- 导出 @ohos.brightness 命名空间
- 提供 setValue 的两个重载

**使用方式**：
```typescript
import { setValue } from '@ohos.brightness';
setValue(128, true);  // continuous update
```

---

### 5. libbrightness_manager.a（静态库）

**来源目标**：`brightness_manager:brightness_manager`

**产物属性**：
| 属性 | 值 |
|------|------|
| **文件名** | libbrightness_manager.a |
| **类型** | ohos_static_library |
| **安装** | 不安装（仅编译时使用） |
| **大小** | ~300-400KB（估算） |

**功能说明**：
- 亮度管理核心逻辑
- 自动亮度算法
- 传感器数据处理
- 配置解析

**使用方式**：
- 被 libdisplaymgrservice.z.so 静态链接
- 不独立发布

**包含组件**：
- BrightnessManager
- BrightnessService
- CalculationManager
- LightLuxManager
- 各种配置解析器

---

## 配置文件产物

### SA 配置文件

**文件**：`system/profile/displaymgr_sa_profile.json`

**来源**：`state_manager/sa_profile:displaymgr_sa_profile`

**内容**：
```json
{
    "process": "powermgr",
    "systemability": [
        {
            "name": 3308,
            "libpath": "libdisplaymgrservice.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

### 系统参数配置

**文件**：`system/etc/param/display.para`

**来源**：`state_manager/service/etc:param_files`

**用途**：亮度相关的系统参数

---

## 运行时加载关系

### 系统启动时序

```
1. 系统启动
   └── SystemAbilityManager 初始化
       └── 加载 SA 配置文件
           └── 发现 displaymgrservice (SA ID 3308)
               └── 启动 powermgr 进程
                   └── 加载 libdisplaymgrservice.z.so
                       └── DisplaySystemAbility::OnStart()
                           └── DisplayPowerMgrService::Init()
                               └── 初始化 ScreenController
                               └── 初始化 BrightnessManager
```

### 应用调用时序

```
JS 应用调用 brightness.setValue()
    └── 加载 libbrightness.so（如未加载）
        └── N-API 入口
            └── 调用 libdisplaymgr.so
                └── 首次调用时：
                    └── 连接 libdisplaymgrservice.z.so
                        └── 建立 IPC 连接
```

### 依赖加载图

```
powermgr 进程启动:
libdisplaymgrservice.z.so
    ├── libipc_core.z.so
    ├── libffrt.so
    ├── libhilog.so
    ├── libhicollie.so
    ├── libeventhandler.so
    ├── libutils.z.so
    ├── libdatashare_consumer.z.so
    ├── librender_service_base.z.so
    └── ... (见 GN external_deps)

JS 应用调用:
libbrightness.so
    ├── libdisplaymgr.so
    │   └── libipc_core.z.so
    │   └── libhilog.so
    ├── libace_napi.z.so
    └── libark_jsruntime.so
```

---

## 产物版本控制

### 版本号

- **组件版本**：3.1（bundle.json）
- **API 版本**：System API
- **兼容性**：向后兼容（IDL 接口版本控制）

### 版本标识

```
libdisplaymgr.so.3.1  （符号链接到 libdisplaymgr.so）
libdisplaymgrservice.z.so.3.1
```

---

## 调试信息

### 符号文件

**位置**：`out/{product}/{variant}/symbols/system/lib/`

**文件**：
```
libdisplaymgrservice.z.so.sym
libdisplaymgr.so.sym
libbrightness.so.sym
```

### Dump 支持

**服务支持 dump**：
```bash
# 获取服务状态
dump displaymgr
# 或
hidumper -s 3308
```

**Dump 输出内容**：
- 当前显示状态
- 当前亮度值
- 自动亮度状态
- 注册的回调数量

---

## 产物大小优化

### 当前优化措施

| 措施 | 说明 |
|------|------|
| 静态库合并 | brightness_manager 静态链接到 service |
| 条件编译 | 传感器、关屏策略等功能可选 |
| CFI 安全 | 控制流完整性检查 |

### 潜在优化空间

1. **功能模块化**：将可选功能编译为独立插件
2. **懒加载**：N-API 模块延迟加载
3. **代码瘦身**：移除调试代码（Release 模式）

---

## 常见问题

### Q: 服务无法启动

**检查项**：
1. `system/profile/displaymgr_sa_profile.json` 是否存在
2. `libdisplaymgrservice.z.so` 是否正确安装
3. 依赖库是否完整

### Q: JS API 调用失败

**检查项**：
1. `libbrightness.so` 是否在 `system/lib/module/`
2. 应用是否有权限调用 System API
3. 服务是否已启动

### Q: Native 客户端链接错误

**检查项**：
1. 是否链接 `-ldisplaymgr`
2. 头文件路径是否正确
3. 是否包含 `platformsdk` 目录

---

## 相关链接

- **GN 构建**：[06_GN_Targets.md](06_GN_Targets.md)
- **目录结构**：[02_Directory_Structure.md](02_Directory_Structure.md)
- **常见问题**：[09_Troubleshooting.md](09_Troubleshooting.md)

---

## 文档更新记录

- **2026-02-07**：初始版本 v1.0，基于构建配置分析生成
