# 内部接口参考

> Native 内部 API 与模块接口

## WallpaperManager

**头文件**: `frameworks/native/include/wallpaper_manager.h`

### 单例获取

```cpp
static WallpaperManager &WallpaperManager::GetInstance()
```

**代码位置**: `wallpaper_manager.cpp:72-76`

### 主要方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `SetWallpaper` | `ErrorCode SetWallpaper(std::string uri, int32_t wallpaperType, const ApiInfo &apiInfo)` | URI 设置壁纸 |
| `SetWallpaper` | `ErrorCode SetWallpaper(std::shared_ptr<OHOS::Media::PixelMap> pixelMap, int32_t wallpaperType, const ApiInfo &apiInfo)` | PixelMap 设置 |
| `GetPixelMap` | `ErrorCode GetPixelMap(int32_t wallpaperType, const ApiInfo &apiInfo, std::shared_ptr<OHOS::Media::PixelMap> &PixelMap)` | 获取壁纸图片 |
| `GetColors` | `ErrorCode GetColors(int32_t wallpaperType, const ApiInfo &apiInfo, std::vector<uint64_t> &colors)` | 获取颜色 |
| `GetWallpaperId` | `int32_t GetWallpaperId(int32_t wallpaperType)` | 获取壁纸 ID |
| `GetFile` | `ErrorCode GetFile(int32_t wallpaperType, int32_t &wallpaperFd)` | 获取文件描述符 |
| `GetWallpaperMinHeight` | `ErrorCode GetWallpaperMinHeight(const ApiInfo &apiInfo, int32_t &minHeight)` | 最小高度 |
| `GetWallpaperMinWidth` | `ErrorCode GetWallpaperMinWidth(const ApiInfo &apiInfo, int32_t &minWidth)` | 最小宽度 |
| `IsChangePermitted` | `bool IsChangePermitted()` | 是否允许修改 |
| `IsOperationAllowed` | `bool IsOperationAllowed()` | 是否允许操作 |
| `ResetWallpaper` | `ErrorCode ResetWallpaper(std::int32_t wallpaperType, const ApiInfo &apiInfo)` | 重置壁纸 |
| `On` | `ErrorCode On(const std::string &type, std::shared_ptr<WallpaperEventListener> listener)` | 订阅事件 |
| `Off` | `ErrorCode Off(const std::string &type, std::shared_ptr<WallpaperEventListener> listener)` | 取消订阅 |
| `SetVideo` | `ErrorCode SetVideo(const std::string &uri, const int32_t wallpaperType)` | 设置视频壁纸 |
| `SetCustomWallpaper` | `ErrorCode SetCustomWallpaper(const std::string &uri, int32_t wallpaperType)` | 自定义壁纸 |
| `SendEvent` | `ErrorCode SendEvent(const std::string &eventType)` | 发送事件 |
| `RegisterWallpaperCallback` | `bool RegisterWallpaperCallback(JScallback callback)` | 注册回调 |

### 内部成员

| 成员 | 类型 | 说明 |
|------|------|------|
| `wallpaperProxy_` | `sptr<IWallpaperService>` | IPC 代理 |
| `wallpaperFdMap_` | `std::map<int32_t, int32_t>` | FD 映射 |
| `listenerMap_` | `std::map<std::string, sptr<WallpaperEventListenerClient>>` | 监听器映射 |
| `wallpaperProxyLock_` | `std::mutex` | 代理锁 |
| `wallpaperFdLock_` | `std::mutex` | FD 锁 |

### 私有方法

| 方法 | 说明 |
|------|------|
| `GetService()` | 获取 IPC 服务代理 |
| `CallService()` | 模板方法，调用服务 |
| `CheckVideoFormat()` | 检查视频格式 |
| `ResetService()` | 重置服务连接 |
| `CreatePixelMapByFd()` | 通过 FD 创建 PixelMap |
| `GetFdByPath()` | 获取路径 FD |
| `CheckWallpaperFormat()` | 检查壁纸格式 |
| `IsDefaultWallpaperResource()` | 检查默认壁纸资源 |

---

## IWallpaperService

**头文件**: `frameworks/native/include/iwallpaper_service.h` (IDL 生成)

### 接口方法

| 方法 | 说明 |
|------|------|
| `SetWallpaper(int fd, int32_t wallpaperType, int32_t length)` | 设置壁纸 |
| `SetAllWallpapers(...)` | 设置多态壁纸 |
| `SetWallpaperByPixelMap(...)` | PixelMap 设置 |
| `GetPixelMap(int32_t wallpaperType, int32_t &size, int &fd)` | 获取壁纸 |
| `GetCorrespondWallpaper(...)` | 获取对应壁纸 |
| `GetColors(int32_t wallpaperType, std::vector<uint64_t> &colors)` | 获取颜色 |
| `GetFile(int32_t wallpaperType, int &wallpaperFd)` | 获取文件描述符 |
| `GetWallpaperId(int32_t wallpaperType)` | 获取 ID |
| `IsChangePermitted(bool &isChangePermitted)` | 是否允许修改 |
| `IsOperationAllowed(bool &isOperationAllowed)` | 是否允许操作 |
| `ResetWallpaper(int32_t wallpaperType)` | 重置壁纸 |
| `On(const std::string &type, const sptr<IWallpaperEventListener> &listener)` | 订阅事件 |
| `Off(const std::string &type, const sptr<IWallpaperEventListener> &listener)` | 取消订阅 |
| `SetVideo(int fd, int32_t wallpaperType, int32_t length)` | 设置视频 |
| `SetCustomWallpaper(int fd, int32_t wallpaperType, int32_t length)` | 自定义设置 |
| `SendEvent(const std::string &eventType)` | 发送事件 |
| `IsDefaultWallpaperResource(...)` | 检查默认资源 |

