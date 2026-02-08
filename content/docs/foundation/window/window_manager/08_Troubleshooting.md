# 问题排查指南

## 目的

本文档提供 Window Manager 子系统常见构建、运行和调试问题的定位方法和解决方案。

## 构建问题

### 1. 编译错误：找不到头文件

**错误信息**:
```
error: 'window.h' file not found
#include "window.h"
         ^~~~~~~~~
```

**原因分析**:
- 头文件路径未正确包含
- 依赖模块未声明

**解决方案**:
```gn
# 在 BUILD.gn 中添加正确的 include_dirs
config("my_module_config") {
  include_dirs = [
    "//foundation/window/window_manager/interfaces/innerkits",
    "//foundation/window/window_manager/interfaces/innerkits/wm",
    "//foundation/window/window_manager/utils/include",
  ]
}

ohos_shared_library("my_module") {
  configs = [ ":my_module_config" ]
  deps = [
    "//foundation/window/window_manager/wm:libwm",
    "//foundation/window/window_manager/utils:libwmutil",
  ]
}
```

**验证**:
```bash
gn desc out/standard/ //my/module:my_module configs
```

---

### 2. 链接错误：未定义引用

**错误信息**:
```
undefined reference to `OHOS::Window::Create(std::string const&, sptr<OHOS::WindowOption>)'
```

**原因分析**:
- 依赖库未正确链接
- 链接顺序问题

**解决方案**:
```gn
ohos_shared_library("my_module") {
  deps = [
    # 确保 libwm 在依赖列表中
    "//foundation/window/window_manager/wm:libwm",
    "//foundation/window/window_manager/dm:libdm",
  ]
  
  # 如果需要静态链接
  deps += [
    "//foundation/window/window_manager/wm:libwm_static",
  ]
}
```

---

### 3. Scene Board 切换问题

**问题**: 修改 `scene_board_enable.gni` 后编译不生效

**解决方案**:
```bash
# 1. 清理输出目录
rm -rf out/standard/

# 2. 重新生成构建配置
gn gen out/standard --args='
  window_manager_use_sceneboard=true
  device_status_enable=true
  window_manager_fold_ability=true
'

# 3. 重新编译
ninja -C out/standard foundation/window/window_manager:wmserver
```

---

## 运行问题

### 4. 窗口创建失败

**症状**: 应用调用 `window.create()` 返回错误

**排查步骤**:

1. **检查日志**:
```bash
# 查看 Window Manager 日志
hilog | grep -i window

# 查看具体错误码
hilog | grep "WMS_LIFE\|WMS_ERROR"
```

2. **验证权限**:
```cpp
// 在代码中检查权限
bool hasPermission = AccessTokenKit::VerifyAccessToken(
    IPCSkeleton::GetCallingTokenID(), 
    "ohos.permission.SYSTEM_FLOAT_WINDOW"
) == RET_SUCCESS;
```

3. **检查系统服务状态**:
```bash
# 检查 WMS 服务是否运行
ps -ef | grep wms

# 检查 SA 注册状态
ability_tool dump -a | grep -i window
```

**常见错误码**:
| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| 1300001 | 重复操作 | 确保窗口未重复创建 |
| 1300002 | 无效窗口状态 | 检查窗口生命周期状态 |
| 1300003 | 无效窗口 | 检查窗口 ID 是否有效 |
| 1300004 | 无权限 | 在 module.json5 中声明权限 |

---

### 5. IPC 调用失败

**症状**: IPC 调用返回 `IPC_STUB_INVALID_DATA` 或超时

**排查步骤**:

1. **检查服务是否注册**:
```bash
# 查看 SystemAbility 列表
cat /system/profile/wms_sa_profile.xml
```

2. **检查 IPC 日志**:
```bash
hilog | grep -i "ipc\|binder"
```

3. **验证接口版本**:
```cpp
// 检查接口版本匹配
if (proxy->GetInterfaceVersion() != EXPECTED_VERSION) {
    TLOGE("Interface version mismatch");
}
```

---

### 6. 显示信息获取失败

**症状**: `display.getDefaultDisplay()` 返回 null

**排查步骤**:

1. **检查 DMS 服务**:
```bash
# 检查 DMS 进程
ps -ef | grep dms

# 查看显示服务日志
hilog | grep -i "DisplayManager\|DMS"
```

2. **检查显示硬件**:
```bash
# 查看显示设备
dumpsys display

# 查看屏幕信息
dumpsys SurfaceFlinger
```

3. **验证权限**:
```xml
<!-- module.json5 -->
"requestPermissions": [
  {
    "name": "ohos.permission.GET_WIFI_INFO"
  }
]
```

---

## 调试技巧

### 7. 启用详细日志

**修改日志级别**:
```cpp
// 在代码中设置日志级别
#include "hilog/log.h"

// 设置 Domain 日志级别
HilogSetLevel(LOG_CORE, LOG_DEBUG);
```

**HiSysEvent 日志**:
```bash
# 查看 Window Manager 事件日志
hisysevent -r | grep -i window

# 查看特定事件
hisysevent -r -o WINDOW_MANAGER
```

---

### 8. 使用调试工具

**窗口信息 Dump**:
```bash
# 导出窗口信息
dumpsys window

# 导出显示信息
dumpsys display
```

**代码中 Dump**:
```cpp
// 在关键位置添加 Dump
WindowInspector::DumpWindowInfo();
WindowInspector::DumpWindowLayout();
```

---

### 9. 性能分析

**启动时间分析**:
```bash
# 使用 hitrace
hitrace -b 20480 -t 5 wm dms graphic

# 分析 trace 文件
python3 /system/bin/trace_analysis.py /data/log/hitrace.txt
```

**内存分析**:
```bash
# 查看进程内存
dumpsys meminfo foundation

# 查看窗口内存
 dumpsys meminfo | grep -i window
```

---

## 常见问题 FAQ

### Q1: 如何判断是否使用 Scene Board？

**A**: 检查运行时或编译时标志
```cpp
// 运行时检查
bool isSceneBoard = SceneBoardJudgement::IsSceneBoardEnabled();

// 编译时检查
#ifdef WINDOW_MANAGER_USE_SCENEBOARD
// Scene Board 代码
#else
// 传统代码
#endif
```

---

### Q2: 窗口显示层级问题

**A**: 检查 ZOrder 和窗口类型
```cpp
// 设置窗口层级
window->SetZOrder(100);

// 检查系统窗口层级限制
if (windowType == WindowType::WINDOW_TYPE_SYSTEM) {
    // 系统窗口有更高优先级
}
```

---

### Q3: 多屏适配问题

**A**: 使用 DisplayManager 获取屏幕信息
```cpp
auto displays = DisplayManager::GetInstance().GetAllDisplays();
for (auto display : displays) {
    auto displayId = display->GetId();
    auto width = display->GetWidth();
    auto height = display->GetHeight();
    // 根据屏幕尺寸调整窗口
}
```

---

### Q4: 权限申请被拒绝

**A**: 检查权限声明和用户授权
```json
// module.json5
{
  "requestPermissions": [
    {
      "name": "ohos.permission.SYSTEM_FLOAT_WINDOW",
      "reason": "$string:float_window_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "always"
      }
    }
  ]
}
```

---

## 联系与支持

- **Issue 追踪**: Gitee OpenHarmony 仓库
- **开发文档**: https://gitee.com/openharmony/docs
- **社区论坛**: https://forums.openharmony.cn

## 相关文档

- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [安全分析](07_Security_Analysis.md)
