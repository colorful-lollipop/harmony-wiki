# 06_Security.md - 安全风险分析

## 1. 安全评估概览

### 1.1 评估结论

astc-encoder 是**安全级别较高**的第三方库：

| 评估维度 | 评分 | 说明 |
|---------|------|------|
| 代码质量 | ⭐⭐⭐⭐⭐ | ARM 官方维护，代码规范严格 |
| 输入验证 | ⭐⭐⭐⭐ | 显式缓冲区大小，防止溢出 |
| 内存管理 | ⭐⭐⭐⭐ | 显式分配/释放，无隐式分配 |
| 依赖风险 | ⭐⭐⭐⭐⭐ | 零外部依赖 |
| 漏洞历史 | ⭐⭐⭐⭐ | 4.7.0 为稳定版本，无已知高危漏洞 |

### 1.2 风险等级

**综合风险等级：低**

理由：
1. 编解码器处理的是图像数据，非可执行代码
2. 所有缓冲区大小显式传递
3. 无网络功能，无权限提升风险
4. 代码经过 ARM 严格测试

---

## 2. 已知 CVE 分析

### 2.1 CVE 搜索结果

通过以下方式搜索 CVE：
- 检查上游 GitHub Issues
- 查阅 CHANGELOG/NEWS
- 检查库内安全相关文档

**结果**：未发现针对 astc-encoder 4.7.0 的公开 CVE。

### 2.2 历史安全问题

根据上游 CHANGELOG-4x.md 分析，4.x 系列修复的安全相关问题：

| 版本 | 问题类型 | 描述 | 严重性 |
|------|---------|------|--------|
| 4.7.0 | 规范合规 | 修复解压器舍入行为匹配 Khronos 规范 | 低 |
| 4.7.0 | 内存对齐 | 避免使用 alignas() 导致的 CPU 兼容性问题 | 低 |
| 4.6.0 | 代码质量 | 减少 reinterpret_cast 避免严格别名违规 | 低 |
| 4.5.0 | 代码质量 | 修复潜在的整数溢出问题 | 中 |

**说明**：上述问题均为代码质量或规范合规问题，无严重安全漏洞。

### 2.3 4.7.0 关键修复

#### 修复 1：sRGB LDR 解压舍入

**问题**：sRGB LDR 解压的端点扩展方法不符合规范，可能导致 LSB 位翻转。

**修复**：使用正确的端点扩展方法。

**影响**：图像质量，非安全问题。

#### 修复 2：decode_unorm8 舍入

**问题**：解压到 8 位输出时舍入规则不符合 decode_unorm8 扩展规范。

**修复**：添加 `ASTCENC_FLG_USE_DECODE_UNORM8` 标志支持。

**影响**：图像质量，非安全问题。

#### 修复 3：alignas 移除

**问题**：某些 CPU 上 alignas(16) 小于原生最小对齐要求。

**修复**：避免在参考 C 实现中使用 alignas()。

**影响**：兼容性，非安全问题。

---

## 3. 潜在安全风险分析

### 3.1 输入处理风险

#### 风险：畸形图像数据

**描述**：
- 恶意构造的图像数据可能导致解压失败或异常
- 极端尺寸或格式错误的图像

**缓解措施**：
```cpp
// astcenc 的防御机制
astcenc_error status = astcenc_compress_image(
    context,
    &image,
    &swizzle,
    output_buffer,
    output_buffer_size,  // 显式指定缓冲区大小
    &output_size,
    thread_index
);

// 所有 API 都显式传递缓冲区大小
// 库内部进行边界检查
```

**评估风险：低**
- 所有输入缓冲区大小显式传递
- 库内部进行边界检查
- 异常返回错误码，不会崩溃

### 3.2 内存管理风险

#### 风险：内存泄漏

**描述**：上下文未正确释放。

**缓解措施**：
```cpp
// 正确的使用模式
astcenc_context* context;
astcenc_context_alloc(&config, 1, &context);

// ... 使用上下文 ...

// 必须释放
astcenc_context_free(context);
```

**评估风险：低**
- 显式内存管理，无隐式分配
- C++ RAII 可轻松管理
- 文档明确说明生命周期

#### 风险：整数溢出

**描述**：极端大图像尺寸导致整数溢出。

