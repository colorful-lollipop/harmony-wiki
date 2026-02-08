# 对外接口 (External API)

## 重要发现：无 N-API 接口

**结论**: 本项目为 **纯 Native C++ 应用**，**无 N-API JS 绑定层**。

**证据**:
- 全局搜索 `napi_`、`NAPI_MODULE`、`napi_define_properties` 等符号，**无任何结果**
- 所有代码均为 C++，直接编译为 `.so` 库
- 应用通过 Ability 框架以 Native 方式运行

**影响**:
- 本应用**不暴露 JavaScript API**
- 对外接口仅限于：
  1. **Ability 生命周期接口**（系统框架调用）
  2. **权限声明**（安装时系统读取）
  3. **Want 参数**（应用间跳转时传递）

---

## Ability 生命周期接口

### 标准 Ability 接口

所有应用均实现以下标准 Ability 接口（由系统框架调用）：

```cpp
// Ability 基类接口
class Ability {
public:
    virtual void OnStart(const Want &want);      // Ability 启动
    virtual void OnInactive();                    // 失活（进入后台）
    virtual void OnActive(const Want &want);      // 激活（进入前台）
    virtual void OnBackground();                  // 后台运行
    virtual void OnStop();                        // 停止销毁
};
```

**代码示例** (`cameraApp/cameraApp/src/main/cpp/camera_ability.cpp`):
```cpp
REGISTER_AA(CameraAbility)

void CameraAbility::OnStart(const Want &want) {
    printf("CameraAbility::OnStart\n");
    SetMainRoute("CameraAbilitySlice");
}

void CameraAbility::OnInactive() {
    printf("CameraAbility::OnInactive\n");
}

void CameraAbility::OnActive(const Want &want) {
    printf("CameraAbility::OnActive\n");
}

void CameraAbility::OnBackground() {
    printf("CameraAbility::OnBackground\n");
}

void CameraAbility::OnStop() {
    printf("CameraAbility::OnStop\n");
}
```

### AbilitySlice 生命周期接口

页面切片接口：

```cpp
class AbilitySlice {
public:
    virtual void OnStart(const Want &want);      // 页面创建
    virtual void OnInactive();                    // 页面失活
    virtual void OnActive(const Want &want);      // 页面激活
    virtual void OnBackground();                  // 页面后台
    virtual void OnStop();                        // 页面销毁
    void SetUIContent(UIView* view);              // 设置 UI 内容
};
```

**代码示例** (`cameraApp/cameraApp/src/main/cpp/camera_ability_slice.cpp`):
```cpp
REGISTER_AS(CameraAbilitySlice)

void CameraAbilitySlice::OnStart(const Want &want) {
    AbilitySlice::OnStart(want);
    cam_manager = new SampleCameraManager();
    cam_manager->SampleCameraCreate();
    // ... UI 初始化
}

void CameraAbilitySlice::OnActive(const Want &want) {
    AbilitySlice::OnActive(want);
    cam_manager->SampleCameraStart(surfaceView->GetSurface());
}

void CameraAbilitySlice::OnBackground() {
    AbilitySlice::OnBackground();
    cam_manager->SampleCameraStop();
}

void CameraAbilitySlice::OnStop() {
    AbilitySlice::OnStop();
    // 清理资源
}
```

---

## 权限声明接口

权限在 `config.json` 中声明，安装时由系统读取。

### cameraApp 权限声明

**文件**: `cameraApp/cameraApp/src/main/config.json:46-97`

