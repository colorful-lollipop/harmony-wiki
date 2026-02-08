# 内部 API

## 概述

Previewer **没有 N-API 接口**，这是一个纯 C++ 桌面应用。项目的对外接口通过 **内部 Kit (Inner Kit)** 提供。

## 内部 Kit 清单

根据 `bundle.json:53-77` 定义：

| Kit 名称 | 类型 | 输出 | 头文件目录 |
|----------|------|------|------------|
| `libide_util` | so | `libide_util.so/dylib/dll` | `util/` |
| `ide_extension` | so | `libide_extension.so/dylib/dll` | `jsapp/rich/external/` |

---

## libide_util.so

### KeyboardHelper

**文件**: `util/KeyboardHelper.h:22-24`

```cpp
class KeyboardHelper {
public:
    static short GetKeyStateByKeyName(const std::string& keyName);
};
```

| 方法 | 签名 | 描述 |
|------|------|------|
| `GetKeyStateByKeyName` | `static short GetKeyStateByKeyName(const std::string& keyName)` | 根据键名获取键盘状态 |

**平台实现**:
- `util/windows/KeyboardHelper.cpp`: Win32 `GetAsyncKeyState`
- `util/unix/KeyboardHelper.cpp`: macOS Carbon API
- `util/linux/KeyboardHelper.cpp`: Linux evdev

### ClipboardHelper

**文件**: `util/ClipboardHelper.h:22-26`

```cpp
class ClipboardHelper {
public:
    static void SetClipboardData(const std::string& data);
    static const std::string GetClipboardData();
};
```

| 方法 | 签名 | 描述 |
|------|------|------|
| `SetClipboardData` | `static void SetClipboardData(const std::string& data)` | 设置剪贴板数据 |
| `GetClipboardData` | `static const std::string GetClipboardData()` | 获取剪贴板数据 |

**平台实现**:
- `util/windows/ClipboardHelper.cpp`: Win32 Clipboard API
- `util/unix/ClipboardHelper.cpp`: macOS Pasteboard
- `util/linux/ClipboardX11.cpp`: X11 Selection

---

## ide_extension.so

### EventRunner

**文件**: `jsapp/rich/external/EventRunner.h:25-44`

```cpp
namespace OHOS::AppExecFwk {
class EventRunner final {
public:
    EventRunner() = default;
    ~EventRunner() = default;
    static EventRunner& Current();
    static EventRunner& GetMainEventRunner();
    void SetMainThreadId(std::thread::id id);
    std::thread::id GetThreadId();
    bool IsCurrentRunnerThread();
    void Run();

    void PushTask(const Callback &callback, std::chrono::steady_clock::time_point targetTime);

private:
    EventRunner(const EventRunner&) = delete;
    EventRunner &operator=(const EventRunner&) = delete;
    std::thread::id threadId;
    EventQueue queue;
    std::mutex mutex;
};
}
```

| 方法 | 签名 | 描述 |
|------|------|------|
| `Current` | `static EventRunner& Current()` | 获取当前线程的事件循环 |
| `GetMainEventRunner` | `static EventRunner& GetMainEventRunner()` | 获取主线程事件循环 |
| `SetMainThreadId` | `void SetMainThreadId(std::thread::id id)` | 设置主线程 ID |
| `GetThreadId` | `std::thread::id GetThreadId()` | 获取线程 ID |
| `IsCurrentRunnerThread` | `bool IsCurrentRunnerThread()` | 检查是否运行在事件循环线程 |
| `Run` | `void Run()` | 启动事件循环 |
| `PushTask` | `void PushTask(const Callback &callback, std::chrono::steady_clock::time_point targetTime)` | 推送延迟任务 |

### EventHandler

**文件**: `jsapp/rich/external/EventHandler.h:22-41`

```cpp
namespace OHOS::AppExecFwk {
class EventHandler final {
public:
    EventHandler() = default;
    ~EventHandler() = default;
    static void SetMainThreadId(std::thread::id id);
    static bool IsCurrentRunnerThread();
    /**
     * Post a task.
     *
     * @param callback Task callback.
     * @param delayTime Process the event after 'delayTime' milliseconds.
     */
    static bool PostTask(const Callback &callback, int64_t delayTime = 0);
    static void Run();

private:
    EventHandler(const EventHandler&) = delete;
    EventHandler &operator=(const EventHandler&) = delete;
};
}
```

| 方法 | 签名 | 描述 |
|------|------|------|
| `SetMainThreadId` | `static void SetMainThreadId(std::thread::id id)` | 设置主线程 ID |
| `IsCurrentRunnerThread` | `static bool IsCurrentRunnerThread()` | 检查当前线程 |
| `PostTask` | `static bool PostTask(const Callback &callback, int64_t delayTime = 0)` | 推送任务（支持延迟） |
| `Run` | `static void Run()` | 运行事件处理器 |

### StageContext

**文件**: `jsapp/rich/external/StageContext.h:35-86`

