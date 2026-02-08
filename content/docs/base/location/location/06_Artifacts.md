# 编译产物说明

> **目的**: 详细说明位置服务组件的所有编译产物、安装路径和运行时加载关系  
> **适用范围**: 集成工程师、系统工程师、需要了解部署细节的人员  
> **最后更新**: 2026-02-05

---

## 1. 产物概览

### 1.1 产物分类

| 分类 | 产物类型 | 说明 |
|------|----------|------|
| NDK 库 | `.so` | C API 库，供 Native 应用使用 |
| SDK 库 | `.so` | Native SDK，供框架内部使用 |
| N-API 库 | `.so` | JS 接口库，供 JS/ETS 应用使用 |
| SA 服务库 | `.so` | 系统能力服务进程库 |
| HAP | `.hap` | 定位权限对话框 |

### 1.2 产物清单

| 产物 | 路径 | 类型 | 说明 |
|------|------|------|------|
| `libohlocation.z.so` | `system/lib64/` | NDK 库 | C API |
| `liblocator_sdk.z.so` | `system/lib64/` | SDK 库 | Native SDK |
| `libgeolocation.z.so` | `system/lib64/module/` | N-API 库 | geolocation |
| `libgeolocationmanager.z.so` | `system/lib64/module/` | N-API 库 | LocationManager |
| `liblbsservice_geocode.z.so` | `system/lib64/sa_dynamic_libs/` | SA 库 | Geocode SA |
| `liblbsservice_locator.z.so` | `system/lib64/sa_dynamic_libs/` | SA 库 | Locator SA |
| `liblbsservice_gnss.z.so` | `system/lib64/sa_dynamic_libs/` | SA 库 | GNSS SA |
| `liblbsservice_network.z.so` | `system/lib64/sa_dynamic_libs/` | SA 库 | Network SA |
| `liblbsservice_passive.z.so` | `system/lib64/sa_dynamic_libs/` | SA 库 | Passive SA |
| `lbsresources` | `system/etc/` | 资源文件 | 位置服务资源 |
| `lbsbase_module` | `system/etc/` | 资源文件 | 基础模块资源 |
| `location_dialog.hap` | `system/app/location_dialog/` | HAP | 权限对话框 |

---

## 2. NDK 库详情

### 2.1 libohlocation.so

**路径**: `system/lib64/libohlocation.z.so`

**用途**: 提供 C API 接口，供 Native 应用调用

**功能**:
- 检查定位开关状态
- 开始/停止定位
- 配置定位参数
- 接收位置回调

**链接方式**:
```cmake
target_link_libraries(myapp PUBLIC libohlocation.z.so)
```

**依赖**:
- libc++
- libuv
- hilog

---

## 3. SDK 库详情

### 3.1 liblocator_sdk.so

**路径**: `system/lib64/liblocator_sdk.z.so`

**用途**: Native SDK，供框架内部模块使用

**功能**:
- 定位请求管理
- SA 通信
- 权限检查

**导出符号**:
- `LocatorImpl`
- `LocationDataManager`
- `PermissionManager`

---

## 4. N-API 库详情

### 4.1 libgeolocation.so

**路径**: `system/lib64/module/libgeolocation.z.so`

**用途**: `@ohos.geolocation` 模块

**导出**:
- `geolocation` 对象
- `LocationRequest`
- `Location`
- `GeoAddress`

### 4.2 libgeolocationmanager.so

**路径**: `system/lib64/module/libgeolocationmanager.z.so`

**用途**: `LocationManager` 类

**功能**:
- `isLocationEnabled()`
- `enableLocation()`
- `disableLocation()`
- `on('locationStateChange')`

---

## 5. SA 服务库详情

### 5.1 运行时加载

所有 SA 服务运行在 `locationhub` 进程中：

| SA | 库文件 | SA ID | 启动方式 |
|----|--------|-------|----------|
| Geocode | `liblbsservice_geocode.z.so` | 2801 | 按需 |
| Locator | `liblbsservice_locator.z.so` | 2802 | 按需 |
| GNSS | `liblbsservice_gnss.z.so` | 2803 | 按需 |
| Network | `liblbsservice_network.z.so` | 2804 | 按需 |
| Passive | `liblbsservice_passive.z.so` | 2805 | 按需 |

### 5.2 依赖关系

```
liblbsservice_locator.z.so
├── liblbsservice_gnss.z.so
├── liblbsservice_network.z.so
├── liblbsservice_passive.z.so
└── liblbsservice_geocode.z.so
```

---

## 6. 资源文件

### 6.1 lbsresources

**路径**: `system/etc/lbsresources/`

**内容**:
- 配置文件
- 权限策略文件
- 地理数据（可选）

### 6.2 lbsbase_module

**路径**: `system/etc/lbsbase_module/`

**内容**:
- 基础模块配置
- 默认参数

---

## 7. HAP 产物

### 7.1 location_dialog.hap

**路径**: `system/app/location_dialog/location_dialog.hap`

**用途**: 定位权限对话框

**触发时机**:
- 用户首次请求精确定位
- 定位开关已开启但权限未授予

**功能**:
- 展示权限请求 UI
- 获取用户授权结果

---

## 8. 运行时加载关系

### 8.1 应用启动时

```
应用进程
    ↓ 加载
libgeolocation.z.so (JS N-API)
    ↓ IPC 调用
locator_sdk.z.so (框架层)
    ↓ IPC 调用
locationhub 进程
    ↓ 加载
lbsservice_locator.z.so
    ↓ 调用
lbsservice_gnss.z.so / lbsservice_network.z.so
```

### 8.2 按需加载

- SA 库：首次 API 调用时加载
- HAP：权限对话框弹出时加载

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [GN 构建配置](05_Build.md) | 构建配置详情 |
| [系统架构](01_Architecture.md) | 架构与组件关系 |
| [C/N-API 接口](02_C_NAPI.md) | API 使用说明 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
