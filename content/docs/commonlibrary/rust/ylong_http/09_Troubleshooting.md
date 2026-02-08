# 常见构建/运行/调试问题

> 本文档总结 ylong_http 项目的常见问题及其解决方案

---

## 目的

本文档的目的是帮助开发者快速定位和解决：
- 构建和编译问题
- 运行时错误和异常
- 性能和调试问题

## 适用范围

- GN 构建问题
- Cargo 构建问题
- 运行时错误
- TLS/SSL 连接问题

---

## 关键结论

### 1. 构建问题

#### 1.1 GN 构建失败

**问题**: GN 构建时找不到依赖

**症状**：
```
ERROR at //commonlibrary/rust/ylong_http/ylong_http/BUILD.gn:17:11: Assignment had no effect.
ERROR: Unresolved dependency //commonlibrary/rust/ylong_http_client/...
```

**原因**：
- ylong_runtime 未编译或安装
- OpenSSL 库未正确配置

**解决方案**：
1. 检查 OpenHarmony 环境配置：
   ```bash
   ohos build sys
   ```

2. 确认 OpenSSL 组件已启用：
   ```bash
   # 查看可用的 GN args
   gn args --list
   ```

3. 检查依赖路径是否正确：
   ```bash
   gn path out/
   ```

**证据**: GN 构建依赖配置（见 [GN 目标梳理](06_GN_Targets.md)）

#### 1.2 Rust 编译错误

**问题**: 编译器报错

**常见错误**：
```
error[E0433]: failed to resolve: could not find `ylong_runtime` in `ylong_http_client`
error: linking with `cc` failed: code signing failed
```

**解决方案**：
1. 清理构建产物并重新构建：
   ```bash
   rm -rf out/
   gn build
   ```

2. 检查 Rust 工具链版本：
   ```bash
   rustc --version
   cargo --version
   ```

3. 使用详细模式查看完整错误信息：
   ```bash
   gn build -v
   ```

#### 1.3 Feature Flags 冲突

**问题**: 某些 feature 组合无法同时启用

**示例**：
```toml
[dependencies]
ylong_http = { path = "../ylong_http", features = ["tokio_base", "ylong_base"] }
```

**原因**：`tokio_base` 和 `ylong_base` 都提供异步运行时，冲突

**解决方案**：
1. 只选择一个运行时：
   - 使用 `ylong_base`（OpenHarmony 推荐）
   - 或使用 `tokio_base`（非 OpenHarmony 环境）

**证据**: `ylong_http/Cargo.toml:18-28` - Feature flags 定义

### 2. 运行时问题

#### 2.1 TLS/SSL 连接失败

**问题**: HTTPS 连接时证书验证失败

**症状**：
```
Error: TlsError(Ssl(SslError(SslErrorCode { code: 33744555, library: "SSL routines", reason: "certificate verify failed" })))
```

**可能原因**：
1. 服务器证书已过期
2. 证书不受信任（不在 CA 列表中）
3. 主机名不匹配（SNI 问题）
4. 证书链不完整

**解决方案**：
1. **检查证书有效性**：
   - 使用浏览器或 OpenSSL 命令验证证书：
     ```bash
     openssl s_client -connect example.com:443 -showcerts
     ```

2. **配置 TLS 验证器**：
   ```rust
   let config = TlsConfigBuilder::new()
       .verifier(Box::new(MyVerifier))
       .build();
   ```

3. **启用详细错误日志**：
   ```rust
   env_logger::Builder::from_env(env_logger::Env::default())
       .filter_module("ylong_http_client", log::LevelFilter::Debug)
       .init();
   ```

**证据**: `ylong_http_client/src/util/config/tls/verifier/mod.rs` - CertVerifier trait

#### 2.2 连接超时

**问题**: 请求长时间未响应

**症状**：
```
Error: Timeout(Timeout(Timeout { connection_timeout: Some(30s), read_timeout: Some(30s) }))
```

**可能原因**：
1. 网络延迟高
2. 服务器响应慢
3. 防火墙阻止

**解决方案**：
1. **增加超时时间**：
   ```rust
   let client = Client::builder()
       .timeout(Timeout::from_secs(60))
       .build(connector);
   ```

2. **分类型超时**：
   - 连接超时：与服务器建立 TCP/TLS 连接
   - 读取超时：读取响应头
   - 写入超时：发送请求数据

3. **使用重试机制**：
   ```rust
   let client = Client::builder()
       .retry(Retry::times(3))
       .build(connector);
   ```

**证据**: `ylong_http_client/src/util/config/settings.rs:182-207` - Timeout 配置

