# API/接口差异分析

## 5.1 概述

OpenHarmony 版本的 protobuf **未对上游 API 进行任何修改**。所有 API 接口与上游版本 5.29.4 保持完全一致。

### 差异统计

| 差异类型 | 数量 | 说明 |
|----------|------|------|
| **新增 API** | 0 | 无 OH 特有 API |
| **行为变更** | 0 | 无 API 行为修改 |
| **禁用功能** | 0 | 无功能禁用 |
| **废弃接口** | 0 | 跟随上游版本 |

## 5.2 API 一致性说明

### 5.2.1 核心 API 列表

| API 类别 | 状态 | 上游版本 |
|----------|------|----------|
| **MessageLite** | ✅ 一致 | 5.29.4 |
| **Message** | ✅ 一致 | 5.29.4 |
| **Descriptor** | ✅ 一致 | 5.29.4 |
| **Arena** | ✅ 一致 | 5.29.4 |
| **RepeatedField** | ✅ 一致 | 5.29.4 |
| **Reflection** | ✅ 一致 | 5.29.4 |
| **DescriptorPool** | ✅ 一致 | 5.29.4 |
| **IO Utils** | ✅ 一致 | 5.29.4 |
| **JSON Util** | ✅ 一致 | 5.29.4 |

### 5.2.2 Protobuf Lite 限制

protobuf_lite 版本在 OH 中与上游 lite 版本功能一致，包括：

**可用功能**：
- 消息序列化/反序列化
- Arena 内存分配
- 基础类型支持
- 扩展字段
- 未知字段处理

**不可用功能**：
- 反射（Reflection）
- 描述符（Descriptor）
- 代码生成器接口
- 动态消息

这些限制**并非 OH 修改**，而是 protobuf_lite 版本的固有限制。

## 5.3 OH 特有配置

### 5.3.1 构建时配置

虽然 API 无差异，但构建时的行为可通过以下方式定制：

```cpp
// 如果定义了 HAVE_HILOG，protobuf 内部会使用 HILOG
// 否则使用标准日志输出
#ifdef HAVE_HILOG
#include <hilog/log.h>
#define PROTOBUF_LOG(level, ...) OH_LOG(level, __VA_ARGS__)
#else
#define PROTOBUF_LOG(level, ...) fprintf(stderr, __VA_ARGS__)
#endif
```

### 5.3.2 性能调优配置

```gn
# 启用优化选项
cflags = [
  "-O2",                    # 优化级别
  "-fomit-frame-pointer",  # 省略帧指针
]
```

## 5.4 ABI 兼容性

### 5.4.1 ABI 风险

protobuf 存在已知的 **ABI 兼容性风险**，主要原因：

| 风险因素 | 说明 |
|----------|------|
| **模板实例化** | 内部大量使用模板，编译器差异可能导致 ABI 不兼容 |
| **虚函数表** | Message 基类的虚函数表结构变化 |
| **字符串实现** | std::string 的实现差异（SSO vs 堆分配） |

### 5.4.2 OH 兼容性措施

| 措施 | 说明 |
|------|------|
| **统一编译器** | OH 构建统一使用指定版本的 Clang |
| **静态链接** | 核心模块建议静态链接 protobuf |
| **版本锁定** | 保持 protobuf 版本稳定 |

### 5.4.3 ABI 检查命令

```bash
# 检查符号导出
nm -D libprotobuf.so | grep " T " | head -20

# 检查 ABI 兼容性
abi-compliance-checker -lib protobuf.xml -old libprotobuf.so -new libprotobuf_new.so
```

## 5.5 使用注意事项

### 5.5.1 跨模块数据传递

```cpp
// 模块 A：序列化
std::string SerializeMessage(const MyMessage& msg) {
  return msg.SerializeAsString();
}

// 模块 B：反序列化
MyMessage DeserializeMessage(const std::string& data) {
  MyMessage msg;
  msg.ParseFromString(data);
  return msg;
}

// ⚠️ 注意事项：
// 1. 两个模块必须使用相同版本的 protobuf
// 2. 建议静态链接到同一版本的库
// 3. .proto 文件定义需要保持一致
```

### 5.5.2 Arena 内存管理

```cpp
// 推荐：使用 Arena 减少内存分配开销
google::protobuf::Arena arena;
MyMessage* msg = google::protobuf::Arena::CreateMessage<MyMessage>(&arena);
// msg 在 arena 销毁时自动释放
```

### 5.5.3 线程安全

```cpp
// protobuf 基本类型是线程安全的
// ⚠️ Message 实例不是线程安全的
// 需要在多线程场景中自行同步
```

## 5.6 升级兼容性

### 5.6.1 上游版本升级

升级到新版本 protobuf 时需注意：

| 升级类型 | 兼容性 | 注意事项 |
|----------|--------|----------|
| **Patch 版本** | ✅ 兼容 | 通常无 API 变更 |
| **Minor 版本** | ⚠️ 谨慎 | 可能有小的 API 变更 |
| **Major 版本** | ❌ 不兼容 | 可能有破坏性变更 |

### 5.6.2 迁移建议

```bash
# 1. 更新 .proto 文件使用新语法
syntax = "proto3";

# 2. 重新编译生成代码
protoc --cpp_out=. your_file.proto

# 3. 编译测试
# 4. 运行单元测试
```

## 5.7 总结

| 维度 | 状态 | 说明 |
|------|------|------|
| **API 一致性** | ✅ 完全一致 | 与上游 5.29.4 API 相同 |
| **ABI 兼容性** | ⚠️ 存在风险 | 需注意编译器/版本匹配 |
| **功能完整性** | ✅ 完整 | lite/full 版本功能完整 |
| **OH 定制** | ❌ 无 | 无 OH 特有 API |
