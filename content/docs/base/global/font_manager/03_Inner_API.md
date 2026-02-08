# 内部 API

## 概述

本章节描述 font_manager 的内部 API，主要供模块内部调用，不对外暴露给应用层。

## FontManagerKits

**定义位置**: `service/inner_api/include/font_manager_kits.h`

**用途**: 定义字体管理的抽象接口，客户端和服务端实现此接口。

```cpp
namespace OHOS {
namespace Global {
namespace FontManager {
class FontManagerKits {
public:
    DISALLOW_COPY_AND_MOVE(FontManagerKits);
    virtual ~FontManagerKits() = default;
    static FontManagerKits& GetInstance();

    virtual int32_t InstallFont(const std::string &fontPath, int &outValue) = 0;
    virtual int32_t UninstallFont(const std::string &fontName, int &outValue) = 0;
    virtual int32_t DataMigration(std::shared_ptr<IDataMigrationListener> listener) = 0;

protected:
    FontManagerKits() = default;
};
} // namespace FontManager
} // namespace Global
} // namespace OHOS
```

### 接口说明

| 方法 | 说明 | 参数 | 返回值 |
|------|------|------|--------|
| InstallFont | 安装字体 | fontPath: 字体路径, outValue: 输出错误码 | ERR_OK 或错误码 |
| UninstallFont | 卸载字体 | fontName: 字体名称, outValue: 输出错误码 | ERR_OK 或错误码 |
| DataMigration | 数据迁移 | listener: 迁移监听器 | ERR_OK 或错误码 |

## FontManagerClient

**定义位置**: `service/client/include/font_manager_client.h`

**用途**: 客户端单例，封装 IPC 调用逻辑。

```cpp
class FontManagerClient : public FontManagerKits, public DelayedSingleton<FontManagerClient> {
public:
    int32_t InstallFont(const std::string &fontPath, int &outValue) override;
    int32_t UninstallFont(const std::string &fontName, int &outValue) override;
    int32_t DataMigration(std::shared_ptr<IDataMigrationListener> listener) override;

private:
    bool PathToRealPath(const std::string& path, std::string& realPath);
};
```

### 关键实现细节

#### InstallFont 实现

```cpp
// service/client/src/font_manager_client.cpp:30-61
int32_t FontManagerClient::InstallFont(const std::string &fontPath, int &outValue)
{
    // 1. 路径校验并转换为真实路径
    std::string realPath;
    if (!PathToRealPath(fontPath, realPath)) {
        outValue = ERR_FILE_NOT_EXISTS;
        return ERR_OK;
    }
    
    // 2. 打开文件获取 fd
    FILE* fp = fopen(realPath.c_str(), "rb");
    if (!fp) {
        outValue = ERR_FILE_NOT_EXISTS;
        return ERR_OK;
    }
    int fd = fileno(fp);
    
    // 3. 获取 SA 代理并调用
    sptr<IFontService> service = FontServiceLoadManager::GetInstance()->GetFontServiceAbility(FONT_SA_ID);
    int32_t ret = service->InstallFont(fd, outValue);
    
    (void)fclose(fp);
    return ret;
}
```

#### 路径校验

```cpp
// service/client/src/font_manager_client.cpp:94-118
bool FontManagerClient::PathToRealPath(const std::string& path, std::string& realPath)
{
    // 1. 空路径检查
    if (path.empty()) {
        return false;
    }
    
    // 2. 路径长度检查
    if (path.length() >= PATH_MAX) {
        return false;
    }
    
    // 3. 转换为真实路径
    char tmpPath[PATH_MAX] = {0};
    if (realpath(path.c_str(), tmpPath) == nullptr) {
        return false;
    }
    
    // 4. 检查文件是否存在
    if (access(realPath.c_str(), F_OK) != 0) {
        return false;
    }
    return true;
}
```

## FontManagerServer

**定义位置**: `service/server/include/font_manager_server.h`

**用途**: SystemAbility 服务端实现，处理字体安装卸载的实际逻辑。

```cpp
class FontManagerServer : public SystemAbility, public FontServiceStub {
public:
    FontManagerServer(int32_t saId, bool runOnCreate);
    ~FontManagerServer() override = default;

    int32_t InstallFont(const int32_t fd, int32_t &outValue) override;
    int32_t UninstallFont(const std::string &fontName, int32_t &outValue) override;
    int32_t DataMigration(const sptr<IDataMigrationCallback>& callback) override;

protected:
    void OnStart(const SystemAbilityOnDemandReason &startReason) override;
    void OnStop(const SystemAbilityOnDemandReason &startReason) override;

private:
    int32_t CheckPermission();  // 权限校验
    void InstallFontInner(const int32_t fd, int32_t &outValue);
    void UninstallFontInner(const std::string &fontName, int32_t &outValue);
    int32_t DataMigrationInner(const sptr<IDataMigrationCallback>& callback);
    // ... 其他方法
};
```

### 权限校验

