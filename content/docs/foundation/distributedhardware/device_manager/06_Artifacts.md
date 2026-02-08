# 编译产物

## 1. 产物清单

### 1.1 核心库产物

| 产物名称 | 类型 | 说明 | 依赖模块 |
|---------|------|------|---------|
| `libdistributeddevicemanager.so` | 共享库 | JS 4.0+ API 接口 | N-API 绑定层 |
| `libdevicemanagersdk.so` | 共享库 | Inner SDK 接口 | IPC 客户端 |
| `libdevicemanagerserviceimpl.so` | 共享库 | Service 实现核心 | 认证、发现模块 |
| `libdevicemanagerservice.so` | 共享库 | Service 骨架 | IPC 骨架 |
| `libdevicemanagerutils.so` | 共享库 | 公共工具库 | 日志、加密 |

### 1.2 应用产物

| 产物名称 | 类型 | 说明 |
|---------|------|------|
| `DeviceManager_UI.hap` | HAP | PIN 码显示界面 |
| `device_manager.cfg` | 配置文件 | SA 服务配置 |

### 1.3 测试产物

| 产物名称 | 类型 | 说明 |
|---------|------|------|
| `lite_devicemanager_test` | 可执行文件 | 轻量单元测试 |
| `devicemanagerservicetest` | 可执行文件 | 服务测试 |
| `devicemanagerutilstest` | 可执行文件 | 工具库测试 |

## 2. 安装路径

### 2.1 系统库路径

```
# JS SDK 库
/system/lib/module/libdistributeddevicemanager.so
/system/lib/module/libdevicemanagersdk.so

# Service 库
/system/lib/libdevicemanagerserviceimpl.so
/system/lib/libdevicemanagerservice.so

# 工具库
/system/lib/libdevicemanagerutils.so
```

### 2.2 系统配置文件

```
# SA 配置
/system/etc/device_manager/device_manager.cfg

# SA 描述文件
/system/profile/device_manager.json
```

### 2.3 HAP 安装路径

```
# 系统应用 HAP
/data/app/el1/bundle/public/device_manager_ui/
└── DeviceManager_UI.hap
```

### 2.4 数据目录

```
# 服务数据目录
/data/service/el1/public/database/distributed_device_manager_service/

# 凭据存储
/data/service/el1/public/database/distributed_device_manager_service/credentials/
```

> 证据来源：`sa_profile/device_manager.cfg:5-7`

## 3. 运行时加载关系

### 3.1 库加载顺序

```
应用进程
    │
    ▼
libhilog.so (系统日志)
    │
    ▼
libipc_core.so (IPC 框架)
    │
    ▼
libdmdevicecache.so (设备缓存)
    │
    ▼
libdevicemanagersdk.so (SDK)
    │
    ▼
libdistributeddevicemanager.so (JS API)
    │
    ▼
应用代码
```

### 3.2 服务进程

```
device_manager 进程
    │
    ├── libhilog.so
    ├── libipc_core.so
    ├── libdmipc_client.so
    │
    ├── libdevicemanagerservice.so (SA 骨架)
    │
    ├── libdevicemanagerserviceimpl.so (核心实现)
    │       │
    │       ├── libdeviceauth.so (设备认证)
    │       ├── libdsoftbus.so (软总线)
    │       ├── libdeviceprofile.so (设备档案)
    │       └── ...
    │
    └── libdevicemanagerutils.so (工具库)
```

### 3.3 动态链接库加载

```cpp
// dlopen 示例
void* handle = dlopen("libdevicemanagersdk.so", RTLD_NOW);
if (handle) {
    typedef IDeviceManager* (*CreateFunc)();
    CreateFunc create = (CreateFunc)dlsym(handle, "CreateDeviceManager");
    // ...
}
```

## 4. 产物验证

### 4.1 库文件验证

```bash
# 检查库依赖
ldd libdistributeddevicemanager.so

# 检查符号导出
nm -D libdistributeddevicemanager.so | grep "T "

# 检查共享库信息
readelf -d libdistributeddevicemanager.so
```

### 4.2 HAP 验证

```bash
# 解压 HAP
unzip DeviceManager_UI.hap -d dm_ui/

# 检查签名
java -jar hapverify.jar DeviceManager_UI.hap

# 检查模块配置
cat dm_ui/config.json
```

### 4.3 SA 配置验证

```bash
# 检查 SA 配置语法
cat device_manager.cfg | python3 -m json.tool

# 检查 SA 注册
hdc shell sa心灵 "device_manager"
```

## 5. 产物尺寸

### 5.1 库大小（典型值）

| 产物 | 文本段 | 数据段 | 总大小 |
|-----|-------|-------|-------|
| libdistributeddevicemanager.so | ~500KB | ~50KB | ~550KB |
| libdevicemanagersdk.so | ~300KB | ~30KB | ~330KB |
| libdevicemanagerserviceimpl.so | ~1.5MB | ~100KB | ~1.6MB |
| libdevicemanagerservice.so | ~200KB | ~20KB | ~220KB |
| libdevicemanagerutils.so | ~100KB | ~10KB | ~110KB |

### 5.2 优化建议

- **Strip**：Release 构建时移除符号表
- **压缩**：使用 LZ4/Zstd 压缩
- **分包**：按需加载功能模块

```bash
# Strip 操作
arm-linux-gnueabi-strip --strip-all libdistributeddevicemanager.so
```

## 6. 版本兼容性

### 6.1 API 版本

| API 版本 | 库版本 | 最低系统版本 |
|---------|-------|-------------|
| JS 4.0+ | v2 | OpenHarmony 4.0.9.2+ |
| Legacy JS | v1 | OpenHarmony 3.2+ |

### 6.2 ABI 兼容性

```cpp
// N-API 版本检查
#if NAPI_VERSION >= 8
// 使用 NAPI 8 特性
#endif
```

### 6.3 SO 版本管理

```gn
# SONAME 设置
soversion = "1.0.0"

# 符号版本
version = "DM_1.0"
```

## 7. 调试符号

### 7.1 调试信息

```bash
# 提取调试信息
objcopy --only-keep-debug libdevicemanagerserviceimpl.so \
    libdevicemanagerserviceimpl.debug

# 分离调试符号
objcopy --strip-debug --add-gnu-debuglink=libdevicemanagerserviceimpl.debug \
    libdevicemanagerserviceimpl.so
```

### 7.2 Crash 分析

```bash
# 使用 addr2line 定位
arm-linux-gnueabi-addr2line -e libdevicemanagerserviceimpl.so \
    -C -f 0x00001234
```

## 8. 签名要求

### 8.1 系统签名

DeviceManager 作为系统组件，需要使用系统签名：

```bash
# 使用系统签名签名 HAP
hdc app sign -mode ohos_provision \
    -profile /path/to/system_profile.p7b \
    DeviceManager_UI.hap
```

### 8.2 权限声明

```json
{
  "bundle": {
    "signature": "/path/to/device_manager.p7b"
  },
  "acl": {
    "permissions": [
      "ohos.permission.MANAGE_SECURE_SETTINGS"
    ]
  }
}
```
