# 调试指南

> 构建、运行与问题定位

## 构建问题

### 问题 1: GN 配置错误

**症状**: `ninja: error: unknown variable 'xxx'`

**解决方案**:
```bash
# 检查 wallpaper.gni 变量
cat wallpaper_mgr/wallpaper.gni

# 确保使用正确的 GN 版本
gn --version
```

---

### 问题 2: 依赖缺失

**症状**: `ninja: error: dependency 'xxx' not found`

**解决方案**:
```bash
# 检查 bundle.json 依赖配置
cat bundle.json | grep -A 50 "deps"

# 常见依赖:
# - graphic_2d: //foundation/graphic/graphic_2d
# - samgr: //foundation/systemabilitysamgr
# - napi: //foundation/arkui/napi
```

---

### 问题 3: 编译产物未找到

**症状**: `error: cannot find -lwallpaper_service`

**解决方案**:
```bash
# 检查 target 是否正确编译
ninja -C out wallpaper_service
ninja -C out wallpapermanager

# 检查输出路径
ls out/default/libs/
```

---

## 运行问题

### 问题 1: SA 注册失败

**症状**: `WallpaperService` not found

**日志**:
```
E/WallpaperManager: Get samgr failed!
```

**排查步骤**:
```bash
# 1. 检查 SA 配置文件
cat services/profile/3705.json

# 2. 检查 rc 脚本
cat services/etc/wallpaperservice.rc

# 3. 检查 SELinux 上下文
ls -Z system/etc/init/wallpaperservice.cfg
```

**代码位置**: `services/src/wallpaper_service.cpp:66`

---

### 问题 2: IPC 调用失败

**症状**: `IPC` error 或 `ERR_DEAD_OBJECT`

**日志**:
```
E/WallpaperManager: Callservice failed with: 32
E/WallpaperManager: Remote is dead, reset service instance.
```

**排查步骤**:
```bash
# 1. 检查服务进程状态
hisysevent -query | grep wallpaper

# 2. 检查 IPC 描述符
adb shell "dumpsys wallpaper_service"

# 3. 检查系统能力
adb shell "dumpsys sa"
```

**代码位置**: `frameworks/native/src/wallpaper_manager.cpp:145-165`

---

### 问题 3: 权限错误

**症状**: `E_NO_PERMISSION` 或权限拒绝

**日志**:
```
E/WallpaperService: Check permission failed!
```

**排查步骤**:
```bash
# 1. 检查应用权限声明
cat config.json | grep -A 10 "reqPermissions"

# 2. 检查运行时权限状态
adb shell "hidumper -s AccessToken"
```

**代码位置**: `services/src/wallpaper_service.cpp:1216-1225`

---

## 调试技巧

### 1. 日志查看

```bash
# 开启 wallpaper 日志
hilog | grep -E "Wallpaper|WALLPAPER"

# 查看详细日志
adb shell "hilog | grep -E 'D0[0-9][0-9][0-9][0-9][0-9]'"
```

**日志标签**:
- `0xD001C20` - WallpaperClient (框架层)
- `0xD002200` - WallpaperExtension (Extension 层)

---

### 2. 服务状态

```bash
# 查看 SA 状态
adb shell "dumpsys battery" | grep -A 5 wallpaper

# 查看壁纸服务
adb shell "dumpsys activity wallpaper"

# 查看进程信息
adb shell "ps -A | grep wallpaper"
```

---

### 3. 文件系统

```bash
# 查看壁纸文件
ls -la /data/service/el1/public/wallpaper/

# 检查权限
adb shell "ls -Z /data/service/el1/public/wallpaper/"
```

---

### 4. 内存分析

```bash
# 查看内存占用
adb shell "dumpsys meminfo wallpaper_service"

# 内存泄漏检测
adb shell "cat /proc/$(pgrep wallpaper_service)/status"
```

---

### 5. Trace 追踪

