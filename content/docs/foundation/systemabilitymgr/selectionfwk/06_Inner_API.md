# 内部 API 参考

本文档描述划词服务子系统内部使用的 C/C++ API，主要面向系统集成者和框架开发者。

## 1. Inner API 概述

### 1.1 Inner API 定义

Inner API 是子系统内部模块间通信使用的 C++ 接口，具有以下特点：
- 仅限系统框架使用
- 不对普通应用暴露
- 可能随版本演进发生变更

**证据来源**: `interfaces/inner_kits/selection_client/include/selection_client.h`

```cpp
class SelectionClient {
public:
    SELECTION_API static SelectionClient& GetInstance();
    bool IsCurrentSelectionApp(int pid);
    int32_t GetSelectionContent(std::string& selectionContent);
    int32_t SetPanelShowingStatus(bool status);
};
```

### 1.2 Inner API 清单

| API 名称 | 返回类型 | 参数 | 功能描述 | 稳定性 |
|---------|---------|------|---------|--------|
| `SelectionClient::GetInstance()` | SelectionClient& | - | 获取单例实例 | Stable |
| `IsCurrentSelectionApp(int pid)` | bool | pid: 进程 ID | 判断是否为当前划词应用 | Stable |
| `GetSelectionContent(string&)` | int32_t | 输出参数: content | 获取选中文本内容 | Stable |
| `SetPanelShowingStatus(bool)` | int32_t | status: 显示状态 | 设置面板显示状态 | Stable |

## 2. SelectionClient API 详解

### 2.1 获取单例

```cpp
static SelectionClient& GetInstance();
```

**功能**: 获取 SelectionClient 的单例实例

