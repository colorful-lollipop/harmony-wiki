# 05 - API/接口差异

## 概述

本文档记录 gRPC 在 OpenHarmony 中的 API 差异，包括 OH 新增、修改或禁用的接口。

**重要说明**: gRPC 在 OH 中的 API 与上游标准 gRPC API **基本一致**，差异主要体现在构建配置和运行时行为上。

## 1. API 兼容性声明

### 1.1 标准 API

gRPC 在 OH 中提供的 API 与上游标准 API **100% 源代码兼容**：

- C API (`grpc/*`) - 完全兼容
- C++ API (`grpcpp/*`) - 完全兼容
- Protocol Buffers 代码生成 - 完全兼容

### 1.2 差异类型

| 类型 | 说明 | 影响 |
|------|------|------|
| **配置差异** | 编译时宏定义不同 | 运行时行为 |
| **依赖差异** | 底层实现库不同 | 功能等价 |
| **性能差异** | 某些优化选项不同 | 性能表现 |

## 2. 配置差异导致的 API 行为差异

### 2.1 DNS 解析行为

**上游默认**: 使用 c-ares 异步 DNS
```cpp
// 上游：异步 DNS，不阻塞线程
auto channel = grpc::CreateChannel(endpoint, creds);
```

**OH 配置**: 禁用 c-ares，使用系统 DNS
```gn
# BUILD.gn
defines = ["GRPC_ARES=0"]
```

**影响**:
- DNS 解析变为同步（阻塞）
- 在大多数情况下行为一致
- 高并发场景可能有性能差异

**代码层面**: 无需修改，行为由库内部处理

### 2.2 Protobuf 版本

**上游**: 默认使用 full protobuf
```cpp
// 支持反射
const google::protobuf::Descriptor* desc = 
    message.GetDescriptor();
```

**OH 配置**: 使用 protobuf lite
```gn
# BUILD.gn
defines = ["GRPC_USE_PROTO_LITE"]
external_deps = ["protobuf:protobuf_lite"]
```

**影响**:
- 不支持反射 API
- 二进制体积减小
- 性能略有提升

**代码层面建议**:
```cpp
// 避免使用反射
// ❌ 不推荐
const Descriptor* desc = msg.GetDescriptor();

// ✅ 推荐：使用生成的类型安全 API
if (msg.has_field()) {
    auto value = msg.field();
}
```

### 2.3 TLS/SSL 配置

**上游**: 使用 BoringSSL
```cpp
// 上游默认
grpc::SslCredentialsOptions ssl_opts;
```

**OH 配置**: 使用系统 OpenSSL
```gn
# BUILD.gn
external_deps = [
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
]
```

**影响**:
- API 完全相同
- 底层实现不同（对应用透明）
- 证书管理方式可能与系统其他组件统一

## 3. 新增/扩展的 API

### 3.1 无 OH 特有 API

gRPC 在 OH 中**没有添加**特有的 API 扩展。

所有接口都是标准 gRPC API，这确保了：
- 代码可移植性
- 与上游文档一致
- 便于维护和升级

### 3.2 可能的扩展点

如果需要 OH 特有的功能，建议通过以下方式实现：

1. **拦截器 (Interceptor)**:
   ```cpp
   class OHInterceptor : public grpc::experimental::Interceptor {
       // 实现 OH 特有的处理逻辑
   };
   ```

2. **自定义 Channel 参数**:
   ```cpp
   grpc::ChannelArguments args;
   args.SetInt("ohos.custom_param", value);
   ```

3. **元数据 (Metadata)**:
   ```cpp
   context.AddMetadata("x-ohos-device-id", device_id);
   ```

## 4. 行为差异详细说明

### 4.1 连接超时

由于 DNS 解析方式不同，默认连接超时行为可能略有差异：

```cpp
// 建议显式设置超时参数
grpc::ChannelArguments args;
args.SetInt(GRPC_ARG_KEEPALIVE_TIME_MS, 10000);
args.SetInt(GRPC_ARG_KEEPALIVE_TIMEOUT_MS, 5000);

auto channel = grpc::CreateCustomChannel(
    endpoint, creds, args);
```

### 4.2 压缩支持

**配置**:
```gn
# zlib 支持
external_deps = ["zlib:libz"]
```

**使用**:
```cpp
// 标准 API，无差异
grpc::ChannelArguments args;
args.SetCompressionAlgorithm(GRPC_COMPRESS_GZIP);
```

### 4.3 日志输出

**上游**: 使用 stderr 或 glog

**OH**: 使用 hilog

**对应用的影响**:
- 日志格式变化
- 使用 `hilog` 查看日志
- API 层面无差异（通过环境变量控制）

```bash
# 查看 gRPC 日志
hilog | grep grpc

# 设置日志级别
export GRPC_VERBOSITY=DEBUG
export GRPC_TRACE=all
```

## 5. 迁移指南

### 5.1 从上游 gRPC 迁移到 OH gRPC

如果您有使用上游 gRPC 的代码，迁移到 OH：

