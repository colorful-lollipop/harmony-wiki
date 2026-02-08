# 常见问题 (FAQ)

> OpenHarmony Camera Framework - 快速解答常见问题

---

## 构建问题

### Q1: 编译相机框架需要哪些前置条件？

**A**: 
- OpenHarmony 完整源码环境
- 安装 gn、ninja 构建工具
- 配置好产品编译环境

```bash
# 检查构建环境
./build/prebuilts_download.sh

# 编译相机框架
./build.sh --product {product_name} --target camera_framework
```

---

### Q2: 如何只编译 camera_service？

**A**:
```bash
./build.sh --product {product_name} \
  --target //foundation/multimedia/camera_framework/services/camera_service
```

---

### Q3: Feature 开关如何配置？

**A**: 在 `multimedia_camera_framework.gni` 中修改：

```gn
declare_args() {
  camera_framework_feature_moving_photo = true   # 开启动态照片
  camera_framework_feature_deferred = false      # 关闭延迟处理
}
```

---

## 运行问题

### Q4: 应用打开相机失败，如何排查？

**A**: 按以下顺序检查：

1. **权限检查**
   ```bash
   # 查看日志确认权限
   hilog -T CAMERA_SVC | grep "permission"
   ```

2. **服务状态**
   ```bash
   # 检查 SA 3008 是否运行
   sa -l | grep 3008
   ```

3. **设备占用**
   ```bash
   # 查看是否有其他应用占用相机
   hidumper -s 3008
   ```

---

### Q5: 相机服务崩溃后如何重启？

**A**:
```bash
# 重启 camera_service
kill -9 $(pidof camera_service)
# 系统会自动重启

# 或者重启整个媒体子系统
service_control stop camera_service
service_control start camera_service
```

---

## 开发问题

### Q6: 如何添加新的 JS API？

**A**:

1. 在 `frameworks/js/camera_napi/src/` 下创建/修改对应的 NAPI 文件
2. 在 `native_module_ohos_camera.cpp` 中注册类和方法
3. 更新 TypeScript 定义文件 `.d.ts`

示例：
```cpp
// 在 photo_output_napi.cpp 中添加方法
static napi_value MyNewApi(napi_env env, napi_callback_info info) {
    // 实现
}

// 在属性表中注册
static napi_property_descriptor photo_output_props[] = {
    // ...
    DECLARE_NAPI_FUNCTION("myNewApi", MyNewApi),
};
```

---

### Q7: 如何调试 IPC 通信问题？

**A**:

1. 启用 IPC 调试日志
   ```cpp
   #define CAMERA_IPC_DEBUG 1
   ```

2. 使用 hilog 查看 IPC 调用
   ```bash
   hilog -T CAMERA_SVC -L DEBUG
   ```

3. 检查 Binder 状态
   ```bash
   cat /sys/kernel/debug/binder/state
   ```

---

### Q8: 相机预览黑屏怎么办？

**A**: 常见原因和解决方案：

| 原因 | 检查方法 | 解决方案 |
|------|----------|----------|
| Surface 未就绪 | 检查 XComponent 加载 | 确保 onLoad 后创建 session |
| 分辨率不匹配 | 检查 profile 支持 | 使用 GetSupportedPreviewProfiles |
| 会话未启动 | 检查 start() 返回值 | 确认 commitConfig 成功 |
| 权限问题 | 检查日志 | 申请 CAMERA 权限 |

---

## 安全问题

### Q9: 如何验证权限检查是否生效？

**A**:

```cpp
// 在代码中添加调试日志
int32_t CheckPermission(uint32_t tokenId, const std::string& permission) {
    MEDIA_DEBUG_LOG("Checking permission %{public}s for token %{public}u", 
        permission.c_str(), tokenId);
    int32_t ret = AccessTokenKit::VerifyAccessToken(tokenId, permission);
    MEDIA_DEBUG_LOG("Permission check result: %{public}d", ret);
    return ret;
}
```

---

### Q10: 如何运行 fuzz 测试？

**A**:

```bash
# 编译 fuzzer
./build.sh --product {product} --target //foundation/multimedia/camera_framework/test/fuzztest/cameramanager_fuzzer

# 运行 fuzzer
./cameramanager_fuzzer corpus/
```

---

## 性能问题

### Q11: 如何优化相机启动时间？

**A**:

1. 使用 `prelaunch` API 预加载相机
2. 复用 CaptureSession，避免重复创建
3. 合理设置预览分辨率

```typescript
// 预加载相机
cameraManager.prelaunch(cameraDevice);

// 复用 session
const session = cameraManager.createCaptureSession();
// 多次使用，最后才 release
```

---

### Q12: 如何减少内存占用？

**A**:

1. 及时释放不用的 Output
2. 使用合适的 Buffer 大小
3. 关闭不需要的功能（如动态照片）

```typescript
// 及时释放
photoOutput.release();
cameraInput.release();
session.release();
```

---

## 其他问题

### Q13: 如何获取相机支持的能力列表？

**A**:

```typescript
// JS API
const capability = cameraManager.getSupportedOutputCapability(cameraDevice);
console.log('Preview profiles:', capability.previewProfiles);
console.log('Photo profiles:', capability.photoProfiles);
console.log('Video profiles:', capability.videoProfiles);
```

---

### Q14: 相机元数据包含哪些信息？

**A**: 包括但不限于：

- 曝光时间、ISO、光圈
- 对焦状态、变焦比例
- 人脸检测结果
- 场景模式识别

```typescript
metadataOutput.on('metadataObjectAvailable', (objects) => {
    for (const obj of objects) {
        console.log('Type:', obj.type);  // FACE, HUMAN_BODY, etc.
        console.log('Rect:', obj.rect);  // 位置矩形
    }
});
```

---

## 获取更多帮助

- **官方文档**: [OpenHarmony 文档](https://docs.openharmony.cn/)
- **源码仓库**: [Gitee - camera_framework](https://gitee.com/openharmony/multimedia_camera_framework)
- **社区论坛**: [OpenHarmony 论坛](https://forums.openharmony.cn/)

---

**最后更新**: 2025-02-07
