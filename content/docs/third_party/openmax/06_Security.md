# 06_Security.md - 安全风险分析

## 概述

OpenMAX IL 是**纯头文件接口库**，本身不包含可执行代码，因此安全风险较低。但需要注意使用方式带来的潜在安全问题。

## 库特性与安全风险

### 纯头文件特性

| 特性 | 安全影响 |
|------|----------|
| **无编译代码** | 无缓冲区溢出、代码注入等运行时漏洞 |
| **宏定义为主** | 无复杂逻辑，无逻辑漏洞 |
| **结构体定义** | 需使用者确保内存安全 |

### 风险等级评估

| 风险类型 | 等级 | 说明 |
|----------|------|------|
| **内存安全漏洞** | 极低 | 无运行时代码 |
| **逻辑漏洞** | 极低 | 无复杂逻辑 |
| **API 误用** | 中 | 依赖使用者正确使用 |
| **供应链攻击** | 低 | 需确保上游来源可信 |

## CVE 与安全公告

### OpenMAX IL 历史 CVE

通过查询 NVD (National Vulnerability Database) 和公开安全公告：

| CVE ID | 影响版本 | 描述 | OH 状态 |
|--------|----------|------|---------|
| 无公开 CVE | - | OpenMAX IL 头文件库历史上无直接安全漏洞报告 | - |

**说明**：
- OpenMAX IL 标准头文件库历史上无直接安全漏洞
- 实际漏洞多出现在**具体实现**中（如芯片厂商的 OMX IL 实现）

### 相关组件 CVE

| CVE ID | 组件 | 描述 | OH 关联 |
|--------|------|------|---------|
| CVE-2017-13279 | Android OMX | 信息泄露漏洞 | OH 实现需独立评估 |
| CVE-2017-13280 | Android OMX | 权限绕过漏洞 | OH 实现需独立评估 |
| CVE-2021-0578 | Android OMX | 远程代码执行 | OH 实现需独立评估 |

**注意**：上述 CVE 是 Android 系统中具体 OMX 实现的漏洞，**非 OpenMAX IL 标准库漏洞**。

## 潜在安全风险

### 风险 1：Buffer 管理不当

```c
// 危险示例：未检查 buffer 大小
OMX_BUFFERHEADERTYPE* buffer;
OMX_AllocateBuffer(handle, &buffer, portIndex, NULL, user_provided_size);
// 如果 user_provided_size 过大或过小，可能导致问题
```

**缓解措施**：
- 驱动层验证 Buffer 大小
- 使用 `OMX_GetParameter` 查询端口要求的 Buffer 大小

### 风险 2：类型混淆

```c
// 潜在问题：强制类型转换
struct CustomBuffer* custom = (struct CustomBuffer*)buffer->pBuffer;
// 如果 buffer 实际类型不匹配，可能导致内存访问错误
```

**缓解措施**：
- 严格遵循 OMX 标准类型定义
- 使用 OH 扩展前验证组件支持情况

### 风险 3：扩展 API 兼容性

```c
// 风险：在旧版本组件上使用新扩展
OMX_SetParameter(handle, OMX_IndexParamRoi, &roiParams);
// 如果组件不支持 ROI，可能返回错误或忽略
```

**缓解措施**：
- 使用 `OMX_GetExtensionIndex` 查询扩展支持
- 检查返回值，提供 fallback 逻辑

### 风险 4：权限控制

```c
// 问题：低权限进程直接访问硬件编解码器
// 可能导致资源耗尽或信息泄露
```

**OH 缓解措施**：
- Codec HDI 层进行权限检查
- 通过 IPC 限制直接硬件访问

## 安全使用建议

### 对于 OMX IL 实现者（芯片厂商）

1. **输入验证**：
   ```c
   // 检查索引范围
   if (nIndex < 0 || nIndex > MAX_INDEX) {
       return OMX_ErrorBadParameter;
   }
   
   // 检查 buffer 大小
   if (buffer->nAllocLen < buffer->nFilledLen) {
       return OMX_ErrorBadParameter;
   }
   ```

2. **资源限制**：
   - 限制最大 Buffer 数量
   - 限制最大分辨率/码率
   - 防止资源耗尽攻击

3. **内存隔离**：
   - 用户态 Buffer 与内核态分离
   - 使用 ION/DMA-BUF 安全内存分配

### 对于 OMX IL 使用者（AV Codec 等）

