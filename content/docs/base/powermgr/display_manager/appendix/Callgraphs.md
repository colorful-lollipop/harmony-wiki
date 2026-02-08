# 关键调用链 - display_manager

> 本文档记录 display_manager 模块的关键调用链，用于调试和性能分析

---

## 调用链说明

每个调用链包含：
- **入口**：调用起始点
- **路径**：函数调用序列
- **出口**：最终结果或硬件操作
- **关键文件**：涉及的源文件

---

## 1. 设置亮度调用链

### 完整路径

```
JS: brightness.setValue(128)
  └─▶ N-API: frameworks/napi/brightness.cpp:81
    └─▶ Brightness::SetValue()
      └─▶ Brightness::BrightnessInfo::SetBrightness()
        └─▶ Client: interfaces/inner_api/native/src/display_power_mgr_client.cpp:164
          └─▶ DisplayPowerMgrClient::SetBrightness(128, 0, false)
            └─▶ GetProxy()
              └─▶ IPC: IDisplayPowerMgr.idl:SetBrightness
                └─▶ Service: service/native/src/display_power_mgr_service.cpp:860
                  └─▶ DisplayPowerMgrService::SetBrightness()
                    └─▶ Permission::IsSystem() ✓
                      └─▶ SetBrightnessInner()
                        └─▶ GetSafeBrightness(128)
                          └─▶ BrightnessManager: brightness_manager/src/brightness_manager.cpp:93
                            └─▶ BrightnessManager::SetBrightness()
                              └─▶ BrightnessService: brightness_manager/src/brightness_service.cpp:712
                                └─▶ BrightnessService::SetBrightness()
                                  └─▶ CanSetBrightness() ✓
                                    └─▶ UpdateBrightness()
                                      └─▶ BrightnessAction: brightness_manager/src/brightness_action.cpp:77
                                        └─▶ BrightnessAction::SetBrightness()
                                          └─▶ Rosen::DisplayManagerLite::GetInstance().SetScreenBrightness()
                                            └─▶ Window Manager / HAL
```

### 关键节点

| 节点 | 文件 | 行号 | 功能 |
|------|------|------|------|
| N-API 入口 | brightness.cpp | 81 | JS 到 Native 转换 |
| Client 调用 | display_power_mgr_client.cpp | 164 | 获取 proxy 并调用 |
| IPC 传输 | IDisplayPowerMgr.idl | 28 | 跨进程通信 |
| Service 入口 | display_power_mgr_service.cpp | 860 | 服务端处理 |
| 权限检查 | display_power_mgr_service.cpp | 337 | IsSystem() 检查 |
| BrightnessManager | brightness_manager.cpp | 93 | 全局亮度管理 |
| BrightnessService | brightness_service.cpp | 712 | 亮度服务逻辑 |
| 硬件操作 | brightness_action.cpp | 77 | 设置硬件亮度 |

---

## 2. 获取亮度调用链

### 完整路径

```
JS: brightness.getValue()
  └─▶ N-API: frameworks/napi/brightness.cpp:70
    └─▶ Brightness::GetValue()
      └─▶ Brightness::BrightnessInfo::GetBrightness()
        └─▶ Client: interfaces/inner_api/native/src/display_power_mgr_client.cpp:231
          └─▶ DisplayPowerMgrClient::GetBrightness(0)
            └─▶ IPC: IDisplayPowerMgr.idl:GetBrightness
              └─▶ Service: service/native/src/display_power_mgr_service.cpp:898
                └─▶ DisplayPowerMgrService::GetBrightness()
                  └─▶ GetBrightnessInner()
                    └─▶ BrightnessManager: brightness_manager/src/brightness_manager.cpp:183
                      └─▶ BrightnessManager::GetBrightness()
                        └─▶ BrightnessService: brightness_manager/src/brightness_service.cpp:781
                          └─▶ BrightnessService::GetBrightness()
                            └─▶ GetSettingBrightness()
                              └─▶ BrightnessSettingHelper: 设置存储读取
```

