# 内部实现细节

> 核心类、结构体职责和资源生命周期

## 目的

本文档描述分布式数据对象组件的内部实现细节，包括核心类的职责、内部 API 契约和资源生命周期管理，供框架层开发者参考。

## 适用范围

- OpenHarmony 标准系统
- 组件版本 3.1.0
- 内部实现视角

---

## 核心类与结构体

### DistributedObject

**职责**: 分布式对象抽象接口，提供数据读写、持久化和绑定功能

**关键方法**:
- `PutDouble/PutString/PutBoolean/PutComplex`: 写入数据
- `GetDouble/GetString/GetBoolean/GetComplex`: 读取数据
- `Save/RevokeSave`: 持久化操作
- `BindAssetStore`: 绑定资产存储
- `GetSessionId`: 获取会话 ID

**位置**: `interfaces/innerkits/distributed_object.h:27-161`

**证据**: `interfaces/innerkits/distributed_object.h`

---

### DistributedObjectStore

**职责**: 对象存储工厂接口，提供对象创建、删除、观察者管理

**关键方法**:
- `GetInstance`: 获取单例
- `CreateObject`: 创建对象实例
- `Get/DeleteObject`: 检索/删除对象
- `Watch/UnWatch`: 注册/取消观察者
- `SetStatusNotifier/SetProgressNotifier`: 设置通知器

**位置**: `interfaces/innerkits/distributed_objectstore.h:32-136`

**证据**: `interfaces/innerkits/distributed_objectstore.h`

---

### FlatObjectStore

**职责**: 对象存储核心实现，管理缓存、观察者和 IPC 连接

**关键成员**:
- `cacheManager_`: 缓存管理器
- `watchers_`: 观察者列表
- `statusNotifiers_`: 状态通知器
- `progressNotifiers_`: 进度通知器

**关键方法**:
- `CreateObject`: 创建对象并初始化存储引擎
- `Watch/UnWatch`: 管理数据变更观察者
- `SetStatusNotifier`: 管理设备上线/下线通知

**位置**: `frameworks/innerkitsimpl/src/adaptor/flat_object_store.cpp`

**证据**: `frameworks/innerkitsimpl/src/adaptor/flat_object_store.cpp`

---

### FlatObjectStorageEngine

**职责**: 存储引擎，封装 KvStore 操作，处理权限检查

**关键方法**:
- `Init`: 初始化存储引擎，验证权限
- `PutXXX/GetXXX`: 读写操作
- `SyncAllData`: 同步数据到远程设备
- `RegisterObserver`: 注册数据变更观察

**权限检查**:
```cpp
int32_t ret = Security::AccessToken::AccessTokenKit::VerifyAccessToken(tokenId, DISTRIBUTED_DATASYNC);
if (ret == Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
    // 允许操作
}
```

**位置**: `frameworks/innerkitsimpl/src/adaptor/flat_object_storage_engine.cpp:43-45`

**证据**: `frameworks/innerkitsimpl/src/adaptor/flat_object_storage_engine.cpp`

---

### ObjectServiceProxy

**职责**: IPC 客户端代理，调用后台服务的 IPC 接口

**关键方法**:
- `Save`: 调用 OBJECTSTORE_SAVE 接口
- `RevokeSave`: 调用 OBJECTSTORE_REVOKE_SAVE 接口
- `Retrieve`: 调用 OBJECTSTORE_RETRIEVE 接口

**位置**: `frameworks/innerkitsimpl/src/object_service_proxy.cpp`

**证据**: `frameworks/innerkitsimpl/src/object_service_proxy.cpp`

---

## 内部 API 契约

### 稳定接口

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `DistributedObject` | 稳定 | 内部实现可替换，接口不变 |
| `DistributedObjectStore` | 稳定 | 内部实现可替换，接口不变 |
| N-API 导出函数 | 不稳定 | 可能随 JS API 变化 |

### 内部实现细节

| 类 | 稳定性 | 说明 |
|------|--------|------|
| `FlatObjectStore` | 不稳定 | 实现细节可能变化 |
| `FlatObjectStorageEngine` | 不稳定 | 实现细节可能变化 |

---

## 资源生命周期

### 对象创建流程

```
DistributedObjectStore::CreateObject()
  ↓
FlatObjectStore::CreateObject()
  ↓
FlatObjectStorageEngine::Init() [权限检查]
  ↓
KvStore::Create()
  ↓
返回 DistributedObject 实例
```

### 观察者注册流程

```
DistributedObjectStore::Watch()
  ↓
FlatObjectStore::Watch()
  ↓
创建 WatcherProxy
  ↓
FlatObjectStorageEngine::RegisterObserver()
  ↓
返回成功
```

### 对象销毁流程

```
DistributedObjectStore::DeleteObject()
  ↓
FlatObjectStore::DeleteObject()
  ↓
移除观察者
  ↓
释放 FlatObjectStorageEngine
  ↓
返回成功
```

### 内存管理策略

- **RAII 模式**: 使用 `std::shared_ptr`、`std::unique_ptr` 管理资源
- **互斥锁**: 使用 `std::mutex` 保护共享数据
- **观察者**: 使用 `std::weak_ptr` 避免循环引用

**证据**: Phase 1 Grep 搜索结果

---

## 证据

- 接口定义: `distributed_object.h`, `distributed_objectstore.h`
- 核心实现: `flat_object_store.cpp`, `flat_object_storage_engine.cpp`
- IPC 代理: `object_service_proxy.cpp`
- 内存管理: Phase 1 Grep 搜索结果

## 相关链接

- [代码地图](./02_CodeMap.md)
- [架构详解](./01_Architecture.md)
- [安全评审](./05_SecurityReview.md)
