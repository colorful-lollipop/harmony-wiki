# 代码地图

> 核心文件定位与代码导航

## 目的

本文档提供项目代码的导航地图，帮助开发者快速定位关键功能的实现位置。

## 适用范围

- OpenHarmony 标准系统
- 组件版本 3.1.0
- 排除测试目录

---

## 顶层目录职责

TODO: 列出 interfaces/、frameworks/ 的子目录职责

## 核心文件定位

| 功能 | 文件路径 | 关键类/函数 |
|------|----------|-------------|
| N-API 入口 | `frameworks/jskitsimpl/src/adaptor/js_module_init.cpp` | `DistributedDataObjectExport` |
| JS 对象封装 | `frameworks/jskitsimpl/src/adaptor/js_distributedobject.cpp` | `JSDistributedObject` |
| JS 存储封装 | `frameworks/jskitsimpl/src/adaptor/js_distributedobjectstore.cpp` | `JSDistributedObjectStore` |
| JS 事件监听 | `frameworks/jskitsimpl/src/adaptor/js_watcher.cpp` | `JSWatcher` |
| 对象存储核心 | `frameworks/innerkitsimpl/src/adaptor/flat_object_store.cpp` | `FlatObjectStore` |
| 存储引擎 | `frameworks/innerkitsimpl/src/adaptor/flat_object_storage_engine.cpp` | `FlatObjectStorageEngine` |
| IPC 客户端 | `frameworks/innerkitsimpl/src/object_service_proxy.cpp` | `ObjectServiceProxy` |
| 权限检查 | `frameworks/innerkitsimpl/src/adaptor/flat_object_storage_engine.cpp` | `VerifyAccessToken` |

## 代码导航图

TODO: 绘制功能到文件路径的映射图

---

## 证据

- 目录结构: Phase 1 扫描结果
- 文件内容: 代码阅读

## 相关链接

- [项目概览](./00_Overview.md)
- [架构详解](./01_Architecture.md)
- [接口文档](./03_Interface.md)