**无需修改的代码**:
- 所有 gRPC API 调用
- protobuf 消息定义
- 服务定义 (.proto 文件)
- 客户端/服务端代码

**需要调整的**:
- 构建配置（Bazel → GN）
- 依赖声明方式
- 日志查看命令

### 5.2 迁移检查清单

- [ ] 确认使用 protobuf lite 兼容的代码
- [ ] 更新 BUILD.gn 依赖声明
- [ ] 测试 DNS 解析超时行为
- [ ] 验证 SSL/TLS 证书配置
- [ ] 更新日志收集方式

## 6. 已知限制

### 6.1 无已知功能限制

gRPC 在 OH 中支持所有标准功能：
- ✅ Unary RPC（单次请求/响应）
- ✅ Server Streaming（服务端流）
- ✅ Client Streaming（客户端流）
- ✅ Bidirectional Streaming（双向流）
- ✅ SSL/TLS 加密
- ✅ 认证和授权
- ✅ 压缩
- ✅ 负载均衡
- ✅ 健康检查

### 6.2 性能考虑

| 场景 | 差异 | 建议 |
|------|------|------|
| DNS 解析 | 同步 vs 异步 | 使用连接池复用 Channel |
| 二进制体积 | 较小（lite） | 已经优化 |
| 日志 | hilog 开销 | 生产环境设置 WARN 级别 |

## 7. API 使用示例

### 7.1 标准 Unary RPC

```cpp
#include <grpcpp/grpcpp.h>
#include "my_service.grpc.pb.h"

// 创建客户端
class MyClient {
public:
    MyClient(const std::string& address) {
        auto channel = grpc::CreateChannel(
            address, 
            grpc::InsecureChannelCredentials()
        );
        stub_ = MyService::NewStub(channel);
    }
    
    std::string CallMethod(const std::string& input) {
        Request request;
        request.set_input(input);
        
        Response response;
        grpc::ClientContext context;
        
        grpc::Status status = stub_->MyMethod(
            &context, request, &response
        );
        
        if (status.ok()) {
            return response.output();
        }
        return "Error: " + status.error_message();
    }

private:
    std::unique_ptr<MyService::Stub> stub_;
};
```

### 7.2 流式 RPC

```cpp
// 服务端流
grpc::Status ServerStream(
    grpc::ServerContext* context,
    const Request* request,
    grpc::ServerWriter<Response>* writer
) {
    for (int i = 0; i < 10; i++) {
        Response resp;
        resp.set_data("chunk " + std::to_string(i));
        writer->Write(resp);
    }
    return grpc::Status::OK;
}

// 客户端流
grpc::Status ClientStream(
    grpc::ServerContext* context,
    grpc::ServerReader<Request>* reader,
    Response* response
) {
    Request req;
    while (reader->Read(&req)) {
        // 处理请求
    }
    response->set_result("done");
    return grpc::Status::OK;
}

// 双向流
grpc::Status BidiStream(
    grpc::ServerContext* context,
    grpc::ServerReaderWriter<Response, Request>* stream
) {
    Request req;
    while (stream->Read(&req)) {
        Response resp;
        resp.set_echo(req.data());
        stream->Write(resp);
    }
    return grpc::Status::OK;
}
```

### 7.3 SSL/TLS 配置

```cpp
// 客户端 SSL
grpc::SslCredentialsOptions ssl_opts;
ssl_opts.pem_root_certs = root_cert;
ssl_opts.pem_private_key = client_key;
ssl_opts.pem_cert_chain = client_cert;

auto creds = grpc::SslCredentials(ssl_opts);
auto channel = grpc::CreateChannel(endpoint, creds);

// 服务端 SSL
grpc::SslServerCredentialsOptions::PemKeyCertPair key_cert;
key_cert.private_key = server_key;
key_cert.cert_chain = server_cert;

grpc::SslServerCredentialsOptions ssl_opts;
ssl_opts.pem_root_certs = root_cert;
ssl_opts.pem_key_cert_pairs.push_back(key_cert);

auto creds = grpc::SslServerCredentials(ssl_opts);
```

## 8. 与 OH 其他通信方式对比

| 特性 | gRPC | OH IPC | SoftBus |
|------|------|--------|---------|
| **API 复杂度** | 中 | 低 | 低 |
| **功能丰富度** | 高 | 中 | 中 |
| **跨网络** | ✅ 支持 | ❌ 不支持 | ✅ 支持（近场）|
| **流式通信** | ✅ 原生支持 | ⚠️ 需自行实现 | ✅ 支持 |
| **二进制体积** | 较大 | 小 | 小 |
| **启动延迟** | 略高 | 低 | 低 |
| **适用场景** | 云服务/复杂 RPC | 本地服务 | 分布式硬件 |

**建议**:
- 需要与云服务通信 → 使用 gRPC
- 本地服务间简单通信 → 使用 OH IPC
- 分布式设备硬件抽象 → 使用 SoftBus

---

**相关文档**:
- [01_Overview.md](./01_Overview.md) - gRPC 简介
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景
