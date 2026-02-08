# 常见问题 (FAQ)

## 构建问题

### Q1: 如何选择开发板并构建项目？

**A**: 使用 `hb` 工具：

```bash
# 进入项目根目录
cd /path/to/camera

# 选择开发板
hb set
# 按提示选择对应的开发板型号

# 构建整个项目
hb build camera_lite

# 构建单个模块
hb build cameraApp_hap
hb build gallery_hap
hb build media_sample
```

**证据**: `README.md:31-36`

---

### Q2: 构建失败，提示找不到头文件？

**A**: 本项目依赖 OpenHarmony 完整源码树。确保：

1. 在完整 OpenHarmony 源码环境下构建
2. 已正确执行 `hb set` 选择开发板
3. 依赖的子系统已同步（如 multimedia_camera_lite, arkui_ui_lite 等）

**依赖路径示例** (来自 `cameraApp/BUILD.gn:24-33`):
```gn
deps = [
    "${aafwk_lite_path}/frameworks/ability_lite:aafwk_abilitykit_lite",
    "//foundation/multimedia/camera_lite/frameworks:camera_lite",
    "//foundation/arkui/ui_lite:ui_lite",
    // ...
]
```

---

### Q3: 如何修改应用权限？

**A**: 编辑对应模块的 `config.json`：

```json
// cameraApp/cameraApp/src/main/config.json
"reqPermissions": [
    {
        "name": "ohos.permission.CAMERA",
        "reason": "USER_GRANT",
        "usedScene": {
            "ability": [".FormAbility"],
            "when": "inuse"
        }
    }
]
```

**权限级别**:
- `SYSTEM_GRANT`: 安装时自动授予
- `USER_GRANT`: 运行时需用户确认

---

## 运行问题

### Q4: HAP 包安装失败？

**A**: 检查以下几点：

1. **签名问题**: 确保证书文件存在且未过期
   - `cert/camera_AppProvision_Release.p7b`
   - 开发测试可使用调试证书

2. **版本兼容性**: 检查 `config.json` 中的 `apiVersion`
   ```json
   "apiVersion": {
       "compatible": 3,
       "target": 4
   }
   ```

3. **设备类型**: 确认设备类型在支持列表中
   ```json
   "deviceType": ["smartWatch", "sportsWatch", ...]
   ```

---

### Q5: 相机无法打开/预览黑屏？

**A**: 检查以下事项：

1. **权限**: 确认已授予 CAMERA 权限
   - 检查 setting 应用中权限开关状态

2. **硬件支持**: 确认设备有相机硬件
   - 查看 `camera_manager.cpp:612-620` 的相机检测逻辑

3. **日志**: 查看系统日志定位问题
   ```cpp
   // camera_manager.cpp:605-633
   printf("camera start init!!! \n");
   camKit = CameraKit::GetInstance();
   // ...
   ```

---

### Q6: 录像文件保存失败？

**A**: 检查以下路径和权限：

1. **存储路径**: 确认 `/userdata/video/` 目录存在且有写入权限

2. **磁盘空间**: 确认设备有足够存储空间

3. **代码位置**: `camera_manager.cpp:248-459`
   ```cpp
   static int CameraGetRecordFd(const char* p) {
       // 获取录像文件描述符
   }
   ```

---

## 开发调试

### Q7: 如何添加调试日志？

**A**: 使用 `printf` 或 `cout`：

```cpp
#include <stdio.h>
#include <iostream>

// C 风格
printf("[DEBUG] Camera started, surface=%p\n", surface);

// C++ 风格
std::cout << "Camera initialized" << std::endl;
```

**注意**: 发布版本应移除或禁用调试日志（参见 [安全风险分析](./Security.md)）

---

### Q8: 如何定位崩溃问题？

**A**: 

1. **查看日志**: 
   ```bash
   hilog | grep cameraApp
   ```