**缓解措施**：
```cpp
// astcenc 内部检查
if (image.dim_x == 0 || image.dim_y == 0 || image.dim_z == 0) {
    return ASTCENC_ERR_BAD_PARAM;
}

// 缓冲区大小计算使用 size_t
size_t buffer_size = (size_t)width * height * channels;
```

**评估风险：低**
- 内部有参数验证
- 使用 size_t 进行大小计算
- 极端尺寸返回错误而非溢出

### 3.3 多线程风险

#### 风险：线程安全问题

**描述**：多线程使用时的竞态条件。

**缓解措施**：
```cpp
// 线程安全设计
// 1. 每个线程使用独立的上下文（推荐）
astcenc_context* context1, *context2;
astcenc_context_alloc(&config, 1, &context1);  // 线程 1
astcenc_context_alloc(&config, 1, &context2);  // 线程 2

// 2. 或使用线程索引（单上下文多线程）
astcenc_context_alloc(&config, 4, &context);  // 4 线程
// 每个线程使用不同 thread_index 调用 API
```

**评估风险：低**
- 明确的多线程使用模型
- 线程索引参数确保线程安全
- 上下文独立，无共享状态问题

### 3.4 OH 特有安全风险

#### 风险：条件编译宏的副作用

**描述**：
- `ASTC_CUSTOMIZED_ENABLE` 等宏可能引入未测试的代码路径
- 特定产品配置下的潜在问题

**分析**：
```cpp
// BUILD.gn 中的宏定义
if (defined(global_parts_info) &&
    (defined(global_parts_info.graphic_graphic_2d_ext) ||
     defined(global_parts_info.product_hmos_sdk_product_hmos_sdk))) {
  defines = [ "ASTC_CUSTOMIZED_ENABLE" ]
}
```

**评估风险：极低**
- 搜索源码未发现这些宏的实际使用
- 可能是预留或历史遗留配置
- TODO(需确认)：确认这些宏在源码中的使用情况

#### 风险：共享库注入

**描述**：共享库可能被恶意替换。

**缓解措施**：
- OH 系统分区（system）只读
- 库文件有签名验证
- 安装位置受保护

**评估风险：低**
- OH 安全机制保护
- 需要 root 权限才能篡改

---

## 4. 攻击面分析

### 4.1 攻击面清单

| 攻击面 | 描述 | 风险等级 |
|--------|------|---------|
| 图像输入 | 恶意构造的图像数据 | 低 |
| ASTC 数据输入 | 恶意构造的压缩数据 | 低 |
| 配置参数 | 极端/非法参数值 | 低 |
| 共享库 | 库文件篡改 | 极低 |
| 内存分配 | 大内存分配导致 OOM | 低 |

### 4.2 攻击场景分析

#### 场景 1：图库缩略图攻击

**攻击向量**：
1. 用户下载恶意构造的图片
2. 图库尝试生成 ASTC 缩略图
3. 触发编解码器漏洞

**防护**：
- 输入图像大小限制
- 编解码器内部边界检查
- 沙箱隔离

#### 场景 2：应用预置图攻击

**攻击向量**：
1. 恶意应用包含构造的 ASTC 资源
2. 系统尝试解压显示
3. 触发解压漏洞

**防护**：
- 应用签名验证
- 资源加载沙箱
- 异常处理

---

## 5. 安全测试建议

### 5.1 Fuzz 测试

**现状**：
- OH 已有 Fuzz 测试：`imagetextureencode_fuzzer`
- 上游提供 Fuzz 测试：`Source/Fuzzers/`

**建议**：
```cpp
// 扩展 Fuzz 测试覆盖
// 1. 畸形图像尺寸
// 2. 极端块大小组合
// 3. 无效质量预设
// 4. 并发操作
```

### 5.2 静态分析

**推荐工具**：
- Coverity Scan
- Clang Static Analyzer
- CodeQL

**关注项**：
- 缓冲区溢出
- 整数溢出
- 空指针解引用
- 内存泄漏

### 5.3 模糊测试参数

| 参数 | 范围 | 关注点 |
|------|------|--------|
| 图像尺寸 | 0 ~ 16384 | 极端尺寸处理 |
| 块大小 | 1x1 ~ 12x12 | 边界值 |
| 质量预设 | 0.0 ~ 100.0 | 非法值处理 |
| 数据类型 | U8/F16/F32 | 类型转换 |

---

## 6. 升级安全建议

