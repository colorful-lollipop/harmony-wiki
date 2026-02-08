# Audio Framework - 安全风险评估

> 本文档对 OpenHarmony 音频框架进行深度安全风险评估，包含具体漏洞点分析和修复建议。

---

## 评估方法

### 风险评级标准

| 评级 | CVSS 范围 | 定义 | 修复优先级 |
|------|-----------|------|-----------|
| **严重 (Critical)** | 9.0-10.0 | 可导致远程代码执行、系统完全 compromise | 立即修复 |
| **高危 (High)** | 7.0-8.9 | 可导致权限提升、敏感数据泄露 | 1周内修复 |
| **中危 (Medium)** | 4.0-6.9 | 可导致功能异常、有限信息泄露 | 1月内修复 |
| **低危 (Low)** | 0.1-3.9 | 轻微安全问题、难以利用 | 下个版本修复 |

### 评估维度

1. **输入验证缺陷** - 类型/长度/范围/null/编码/路径遍历
2. **内存安全问题** - 缓冲区/Use-After-Free/双重释放
3. **权限与鉴权** - 权限校验、身份验证、访问控制
4. **并发安全** - 竞态条件、TOCTOU、线程安全
5. **逻辑漏洞** - 错误处理、资源耗尽、信息泄露

---

## 1. 输入验证缺陷

### 1.1 N-API 参数验证分析

#### R1: 参数类型验证缺失

**状态**: TODO(待分析)

**位置**: `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp`

**证据**: 需验证每个 N-API 方法的参数类型检查逻辑

**触发路径**:
```
JS App → N-API → Native 层
```

**潜在影响**: 类型混淆可能导致内存损坏

**修复建议**: 
- 使用 NAPI 类型检查宏严格验证所有参数类型
- 参考示例:
```cpp
napi_valuetype valueType = napi_undefined;
napi_typeof(env, args[0], &valueType);
if (valueType != napi_number) {
    return NapiAudioError::ThrowError(env, NAPI_ERR_INPUT_INVALID);
}
```

---

#### R2: 音量参数范围验证

**状态**: 待验证

**位置**: `frameworks/js/napi/audiomanager/napi_audio_volume_manager.cpp`

**证据**: 
```cpp
// frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:49-50
static constexpr double MIN_LOUDNESS_GAIN_IN_DOUBLE = -90.0;
static constexpr double MAX_LOUDNESS_GAIN_IN_DOUBLE = 24.0;
```

**分析**: 音量参数有范围定义，需验证是否所有入口都有检查

**修复建议**: 确保所有音量设置接口都进行范围校验

---

### 1.2 路径遍历风险

#### R3: cacheDir 路径验证

**状态**: TODO(需确认)

**位置**: 
- `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:251`
- `frameworks/js/napi/audiocapturer/napi_audio_capturer.cpp:152`

**证据**:
```cpp
rendererNapi->audioRenderer_ = AudioRenderer::Create(cacheDir, rendererOptions);
napiCapturer->audioCapturer_ = AudioCapturer::Create(capturerOptions, cacheDir);
```

**风险**: cacheDir 从 JS 层传递，如果未验证可能包含 `../` 导致路径遍历

**触发路径**:
```
JS createAudioRenderer({cacheDir: "../../../system/data"})
  → NapiAudioRenderer::Create
    → AudioRenderer::Create
      → 可能访问任意目录
```

**影响评估**: 中危 - 可能访问敏感文件

**修复建议**:
```cpp
// 验证路径合法性
if (!IsPathWithinSandbox(cacheDir, appSandbox)) {
    return NapiAudioError::ThrowError(env, NAPI_ERR_INVALID_PATH);
}
```

---

### 1.3 XML 解析安全风险

#### R4: XXE (XML External Entity) 攻击

**状态**: TODO(需确认)

**位置**: 
- `services/audio_policy/server/infra/config/parser/audio_xml_parser.cpp`

**分析**: 音频框架解析多个 XML 配置文件，如果解析器未禁用外部实体，可能存在 XXE 风险

**配置文件清单**:
- audio_effect_config.xml
- audio_volume_config.xml
- audio_strategy_router.xml
- audio_interrupt_policy_config.xml
- audio_device_privacy.xml

