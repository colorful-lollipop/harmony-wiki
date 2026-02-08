# 内部 API (Inner API)

> bundle_tool 模块接口、依赖方向与稳定性说明

## 模块结构

```
frameworks/
├── include/
│   ├── bundle_command.h          # Bundle 管理命令
│   ├── bundle_command_common.h   # 公共命令逻辑
│   ├── shell_command.h           # Shell 命令基类
│   ├── quick_fix_command.h       # 快速修复命令
│   ├── quick_fix_status_callback_host_impl.h
│   ├── bundle_test_tool.h        # 测试工具
│   ├── status_receiver_impl.h    # 状态接收
│   └── bundle_tool_callback/      # IPC 回调
│       ├── i_bundle_tool_callback.h
│       └── bundle_tool_callback_stub.h
└── src/
    ├── main.cpp                  # 程序入口
    ├── bundle_command.cpp         # 命令实现
    ├── bundle_command_common.cpp  # 公共命令
    ├── shell_command.cpp          # Shell 命令基类
    ├── quick_fix_command.cpp      # 快速修复
    ├── bundle_test_tool.cpp       # 测试工具
    ├── status_receiver_impl.cpp   # 状态接收
    ├── quick_fix_status_callback_host_impl.cpp
    └── bundle_tool_callback/
        └── bundle_tool_callback_stub.cpp
```

---

## ShellCommand 基类

### 类定义

**文件**: `include/shell_command.h:34`

```cpp
class ShellCommand {
public:
    ShellCommand(int argc, char *argv[], std::string name);
    virtual ~ShellCommand();

    ErrCode OnCommand();
    std::string ExecCommand();
    std::string GetCommandErrorMsg() const;
    std::string GetUnknownOptionMsg(std::string &unknownOption) const;
    std::string GetMessageFromCode(const int32_t code) const;

    virtual ErrCode CreateCommandMap() = 0;
    virtual ErrCode CreateMessageMap() = 0;
    virtual ErrCode Init() = 0;

protected:
    static constexpr int MIN_ARGUMENT_NUMBER = 2;
    int argc_;
    char **argv_;
    std::string cmd_;
    std::vector<std::string> argList_;
    std::string name_;
    std::map<std::string, std::function<int()>> commandMap_;
    std::map<int32_t, std::string> messageMap_;
    std::string resultReceiver_ = "";
};
```

### 公共方法

| 方法 | 访问级别 | 返回类型 | 功能 |
|------|----------|----------|------|
| `ShellCommand()` | public | - | 构造函数 |
| `~ShellCommand()` | public | virtual | 析构函数 |
| `OnCommand()` | public | ErrCode | 执行命令 |
| `ExecCommand()` | public | std::string | 获取执行结果 |
| `GetCommandErrorMsg()` | public | std::string | 获取错误消息 |
| `GetUnknownOptionMsg()` | public | std::string | 获取未知选项消息 |
| `GetMessageFromCode()` | public | std::string | 根据错误码获取消息 |
| `CreateCommandMap()` | public virtual | ErrCode | 注册命令映射 |
| `CreateMessageMap()` | public virtual | ErrCode | 注册消息映射 |
| `Init()` | public virtual | ErrCode | 初始化 |

### 稳定性

- **稳定性**: **稳定**
- **说明**: 所有命令均继承此类，接口不变

---

## BundleManagerShellCommand

### 类定义

**文件**: `include/bundle_command.h:265`

```cpp
class BundleManagerShellCommand : public ShellCommand {
public:
    BundleManagerShellCommand(int argc, char *argv[]);
    ~BundleManagerShellCommand() override {}

private:
    ErrCode CreateCommandMap() override;
    ErrCode CreateMessageMap() override;
    ErrCode Init() override;

    // 17 个命令处理方法...
};
```

### 继承关系

```
ShellCommand
    └── BundleManagerShellCommand
```

### 稳定性

- **稳定性**: **稳定**
- **说明**: 对外接口，命令集相对固定

---

## 公共业务方法

### InstallOperation

```cpp
// 文件: bundle_command.h:312
int32_t InstallOperation(
    const std::vector<std::string> &bundlePaths,
    InstallParam &installParam,
    int32_t waittingTime,
    std::string &resultMsg) const;
```

**功能**: 安装 HAP/HSP 包

**调用链**:
```
InstallOperation()
  └── bundleInstallerProxy_->InstallBundle(bundlePaths, installParam)
        └── IPC → BundleManagerService
```

### UninstallOperation

```cpp
// 文件: bundle_command.h:314
int32_t UninstallOperation(
    const std::string &bundleName,
    const std::string &moduleName,
    InstallParam &installParam) const;
```

**功能**: 卸载应用或模块

### CleanBundleCacheFilesOperation

```cpp
// 文件: bundle_command.h:323
bool CleanBundleCacheFilesOperation(
    const std::string &bundleName,
    int32_t userId,
    int32_t appIndex = 0) const;
```