**线程安全**: 是 (内部实现为 Meyer's Singleton)

**示例**:
```cpp
auto& client = SelectionClient::GetInstance();
```

### 2.2 判断划词应用

```cpp
bool IsCurrentSelectionApp(int pid);
```

**功能**: 判断指定 pid 的进程是否为当前划词应用

**参数**:
| 参数名 | 类型 | 说明 |
|-------|------|------|
| pid | int | 待检查的进程 ID |

**返回值**:
| 值 | 说明 |
|---|------|
| true | 是当前划词应用 |
| false | 不是当前划词应用 |

**调用链**:
```
IsCurrentSelectionApp()
    ↓
SelectionClient::IsCurrentSelectionApp()
    ↓
IPC Proxy → ISelectionService::IsCurrentSelectionApp()
    ↓
SelectionService::IsCurrentSelectionApp()
```

### 2.3 获取选中文本

```cpp
int32_t GetSelectionContent(std::string& selectionContent);
```

**功能**: 获取当前选中的文本内容

**参数**:
| 参数名 | 类型 | 说明 |
|-------|------|------|
| selectionContent | std::string& | 输出参数，返回选中文本 |

**返回值**:
| 返回值 | 说明 |
|-------|------|
| 0 | 成功 |
| 非 0 | 错误码 |

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 0 | 成功 |
| INVALID_DATA | 参数错误 |
| SERVICE_UNAVAILABLE | 服务不可用 |
| TIMEOUT | 操作超时 |

**长度限制**: 选中文本最大长度为 6000 字节

**示例**:
```cpp
std::string content;
int32_t ret = SelectionClient::GetInstance().GetSelectionContent(content);
if (ret == 0) {
    SELECTION_HILOGI("Selection content: %{public}s", content.c_str());
}
```

### 2.4 设置面板状态

```cpp
int32_t SetPanelShowingStatus(bool status);
```

**功能**: 设置划词面板的显示/隐藏状态

**参数**:
| 参数名 | 类型 | 说明 |
|-------|------|------|
| status | bool | true: 显示面板, false: 隐藏面板 |

**返回值**:
| 返回值 | 说明 |
|-------|------|
| 0 | 成功 |
| 非 0 | 错误码 |

## 3. 头文件清单

### 3.1 公开头文件

| 头文件路径 | 暴露内容 | 使用场景 |
|-----------|---------|---------|
| `selection_client.h` | SelectionClient 类定义 | 客户端调用 |
| `visibility.h` | API可见性宏定义 | 编译配置 |

### 3.2 头文件内容

**selection_client.h**:

```cpp
/*
 * Copyright (c) 2025 Huawei Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 */

#ifndef SELECTION_CLIENT_H
#define SELECTION_CLIENT_H

#include <cstdint>
#include <string>
#include "visibility.h"

class SelectionClient {
public:
    SELECTION_API static SelectionClient& GetInstance();
    bool IsCurrentSelectionApp(int pid);
    int32_t GetSelectionContent(std::string& selectionContent);
    int32_t SetPanelShowingStatus(bool status);
};

#endif // SELECTION_CLIENT_H
```

**visibility.h**:

```cpp
/*
 * Copyright (c) 2025 Huawei Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 */

#ifndef VISIBILITY_H
#define VISIBILITY_H

#ifdef _WIN32
    #define SELECTION_API __declspec(dllexport)
#else
    #define SELECTION_API __attribute__((visibility("default")))
#endif

#endif // VISIBILITY_H
```

## 4. 编译配置

### 4.1 导出配置

**证据来源**: `interfaces/inner_kits/selection_client/BUILD.gn`

```gn
config("selection_client_native_public_config") {
  visibility = [
    "./*",
    "//foundation/systemabilitymgr/selectionfwk/interfaces/inner_kits/selection_client/*",
  ]
  include_dirs = [
    "include",
  ]
}

ohos_shared_library("selection_client") {
  # ...
  version_script = "selection_client.versionscript"
  innerapi_tags = [ "platformsdk" ]
  # ...
}
```

### 4.2 版本脚本

**selection_client.versionscript**:

```text
{
    global:
        SelectionClient*;
        GetInstance*;
    local:
        *;
};
```

## 5. 使用指南

### 5.1 集成步骤

1. **添加依赖**: 在 `BUILD.gn` 中添加
   ```gn
   external_deps = [
     "selectionfwk:selection_client",
   ]
   ```

2. **包含头文件**: 
   ```cpp
   #include "selection_client.h"
   ```

3. **调用 API**:
   ```cpp
   auto& client = SelectionClient::GetInstance();
   ```

### 5.2 注意事项

| 注意事项 | 说明 |
|---------|------|
| 线程安全 | 所有 API 均为线程安全 |
| 生命周期 | SelectionClient 为单例，长期有效 |
| 错误处理 | 必须检查返回值 |
| 性能考虑 | IPC 调用，避免高频调用 |

## 6. 服务端内部 API

### 6.1 SelectionService 主类

**头文件**: `service/include/selection_service.h`

```cpp
class SelectionService : public SystemAbility {
public:
    static sptr<SelectionService> GetInstance();
    
    // IPC 接口实现
    ErrCode RegisterListener(const sptr<ISelectionListener>& listener);
    ErrCode UnregisterListener(const sptr<ISelectionListener>& listener);
    ErrCode IsCurrentSelectionApp(int pid, bool& resultValue);
    ErrCode GetSelectionContent(std::string& selectionContent);
    ErrCode SetPanelShowingStatus(bool status);
    
protected:
    virtual void OnStart() override;
    virtual void OnStop() override;
};
```

### 6.2 配置管理 API

**头文件**: `service/include/selection_config.h`

```cpp
class SelectionConfig {
public:
    bool GetEnable() const;
    void SetEnable(bool enable);
    std::string GetApplicationInfo() const;
    void SetApplicationInfo(const std::string& info);
};
```

### 6.3 输入监控 API

**头文件**: `service/include/selection_input_monitor.h`

```cpp
class SelectionInputMonitor : public MMI::InputObserver {
public:
    virtual void OnInputEvent(std::shared_ptr<MMI::KeyEvent> event) override;
    virtual void OnInputEvent(std::shared_ptr<MMI::PointerEvent> event) override;
};
```

---

**相关链接**:

- [返回 SUMMARY](./SUMMARY.md)
- [架构说明](./02_Architecture.md)
- [N-API 参考](./03_NAPI_Reference.md)
- [构建系统](./04_Build_System.md)