---

## 3. 自动亮度调用链

### 传感器数据流

```
Sensor HAL
  └─▶ SensorEventCallback
    └─▶ BrightnessService: brightness_manager/src/brightness_service.cpp:474
      └─▶ BrightnessService::AmbientLightCallback()
        └─▶ ProcessLightLux()
          └─▶ LightLuxManager: brightness_manager/src/light_lux_manager.cpp:312
            └─▶ LightLuxManager::IsNeedUpdateBrightness()
              └─▶ UpdateLuxBuffer()
                ├─▶ CalcSmoothLux() - 加权平均滤波
                ├─▶ GetNextBrightenTime() - 亮屏防抖
                └─▶ GetNextDarkenTime() - 暗屏防抖
          └─▶ UpdateCurrentBrightnessLevel()
            └─▶ BrightnessCalculationManager: brightness_manager/src/calculation_manager.cpp
              └─▶ GetInterpolatedValue() - 曲线插值计算
            └─▶ SetBrightnessLevel()
              └─▶ UpdateBrightness()
                └─▶ ScreenController: service/native/src/screen_controller.cpp
                  └─▶ ScreenController::SetBrightness()
                    └─▶ 硬件亮度设置
```

### 自动亮度开关

```
JS: brightness.setMode(1)  // 自动亮度开启
  └─▶ N-API: frameworks/napi/brightness.cpp:139
    └─▶ Brightness::SetMode()
      └─▶ Client: display_power_mgr_client.cpp
        └─▶ DisplayPowerMgrClient::AutoAdjustBrightness(true)
          └─▶ IPC: IDisplayPowerMgr.idl:AutoAdjustBrightness
            └─▶ Service: display_power_mgr_service.cpp:463
              └─▶ DisplayPowerMgrService::AutoAdjustBrightnessInner(true)
                └─▶ BrightnessManager: brightness_manager.cpp
                  └─▶ BrightnessManager::AutoAdjustBrightness(true)
                    └─▶ LightLuxManager: 开始监听传感器
```

---

## 4. 显示状态变更调用链

### 设置显示状态

```
WindowManager / PowerManager
  └─▶ Client: display_power_mgr_client.cpp:37
    └─▶ DisplayPowerMgrClient::SetScreenDisplayState()
      └─▶ IPC: IDisplayPowerMgr.idl:SetScreenDisplayState
        └─▶ Service: display_power_mgr_service.cpp
          └─▶ DisplayPowerMgrService::SetScreenDisplayState()
            └─▶ Permission::IsSystem() ✓
              └─▶ ScreenController: service/native/src/screen_controller.cpp
                └─▶ ScreenController::UpdateState()
                  ├─▶ action_->SetDisplayState() - 硬件状态
                  ├─▶ NotifyStateChange() - 通知回调
                  └─▶ DisplayPowerMgrService::NotifyStateChangeCallback()
                    └─▶ IDisplayPowerCallback::OnDisplayStateChanged()
                      └─▶ 注册的应用回调
```

---

## 5. 回调注册调用链

### 注册显示状态回调

```
Native App
  └─▶ Client: display_power_mgr_client.cpp
    └─▶ DisplayPowerMgrClient::RegisterCallback(callback)
      └─▶ IPC: IDisplayPowerMgr.idl:RegisterCallback
        └─▶ Service: display_power_mgr_service.cpp:472
          └─▶ DisplayPowerMgrService::RegisterCallbackInner()
            ├─▶ Permission::IsSystem() ✓
            ├─▶ callback_ = callback
            └─▶ AddDeathRecipient(cbDeathRecipient_)
              └─▶ 监听客户端进程死亡
```

### 注册数据监听

