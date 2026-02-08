# 项目概览

## 项目定位

`font_manager` 是 OpenHarmony 全球化管理（globalization）子系统的核心组件，负责管理系统字体的安装和卸载功能。

**核心能力**：
- 第三方字体安装：为系统应用提供安装自定义字体的能力
- 字体卸载管理：支持卸载已安装的字体
- 数据迁移支持：支持字体数据的跨设备迁移

## 系统能力

| 属性 | 值 |
|------|-----|
| 子系统 | global |
| 系统能力 | SystemCapability.Global.FontManager |
| 组件名 | font_manager |
| 依赖组件 | ability_base, ability_runtime, access_token, hilog, ipc, safwk, samgr 等 |

## 目录结构

```
/base/global/font_manager
├── frameworks/fontmgr         # 核心框架代码
│   ├── include               # 头文件
│   │   ├── font_manager.h    # 字体管理核心类
│   │   ├── font_config.h     # 字体配置管理
│   │   ├── data_migration_manager.h
│   │   ├── font_event_publish.h
│   │   ├── font_manager_utils.h
│   │   └── hisysevent_adapter.h
│   └── src                   # 实现代码
│       ├── font_manager.cpp  # 字体安装卸载核心逻辑
│       ├── font_config.cpp
│       ├── font_manager_utils.cpp
│       ├── font_event_publish.cpp
│       ├── hisysevent_adapter.cpp
│       └── data_migration_manager.cpp
├── interfaces                # API
│   ├── ani                   # ANI 接口
│   └── js/kits               # ArkTS API (N-API)
│       ├── src/
│       │   ├── font_manager_napi.cpp  # N-API 注册入口
│       │   ├── font_manager_addon.cpp # N-API 实现
│       │   ├── js_data_migration_listener.cpp
│       │   └── js_func_ref_holder.cpp
│       └── include/
│           ├── font_manager_addon.h
│           └── font_napi_callback.h
├── sa_profile                # SystemAbility 配置
│   └── 66262.json            # SA ID: 66262
├── service                   # 服务端和客户端
│   ├── client                # 客户端实现
│   ├── server                # 服务端实现
│   ├── inner_api             # 内部 API
│   ├── IFontService.idl      # IPC 接口定义
│   └── BUILD.gn
├── common                    # 公共代码
│   └── include               # 公共头文件
│       ├── font_define.h     # 错误码和常量定义
│       ├── font_hilog.h      # 日志宏定义
│       └── idata_migration_listener.h
├── bundle.json               # 组件配置
├── README.md                 # 项目说明
└── README_zh.md              # 中文说明
```

## 运行环境

| 条件 | 说明 |
|------|------|
| 系统类型 | standard（标准系统） |
| 开发语言 | ArkTS / C++ |
| 许可协议 | Apache License 2.0 |

## 关键概念

### SystemAbility (SA)

font_manager 以 SystemAbility 形式运行，SA ID 为 **66262**。

```json
// sa_profile/66262.json
{
    "process": "font_manager_server",
    "systemability": [{
        "name": 66262,
        "libpath": "libfont_manager_server.z.so",
        "run-on-create": false,
        "distributed": false
    }]
}
```

### 字体安装路径

字体文件安装到以下路径：

```cpp
// common/include/font_define.h:56-60
INSTALL_PATH_PREFIX  = "/data/service/el1/"
INSTALL_PATH_SUFFIX  = "/for-all-app/fonts/"
FULL_PATH            = "/data/service/el1/{userId}/for-all-app/fonts/"
FONT_CONFIG_FILE     = "install_fontconfig.json"
```

### 权限要求

字体安装/卸载操作需要以下权限：

```cpp
// service/server/src/font_manager_server.cpp:38
static const std::string PERMISSION_UPDATE_FONT = "ohos.permission.UPDATE_FONT";
```

## 相关文档

- [架构说明](01_Architecture.md)
- [N-API 参考](02_NAPI_Reference.md)
- [构建目标](04_Build_Targets.md)
