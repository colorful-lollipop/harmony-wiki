# AI Engine 常见问题

## 构建问题

### Q1：编译 ai_engine 失败，提示找不到 samgr_lite

**问题描述**：
```
error: cannot find //distributedschedule_samgr_lite/samgr:samgr
```

**原因**：未正确设置产品配置

**解决方案**：
```bash
# 1. 设置编译路径
hb set -root <project_root>

# 2. 选择产品（确保包含 ai 子系统）
hb set -p

# 3. 选择 hispark_taurus 或其他支持的产品
# 4. 重新编译
hb build ai_engine
```

**证据**：`README.md:58-75` 编译步骤说明

---

### Q2：插件未编译到目标产物

**问题描述**：libasr_keyword_spotting.so 未生成

**原因**：条件编译，仅 hispark_taurus 开发板支持 NNIE 插件

**解决方案**：
```bash
# 确认开发板配置
hb set -p | grep hispark

# 如需支持其他开发板，需移植 NNIE 适配层
# 或禁用 NNIE 相关代码
```

**证据**：`services/server/plugin/asr/keyword_spotting/BUILD.gn` NNIE 依赖

---

### Q3：编译时提示 cflags 冲突

**问题描述**：
```
error: undefined reference to '__cxa_atexit'
```

**原因**：链接顺序或库配置问题

**解决方案**：
```bash
# 确保添加 -lstdc++ 链接选项
hb build -f --clean
```

**证据**：`services/server/BUILD.gn:25` ldflags 定义

---

## 运行问题

### Q4：ai_server 启动失败

**问题描述**：服务启动后立即退出

**排查步骤**：
```bash
# 1. 查看系统日志
hilog | grep -i ai

# 2. 检查 SAMGR 注册状态
hilog | grep -i samgr

# 3. 手动启动测试
./ai_server
```

**常见原因**：
- SAMGR 未启动（系统初始化问题）
- 插件配置文件缺失
- 权限不足

**证据**：`services/server/communication_adapter/source/start_server.c` 服务启动入口

---

### Q5：客户端连接超时

**问题描述**：
```
KWS_RETCODE_INIT_ERROR (1001)
```

**排查步骤**：
```bash
# 1. 确认 ai_server 正在运行
ps | grep ai_server

# 2. 检查 SAMGR 服务注册
hilog | grep AI_SERVICE

# 3. 检查网络连通性（分布式场景）
ping <remote_ip>
```

**常见原因**：
- ai_server 未启动
- SAMGR 服务注册失败
- 网络配置问题（分布式场景）

---

### Q6：推理结果异常

**问题描述**：SyncExecute 返回成功，但结果无效

**排查步骤**：
```bash
# 1. 检查输入数据格式
# KWS: 16kHz PCM, 16-bit, mono

# 2. 检查输出解析
# 返回值: int32_t 数组，表示检测到的关键词索引
```

**示例代码**：
```cpp
// 正确的数据格式
Array<int16_t> audioInput;
audioInput.data = pcmBuffer;     // 16kHz, 16-bit PCM
audioInput.size = bufferSize / 2; // 采样点数量

int32_t ret = kws.SyncExecute(audioInput);
if (ret != KWS_RETCODE_SUCCESS) {
    HILOGE("KWS failed: %d", ret);
}
```

---

## 调试问题

### Q7：如何启用调试日志

**解决方案**：
```cpp
// 在代码中设置日志级别
#include "aie_log.h"

// 启用 DEBUG 日志
HILOG_DEBUG("[Module] debug message");
HILOG_INFO("[Module] info message");
HILOG_ERROR("[Module] error message");
```

**证据**：`services/common/utils/log/aie_log.h` 日志接口

---

### Q8：如何调试插件加载问题

**问题描述**：插件加载失败

**排查步骤**：
```bash
# 1. 检查插件配置文件
cat /system/etc/ai_engine_plugin.ini

# 2. 验证插件路径
ls -la /system/lib/libasr_keyword_spotting.so

# 3. 检查 dlsym 错误
hilog | grep dlopen
hilog | grep PLUGIN_INTERFACE
```

**证据**：`services/server/plugin_manager/source/plugin_manager.cpp` 插件加载逻辑

---

### Q9：如何追踪 IPC 调用

**解决方案**：使用 Hildump 工具

```bash
# 启用 IPC 追踪
hildump --ipc-trace on

# 运行测试
./ai_test_function_door

# 查看追踪结果
hildump --ipc-trace dump
```

