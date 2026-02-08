# Data Share 概览

## 目的

本文档介绍 OpenHarmony Data Share 部件的项目定位、核心能力和运行环境，帮助开发者快速理解项目的整体架构和设计目标。

## 适用范围

- 初次接触 Data Share 项目的开发者
- 需要了解数据共享机制的系统架构师
- 评估是否使用 Data Share 进行应用开发的技术决策者

## 项目定位

**Data Share（数据共享）** 是 OpenHarmony 分布式数据管理子系统的核心部件，提供跨应用数据共享的标准化机制。

### 解决的问题

| 问题场景 | Data Share 解决方案 |
|---------|-------------------|
| 电话簿、短信、媒体库数据需要被其他应用访问 | 提供标准化的数据访问接口 |
| 账号、密码等敏感数据需要限制访问 | 通过权限系统控制访问范围 |
| 不同数据提供方使用不同的存储方式 | 统一访问接口，屏蔽底层差异 |
| 数据提供方需要繁琐封装才能共享数据 | 框架自动处理 IPC、序列化等 |

### 核心设计目标

1. **统一访问方式** - 无论数据存储在数据库、文件还是内存，访问接口一致
2. **安全可控** - 基于 AccessToken 的细粒度权限控制
3. **高效传输** - 共享内存机制优化大数据量传输
4. **双模式支持** - Silent 模式（直接访问）和 Non-Silent 模式（通过 Extension）

## 核心能力

### 1. 数据 CRUD 操作

```javascript
// 创建 DataShareHelper 实例
let helper = dataShare.createDataShareHelper(context, uri, options);

// 插入数据
helper.insert(uri, valuesBucket);

// 查询数据
let resultSet = helper.query(uri, predicates, columns);

// 更新数据
helper.update(uri, predicates, valuesBucket);

// 删除数据
helper.delete(uri, predicates);
```

### 2. 批量操作

支持批量插入、批量更新，减少 IPC 往返次数：

```javascript
// 批量插入
helper.batchInsert(uri, valuesBuckets);

// 批量更新
helper.batchUpdate(operations);
```

### 3. 数据观察

```javascript
// 注册数据变化观察者
helper.registerObserver(uri, observer);

// 通知数据变化
helper.notifyChange(uri);
```

### 4. 模板订阅

支持基于模板的 RDB 数据订阅和发布数据订阅：

```javascript
// 订阅 RDB 数据变化
helper.subscribeRdbData(uris, templateId, callback);

// 订阅发布数据变化
helper.subscribePublishedData(uris, subscriberId, callback);
```

### 5. 文件操作

```javascript
// 打开远程文件
let fd = helper.openFile(uri, "r");

// 获取文件类型
let types = helper.getFileTypes(uri, mimeTypeFilter);
```

## 运行环境

### 系统要求

| 组件 | 版本要求 | 说明 |
|------|---------|------|
| OpenHarmony | 3.2.0+ | 标准系统 |
| API 级别 | API 9+ | 支持 Stage 模型 |

### 权限要求

- **系统应用** - 可以访问所有 Data Share 接口
- **普通应用** - 需要申请特定的读写权限（由数据提供方定义）

### 系统能力 (Syscap)

```
SystemCapability.DistributedDataManager.DataShare.Core
SystemCapability.DistributedDataManager.DataShare.Consumer
SystemCapability.DistributedDataManager.DataShare.Provider
```

## 关键概念

### 数据提供方 (Provider)

提供数据及实现相关业务的应用程序，也称为生产者或服务端。

**代码位置**: `frameworks/native/provider/`

### 数据访问方 (Consumer)

访问数据提供方所提供的数据或业务的应用程序，也称为消费者或客户端。

**代码位置**: `frameworks/native/consumer/`

### 数据集 (ValuesBucket)

用户要插入的数据集合，以键值对形式存在：

```cpp
// interfaces/inner_api/common/include/datashare_values_bucket.h
std::map<std::string, DataShareValueObject::Type> valuesMap;
```

### 结果集 (ResultSet)

查询后返回的结果集合，支持灵活的数据访问方式：

```javascript
// 遍历结果集
while (resultSet.goToNextRow()) {
    let value = resultSet.getString(columnIndex);
}
```

### 谓词 (Predicates)

数据筛选条件，用于构建查询条件：

```javascript
let predicates = new dataShare.DataSharePredicates();
predicates.equalTo("name", "test").and().greaterThan("age", 18);
```

### URI 格式

**Silent 访问**:
```
datashareproxy://bundleName/moduleName/DB00/TBL01?Proxy=true
```

**Non-Silent 访问**:
```
datashare:///bundleName/moduleName/DB00/TBL01
```

## 架构概览

```
┌─────────────────────────────────────────────────────────────────┐
│                         Application Layer                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │   JS/ETS    │  │  Cangjie    │  │       Native C++        │  │
│  │   (NAPI)    │  │    (FFI)    │  │       (Inner API)       │  │
│  └──────┬──────┘  └──────┬──────┘  └───────────┬─────────────┘  │
└─────────┼────────────────┼─────────────────────┼────────────────┘
          │                │                     │
          └────────────────┴─────────────────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────────┐
│                        Framework Layer                             │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                   DataShareHelper (Consumer)                 │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │  │
│  │  │   Silent    │  │ Non-Silent  │  │   DataProxyHandle   │  │  │
│  │  │   Mode      │  │    Mode     │  │                     │  │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │  │
│  └─────────┼────────────────┼────────────────────┼─────────────┘  │
│            │                │                    │                │
│            └────────────────┴────────────────────┘                │
│                              │                                    │
│  ┌───────────────────────────▼────────────────────────────────┐  │
│  │              DataShareExtAbility (Provider)                 │  │
│  │         ┌──────────────┐  ┌───────────────────┐            │  │
│  │         │  JavaScript  │  │      ArkTS        │            │  │
│  │         │  Extension   │  │   Extension       │            │  │
│  │         └──────────────┘  └───────────────────┘            │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼─────────────────────────────────────┐
│                         IPC Layer                                  │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    DataShareProxy / Stub                     │  │
│  │              (基于 Binder 的 IPC 通信机制)                    │  │
│  └─────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

## 相关文档

- [目录结构](02_Directory_Structure.md) - 详细的模块划分和文件组织
- [架构设计](05_Architecture.md) - 详细的架构图和线程模型
- [N-API 参考](04_NAPI_Reference.md) - 完整的 JS API 文档

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [分布式数据管理子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/分布式数据管理子系统.md)
- [Data Share 代码仓库](https://gitee.com/openharmony/distributeddatamgr_data_share)

## 关键结论

1. **Data Share 是 OpenHarmony 标准化的跨应用数据共享机制**，提供统一接口访问不同存储方式的数据
2. **支持双模式访问**：Silent 模式适合高频访问，Non-Silent 模式需要 Provider 保持运行
3. **基于 AccessToken 的权限控制**，支持细粒度的读写权限管理
4. **共享内存优化大数据传输**，提升查询性能
5. **完整的观察者和订阅机制**，支持数据变化实时通知
