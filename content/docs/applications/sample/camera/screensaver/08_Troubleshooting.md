# 故障排查指南

## 1. 构建问题

### 1.1 编译错误：头文件找不到

**错误信息**：
```
fatal error: 'ability_loader.h' file not found
```

**原因**：
- 构建环境未正确配置 OpenHarmony 框架路径

**解决方案**：
```bash
# 1. 确保在 OpenHarmony 源码根目录执行
source build.sh

# 2. 使用 hb 工具链构建
hb build -f

# 3. 检查环境变量
echo $OHOS_SDK_HOME
```

**相关文件**：`BUILD.gn:33-39`

---

### 1.2 链接错误：符号未定义

**错误信息**：
```
undefined reference to 'Screen::GetInstance()'
```

**原因**：
- 未链接 `ui_lite` 或 `surface_lite` 依赖

**解决方案**：
```bash
# 确保 BUILD.gn 中包含正确依赖
# 检查 deps 配置
deps = [
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/graphic/surface_lite",
]
```

**证据**：`BUILD.gn:22-30`

---

### 1.3 HAP 打包失败

**错误信息**：
```
error: cert_profile not found
```

**原因**：
- 签名证书路径配置错误

**解决方案**：
```bash
# 1. 检查证书文件是否存在
ls -la cert/

# 2. 确认 BUILD.gn 中路径配置正确
cert_profile = "cert/com.huawei.screensaver_AppProvision_release.p7b"
```

**证据**：`BUILD.gn:54`

---

## 2. 运行问题

### 2.1 屏保无法启动

**现象**：
- 屏保应用未在预期时间触发

**排查步骤**：
```bash
# 1. 检查应用是否正确安装
hdc shell bm dump -a | grep screensaver

# 2. 检查应用签名
hdc shell bm dump -n com.huawei.screensaver

# 3. 查看系统日志
hdc shell hidumper -a ability
```

**可能原因**：
- 应用未正确签名
- 系统屏保服务未启用
- 设备类型不支持

**相关配置**：`config.json:22-24`

---

### 2.2 图片不显示

**现象**：
- 屏保界面显示黑屏，无图片动画

**排查步骤**：
```bash
# 1. 检查图片资源是否存在
hdc shell ls /storage/app/run/com.huawei.screensaver/screensaver/resources/base/media/

# 2. 检查权限
hdc shell ls -la /storage/app/run/com.huawei.screensaver/
```

**可能原因**：
- 图片资源未正确打包
- 资源路径配置错误

**相关配置**：`ui_config.h:25-34`

---

### 2.3 点击无响应

**现象**：
- 点击屏幕无法退出屏保

**排查步骤**：
```bash
# 1. 检查触摸事件是否正常
hdc shell hidumper -a input

# 2. 查看应用日志
hdc shell logcat | grep -i screensaver
```

**可能原因**：
- 触摸事件被其他应用拦截
- EventListener 回调异常
- 焦点问题

**相关代码**：`screensaver_ability_slice.cpp:53-58`

---

## 3. 性能问题

### 3.1 动画卡顿

**现象**：
- 图片切换不流畅，有掉帧现象

**排查步骤**：
1. 检查设备 GPU 能力
2. 检查图片资源大小
3. 确认动画间隔设置

**优化建议**：
```cpp
// 减少动画间隔
static constexpr uint16_t IMAGE_ANIMATOR_TIME_S = 2 * 1000;

// 或降低图片分辨率
```

**相关配置**：`ui_config.h:22-23`

---

### 3.2 内存占用过高

**排查方法**：
```bash
# 查看内存使用
hdc shell cat /proc/meminfo

# 查看应用内存
hdc shell cat /proc/<pid>/status
```

**可能原因**：
- 图片未正确释放
- 内存泄漏

**相关代码**：`screensaver_ability_slice.cpp:33-44`

---

## 4. 日志与调试

### 4.1 日志输出

代码中使用 `printf` 输出调试日志：
```cpp
printf("ScreensaverAbilitySlice::OnInactive\n");
```

**查看日志**：
```bash
hdc shell logcat | grep -E "screensaver|Screensaver"
```

---

### 4.2 调试技巧

**方法 1：使用断点**
- 在 IDE 中设置断点（如 DevEco Studio）
- 使用调试器附加进程

**方法 2：打印变量**
```cpp
printf("Screen width: %d, height: %d\n",
       Screen::GetInstance().GetWidth(),
       Screen::GetInstance().GetHeight());
```

---

## 5. 常见问题 FAQ

### Q1: 如何修改屏保图片？

**回答**：
1. 替换 `resources/base/media/` 目录下的图片
2. 或修改 `ui_config.h` 中的路径配置
3. 重新编译 HAP

---

### Q2: 如何调整动画速度？

**回答**：
修改 `ui_config.h` 中的 `IMAGE_ANIMATOR_TIME_S`：
```cpp
static constexpr uint16_t IMAGE_ANIMATOR_TIME_S = 2 * 1000;  // 改为 3 * 1000 则为 3 秒
```

---

### Q3: 如何禁用点击退出？

**回答**：
注释或修改 `screensaver_ability_slice.cpp:53-58` 中的点击回调。

---

## 6. 文档导航

- **返回**：[安全评审](07_Security.md) → 安全风险
- **相关**：[构建系统](06_Build.md) → 编译配置
- **相关**：[内部 API](05_Inner_API.md) → 模块接口