```cpp
namespace OHOS::Ide {
class HspInfo {
public:
    std::string moduleName;
    std::string resourcePath;
    std::vector<uint8_t> moduleJsonBuffer;
};

class StageContext {
public:
    static StageContext& GetInstance();
    const std::optional<std::vector<uint8_t>> ReadFileContents(const std::string& filePath) const;
    void SetLoaderJsonPath(const std::string& assetPath);
    void SetHosSdkPath(const std::string& hosSdkPathValue);
    void GetModulePathMapFromLoaderJson();
    std::string GetHspAceModuleBuild(const std::string& hspConfigPath);
    void ReleaseHspBuffers();
    std::map<std::string, std::string> ParseMockJsonFile(const std::string& mockJsonFilePath);
    std::vector<uint8_t>* GetModuleBuffer(const std::string& inputPath);
    std::vector<uint8_t>* GetLocalModuleBuffer(const std::string& moduleName);
    std::vector<uint8_t>* GetCloudModuleBuffer(const std::string& moduleName);
    std::vector<uint8_t>* GetSystemModuleBuffer(const std::string& inputPath, const std::string& moduleName);
    std::vector<uint8_t>* GetModuleBufferFromHsp(const std::string& hspFilePath, const std::string& fileName);
    void SetPkgContextInfo(std::map<std::string, std::string>& pkgContextInfoJsonStringMap,
        std::map<std::string, std::string>& packageNameList);
    void GetModuleInfo(std::vector<HspInfo>& dependencyHspInfos);
};
}
```

| 方法 | 签名 | 描述 |
|------|------|------|
| `GetInstance` | `static StageContext& GetInstance()` | 获取单例 |
| `ReadFileContents` | `const std::optional<std::vector<uint8_t>> ReadFileContents(const std::string& filePath) const` | 读取文件内容 |
| `SetLoaderJsonPath` | `void SetLoaderJsonPath(const std::string& assetPath)` | 设置 loader.json 路径 |
| `SetHosSdkPath` | `void SetHosSdkPath(const std::string& hosSdkPathValue)` | 设置 SDK 路径 |
| `GetModulePathMapFromLoaderJson` | `void GetModulePathMapFromLoaderJson()` | 解析 loader.json |
| `GetHspAceModuleBuild` | `std::string GetHspAceModuleBuild(const std::string& hspConfigPath)` | 获取 HSP 构建路径 |
| `ReleaseHspBuffers` | `void ReleaseHspBuffers()` | 释放 HSP 缓冲区 |
| `ParseMockJsonFile` | `std::map<std::string, std::string> ParseMockJsonFile(const std::string& mockJsonFilePath)` | 解析 Mock JSON |
| `GetModuleBuffer` | `std::vector<uint8_t>* GetModuleBuffer(const std::string& inputPath)` | 获取模块缓冲区 |
| `GetLocalModuleBuffer` | `std::vector<uint8_t>* GetLocalModuleBuffer(const std::string& moduleName)` | 获取本地 HSP |
| `GetCloudModuleBuffer` | `std::vector<uint8_t>* GetCloudModuleBuffer(const std::string& moduleName)` | 获取云端 HSP |
| `GetSystemModuleBuffer` | `std::vector<uint8_t>* GetSystemModuleBuffer(const std::string& inputPath, const std::string& moduleName)` | 获取系统模块 |
| `GetModuleBufferFromHsp` | `std::vector<uint8_t>* GetModuleBufferFromHsp(const std::string& hspFilePath, const std::string& fileName)` | 从 HSP 提取 |
| `SetPkgContextInfo` | `void SetPkgContextInfo(...)` | 设置包上下文 |
| `GetModuleInfo` | `void GetModuleInfo(std::vector<HspInfo>& dependencyHspInfos)` | 获取模块依赖 |

### JsMockUtil

**文件**: `jsapp/rich/external/JsMockUtil.h:23-32`

```cpp
namespace OHOS::Ide {
class JsMockUtil {
public:
    class AbcInfo {
    public:
        const uint8_t *buffer;
        std::size_t bufferSize;
    };
    static const AbcInfo GetAbcBufferInfo();
};
}
```

| 方法 | 签名 | 描述 |
|------|------|------|
| `GetAbcBufferInfo` | `static const AbcInfo GetAbcBufferInfo()` | 获取 ABC 字节码缓冲区信息 |

---

## 单例模式汇总

| 类 | 获取方法 | 头文件 |
|----|----------|--------|
| `StageContext` | `GetInstance()` | `StageContext.h:37` |
| `CommandParser` | `GetInstance()` | `CommandParser.h:55` |
| `CommandLineInterface` | `GetInstance()` | `CommandLineInterface.h:30` |
| `TraceTool` | `GetInstance()` | `TraceTool.h:29` |
| `WebSocketServer` | `GetInstance()` | `WebSocketServer.h:29` |
| `CppTimerManager` | `GetTimerManager()` | `CppTimerManager.h:34` |
| `JsAppImpl` | `GetInstance()` | `JsAppImpl.h:33/61` |
| `VirtualScreenImpl` | `GetInstance()` | `VirtualScreenImpl.h:31/37` |

---

## 相关文档

- 架构: [02_Architecture.md](./02_Architecture.md)
- 通信协议: [03_Communication_Protocol.md](./03_Communication_Protocol.md)
- 安全评审: [07_Security_Review.md](./07_Security_Review.md)
