# 安全风险评审

## 目的

本文档介绍 media_lite 项目的安全风险分析，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

- 适用于安全工程师
- 适用于架构师
- 适用于代码审计人员
- 基于 2026-02-06 的代码分析

---

## 攻击面分析

### 1. N-API（JSI）接口攻击面

**入口点**：
- `audio_module.cpp` - JSI 模块，暴露播放器功能到 JS 层
- 主要 API：`play()`, `pause()`, `stop()`, `src`, `currentTime`, `volume`, `muted`

**攻击向量**：
- 恶意 URI 注入（通过 `src` 属性）
- 参数篡改（通过 setter 方法）
- 内存耗尽（频繁创建事件监听器）
- 竞态条件（多线程访问共享状态）

**当前防护措施**：
- ✅ 参数数量检查（argsSize）
- ✅ 对象类型检查（JSI::ValueIsObject）
- ✅ 函数类型检查（JSI::ValueIsFunction）
- ✅ 值范围检查（volume: 0-1, currentTime >= 0）

**缺失防护**：
- ❌ URI 路径遍历防护（TODO）
- ❌ 恶意文件格式检测（TODO）
- ❌ 资源耗尽防护（TODO）

---

### 2. IPC 服务攻击面

**入口点**：
- `player_server.cpp` - Player 服务端，通过 SAMGR 暴露
- `recorder_service.cpp` - Recorder 服务端，通过 SAMGR 暴露

**攻击向量**：
- IPC 参数篡改
- 恶意服务注入
- 拒绝服务（DoS）
- 权限绕过

**当前防护措施**：
- ✅ 使用 SAMGR 服务管理机制
- ✅ 使用 IpcIo 进行参数序列化
- ✅ 进程隔离（独立服务进程）

**缺失防护**：
- ❌ IPC 调用频率限制（TODO）
- ❌ 恶意载荷大小限制（TODO）

---

### 3. 文件操作攻击面

**入口点**：
- `player_impl.cpp` - 播放器实现，打开和读取媒体文件
- `recorder_impl.cpp` - 录音器实现，写入媒体文件

**攻击向量**：
- 路径遍历攻击（通过 `SetOutputPath/SetSource`）
- 符号链接攻击
- 任意文件写入（通过 `SetOutputPath`）
- 文件描述符泄露（通过 `SetOutputFile`）

**当前防护措施**：
- ✅ 权限检查（WRITE_MEDIA）
- ✅ 权限检查（READ_MEDIA）

**缺失防护**：
- ❌ 路径规范化（TODO）
- ❌ 文件扩展白名单（TODO）
- ❌ 符号链接攻击防护（TODO）

---

### 4. 权限系统攻击面

**入口点**：
- `player.cpp` - Player 权限检查入口
- `recorder.cpp` - Recorder 权限检查入口

**权限要求**：
- `ohos.permission.MODIFY_AUDIO_SETTINGS` - 修改音频设置
- `ohos.permission.READ_MEDIA` - 读取媒体文件
- `ohos.permission.MICROPHONE` - 麦克风访问
- `ohos.permission.WRITE_MEDIA` - 写入媒体文件

**攻击向量**：
- 权限提升
- 权限绕过
- 恶意应用伪装

**当前防护措施**：
- ✅ 集中式权限管理（permission_lite）
- ✅ 绑定检查（bundle 签名验证由底层处理）

**缺失防护**：
- ⚠️ access token 校验由 permission_lite 处理，本项目未直接实现（外部依赖）
- ⚠️ UID 校验由 permission_lite 处理，本项目未直接实现（外部依赖）
- ⚠️ bundle name 校验由 permission_lite 处理，本项目未直接实现（外部依赖）

---

## 信任边界

### 边界 1：JS 应用 → AudioModule

**边界**：
- JS 应用在独立的进程
- AudioModule 在应用进程空间
- 通过 JSI 接口通信

**信任假设**：
- JS 应用已通过权限检查
- JS 应用代码可信（但在本项目不验证）