---

## WallpaperEventListener

**头文件**: `frameworks/native/include/wallpaper_event_listener.h`

### 接口方法

```cpp
virtual void OnColorsChanged(const std::vector<uint64_t> &colors, WallpaperType wallpaperType) = 0;
virtual void OnWallpaperChanged(WallpaperType wallpaperType) = 0;
```

---

## WallpaperService

**头文件**: `services/include/wallpaper_service.h`

### 公共方法

| 方法 | 说明 |
|------|------|
| `SetWallpaper(int fd, int32_t wallpaperType, int32_t length)` | 设置壁纸 |
| `GetPixelMap(int32_t wallpaperType, int32_t &size, int &fd)` | 获取壁纸 |
| `GetColors(int32_t wallpaperType, std::vector<uint64_t> &colors)` | 获取颜色 |
| `IsChangePermitted(bool &isChangePermitted)` | 权限检查 |
| `IsOperationAllowed(bool &isOperationAllowed)` | 操作检查 |
| `On/Off` | 事件订阅 |

### 生命周期

| 方法 | 说明 |
|------|------|
| `OnStart()` | 服务启动 |
| `OnStop()` | 服务停止 |
| `OnAddSystemAbility(int32_t systemAbilityId, const std::string &deviceId)` | SA 添加 |

### 内部方法

| 方法 | 说明 |
|------|------|
| `InitData()` | 初始化数据 |
| `InitUsersOnBoot()` | 初始化用户 |
| `CheckCallingPermission()` | 权限检查 |
| `IsSystemApp()` | 系统应用检查 |
| `IsNativeSa()` | Native SA 检查 |
| `CheckUserPermissionById()` | 用户权限检查 |
| `SaveColor()` | 保存颜色 |
| `GetWallpaperDir()` | 获取壁纸目录 |
| `WritePixelMapToFile()` | 写入 PixelMap |

---

## 错误码

**头文件**: `utils/include/wallpaper_common.h`

```cpp
enum ErrorCode : int32_t {
    NO_ERROR = 0,
    E_OK = WALLPAPER_ERR_OFFSET,
    E_ERROR = 1,
    E_NOT_SYSTEM_APP,
    E_PARAMETERS_INVALID,
    E_PERMISSION_DENIED,
    E_DEAD_OBJECT,
    E_NO_PERMISSION,
    E_USER_IDENTITY_ERROR,
    E_CHECK_DESCRIPTOR_ERROR,
    E_PICTURE_OVERSIZED,
    E_GET_WALLPAPER_FAILED,
    E_SET_WALLPAPER_FAILED,
    E_IMAGE_ENCODE_FAILED,
    E_ADAPTER_FAIL,
    E_INIT_FAIL,
    E_ALREADY_EXIST,
    E_NOT_CONTENT,
    E_URI_ERROR,
    E_FILE_OEPN_FAIL,
    E_IPC_ERROR,
    E_LOAD_FAIL,
    E_WAIT_ASYNC_RESULT_FAIL,
};
```

---

## 常量定义

**头文件**: `utils/include/wallpaper_manager_common_info.h`

```cpp
enum WallpaperType {
    WALLPAPER_SYSTEM = 0,
    WALLPAPER_LOCKSCREEN = 1
};

enum WallpaperResourceType {
    DEFAULT = 0,
    PICTURE = 1,
    VIDEO = 2,
    PACKAGE = 3
};

enum FoldState {
    NORMAL = 0,
    UNFOLD_1 = 1,
    UNFOLD_2 = 2
};

enum RotateState {
    PORT = 0,
    LAND = 1
};
```

---

## 使用示例

### Native 应用调用示例

```cpp
#include "wallpaper_manager.h"

using namespace OHOS::WallpaperMgrService;

void SetWallpaperExample()
{
    auto &manager = WallpaperManager::GetInstance();
    
    // 设置壁纸
    ErrorCode ret = manager.SetWallpaper("/data/test.jpg", WALLPAPER_SYSTEM, {});
    if (ret != E_OK) {
        HILOG_ERROR("Set wallpaper failed: %{public}d", ret);
        return;
    }
    
    // 获取壁纸
    std::shared_ptr<OHOS::Media::PixelMap> pixelMap;
    ret = manager.GetPixelMap(WALLPAPER_SYSTEM, {}, pixelMap);
    if (ret != E_OK) {
        HILOG_ERROR("Get pixel map failed: %{public}d", ret);
        return;
    }
    
    // 获取颜色
    std::vector<uint64_t> colors;
    ret = manager.GetColors(WALLPAPER_SYSTEM, {}, colors);
    if (ret == E_OK) {
        HILOG_INFO("Colors size: %{public}zu", colors.size());
    }
}
```

### 事件监听示例

```cpp
#include "wallpaper_event_listener.h"

class WallpaperListener : public WallpaperEventListener {
public:
    void OnColorsChanged(const std::vector<uint64_t> &colors, WallpaperType wallpaperType) override {
        HILOG_INFO("Colors changed, type: %{public}d", wallpaperType);
    }
    
    void OnWallpaperChanged(WallpaperType wallpaperType) override {
        HILOG_INFO("Wallpaper changed, type: %{public}d", wallpaperType);
    }
};

void RegisterListenerExample()
{
    auto &manager = WallpaperManager::GetInstance();
    auto listener = std::make_shared<WallpaperListener>();
    
    ErrorCode ret = manager.On("colorChange", listener);
    if (ret != E_OK) {
        HILOG_ERROR("Register listener failed: %{public}d", ret);
    }
}
```

---

## 相关文档

- API 参考: [03_API.md](03_API.md)
- 架构设计: [04_Architecture.md](04_Architecture.md)
- 安全评估: [06_Security.md](06_Security.md)