---

### Q10：崩溃时如何定位

**解决方案**：
```bash
# 1. 获取崩溃日志
hilog | grep -A 20 "stack backtrace"

# 2. 使用 addr2line 解析地址
addr2line -e ai_server 0x12345 -C -f

# 3. 检查 core dump
gdb ai_server core
```

**证据**：`services/server/BUILD.gn:24` -Wl,-Map=server.map 生成符号映射

---

## 开发问题

### Q11：如何添加新插件

**步骤**：
```cpp
// 1. 创建插件类，继承 IPlugin
class NewPlugin : public IPlugin {
public:
    const long long GetVersion() const override { return 1; }
    const char *GetName() const override { return "NewPlugin"; }
    const char *GetInferMode() const override { return "sync"; }
    // ... 其他方法实现
};

// 2. 使用 PLUGIN_INTERFACE_IMPL 宏导出
PLUGIN_INTERFACE_IMPL(NewPlugin)

// 3. 添加 BUILD.gn 配置
// 4. 更新插件配置文件
```

**证据**：`services/server/plugin/asr/keyword_spotting/kws_plugin.cpp` 插件示例

---

### Q12：同步和异步模式如何选择

**选择标准**：

| 场景 | 推荐模式 | 说明 |
|------|----------|------|
| 实时响应（<100ms） | 同步 | 简单直接，阻塞等待结果 |
| 大模型推理（>500ms） | 异步 | 避免阻塞，不卡死应用 |
| 流式处理 | 异步 | 持续接收数据，回调返回结果 |

**配置方式**：
```cpp
// AlgorithmInfo.isAsync 决定
AlgorithmInfo algoInfo;
algoInfo.isAsync = true;  // 异步模式
```

---

### Q13：如何处理插件异常

**建议**：
```cpp
int32_t MyPlugin::SyncProcess(IRequest *request, IResponse *&response)
{
    // 1. 参数校验
    if (request == nullptr) {
        return RETCODE_FAILURE;
    }

    // 2. 异常捕获
    try {
        // 推理逻辑
    } catch (const std::exception &e) {
        HILOGE("Plugin exception: %s", e.what());
        return RETCODE_FAILURE;
    }

    // 3. 资源清理确保
    return RETCODE_SUCCESS;
}
```

---

### Q14：内存泄漏如何排查

**工具**：
```bash
# 使用 hiebpd 内存检测
hiebpd --enable ai_server

# 运行测试
./ai_test_function_door

# 查看泄漏报告
hiebpd --report
```

**常见泄漏场景**：
- 未调用 Destroy 释放 SDK
- 插件 Release 未释放模型内存
- 异步回调未注销

---

### Q15：如何贡献代码

**步骤**：
```bash
# 1. Fork 仓库
git remote add fork <your_fork_url>

# 2. 创建分支
git checkout -b feature/new_algorithm

# 3. 开发并测试
hb build ai_engine
hb test ai_engine

# 4. 提交 PR
git push fork feature/new_algorithm
```

**注意**：
- 遵循 OpenHarmony 代码规范
- 添加必要注释和单元测试
- 更新相关文档

---

## 性能问题

### Q16：推理延迟过高

**优化建议**：
```cpp
// 1. 使用异步模式，避免阻塞
algoInfo.isAsync = true;

// 2. 批处理推理（如果支持）
// 将多个输入合并为一批

// 3. 模型量化
// 使用 INT8 量化替代 FP32
```

---

### Q17：内存占用过高

**排查方法**：
```bash
# 查看内存使用
cat /proc/<pid>/status | grep -i vm

# 内存分析
hiebpd --leak-check ai_server
```

**常见原因**：
- 模型未及时卸载
- 异步回调未释放
- 大数据共享内存未清理

**解决方案**：
```cpp
// 确保及时释放
kws.Destroy();  // 释放客户端资源

// 服务端及时卸载插件
ClientRelease(); // 触发服务端 Release
```

---

## 故障排查流程图

```mermaid
graph TD
    A[发现问题] --> B{编译问题?}
    B -->|是| C[检查产品配置]
    B -->|否| D{运行问题?}
    C --> E[hb set -p]
    D --> F{连接失败?}
    F -->|是| G[检查 ai_server 状态]
    F -->|否| H{推理失败?}
    G --> I[ps | grep ai_server]
    H --> J[检查输入数据格式]
    I --> K[查看 hilog 日志]
    K --> L[定位根因]
```