**风险**：
- 恶意 JS 代码可绕过检查
- 任意 API 调用

### 边界 2：AudioModule → Player 框架

**边界**：
- Player 框架在应用进程空间
- Player 可能在服务进程空间（Passthrough 模式）

**信任假设**：
- Player 框架实现可信
- 媒体文件来自可信源

**风险**：
- 恶意 URI 可触发未知行为
- 恶意媒体文件可导致崩溃

### 边界 3：Player/Recorder 框架 → IPC

**边界**：
- Framework 层通过 IPC（SAMGR）调用服务层
- 服务层在独立进程（media_server）

**信任假设**：
- 调用者有权限（通过 permission_lite 检查）
- 调用者身份可通过 GetCallingPid() 获取

**风险**：
- 权限提升漏洞
- 服务假冒（如果身份验证薄弱）

### 边界 4：IPC → Services

**边界**：
- IPC 层通过 SAMGR 传输数据
- 服务层接收并处理请求

**信任假设**：
- IpcIo 序列化安全
- 调用者身份已验证

**风险**：
- 参数篡改
- 重放攻击

---

## 可被利用点

### 1. 路径遍历漏洞（高风险）

**证据**：
- 文件：`frameworks/player_lite/binder/player.cpp:46-48`, `frameworks/recorder_lite/recorder.cpp:447`
- API：`Player::SetSource()`, `Recorder::SetOutputPath()`

**触发路径**：
```javascript
// JS 应用
audio.src = "../../etc/passwd";
```

**影响**：
- 攻击者可读取任意系统文件
- 可能导致敏感信息泄露

**修复建议**：
```cpp
// 在 player.cpp 和 recorder.cpp 中添加路径规范化
#include <libgen.h>

// 限制在允许的目录内
const char* ALLOWED_PATHS[] = {
    "/data/",
    "/storage/",
    "/sdcard/"
};

// 标准化路径并验证
std::string NormalizePath(const std::string& input) {
    std::string resolved = input;
    // 移除 "../" 序列
    size_t pos = 0;
    while ((pos = resolved.find("../", pos)) != std::string::npos) {
        resolved.replace(pos, 3, "");
    }
    return resolved;
}
```

---

### 2. 参数类型混淆漏洞（中风险）

**证据**：
- 文件：`interfaces/kits/player_lite/js/builtin/src/audio_module.cpp:107-108`
- API：`AudioModule::GetPlayState()`

**触发路径**：
```javascript
// 传入非对象参数
audio.getPlayState("invalid_type");
```

**影响**：
- 可能导致类型混淆攻击
- 方法返回 false，但应用可能继续执行

**修复建议**：
- 当前已有基础类型检查，但建议：
- 添加详细的错误信息
- 使用 JSI::CreateError() 返回错误对象
- 记录所有无效调用到日志

---

### 3. 线程竞态条件（中风险）

**证据**：
- 文件：`interfaces/kits/player_lite/js/builtin/src/audio_player.cpp:128-227`
- 竞态变量：`isRunning_`, `status_`, `src_`
- 同步原语：`pthread_mutex_t`, `pthread_cond_t`

**触发场景**：
```javascript
// 快速连续调用 stop() 和 play()
audio.stop();
audio.play(); // 可能在状态更新前被调用
```

**影响**：
- 状态不一致
- 潜在的未定义行为

**修复建议**：
- 当前使用了 mutex 和 condition，但建议：
- 在状态转换时添加原子操作
- 明确锁的持有顺序

---

### 4. 内存泄漏风险（中风险）

**证据**：
- 文件：`interfaces/kits/player_lite/js/builtin/src/audio_player.cpp:55-62, 294-302`
- 泄漏点：事件监听器（AudioEventListener）
- 问题：旧监听器未被释放（虽然代码有删除逻辑）

**触发场景**：
```javascript
// 频繁设置事件监听器
audio.onplay = callback1;
audio.onplay = callback2; // callback1 泄漏
```

**影响**：
- 内存占用增长
- 可能导致 OOM（内存不足）