```cpp
// service/server/src/font_manager_server.cpp:213-223
int32_t FontManagerServer::CheckPermission()
{
    uint32_t callerToken = IPCSkeleton::GetCallingTokenID();
    int result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, PERMISSION_UPDATE_FONT);
    if (result != Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        FONT_LOGE("FontManagerServer caller process doesn't have permission.");
        return ERR_NO_PERMISSION;
    }
    return ERR_OK;
}
```

### SA 生命周期管理

```cpp
// service/server/src/font_manager_server.cpp:194-206
void FontManagerServer::OnStart(const SystemAbilityOnDemandReason &startReason)
{
    // 1. 创建事件处理器
    handler_ = std::make_shared<AppExecFwk::EventHandler>(
        AppExecFwk::EventRunner::Create(true));
    
    // 2. 添加延迟卸载任务（空闲 10 秒后卸载）
    AddUnloadFontServiceTask();
    
    // 3. 发布服务
    bool status = Publish(this);
}
```

## FontManager (Core)

**定义位置**: `frameworks/fontmgr/include/font_manager.h`

**用途**: 字体安装卸载的核心业务逻辑。

```cpp
class FontManager : public DelayedSingleton<FontManager> {
public:
    int32_t InstallFont(const int32_t &fd, const int32_t userId);
    int32_t UninstallFont(const std::string &fontFullName, const int32_t userId);

private:
    std::string GetFormatFullName(const std::vector<std::string> &fullNameVector);
    std::string CopyFileForInstall(const std::string &installPath, const std::string &fileName, const int32_t &fd);
    std::string SandBoxPathToRealPath(const std::string &installPath, const std::string &path);
    FontConfig& SafeGetOrCreateConfig(int32_t userId, const std::string& configPath);
    std::unordered_map<int32_t, FontConfig> configMap_;
    std::mutex mapLock_;
};
```

### InstallFont 核心逻辑

```cpp
// frameworks/fontmgr/src/font_manager.cpp:39-85
int32_t FontManager::InstallFont(const int32_t &fd, const int32_t userId)
{
    // 1. 确定安装路径
    std::string installPath = INSTALL_PATH_PREFIX + std::to_string(userId) + INSTALL_PATH_SUFFIX;
    
    // 2. 初始化安装路径和配置
    FontManagerUtils::CheckAndInitInstallPath(installPath);
    auto& fontConfig = SafeGetOrCreateConfig(userId, installPath + FONT_CONFIG_FILE);
    
    // 3. 获取字体全名
    std::vector<std::string> fullNameVector = FontManagerUtils::GetFullNamesByFd(fd);
    if (fullNameVector.size() == 0) {
        return ERR_FILE_VERIFY_FAIL;
    }
    
    // 4. 检查是否已安装
    for (const auto &fullName : fullNameVector) {
        if (fontConfig.GetFontFileByName(fullName)) {
            return ERR_INSTALLED_ALRADY;
        }
    }
    
    // 5. 检查安装数量限制
    if (fontConfig.GetInstalledFontsNum() >= MAX_INSTALL_NUM) {
        return ERR_MAX_FILE_COUNT;
    }
    
    // 6. 复制文件
    std::string destPath = CopyFileForInstall(installPath, fileName, fd);
    if (destPath.empty()) {
        return ERR_COPY_FAIL;
    }
    
    // 7. 更新配置
    fontConfig.InsertFontRecord(INSTALL_PATH_APP + realFileName, fullNameVector);
    
    // 8. 发布事件
    FontEventPublish::PublishFontUpdate(FontEventType::INSTALL, fullName, userId);
    
    return ERR_OK;
}
```

## FontConfig

**定义位置**: `frameworks/fontmgr/include/font_config.h`

**用途**: 管理字体配置文件 `install_fontconfig.json`。

### 配置格式

```json
{
    "fonts": [
        {
            "path": "/data/service/el1/100/for-all-app/fonts/myfont.ttf",
            "fullName": ["MyFont", "MyFont-Bold"]
        }
    ]
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| CheckAndUpdateFontRecord | 检查并更新字体记录 |
| GetFontFileByName | 根据字体全名获取路径 |
| InsertFontRecord | 插入字体记录 |
| DeleteFontRecord | 删除字体记录 |
| GetInstalledFontsNum | 获取已安装字体数量 |

## 依赖方向

```
┌─────────────────────┐
│   FontManagerAddon  │  (N-API 层)
└──────────┬──────────┘
           │ 调用
           ↓
┌─────────────────────┐
│  FontManagerClient  │  (客户端层)
└──────────┬──────────┘
           │ IPC 调用
           ↓
┌─────────────────────┐
│  FontManagerServer  │  (服务端层)
└──────────┬──────────┘
           │ 调用
           ↓
┌─────────────────────┐
│    FontManager      │  (核心框架层)
└──────────┬──────────┘
           │
    ┌──────┼──────┐
    ↓      ↓      ↓
FontConfig  Utils  Events
```

## 相关文档

- [N-API 参考](02_NAPI_Reference.md)
- [架构说明](01_Architecture.md)
- [构建目标](04_Build_Targets.md)
