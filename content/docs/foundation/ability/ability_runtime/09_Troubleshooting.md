# 常见问题与定位路径

## 概述

本文档汇总 ability_runtime 开发和调试过程中的常见问题，提供定位路径和解决方案。

## 编译问题

### 问题 1：GN 编译报错

**现象**：
```
error: cannot find header file
```

**定位路径**：
1. 检查头文件路径配置
2. 验证 include_dirs 配置
3. 确认依赖是否正确声明

**解决方案**：
```bash
# 清理后重新构建
./build.sh --clean
./build.sh --product-name <product> --build-target ability_runtime
```

**相关配置**：
```
BUILD.gn 中的 include_dirs 配置
```

### 问题 2：N-API 模块未注册

**现象**：
```
Cannot find module 'abilityManager'
```

**定位路径**：
1. 检查模块是否在 `frameworks/js/napi/BUILD.gn` 的 deps 中声明
2. 验证 `napi_module_register()` 是否被调用
3. 检查模块文件名是否正确

**代码证据**：
```cpp
// 检查模块注册
grep -r "napi_module_register" frameworks/js/napi/ability_manager/
```

### 问题 3：符号未定义

**现象**：
```
undefined reference to 'xxx'
```

**定位路径**：
1. 检查 `deps` 和 `public_deps` 配置
2. 验证源文件是否在 `sources` 中声明
3. 检查库链接顺序

## 运行时问题

### 问题 1：Ability 启动失败

**错误码**：16000001 ~ 16000010

**定位步骤**：
```bash
# 1. 查看日志
hilog | grep -i "ability"

# 2. 使用 aa dump 查看状态
aa dump

# 3. 检查权限
```

**常见原因**：
- Bundle 名称错误
- Ability 不存在
- 权限不足
- 目标应用已冻结

**日志位置**：
- `hilog`：主日志
- `faultloggerd`：故障日志
- `/data/log/`：应用日志

### 问题 2：连接 Service 失败

**错误码**：16000003（IPC 错误）

**定位步骤**：
```bash
# 1. 查看服务是否运行
aa dump

# 2. 检查连接回调
hilog | grep -i "connect"

# 3. 验证权限配置
```

**解决方案**：
- 确认 Service 已在 `module.json5` 中声明
- 检查 `connectAbility` 权限
- 验证回调实现正确

### 问题 3：生命周期回调未触发

**现象**：
```
onCreate/onStart/onForeground 未被调用
```

**定位路径**：
1. 检查 Ability 实现是否继承正确基类
2. 验证生命周期方法是否 `override`
3. 检查 AMS 日志

**代码证据**：
```cpp
// 正确的生命周期实现
class MyAbility : public UIAbility {
public:
    void OnCreate(const Want &want) override {
        // 实现
    }
};
```

### 问题 4：IPC 调用超时

**现象**：
```
IPC timeout, caller: xxx
```

**定位路径**：
1. 检查目标服务是否正常运行
2. 查看是否有死锁
3. 检查系统负载

**相关日志**：
```
hilog | grep -i "IPC"
hilog | grep -i "timeout"
```

### 问题 5：内存泄漏

**现象**：
```
High memory usage detected
```

**定位工具**：
- `hitrace`：性能追踪
- `hisysevent`：系统事件
- `memmgr`：内存管理

**定位步骤**：
```bash
# 1. 查看内存使用
hitrace --mem

# 2. 分析堆内存
hitrace --heap
```

## 调试方法

### 1. 日志调试

**HiLog 使用**：
```cpp
#include "hilog/log.h"

constexpr OHOS::HiviewDFX::HiLogLabel LABEL = {LOG_CORE, 0, "MyTag"};

HILOG_INFO(LABEL, "Message: %{public}d", value);
```

**日志级别**：
- DEBUG
- INFO
- WARN
- ERROR
- FATAL

### 2. 断点调试

**使用 GDB**：
```bash
# 附加到进程
gdb pid

# 设置断点
break ability_manager_service.cpp:123
run

# 查看调用栈
backtrace
```

### 3. 能力调试

**aa 命令调试**：

| 命令 | 说明 |
|------|------|
| `aa start` | 启动 Ability |
| `aa dump` | 打印 Ability 状态 |
| `aa force-stop` | 强制停止应用 |

### 4. 性能分析

**HiTrace 使用**：
```bash
# 启动追踪
hitrace --trace ability

# 停止并分析
hitrace --stop trace.bin

# 查看分析结果
hitrace --parse trace.bin
```

## 常见错误码汇总

### Ability 错误码（160000xx）

| 错误码 | 说明 | 解决方案 |
|--------|------|---------|
| 16000001 | 指定的能力不存在 | 检查 Bundle/Ability 名称 |
| 16000002 | 参数不合法 | 验证参数格式 |
| 16000003 | IPC 错误 | 检查目标服务状态 |
| 16000004 | 目标应用已冻结 | 等待或强制停止 |
| 16000005 | 权限不足 | 检查权限配置 |
| 16000006 | 操作被禁止 | 检查操作是否允许 |
| 16000007 | 任务栈已满 | 清理任务 |

### 权限错误码

| 错误码 | 说明 |
|--------|------|
| 201 | 参数不合法 |
| 202 | 非系统应用调用系统 API |
| 401 | 参数错误 |

### IPC 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 失败 |
| 1 | 未知错误 |

## 定位路径速查

| 问题类型 | 定位方法 | 关键日志 |
|---------|---------|---------|
| 启动失败 | `aa dump`, `hilog` | ability_manager |
| 连接失败 | `hilog` | connect_manager |
| 崩溃 | `faultloggerd`, `hilog` | fault |
| 性能问题 | `hitrace` | trace |
| 内存问题 | `hitrace --mem` | memmgr |

## 相关文档

- [N-API 参考](04_NAPI_Reference.md)
- [架构说明](03_Architecture.md)
- [aa 命令文档](tools/aa/)