**修复建议**：
- ✅ 当前代码已有删除逻辑：
```cpp
void AudioModule::SetOnPlayListener(AudioEventListener *listener) {
    if (onPlayListener_ != nullptr) {
        delete onPlayListener_;  // 删除旧监听器
    }
    onPlayListener_ = listener;
}
```
- 建议使用智能指针（shared_ptr）管理监听器生命周期

---

### 5. 权限检查绕过（低风险）

**证据**：
- 文件：`frameworks/player_lite/binder/player.cpp:58-71`, `frameworks/recorder_lite/recorder.cpp:203-218`
- API：`Player::Prepare()`, `Recorder::Prepare()`

**当前检查逻辑**：
```cpp
if (CheckSelfPermission("ohos.permission.READ_MEDIA") != GRANTED) {
    MEDIA_WARNING_LOG("Process can not read media.");
    return MEDIA_PERMISSION_DENIED;
}
```

**影响**：
- 恶意应用可能在权限未授予时仍能部分功能
- 信息泄露

**修复建议**：
- 当前实现基本正确
- 建议在所有敏感操作入口添加权限检查（当前仅在 Prepare 中）
- 确保错误码正确返回

---

### 6. IPC 参数篡改风险（中风险）

**证据**：
- 文件：`frameworks/player_lite/binder/player_client.cpp:67-73`, `services/player_lite/server/src/player_server.cpp`
- IPC 序列化：`WriteInt32`, `WriteRawData`

**攻击向量**：
- 通过修改 IPC 缓冲区篡改参数
- 利用整数溢出改变参数值

**影响**：
- 未授权操作
- 服务崩溃

**修复建议**：
- 添加参数边界检查
- 使用安全的序列化/反序列化库
- 在服务端二次验证参数

---

### 7. 文件描述符泄露风险（低风险）

**证据**：
- 文件：`interfaces/kits/recorder_lite/recorder.h:460`
- API：`Recorder::SetOutputFile(int32_t fd)`

**风险场景**：
```cpp
// 应用传递任意 FD
int malicious_fd = open("/etc/passwd", O_RDONLY);
recorder.SetOutputFile(malicious_fd);
```

**影响**：
- 敏感文件泄露
- 任意文件操作

**修复建议**：
- 验证 FD 路径（如果可能）
- 限制 FD 的使用范围
- 使用路径替代 FD（如果可能）

---

## 安全加固建议

### 高优先级

1. **路径规范化**：添加到 `SetOutputPath` 和 `SetSource`
2. **参数深度验证**：对 IPC 和文件参数进行二次验证
3. **输入大小限制**：限制 URI 长度和参数大小
4. **速率限制**：添加 IPC 调用频率限制

### 中优先级

1. **内存管理**：使用智能指针管理资源
2. **错误处理**：完善错误码定义和日志
3. **状态机验证**：确保状态转换的一致性
4. **线程安全审查**：审查所有共享状态的访问

### 低优先级

1. **代码审计**：定期进行安全审计
2. **模糊测试**：对 IPC 接口进行模糊测试
3. **文档更新**：及时更新安全相关文档

---

## 安全检查范围

### 已检查范围

- [x] N-API（JSI）接口参数校验
- [x] IPC 通信安全机制
- [x] 文件操作权限检查
- [x] 线程同步机制
- [x] 内存管理机制

### 未检查范围（局限性）

- [ ] IPC 底层序列化安全性（IpcIo 实现）
- [ ] SAMGR 服务注册安全性
- [ ] permission_lite 的 access token/UID/bundle name 验证机制（外部依赖）
- [ ] Surface 生命周期的安全性
- [ ] 播放引擎（histreamer）和录制引擎的安全性
- [ ] 硬件 SDK（hardware_media_sdk）的安全性

---

## 相关跳转

- [项目概览](00_Overview.md)
- [目录结构与模块职责](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [对外 JSI 接口](03_JSI_Interfaces.md)
- [常见问题](08_FAQ.md)
