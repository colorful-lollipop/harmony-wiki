# 编译产物

## 概述

本章节描述 font_manager 编译生成的产物，包括动态库、配置文件及其安装路径。

## 产物清单

### 动态库

| 产物 | 类型 | 说明 | 依赖 |
|------|------|------|------|
| `libfontmanager.z.so` | 共享库 | N-API 接口层，供应用调用 | libfont_manager_client.z.so |
| `libfont_manager_client.z.so` | 共享库 | 客户端 IPC 层 | IPC, samgr |
| `libfont_manager_server.z.so` | SA 共享库 | 服务端实现（SA） | SAFW, access_token, fontmgr |

### 配置文件

| 产物 | 说明 | 安装路径 |
|------|------|----------|
| `66262.json` | SA 配置文件 | /system/etc/sa_config/ |
| `font_manager_server.cfg` | 服务配置 | /system/etc/ |

### 其他产物

| 产物 | 说明 |
|------|------|
| IDL 生成的代码 | IFontService.h/cpp/proxy/stub 等 |

## 运行时加载关系

```
应用进程
    │
    ├── libfontmanager.z.so (dlopen)
    │       │
    │       └── libfont_manager_client.z.so (dep)
    │               │
    │               └── IPC 调用
    │                       │
    │                       ↓
    │               font_manager_server 进程
    │                       │
    │                       └── libfont_manager_server.z.so
    │                               │
    │                               └── libfontmgr.z.so (dep)
```

## 安装路径

### 系统库路径

```
/system/lib/module/
    └── libfontmanager.z.so          # N-API 模块

/system/lib/
    ├── libfont_manager_client.z.so  # 客户端库
    └── libfont_manager_server.z.so  # 服务端库

/system/lib/module/libfont_manager_server.z.so  # SA 库（可能）
```

### SA 配置路径

```
/system/etc/sa_config/
    └── 66262.json                   # SA ID: 66262

/system/etc/
    └── font_manager_server.cfg      # 服务配置
```

### 字体安装路径

```
/data/service/el1/{userId}/
    └── for-all-app/fonts/
        ├── myfont.ttf               # 安装的字体文件
        └── install_fontconfig.json  # 字体配置
```

## SA 配置详情

```json
// sa_profile/66262.json
{
    "process": "font_manager_server",
    "systemability": [
        {
            "name": 66262,
            "libpath": "libfont_manager_server.z.so",
            "run-on-create": false,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

| 字段 | 值 | 说明 |
|------|-----|------|
| name | 66262 | SystemAbility ID |
| libpath | libfont_manager_server.z.so | SA 实现库路径 |
| run-on-create | false | 按需启动（首次调用时启动） |
| distributed | false | 非分布式 SA |
| dump_level | 1 | 调试 dumpsys 级别 |

## 服务端加载流程

```cpp
// 1. SAMgr 收到请求需要启动 SA 66262
// 2. 加载 libfont_manager_server.z.so
// 3. 调用 REGISTER_SYSTEM_ABILITY_BY_ID 注册
// 4. FontManagerServer 构造
// 5. OnStart() 被调用
// 6. Publish() 发布服务
// 7. 等待客户端调用
```

## 客户端连接流程

```cpp
// 1. FontManagerClient::InstallFont()
// 2. FontServiceLoadManager::GetFontServiceAbility(FONT_SA_ID)
// 3. IPCSkeleton::GetRemoteObject() 获取 SA 代理
// 4. 通过 Binder IPC 调用服务端
// 5. 服务端处理完成后返回结果
```

## 产物与 Target 映射

| Target | 输出 | 安装路径 |
|--------|------|----------|
| `fontmanager` | libfontmanager.z.so | /system/lib/module/ |
| `font_manager_client` | libfont_manager_client.z.so | /system/lib/ |
| `font_manager_server` | libfont_manager_server.z.so | /system/lib/ 或 /system/lib/module/ |
| `font_server_profile` | 66262.json | /system/etc/sa_config/ |
| `font_sa_etc` | font_manager_server.cfg | /system/etc/ |

## 产物验证

### 检查 N-API 模块

```bash
# 查看模块导出符号
nm -D /system/lib/module/libfontmanager.z.so | grep "T "

# 查看模块依赖
ldd /system/lib/module/libfontmanager.z.so
```

### 检查 SA 配置

```bash
# 查看 SA 配置
cat /system/etc/sa_config/66262.json

# 查看运行中的 SA
dumpsys --dump-service-list
```

### 检查字体安装

```bash
# 查看已安装字体
ls -la /data/service/el1/0/for-all-app/fonts/

# 查看字体配置
cat /data/service/el1/0/for-all-app/fonts/install_fontconfig.json
```

## 相关文档

- [构建目标](04_Build_Targets.md)
- [故障排查](07_Troubleshooting.md)
- [安全评审](06_Security_Review.md)