```
Native App (APS / UI)
  └─▶ Client: display_power_mgr_client.cpp:69
    └─▶ DisplayPowerMgrClient::RegisterDataChangeListener()
      └─▶ IPC: IDisplayPowerMgr.idl:RegisterDataChangeListener
        └─▶ Service: display_power_mgr_service.cpp:1059
          └─▶ RegisterDataChangeListener()
            ├─▶ Permission::IsSystem() ✓
            ├─▶ 验证 callerId 长度
            ├─▶ 验证 params 长度
            └─▶ BrightnessManager::RegisterDataChangeListener()
              └─▶ 添加到 listener map
```

---

## 6. 服务生命周期调用链

### 服务启动

```
SystemAbilityManager::OnStart()
  └─▶ DisplaySystemAbility: service/native/src/display_system_ability.cpp
    └─▶ DisplaySystemAbility::OnStart()
      └─▶ DisplayPowerMgrService::GetInstance()
        └─▶ DelayedSpSingleton 创建单例
      └─▶ DisplayPowerMgrService::Init()
        ├─▶ CreateScreenController() - 创建屏幕控制器
        ├─▶ BrightnessManager::Init() - 初始化亮度管理
        ├─▶ RegisterBootCompletedCallback() - 注册启动完成回调
        └─▶ RegisterSettingObservers() - 注册设置观察
```

### 服务停止

```
SystemAbilityManager::OnStop()
  └─▶ DisplaySystemAbility::OnStop()
    └─▶ DisplayPowerMgrService::Deinit()
      ├─▶ BrightnessManager::DeInit()
      ├─▶ UnregisterSettingObservers()
      └─▶ 清理 ScreenController
```

---

## 7. 渐变动画调用链

### 启动渐变

```
SetBrightness(targetValue, gradualDuration)
  └─▶ ScreenController: service/native/src/screen_controller.cpp
    └─▶ ScreenController::SetBrightness(value, gradualDuration)
      └─▶ animator_->StartAnimation()
        └─▶ GradualAnimator: service/native/src/gradual_animator.cpp
          └─▶ GradualAnimator::StartAnimation()
            ├─▶ 计算动画参数
            ├─▶ PostAnimationTask() - 投递动画任务
            └─▶ 循环：OnAnimationCallback()
              ├─▶ 计算插值亮度
              ├─▶ AnimateCallback::OnChanged()
              │   └─▶ ScreenController::OnBrightnessChanged()
              │     └─▶ action_->SetBrightness()
              └─▶ 继续或结束动画
```

---

## 性能热点

### 高频调用路径

| 调用链 | 频率 | 优化建议 |
|--------|------|----------|
| 传感器回调 → 亮度计算 | 高频（传感器采样率） | 批量处理，防抖 |
| 渐变动画回调 | 60Hz（每帧） | 减少锁竞争 |
| 亮度设置 | 用户操作 | 使用连续模式 |

### 耗时操作

| 操作 | 耗时 | 位置 |
|------|------|------|
| IPC 调用 | ~1-5ms | Client ↔ Service |
| 亮度曲线计算 | ~0.1ms | CalculationManager |
| 硬件亮度设置 | ~5-20ms | HAL |
| 渐变动画 | 用户配置（通常 200-500ms） | GradualAnimator |

---

## 调试技巧

### 打印调用栈

```cpp
#include <utils/backtrace_local.h>

void DebugCallStack() {
    std::string backtrace = GetBacktrace();
    DISPLAY_HILOGD("Call stack: %{public}s", backtrace.c_str());
}
```

### 使用 htrace

```bash
# 跟踪亮度相关调用
htrace -t brightness

# 生成火焰图
htrace -g brightness.trace
```

### 日志标记

在关键节点添加日志标记：
```cpp
DISPLAY_HILOGD(COMP_FWK, "[CALLCHAIN] Enter SetBrightness, value=%{public}u", value);
```

---

## 相关链接

- **架构文档**：[../03_Architecture.md](../03_Architecture.md)
- **内部 API**：[../05_Internal_API.md](../05_Internal_API.md)
- **N-API 接口**：[../04_NAPI_Interface.md](../04_NAPI_Interface.md)
