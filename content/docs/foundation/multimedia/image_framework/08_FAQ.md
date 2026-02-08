# 常见问题

## 目的

本文档汇总 Image Framework 的常见问题及解决方案。

## 构建问题

### Q1: 编译错误 "undefined reference to `skia::xxx`"

**原因**: Skia 库未正确链接

**解决**:
```gn
# 检查 BUILD.gn 中是否添加 skia 依赖
external_deps += [ "skia:skia_canvaskit" ]
```

### Q2: 找不到插件元数据文件

**原因**: `.pluginmeta` 文件未正确安装

**解决**:
```bash
# 确保编译时包含插件目标
ninja -C out plugins

# 检查产物目录
ls out/ohos-arm64-release/system/etc/image/*.pluginmeta
```

### Q3: 硬件解码编译失败

**原因**: 硬件解码依赖未满足

**解决**:
```gni
# 检查 gn 参数
enable_jpeg_hw_decode = true
enable_heif_hw_decode = true

# 检查依赖组件是否存在
# - hdf_drivers_interface_display
# - drivers_interface_codec
```

### Q4: iOS/Android 交叉编译失败

**原因**: 平台配置不完整

**解决**:
```gni
# 检查交叉编译配置
if (use_clang_ios) {
  # 使用 ohos_source_set 而非 ohos_shared_library
}
```

---

## 运行问题

### Q5: 图片解码失败，返回 "UNSUPPORTED_FORMAT"

**排查步骤**:
1. 检查文件头是否正确
2. 检查对应格式插件是否已加载
3. 检查日志获取详细错误

```cpp
// 启用调试日志
#define IMAGE_DEBUG_FLAG
```

### Q6: 大图解码导致 OOM

**解决**:
```typescript
// 使用采样率缩小尺寸
const options = {
    sampleSize: 4,  // 1/4 尺寸
    desiredSize: { width: 1024, height: 1024 }
};
const pixelMap = await imageSource.createPixelMap(options);
```

### Q7: PixelMap 操作返回 "PixelMap not editable"

**原因**: PixelMap 不可编辑属性被设置

**解决**:
```typescript
// 创建时指定可编辑
const options = {
    editable: true
};
const pixelMap = await imageSource.createPixelMap(options);
```

### Q8: HDR 图片无法显示

**原因**: 
1. 设备不支持 HDR
2. 颜色空间设置错误

**解决**:
```typescript
// 检查 HDR 支持
const isHdr = pixelMap.isHdr();

// 转换为 SDR 显示
if (isHdr) {
    await pixelMap.toSdr();
}
```

---

## 性能问题

### Q9: 图片解码速度慢

**优化建议**:
1. 使用硬件解码（如可用）
2. 调整 `sampleSize` 减少解码尺寸
3. 使用 `desiredRegion` 裁剪需要的区域
4. 复用 PixelMap 避免重复解码

```typescript
// 最优解码选项
const options = {
    sampleSize: 2,
    desiredPixelFormat: image.PixelMapFormat.RGBA_8888,
    allocatorType: image.AllocatorType.DMA  // 零拷贝
};
```

### Q10: 内存占用过高

**优化建议**:
1. 及时释放不再使用的 PixelMap
2. 使用 DMA 分配器
3. 启用 Purgeable PixelMap（如支持）

```typescript
// 及时释放
pixelMap.release();

// 使用 DMA
const options = {
    allocatorType: image.AllocatorType.DMA
};
```

---

## 调试问题

### Q11: 如何启用详细日志

**解决**:
```cpp
// 在代码中设置日志级别
#define IMAGE_DEBUG_FLAG
#define LOG_DOMAIN LOG_TAG_DOMAIN_ID_IMAGE
```

或在运行时：
```bash
# 设置日志级别
hilog -b D -T image
```

### Q12: 如何分析解码性能

**方法**:
```cpp
// 使用 ImageTrace
#include "image_trace.h"

{
    ImageTrace trace("DecodeOperation");
    // 解码操作
}
```

### Q13: 如何调试插件加载问题

**步骤**:
1. 检查插件元数据文件是否存在
2. 检查插件库文件权限
3. 启用插件管理器调试日志

```bash
# 检查插件
ls -la /system/lib64/lib*plugin.so
ls -la /system/etc/image/*.pluginmeta

# 检查日志
grep "PluginServer" /var/log/hilog
```

---

## API 使用问题

### Q14: 异步方法和同步方法的区别

| 特性 | 异步方法 | 同步方法 |
|------|----------|----------|
| 返回值 | Promise | 直接值 |
| 阻塞 | 否 | 是 |
| 适用场景 | UI 线程 | 后台线程 |
| 示例 | `createPixelMap()` | `createPixelMapSync()` |

### Q15: 如何在 Worker 线程使用

**示例**:
```typescript
import worker from '@ohos.worker';

// Worker 线程中
const imageSource = image.createImageSource(filePath);
const pixelMap = await imageSource.createPixelMap();
// 处理完成后传回主线程
```

### Q16: 如何处理动图 (GIF)

**示例**:
```typescript
// 获取帧数
const frameCount = await imageSource.getFrameCount();

// 获取帧延迟
const delayTimes = await imageSource.getDelayTime();

// 解码指定帧
for (let i = 0; i < frameCount; i++) {
    const options = { index: i };
    const frame = await imageSource.createPixelMap(options);
    frames.push(frame);
}
```

---

## 平台差异问题

### Q17: iOS/Android 平台的限制

**已知限制**:
- Picture API 在 iOS/Android 不可用
- 部分 HDR 功能受限
- 硬件解码不可用

**代码示例**:
```typescript
// 检查平台能力
import deviceInfo from '@ohos.deviceInfo';

if (deviceInfo.osFullName.includes('OpenHarmony')) {
    // 使用完整功能
    const picture = await imageSource.createPicture();
}
```

---

## 安全相关问题

### Q18: 如何处理不受信任的图片

**建议**:
1. 限制文件大小
2. 限制图片尺寸
3. 验证图片格式
4. 在沙箱中解码

```typescript
// 安全检查示例
async function safeDecodeImage(source: image.ImageSource) {
    // 检查尺寸
    const info = await source.getImageInfo();
    if (info.size.width > 10000 || info.size.height > 10000) {
        throw new Error('Image too large');
    }
    
    // 安全解码
    return await source.createPixelMap({
        sampleSize: 1,
        editable: false  // 只读
    });
}
```

---

## 错误码速查

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| 0 | SUCCESS | 成功 |
| 401 | PARAMETER_ERROR | 参数错误 |
| 7600101 | UNSUPPORTED_OPERATION | 不支持的操作 |
| 7600102 | OUT_OF_MEMORY | 内存不足 |
| 7600103 | RESOURCE_UNAVAILABLE | 资源不可用 |
| 7600104 | INVALID_PARAMETER | 无效参数 |
| 7600105 | DECODING_FAILED | 解码失败 |
| 7600106 | ENCODING_FAILED | 编码失败 |
| 7600107 | CROP_FAILED | 裁剪失败 |

---

## 获取帮助

### 日志收集

```bash
# 收集 Image Framework 日志
hilog | grep -E "image|Image|IMAGE" > image_log.txt

# 收集崩溃日志
ls -la /data/log/faultlog/
```

### 调试构建

```gn
# 开启调试符号
declare_args() {
  is_debug = true
  symbol_level = 2
}
```

---

## 相关文档

- [N-API 接口](03_NAPI_Reference.md) - API 参考
- [架构说明](02_Architecture.md) - 系统设计
- [安全风险](07_Security_Risks.md) - 安全分析