**功能**: 清理应用缓存

### CleanBundleDataFilesOperation

```cpp
// 文件: bundle_command.h:324
bool CleanBundleDataFilesOperation(
    const std::string &bundleName,
    int32_t userId,
    int32_t appIndex = 0) const;
```

**功能**: 清理应用数据

### SetApplicationEnabledOperation

```cpp
// 文件: bundle_command.h:326
bool SetApplicationEnabledOperation(
    const AbilityInfo &abilityInfo,
    bool isEnable,
    int32_t userId) const;
```

**功能**: 使能或禁用应用

### GetUdid

```cpp
// 文件: bundle_command.h:317
std::string GetUdid() const;
```

**功能**: 获取设备 UDID

---

## QuickFixCommand

### 类定义

**文件**: `include/quick_fix_command.h`

```cpp
class QuickFixCommand : public ShellCommand {
public:
    QuickFixCommand(int argc, char *argv[]);
    ~QuickFixCommand() override {}

private:
    ErrCode CreateCommandMap() override;
    ErrCode CreateMessageMap() override;
    ErrCode Init() override;

    ErrCode RunAsQuickFixCommand() override;
};
```

### 稳定性

- **稳定性**: **稳定**
- **条件**: `quick_fix_bm` feature flag 启用

---

## 回调接口

### IBundleToolCallback

**文件**: `include/bundle_tool_callback/i_bundle_tool_callback.h`

**功能**: 快速修复进度回调

**实现类**:
- `BundleToolCallbackStub`: `bundle_tool_callback_stub.cpp`
- `StatusReceiverImpl`: `status_receiver_impl.cpp`

### 稳定性

- **稳定性**: **稳定**
- **说明**: IPC 回调接口，跨进程通信使用

---

## 依赖方向

### 内部依赖

```
main.cpp
    └── BundleManagerShellCommand
            ├── ShellCommand
            ├── IBundleMgr (IPC)
            ├── IBundleInstaller (IPC)
            └── QuickFixCommand

bundle_command.cpp
    └── 所有命令实现
```

### 外部依赖

| 模块 | 依赖类型 | 用途 |
|------|----------|------|
| `ability_base` | external_deps | Want 结构 |
| `ability_runtime` | external_deps | AppManager, QuickFixManager |
| `bundle_framework` | external_deps | Bundle 管理核心 |
| `ipc` | external_deps | IPC 通信 |
| `samgr` | external_deps | 系统服务 |
| `os_account` | external_deps | 用户账号 |
| `hilog` | external_deps | 日志 |
| `common_event_service` | external_deps | 公共事件 |

**证据来源**: `frameworks/BUILD.gn:60-74`

---

## 稳定性标注

### 稳定接口

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| ShellCommand 基类 | **稳定** | 无接口变更 |
| BundleManagerShellCommand | **稳定** | 固定命令集 |
| IBundleMgr | **稳定** | IPC 接口契约 |
| IBundleInstaller | **稳定** | IPC 接口契约 |
| IBundleToolCallback | **稳定** | IPC 接口契约 |

### 可选接口

| 接口 | 稳定性 | 条件 |
|------|--------|------|
| QuickFixCommand | **可选** | `quick_fix_bm=true` |
| os_account | **可选** | `account_enable_bm=true` |
| Overlay | **可选** | `overlay_install_bm=true` |
| 分布式 | **可选** | `distributed_bundle_framework_bm=true` |

**证据来源**: `bundletool.gni:24-39`

---

## 可替换点

### 命令处理可替换

BundleManagerShellCommand 继承 ShellCommand，可通过派生类替换命令集：

```cpp
// ShellCommand 基类定义 (shell_command.h:34)
class ShellCommand {
    // 虚方法
    virtual ErrCode CreateCommandMap() = 0;  // 可替换命令注册
    virtual ErrCode CreateMessageMap() = 0;  // 可替换错误映射
    virtual ErrCode Init() = 0;              // 可替换初始化
};
```

### 回调可替换

IBundleToolCallback 实现可替换：

```cpp
// i_bundle_tool_callback.h
class IBundleToolCallback : public IRemoteStub<IBundleToolCallback> {
public:
    // 回调接口方法
};
```

### IPC 代理可替换

bundleMgrProxy_ 和 bundleInstallerProxy_ 通过接口获取，可注入替换：

```cpp
// bundle_command.h:343-344
sptr<IBundleMgr> bundleMgrProxy_;           // 可替换
sptr<IBundleInstaller> bundleInstallerProxy_;  // 可替换
```

---

## 相关文档

- [01_Architecture.md](./01_Architecture.md) - 架构设计
- [02_Command_Reference.md](./02_Command_Reference.md) - 命令参考
- [04_Build.md](./04_Build.md) - 构建系统