```bash
# 开启 hitrace
adb shell "hitrace --wallpaper --time 5 -b 10240 > wallpaper_trace.txt"

# 分析 trace
hiview-trace open wallpaper_trace.txt
```

---

## 常见问题 FAQ

### Q1: setWallpaper 返回成功但壁纸未更新

**可能原因**:
1. 写入路径错误
2. 文件格式不支持
3. 权限问题

**排查**:
```bash
# 检查壁纸文件
ls -la /data/service/el1/public/wallpaper/0/system/

# 检查日志
hilog | grep -E "SetWallpaper|WriteToFile"
```

---

### Q2: getPixelMap 返回空

**可能原因**:
1. 未设置壁纸
2. PixelMap 创建失败
3. FD 已关闭

**排查**:
```bash
# 检查壁纸是否存在
adb shell "ls -la /data/service/el1/public/wallpaper/"

# 检查 FD 状态
adb shell "cat /proc/$(pgrep wallpaper_service)/fd"
```

---

### Q3: on('colorChange') 回调不触发

**可能原因**:
1. 回调函数格式错误
2. 事件类型错误
3. 监听器未正确注册

**排查**:
```javascript
// 检查回调格式
wallpaper.on('colorChange', (colors, type) => {
    console.log('颜色变化:', colors, type);
});

// 正确的壁纸类型
wallpaper.on('colorChange', (colors, wallpaperType) => {
    // wallpaperType: 0=系统, 1=锁屏
});
```

---

### Q4: V9 API 返回 E_NOT_SYSTEM_APP

**可能原因**:
1. 调用者不是系统应用
2. TokenID 校验失败

**排查**:
```bash
# 检查调用者身份
adb shell "dumpsys app xxx"
```

---

### Q5: 视频壁纸设置失败

**可能原因**:
1. 视频格式不支持
2. 视频过大 (>100MB)
3. 编码格式不支持

**排查**:
```bash
# 检查视频格式
ffprobe -v error -select_streams v:0 -show_entries stream=codec_name,width,height -of csv=p=0 video.mp4

# 检查视频大小
ls -lh video.mp4
```

---

## 诊断工具

### wallpaper_dump

```bash
# 打印壁纸信息
adb shell "wallpaper_dump"
```

**输出示例**:
```
Wallpaper Service Info:
  User: 0
  System Wallpaper: /data/service/el1/public/wallpaper/0/system/wallpaper_home
  Lock Wallpaper: /data/service/el1/public/wallpaper/0/lockscreen/wallpaper_lock
  Colors: [0xFF123456, 0xFF654321]
```

---

### hisysevent 查看

```bash
# 查看 wallpaper 相关事件
hisysevent -query | grep wallpaper
```

**事件类型**:
- `WALLPAPER_SERVICE_FAULT`
- `WALLPAPER_SET`
- `WALLPAPER_GET`

---

## 性能优化

### 1. 壁纸加载优化

```cpp
// 使用异步加载
ErrorCode WallpaperManager::GetPixelMapAsync(int32_t wallpaperType, 
    std::function<void(std::shared_ptr<PixelMap>)> callback)
{
    // ThreadPool 中执行
}
```

---

### 2. 内存优化

```cpp
// 及时关闭 FD
void WallpaperManager::CloseWallpaperFd(int32_t wallpaperType)
{
    std::lock_guard<std::mutex> lock(wallpaperFdLock_);
    auto iter = wallpaperFdMap_.find(wallpaperType);
    if (iter != wallpaperFdMap_.end()) {
        close(iter->second);
        wallpaperFdMap_.erase(iter);
    }
}
```

---

## 相关文档

- API 参考: [03_API.md](03_API.md)
- 架构设计: [04_Architecture.md](04_Architecture.md)
- 安全评估: [06_Security.md](06_Security.md)
- Native 内部接口: [07_InnerAPI.md](07_InnerAPI.md)