**影响评估**: 中危 - 可能导致文件读取、SSRF

**修复建议**: 确保 XML 解析器配置禁用外部实体
```cpp
// libxml2 示例
xmlParserCtxtPtr ctxt = xmlNewParserCtxt();
ctxt->options |= XML_PARSE_NOENT;  // 禁用实体扩展
ctxt->options |= XML_PARSE_DTDLOAD; // 谨慎加载 DTD
// 或者使用 xmlReadFile 时设置 XML_PARSE_NOENT
```

---

## 2. 内存安全问题

### 2.1 缓冲区管理

#### R5: 音频数据缓冲区溢出

**状态**: TODO(待分析)

**位置**: 
- `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:write`
- 共享内存缓冲区操作

**证据**: 需要进一步分析 write 方法的实现

**潜在风险**:
- JS 传递的 buffer 长度与实际长度不符
- Native 层写入时未验证长度

**影响评估**: 高危 - 可能导致缓冲区溢出

**修复建议**:
- 严格验证 buffer 长度参数
- 使用安全拷贝函数
```cpp
size_t actualLen = 0;
napi_get_arraybuffer_info(env, buffer, &data, &actualLen);
if (userProvidedLen > actualLen) {
    return NapiAudioError::ThrowError(env, NAPI_ERR_BUFFER_OVERFLOW);
}
```

---

### 2.2 共享内存安全

#### R6: 共享内存竞态条件

**状态**: TODO(待分析)

**位置**: `services/audio_service/common/include/va_shared_buffer.h`

**分析**: OHAudioBuffer 用于跨进程音频数据传输

**潜在风险**:
- 生产者-消费者同步问题
- 缓冲区溢出
- Use-After-Free

**影响评估**: 高危 - 可能导致内存损坏或信息泄露

---

## 3. 权限与鉴权

### 3.1 麦克风权限

#### R7: 麦克风访问权限检查

**状态**: 部分验证

**位置**: 
- AudioCapturer 创建和启动流程
- IPC 接口权限检查

**分析**: 音频采集需要 `ohos.permission.MICROPHONE` 权限

**触发路径**:
```
JS App → NapiAudioCapturer::Create → AudioCapturer::Create
  → AudioPolicyServer::CreateCapturerClient
    → 权限检查点
```

**验证项**:
- [x] N-API 层是否有权限检查？
- [x] IPC 层是否有权限检查？
- [ ] 权限检查是否可被绕过？

**影响评估**: 严重 - 可能导致未经授权的录音

**修复建议**: 
- 在 N-API 层进行初步权限检查
- 在 IPC 服务端进行最终权限校验
- 使用 AccessTokenKit 进行权限验证

---

### 3.2 IPC 接口权限

#### R8: IPC 方法权限控制

**状态**: TODO(需确认)

**位置**: 
- `services/audio_service/server/src/audio_server.cpp`
- `services/audio_policy/server/service/service_main/src/audio_policy_server.cpp`

**分析**: 376+ 个 IPC 接口方法需要适当的权限控制

**高风险 IPC 方法**:
| 方法 | 当前权限检查 | 风险 |
|------|-------------|------|
| SET_MICROPHONE_MUTE | TODO | 高 |
| CREATE_CAPTURER_CLIENT | TODO | 高 |
| FORCE_STOP_AUDIO_STREAM | TODO | 中 |
| SET_SYSTEM_VOLUMELEVEL | TODO | 中 |

**影响评估**: 高危 - 可能导致权限提升

**修复建议**: 每个 IPC 方法都应有明确的权限声明和检查

---

## 4. 并发安全

### 4.1 多线程安全

#### R9: N-API 对象生命周期

**状态**: 待验证

**位置**: `frameworks/js/napi/audiorenderer/napi_audio_renderer.cpp:45-47`

**证据**:
```cpp
static __thread napi_ref g_rendererConstructor = nullptr;
mutex NapiAudioRenderer::createMutex_;
int32_t NapiAudioRenderer::isConstructSuccess_ = SUCCESS;
```

**分析**: 使用了 `__thread` 存储线程局部变量，`createMutex_` 保护创建过程

