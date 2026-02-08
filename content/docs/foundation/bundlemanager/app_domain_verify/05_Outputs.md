# 编译产物清单

> 本文档描述 app_domain_verify 部件的编译产物，包括文件列表、安装路径和运行时加载关系。

## 1. 共享库 (.so)

### 1.1 系统库

| 产物 | 源文件 | 安装路径 | 说明 |
|-----|-------|---------|------|
| `libapp_domain_verify_common.so` | interfaces/inner_api/common | system/lib | 公共数据结构和接口 |
| `libapp_domain_verify_frameworks_common.so` | frameworks/common | system/lib | 框架公共代码 |
| `libapp_domain_verify_app_details_rdb.so` | frameworks/app_details_rdb | system/lib | RDB 数据管理 |
| `libapp_domain_verify_extension_framework.so` | frameworks/extension | system/lib | 扩展框架 |
| `libapp_domain_verify_agent_verifier.so` | frameworks/verifier | system/lib | 校验器实现 |
| `libapp_domain_verify_mgr_client.so` | interfaces/inner_api/client | system/lib | 客户端接口 |
| `libapp_domain_verify_agent_client.so` | interfaces/inner_api/client | system/lib | Agent 客户端 |
| `libapp_domain_verify_mgr_service.so` | services | system/lib | Manager Service |
| `libapp_domain_verify_agent_service.so` | services | system/lib | Agent Service |

### 1.2 模块库

| 产物 | 源文件 | 安装路径 | 说明 |
|-----|-------|---------|------|
| `libappdomainverify_napi.so` | interfaces/kits/js/jsi | system/lib/module/bundle/ | N-API 实现 |

### 1.3 ANI 库

| 产物 | 源文件 | 安装路径 | 说明 |
|-----|-------|---------|------|
| `libapp_domain_verify_ani.so` | interfaces/kits/js/ani | system/lib/ | ANI 接口实现 |

## 2. 字节码文件 (.abc)

| 产物 | 源文件 | 安装路径 | 说明 |
|-----|-------|---------|------|
| `app_domain_verify_ets.abc` | interfaces/kits/js/ani | system/framework/ | ArkTS 字节码 |

## 3. 配置文件

### 3.1 API 报告配置

| 产物 | 源文件 | 安装路径 | 说明 |
|-----|-------|---------|------|
| `api_report.conf` | etc/ | system/etc/app_domain_verify/ | API 上报配置 |

### 3.2 服务配置

| 产物 | 源文件 | 安装路径 | 说明 |
|-----|-------|---------|------|
| `app_domain_verify_agent.cfg` | etc/init/ | system/etc/init/ | Agent 服务配置 |

**配置文件内容**:

```json
{
    "jobs": [{
        "name": "services:app_domain_verify_agent_service",
        "cmds": [
            "mkdir /data/service/el1/public/app_domain_verify_agent_service 0770 app_domain_verify system",
            "chown app_domain_verify system /data/service/el1/public/app_domain_verify_agent_service",
            "chmod 0770 /data/service/el1/public/app_domain_verify_agent_service"
        ]
    }],
    "services": [{
        "name": "app_domain_verify_agent",
        "path": ["/system/bin/sa_main", "/system/profile/app_domain_verify_agent.json"],
        "uid": "app_domain_verify",
        "gid": ["app_domain_verify", "system", "shell", "netsys_socket"],
        "caps": [],
        "ondemand": true,
        "permission": [
            "ohos.permission.INTERNET",
            "ohos.permission.MANAGE_SECURE_SETTINGS",
            "ohos.permission.GET_BUNDLE_INFO",
            "ohos.permission.GET_NETWORK_INFO"
        ],
        "secon": "u:r:app_domain_verify_agent:s0",
        "sandbox": 0
    }]
}
```

## 4. SA 配置文件

| 产物 | 源文件 | 安装路径 | SA ID | 说明 |
|-----|-------|---------|-------|------|
| `6200.json` | profile/ | system/profile/ | 6200 | Manager Service SA 配置 |
| `6201.json` | profile/ | system/profile/ | 6201 | Agent Service SA 配置 |

### 4.1 Manager Service SA 配置 (6200.json)