### 6.1 版本跟踪

**当前版本**：4.7.0（2024-01）

**建议跟踪**：
- 订阅上游安全通告
- 关注 GitHub Security Advisory
- 定期检查 CHANGELOG

### 6.2 升级检查清单

| 检查项 | 说明 |
|--------|------|
| CVE 检查 | 新版本修复的 CVE |
| API 变更 | 是否影响现有代码 |
| 行为变更 | 压缩/解压结果是否一致 |
| 性能回归 | 速度是否有明显下降 |
| 兼容性 | 旧数据能否正常解压 |

### 6.3 应急响应

**如果发现严重漏洞**：

1. **立即评估**：
   - 漏洞是否影响 OH 使用场景
   - 攻击难度和影响范围

2. **临时缓解**：
   - 限制输入图像尺寸
   - 增加输入验证
   - 禁用特定功能

3. **修复升级**：
   - 同步上游安全修复
   - 验证修复有效性
   - 发布安全更新

---

## 7. 安全最佳实践

### 7.1 开发者建议

#### 输入验证

```cpp
// 在调用 astcenc 前验证输入
bool validate_image(const astcenc_image* image) {
    // 检查尺寸限制
    if (image->dim_x == 0 || image->dim_x > MAX_IMAGE_WIDTH) {
        return false;
    }
    if (image->dim_y == 0 || image->dim_y > MAX_IMAGE_HEIGHT) {
        return false;
    }
    
    // 检查数据类型
    if (image->data_type != ASTCENC_TYPE_U8 &&
d image->data_type != ASTCENC_TYPE_F16 &&
        image->data_type != ASTCENC_TYPE_F32) {
        return false;
    }
    
    return true;
}
```

#### 错误处理

```cpp
// 始终检查返回值
astcenc_error status = astcenc_compress_image(...);
if (status != ASTCENC_SUCCESS) {
    // 记录错误日志
    LOGE("ASTC compress failed: %s", astcenc_get_error_string(status));
    // 清理资源
    // 返回错误
    return ERROR_COMPRESS_FAILED;
}
```

#### 资源限制

```cpp
// 限制并发线程数
const unsigned int MAX_THREADS = 8;
unsigned int thread_count = std::min(hardware_threads, MAX_THREADS);

// 限制图像尺寸
const unsigned int MAX_DIMENSION = 8192;
if (image_width > MAX_DIMENSION || image_height > MAX_DIMENSION) {
    return ERROR_IMAGE_TOO_LARGE;
}
```

### 7.2 系统集成建议

#### 沙箱隔离

- 在独立进程中运行图像处理
- 限制文件系统访问
- 限制网络访问

#### 监控告警

- 监控异常高的内存使用
- 监控长时间运行的压缩任务
- 记录失败的编解码尝试

---

## 8. 总结

### 8.1 安全状态

| 项目 | 状态 |
|------|------|
| 已知 CVE | 无 |
| 严重漏洞 | 无 |
| 代码质量 | 高 |
| 维护活跃度 | 高 |

### 8.2 风险评估

| 风险类型 | 等级 | 缓解措施 |
|---------|------|---------|
| 缓冲区溢出 | 低 | 显式缓冲区大小 |
| 整数溢出 | 低 | 内部参数检查 |
| 内存泄漏 | 低 | 显式内存管理 |
| DoS | 低 | 资源限制 |
| 代码注入 | 极低 | 无代码执行路径 |

### 8.3 维护建议

1. **定期升级**：跟踪上游安全修复
2. **Fuzz 测试**：持续运行模糊测试
3. **静态分析**：定期执行代码扫描
4. **监控告警**：部署运行时监控

---

## 附录：安全参考资源

### 上游资源
- [GitHub Security Advisories](https://github.com/ARM-software/astc-encoder/security)
- [CHANGELOG-4x.md](../Docs/ChangeLog-4x.md)
- [Issue Tracker](https://github.com/ARM-software/astc-encoder/issues)

### OH 安全资源
- OpenHarmony 安全响应中心
- 图像框架安全审查记录

### 工具资源
- [OSS-Fuzz](https://github.com/google/oss-fuzz) - 开源模糊测试
- [Coverity Scan](https://scan.coverity.com/) - 静态分析
- [Clang Static Analyzer](https://clang-analyzer.llvm.org/) - 静态分析