**潜在风险**:
- 析构时的竞态条件
- 回调函数的多线程安全

---

### 4.2 音频流状态机

#### R10: 状态机竞态条件

**状态**: TODO(待分析)

**分析**: AudioRenderer/AudioCapturer 有复杂的状态机 (IDLE, RUNNING, PAUSED, STOPPED)

**潜在风险**:
- 状态转换竞态条件
- 重复 start/stop 导致的问题

---

## 5. 逻辑漏洞

### 5.1 资源耗尽

#### R11: 资源创建限制缺失

**状态**: TODO(需确认)

**分析**: 是否可以创建无限数量的 AudioRenderer/AudioCapturer 实例？

**攻击示例**:
```javascript
for (let i = 0; i < 10000; i++) {
    audio.createAudioRenderer({});
}
```

**影响评估**: 中危 - 可能导致 DoS

**修复建议**: 实施资源配额限制

---

### 5.2 信息泄露

#### R12: 日志敏感信息

**状态**: TODO(需确认)

**位置**: 所有 AUDIO_INFO_LOG, AUDIO_WARNING_LOG 调用

**分析**: 检查日志是否输出敏感信息（如文件路径、内存地址、音频数据）

**影响评估**: 低危 - 信息泄露

---

## 6. 风险汇总表

| ID | 风险描述 | 评级 | 状态 | 位置 |
|----|---------|------|------|------|
| R1 | N-API 参数类型验证缺失 | 中 | TODO | napi_*.cpp |
| R2 | 音量参数范围验证 | 低 | 待验证 | volume_manager |
| R3 | cacheDir 路径遍历 | 中 | TODO | renderer/capturer |
| R4 | XXE 攻击 | 中 | TODO | xml_parser.cpp |
| R5 | 缓冲区溢出 | 高 | TODO | write 方法 |
| R6 | 共享内存竞态条件 | 高 | TODO | va_shared_buffer |
| R7 | 麦克风权限绕过 | 严重 | 部分验证 | capturer 流程 |
| R8 | IPC 权限控制缺失 | 高 | TODO | ipc server |
| R9 | N-API 对象生命周期 | 中 | 待验证 | napi_*.cpp |
| R10 | 状态机竞态条件 | 中 | TODO | audio_stream |
| R11 | 资源耗尽 DoS | 中 | TODO | create 方法 |
| R12 | 日志信息泄露 | 低 | TODO | 所有日志点 |

---

## 7. 修复建议汇总

### 7.1 立即修复 (严重/高危)

1. **R7 - 麦克风权限检查**
   - 确保所有音频采集入口都有严格的权限验证
   - 使用 AccessTokenKit 进行校验

2. **R5 - 缓冲区溢出防护**
   - 对所有 buffer 操作进行长度验证
   - 使用安全拷贝函数

3. **R8 - IPC 权限控制**
   - 审查所有 376+ 个 IPC 方法的权限声明
   - 添加缺失的权限检查

### 7.2 短期修复 (中危)

4. **R3 - 路径遍历防护**
   - 验证所有文件路径参数
   - 限制在应用沙盒内

5. **R4 - XXE 防护**
   - 禁用 XML 外部实体解析
   - 使用安全的 XML 解析配置

6. **R6 - 共享内存安全**
   - 添加边界检查
   - 使用同步原语保护

### 7.3 长期改进 (低危)

7. **R11 - 资源配额**
   - 实施实例数量限制
   - 添加资源监控

8. **R12 - 日志审计**
   - 审查所有日志输出
   - 移除敏感信息

---

## 8. 安全测试建议

### 8.1 静态分析
- 使用 CodeQL 进行 N-API 参数分析
- 使用 Coverity 进行内存安全分析
- 使用 Bandit 进行安全配置检查

### 8.2 动态测试
- Fuzzing N-API 接口
- IPC 方法 fuzzing
- 配置文件畸形数据测试

### 8.3 渗透测试
- 权限绕过测试
- 路径遍历测试
- XXE 测试

---

## 9. 参考资料

- [OpenHarmony 安全开发指南](https://gitee.com/openharmony/docs)
- [N-API 安全最佳实践](https://nodejs.org/api/n-api.html)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)

---

*最后更新: 2026-02-07*