```json
{
  "process": "foundation",
  "systemability": [
    {
      "name": 6200,
      "libpath": "libapp_domain_verify_mgr_service.z.so",
      "run-on-create": true,
      "distributed": false,
      "min_hdi_proxy_version": [],
      "dump_level": 1
    }
  ]
}
```

### 4.2 Agent Service SA 配置 (6201.json)

```json
{
  "process": "app_domain_verify_agent",
  "systemability": [
    {
      "name": 6201,
      "libpath": "libapp_domain_verify_agent_service.z.so",
      "run-on-create": false,
      "distributed": false,
      "dump_level": 1,
      "min_hdi_proxy_version": [],
      "start-on-demand": {
        "commonevent":[
          {
            "name":"usual.event.BOOT_COMPLETED"
          }
        ],
        "timedevent":[
          {
              "name":"loopevent",
              "value":"86400",
              "persistence":true
          }
        ]
      }
    }
  ]
}
```

## 5. 运行时加载关系

### 5.1 服务加载顺序

```
1. 系统启动
   │
   ▼
2. foundation 进程启动
   │
   ▼
3. SystemAbilityManager 加载 SA 6200 (Manager Service)
   │
   ▼
4. libapp_domain_verify_mgr_service.so 加载
   │
   ├─► 依赖 libapp_domain_verify_common.so
   ├─► 依赖 libapp_domain_verify_app_details_rdb.so
   └─► 依赖 libapp_domain_verify_agent_client.so
   
5. 用户触发需要校验的操作
   │
   ▼
6. SA 6201 按需启动 (Agent Service)
   │
   ▼
7. libapp_domain_verify_agent_service.so 加载
   │
   ├─► 依赖 libapp_domain_verify_mgr_client.so
   ├─► 依赖 libapp_domain_verify_frameworks_common.so
   ├─► 依赖 libapp_domain_verify_extension_framework.so
   ├─► 依赖 libapp_domain_verify_agent_verifier.so
   └─► 依赖 libapp_domain_verify_app_details_rdb.so
```

### 5.2 应用调用加载

```
应用进程
   │
   ▼
libnapi.z.so (N-API 框架)
   │
   ▼
libappdomainverify_napi.so (按需加载)
   │
   ├─► 依赖 libapp_domain_verify_mgr_client.so
   └─► 依赖 libapp_domain_verify_common.so
   
   │
   ▼
Binder IPC 到 foundation 进程
   
   │
   ▼
libapp_domain_verify_mgr_service.so (Manager Service)
```

### 5.3 依赖汇总

```
libapp_domain_verify_common.so
├── libhilog.so
├── libipc.so
└── libsafwk.so

libapp_domain_verify_frameworks_common.so
├── libapp_domain_verify_common.so
├── libhilog.so
├── libhisysevent.so
├── libaccess_token.so
├── libnetstack.so
└── libffrt.so

libapp_domain_verify_app_details_rdb.so
├── libapp_domain_verify_frameworks_common.so
└── librdb.so

libapp_domain_verify_mgr_service.so
├── libapp_domain_verify_agent_client.so
├── libapp_domain_verify_common.so
└── libapp_domain_verify_app_details_rdb.so

libapp_domain_verify_agent_service.so
├── libapp_domain_verify_mgr_client.so
├── libapp_domain_verify_common.so
├── libapp_domain_verify_app_details_rdb.so
├── libapp_domain_verify_frameworks_common.so
├── libapp_domain_verify_extension_framework.so
└── libapp_domain_verify_agent_verifier.so

libappdomainverify_napi.so
├── libapp_domain_verify_mgr_client.so
└── libapp_domain_verify_common.so
```

## 6. 数据目录

### 6.1 运行时数据目录

| 目录 | 权限 | 说明 |
|-----|------|------|
| `/data/service/el1/public/app_domain_verify_agent_service/` | 770 app_domain_verify system | Agent Service 数据目录 |

### 6.2 数据库文件

| 文件 | 说明 |
|-----|------|
| `app_domain_verify.db` | 域名校验结果数据库 |

## 7. 相关文档

| 文档 | 链接 |
|-----|------|
| GN 构建配置 | [04_GN_Build.md](./04_GN_Build.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 安全风险评审 | [06_Security.md](./06_Security.md) |