2. **关键生命周期**: 检查 Ability/AbilitySlice 的 OnStart/OnStop
   - `camera_ability.cpp`
   - `camera_ability_slice.cpp`

3. **空指针检查**: 确保指针使用前已初始化
   ```cpp
   // camera_ability_slice.cpp:509
   cam_manager = new SampleCameraManager();
   if (cam_manager == nullptr) {
       // 错误处理
   }
   ```

---

### Q9: 独立示例程序（media_sample）如何运行？

**A**: 

1. 构建 `media_sample`:
   ```bash
   hb build media_sample
   ```

2. 推送到设备:
   ```bash
   hdc file send out/dev_tools/camera_sample /data/
   ```

3. 运行:
   ```bash
   hdc shell
   cd /data
   chmod +x camera_sample
   ./camera_sample
   ```

**注意**: 这些示例直接访问相机硬件，不经过 Ability 框架

---

## 架构理解

### Q10: 为什么找不到 N-API 接口？

**A**: 本项目是 **纯 Native C++ 应用**，**无 N-API JS 绑定层**。

**证据**:
- 全局搜索 `napi_`、`NAPI_MODULE` 等符号，无结果
- 所有代码直接编译为 `.so`，通过 Ability 框架运行

**对比**:
| 特性 | 本项目 | 标准 OpenHarmony |
|------|--------|------------------|
| 开发语言 | C++ | JS/ArkTS + C++ |
| API 类型 | Native Ability | N-API + Ability |
| 设备类型 | Lite (轻量) | 标准设备 |

---

### Q11: Ability 和 AbilitySlice 的关系？

**A**: 

- **Ability**: 应用入口，管理整个应用生命周期
- **AbilitySlice**: 页面切片，管理单个页面

**典型关系**:
```
CameraAbility (Ability)
    └── CameraAbilitySlice (AbilitySlice) - 相机主页面

GalleryAbility (Ability)
    ├── GalleryAbilitySlice - 图库主页面
    ├── PictureAbilitySlice - 图片查看页面
    └── PlayerAbilitySlice - 视频播放页面
```

**代码示例**:
```cpp
// camera_ability.cpp
void CameraAbility::OnStart(const Want &want) {
    SetMainRoute("CameraAbilitySlice");  // 设置主页面
}
```

---

### Q12: 如何理解模块依赖关系？

**A**: 查看 `BUILD.gn` 中的 `deps`：

```gn
# cameraApp/BUILD.gn
deps = [
    "//foundation/multimedia/camera_lite/frameworks:camera_lite",
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
]
```

**依赖方向**: 应用 -> 系统服务  
**注意**: 应用模块之间**无直接依赖**，避免循环依赖

---

## 其他问题

### Q13: 如何修改应用包名？

**A**: 编辑 `config.json` 中的 `bundleName`:

```json
{
  "app": {
    "bundleName": "com.yourcompany.camera",
    "vendor": "yourcompany",
    ...
  }
}
```

**注意**: 修改后需同步更新：
- 签名证书
- 代码中硬编码的包名（如有）

---

### Q14: 支持哪些设备类型？

**A**: 查看 `config.json` 的 `deviceType`:

```json
"deviceType": [
    "phone",
    "tv", 
    "tablet",
    "pc",
    "car",
    "smartWatch",
    "sportsWatch",
    "smartVision"
]
```

**本项目主要面向**: IoT/轻量级设备（smartWatch, sportsWatch 等）

---

## 获取帮助

如上述问题无法解决，建议：

1. 查看 [架构说明](./Architecture.md) 理解系统架构
2. 查看 [安全风险分析](./Security.md) 排查安全问题
3. 参考 OpenHarmony 官方文档：https://gitee.com/openharmony/docs

---

## 相关链接

- [目录结构](./Structure.md) - 代码组织
- [GN 构建系统](./Build_System.md) - 构建配置
- [编译产物](./Build_Outputs.md) - 输出说明