```json
{
  "module": {
    "reqPermissions": [
      {
        "name": "ohos.permission.CAMERA",
        "reason": "USER_GRANT",
        "usedScene": {
          "ability": [".FormAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.MODIFY_AUDIO_SETTINGS",
        "reason": "SYSTEM_GRANT",
        "usedScene": {
          "ability": [".FormAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.READ_MEDIA",
        "reason": "USER_GRANT",
        "usedScene": {
          "ability": [".FormAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.MICROPHONE",
        "reason": "USER_GRANT",
        "usedScene": {
          "ability": [".FormAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.WRITE_MEDIA",
        "reason": "USER_GRANT",
        "usedScene": {
          "ability": [".FormAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

### 权限级别说明

| 权限 | 级别 | 授权方式 | 用途 |
|------|------|----------|------|
| `ohos.permission.CAMERA` | normal | user_grant | 访问相机硬件 |
| `ohos.permission.MICROPHONE` | normal | user_grant | 访问麦克风 |
| `ohos.permission.READ_MEDIA` | normal | user_grant | 读取媒体文件 |
| `ohos.permission.WRITE_MEDIA` | normal | user_grant | 写入媒体文件 |
| `ohos.permission.MODIFY_AUDIO_SETTINGS` | normal | system_grant | 修改音频设置 |

**注意**: 本应用**无权限校验代码**，依赖系统框架在调用底层服务时进行权限检查。

---

## Want 参数接口

Want 用于 Ability 间通信和参数传递。

### 典型使用场景

**1. 应用内页面跳转**
```cpp
// gallery -> picture view
void GalleryAbilitySlice::OnClick(UIView& view) {
    Want want;
    want.data = imageName;  // 图片文件名
    want.dataLength = strlen(imageName);
    StartAbility(want);
}
```

**2. launcher 启动其他应用**
```cpp
// launcher/app_manage.cpp
void AppManage::StartAbility(const char* bundleName, const char* abilityName) {
    Want want;
    want.element->SetBundleName(bundleName);
    want.element->SetAbilityName(abilityName);
    StartAbility(want);
}
```

**3. setting 显示应用详情**
```cpp
// setting/app_info_ability_slice.cpp:145
memcpy_s(bundleName_, sizeof(bundleName_), want.data, want.dataLength);
```

### Want 结构

```cpp
struct Want {
    ElementName* element;    // 目标组件名
    void* data;              // 自定义数据指针
    int dataLength;          // 数据长度
    // ... 其他字段
};
```

---

## 系统服务调用接口

本应用通过 SDK 间接调用系统服务，主要依赖以下接口：

### CameraKit 接口

```cpp
// 来自 camera_lite 库
class CameraKit {
public:
    static CameraKit* GetInstance();                    // 获取单例
    list<string> GetCameraIds();                        // 获取相机列表
    void CreateCamera(string &cameraId, 
                      CameraStateCallback &callback, 
                      EventHandler &handler);           // 创建相机实例
};
```

**代码位置**: `cameraApp/cameraApp/src/main/cpp/camera_manager.cpp:605-632`

### Recorder 接口

```cpp
// 来自 recorder_lite 库
class Recorder {
public:
    int SetVideoSource(VideoSourceType source);         // 设置视频源
    int SetOutputPath(const string &path);              // 设置输出路径
    int Prepare();                                       // 准备录制
    int Start();                                         // 开始录制
    int Stop();                                          // 停止录制
    int Release();                                       // 释放资源
};
```

**代码位置**: `cameraApp/cameraApp/src/main/cpp/camera_manager.cpp:434-491`

### Player 接口

```cpp
// 来自 player_lite 库
class Player {
public:
    int SetSource(const string &path);                  // 设置媒体源
    int Prepare();                                       // 准备播放
    int Start();                                         // 开始播放
    int Pause();                                         // 暂停
    int Stop();                                          // 停止
    int Release();                                       // 释放资源
};
```

**代码位置**: `gallery/src/player_ability_slice.cpp`

---

## 接口清单表

| 接口类型 | 接口名称 | 所属模块 | 调用方 | 实现位置 |
|----------|----------|----------|--------|----------|
| Ability | OnStart/OnStop | cameraApp | 系统框架 | camera_ability.cpp |
| Ability | OnStart/OnStop | gallery | 系统框架 | gallery_ability.cpp |
| Ability | OnStart/OnStop | launcher | 系统框架 | main_ability.cpp |
| Ability | OnStart/OnStop | setting | 系统框架 | setting_main_ability.cpp |
| AbilitySlice | OnStart/OnActive/OnBackground/OnStop | 所有模块 | 系统框架 | *_ability_slice.cpp |
| Want 参数 | data/dataLength | cameraApp/gallery/launcher/setting | 应用间 | config.json |
| 权限 | ohos.permission.CAMERA | cameraApp | 系统 | config.json |
| 权限 | ohos.permission.MICROPHONE | cameraApp | 系统 | config.json |
| 权限 | ohos.permission.READ_MEDIA | cameraApp | 系统 | config.json |
| 权限 | ohos.permission.WRITE_MEDIA | cameraApp | 系统 | config.json |

---

## 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| Ability 生命周期 | **稳定** | OpenHarmony 核心 API |
| Want 参数传递 | **稳定** | OpenHarmony 核心 API |
| CameraKit | **稳定** | 多媒体子系统公开 API |
| Recorder/Player | **稳定** | 多媒体子系统公开 API |
| Permission API | **稳定** | 安全子系统公开 API |

---

## 相关链接

- [架构说明](./Architecture.md) - 接口在架构中的位置
- [内部接口](./Internal_API.md) - 模块内部接口
- [安全风险分析](./Security.md) - 接口安全考虑