#### 2.3 DNS 解析失败

**问题**: 无法解析域名

**症状**：
```
Error: DnsError(io::Error { kind: WouldBlock, message: "resolving timed out" }))
```

**可能原因**：
1. DNS 服务器无响应
2. 网络不可达
3. DNS 缓存过期

**解决方案**：
1. **使用 IP 地址**：
   ```rust
   let request = Request::builder()
       .url("https://192.168.1.1/api/data")
       .body(...);
   ```

2. **配置自定义 DNS 解析器**：
   ```rust
   use ylong_http_client::async_impl::dns::Resolver;

   let client = Client::with_dns_resolver(MyResolver);
   ```

3. **检查网络连接**：
   ```bash
   ping example.com
   nslookup example.com
   ```

**证据**: `ylong_http_client/src/async_impl/dns/mod.rs` - Resolver trait

#### 2.4 连接池耗尽

**问题**: 所有连接被占用，无法创建新连接

**症状**：
```
Error: IoError(io::Error { kind: WouldBlock, message: "max idle connections reached" }))
```

**可能原因**：
1. 连接泄漏（使用后未正确释放）
2. 连接池大小配置过小
3. 长时间运行导致连接积累

**解决方案**：
1. **增加连接池大小**：
   ```rust
   let client = Client::builder()
       .max_idle_connections(1000)
       .build(connector);
   ```

2. **启用连接池监控**：
   - 使用 `Interceptor` 监控连接创建和释放
   - 记录连接使用统计信息

3. **检查连接释放逻辑**：
   - 确保所有 Response 被完整读取后调用 `release`
   - 不要手动持有 Connection 引用

**证据**: `ylong_http_client/src/async_impl/pool.rs` - ConnPool 实现

### 3. 性能问题

#### 3.1 连接复用率低

**症状**：每次请求都创建新连接，性能差

**原因**：
1. 连接池配置不当
2. 服务器强制关闭连接（Connection: close）
3. HTTP/1.1 使用 Connection: close 而非 keep-alive

**解决方案**：
1. **使用 HTTP/1.1 Keep-Alive**：
   ```rust
   let request = Request::builder()
       .header("Connection", "keep-alive")
       .body(...);
   ```

2. **增大连接池大小**：
   ```rust
   let client = Client::builder()
       .max_idle_connections(100)
       .build(connector);
   ```

3. **监控连接复用统计**：
   ```rust
   use ylong_http_client::util::information::ConnInfo;

   // ConnInfo 包含连接复用信息
   ```

**证据**: `ylong_http_client/src/util/information.rs` - ConnInfo 定义

#### 3.2 内存占用高

**症状**：程序占用过多内存

**可能原因**：
1. 响应 Body 一次性全部加载到内存
2. 连接池过大
3. Body 未正确释放

**解决方案**：
1. **流式读取 Body**：
   ```rust
   // 使用 streaming Body 而非一次性加载
   let response = client.request(request).await?;
   let mut body = response.body();
   let mut buf = [0u8; 4096];
   while let Ok(n) = body.data(&mut buf).await {
       if n == 0 { break; }
       // 处理数据块
   }
   ```

2. **使用 EmptyBody**：
   ```rust
   // 不需要响应体时使用 EmptyBody
   use ylong_http::body::EmptyBody;
   ```

3. **调整连接池大小**：
   - 根据可用内存调整最大连接数
   - 定期清理空闲连接

**证据**: `ylong_http/src/body/mod.rs` - EmptyBody 定义

### 4. 调试技巧

#### 4.1 启用日志

**使用环境变量**：
```bash
# 启用详细日志
export RUST_LOG=ylong_http_client=debug

# 启用 OpenSSL 日志
export OPENSSL_DEBUG=1024
```

**证据**: ylong_http_client 错误处理支持日志

#### 4.2 使用测试工具

**单元测试**：
```bash
# 运行所有单元测试
gn test //commonlibrary/rust/ylong_http/ylong_http_client:unittest

# 运行特定测试
gn test //commonlibrary/rust/ylong_http/ylong_http_client:unittest --test-filter="test_*"
```

**示例代码**：
- `ylong_http/examples/` - 各种使用示例
- `ylong_http_client/examples/` - 客户端使用示例

**证据**: 目录结构和 README 示例说明

---

## 相关跳转

- **[GN 目标梳理](06_GN_Targets.md)** - 构建配置和依赖
- **[对外 API](04_External_API.md)** - API 使用方法和配置

---

**文档版本**: 1.0
**作者**: Wiki 生成器（基于代码证据）
**最后更新**: 2026-02-06