1. **参数校验**：
   ```c
   // 检查参数范围
   if (targetBitrate < MIN_BITRATE || targetBitrate > MAX_BITRATE) {
       return ERR_INVALID_PARAM;
   }
   ```

2. **错误处理**：
   ```c
   OMX_ERRORTYPE err = OMX_SetParameter(...);
   if (err != OMX_ErrorNone) {
       // 记录日志，优雅降级
       return HandleOmxError(err);
   }
   ```

3. **超时保护**：
   ```c
   // 防止 OMX 调用阻塞
   // 使用异步回调机制
   // 设置合理的超时时间
   ```

## OH 安全机制

### 1. 分层架构隔离

```
应用层 (沙箱)
    ↓ IPC
AV Codec Service (特权进程)
    ↓ HDI IPC
Codec HDI Service (系统进程)
    ↓ 内核态
Vendor OMX IL (驱动层)
    ↓
硬件 VPU
```

**安全收益**：
- 应用无法直接访问硬件
- 每层都有权限检查
- 攻击面逐层缩小

### 2. 权限控制

```xml
<!-- config.json 中声明权限 -->
"reqPermissions": [
    {
        "name": "ohos.permission.MICROPHONE",
        "reason": "录音"
    },
    {
        "name": "ohos.permission.CAMERA",
        "reason": "录像"
    }
]
```

### 3. 进程隔离

| 进程 | 权限 | 说明 |
|------|------|------|
| **应用进程** | 受限 | 无法直接访问 OMX |
| **AV Codec Service** | 系统 | 代理编解码请求 |
| **Codec HDI** | 系统 | 硬件抽象层 |
| **Vendor OMX** | 内核 | 直接硬件访问 |

## 安全升级策略

### 上游版本升级

| 场景 | 建议 |
|------|------|
| **Khronos 发布新版本** | 评估新功能和安全改进，制定迁移计划 |
| **安全公告** | 关注 Khronos 安全公告，及时评估影响 |
| **芯片厂商更新** | 协调芯片厂商同步更新 OMX 实现 |

### codec_omx_ext.h 维护

| 场景 | 建议 |
|------|------|
| **添加新扩展** | 进行安全评审，检查参数范围 |
| **修改已有扩展** | 评估向后兼容性，渐进式更新 |
| **废弃扩展** | 标记 deprecated，保留一段时间后移除 |

## 安全测试建议

### Fuzz 测试

已存在的 Fuzz 测试：
- `hwvvcdecoderserver_fuzzer` - VVC 解码器 Fuzz
- `hwavcencoderserver_fuzzer` - AVC 编码器 Fuzz
- `hwhevcdecoderserver_fuzzer` - HEVC 解码器 Fuzz
- `hwhevcencoderserver_fuzzer` - HEVC 编码器 Fuzz

**建议**：
- 覆盖所有 OMX 命令
- 覆盖参数边界值
- 覆盖错误处理路径

### 渗透测试

| 测试项 | 方法 |
|--------|------|
| **非法参数** | 传入超出范围的枚举值、负值、极大值 |
| **竞争条件** | 多线程并发操作 OMX 组件 |
| **资源耗尽** | 创建大量组件、分配大量 Buffer |
| **状态机违规** | 在不恰当的状态执行命令 |

## 应急响应

### 发现安全漏洞时的流程

1. **评估影响**
   - 确定影响范围（是否影响 OH）
   - 确定严重等级

2. **临时缓解**
   - 配置关闭受影响功能
   - 发布安全公告

3. **修复发布**
   - 开发修复补丁
   - 安全测试验证
   - 发布安全更新

4. **事后分析**
   - 根因分析
   - 改进开发流程

## 总结

| 项目 | 结论 |
|------|------|
| **直接风险** | 极低 - 纯头文件库 |
| **间接风险** | 中 - 依赖实现者和使用者的正确使用 |
| **主要威胁** | OMX IL 实现中的内存安全问题 |
| **缓解措施** | 分层架构、权限控制、输入验证 |
| **维护建议** | 关注实现层安全，定期安全测试 |

## 参考资源

- [Khronos Group Security](https://www.khronos.org/security/)
- [NVD - National Vulnerability Database](https://nvd.nist.gov/)
- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/tree/master/zh-cn/device-dev/security)
- [Android MediaCodec Security](https://source.android.com/security/bulletin)
