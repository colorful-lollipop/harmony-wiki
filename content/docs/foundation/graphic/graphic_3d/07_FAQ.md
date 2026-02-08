# 常见问题与定位

## 构建问题

### Q1: 编译报错 "undefined reference to..."

**症状**:
```
ld.lld: error: undefined reference to 'OHOS::Render3D::SceneAdapter::LoadPluginsAndInit()'
```

**原因**: 依赖库未正确链接或链接顺序错误。

**解决**:
1. 检查 `BUILD.gn` 中的 `deps` 和 `external_deps`
2. 确保依赖库已构建
3. 检查链接顺序（被依赖的库应在后面）

```gn
# 正确的依赖顺序
deps = [
  ":my_source",              # 自己的源码
  "../other:other_lib",      # 同级依赖
  "//foundation/...:base",   # 基础库最后
]
```

**定位路径**:
- 查看 `ld.lld` 输出的未定义符号
- 使用 `gn path` 查找依赖关系
- 检查 `bundle.json` 中的 `deps` 声明

---

### Q2: Shader编译失败

**症状**:
```
error: invalid shader file
error: compilation failed for shader.vert
```

**原因**: 
1. Shader语法错误
2. 缺少GLSL扩展支持
3. 着色器编译工具未正确配置

**解决**:
1. 检查Shader语法（`lume/Lume*/api/*/shaders/`）
2. 确认 `lume_config.gni` 中的渲染后端配置
3. 手动运行着色器编译工具测试

**定位路径**:
- 检查 `lume/LumeBinaryCompile/LumeShaderCompiler/` 编译日志
- 查看生成的shader头文件 (`*.h` 格式的GLSL)

---

### Q3: Taihe生成代码失败

**症状**:
```
ERROR: Taihe IDL generation failed
```

**原因**:
1. IDL文件语法错误
2. 依赖的IDL文件缺失
3. 输出目录权限问题

**解决**:
1. 检查 `kits/ets/taihe/idl/*.taihe` 文件语法
2. 确保所有依赖的IDL文件存在
3. 清理输出目录重新生成

```bash
rm -rf out/*/graphics3d/
hb build //foundation/graphic/graphic_3d/kits/ets:run_taihe
```

---

## 运行问题

### Q4: 3D场景无法显示（黑屏）

**症状**: 应用启动后3D区域黑屏，无内容显示。

**可能原因**:
1. 插件未正确加载
2. 渲染上下文初始化失败
3. GLTF模型加载失败

**排查步骤**:

1. **检查插件加载**:
```bash
# 查看系统日志
hilog | grep -i "plugin\|agp\|3d"
```

2. **验证插件存在**:
```bash
ls /system/lib64/graphics3d/
# 应包含: libPluginAGPRender.z.so, libPluginAGP3D.z.so, ...
```

3. **检查渲染上下文**:
```cpp
// 在代码中添加日志
WIDGET_LOGD("Graphics context created: %p", graphicsContext.get());
WIDGET_LOGD("Render context valid: %d", renderContext->IsValid());
```

**定位路径**:
- `3d_widget_adapter/src/ohos/graphics_manager.cpp` - 图形管理器
- `3d_widget_adapter/src/widget_adapter.cpp` - 适配器初始化
- `kits/js/src/SceneJS.cpp` - Scene创建

---

### Q5: GLTF模型加载失败

**症状**:
```
E/AGP: Failed to load GLTF: invalid format
E/AGP: GLTF parsing failed at line 123
```

**可能原因**:
1. GLTF/GLB文件格式错误
2. 依赖的资源文件缺失（纹理、bin文件）
3. 文件路径错误

**排查步骤**:

1. **验证GLTF文件**:
```bash
# 使用GLTF Validator验证
gltf-validator model.gltf
```

2. **检查文件路径**:
```javascript
// 确保使用正确的URI格式
const scene = await Scene.load('file:///data/.../model.gltf');
// 或
const scene = await Scene.load('bundle://.../model.gltf');
```

3. **检查资源完整性**:
```bash
# GLTF分离格式需要同名bin文件
ls model.gltf model.bin

# 检查纹理路径
unzip -l model.gltf | grep textures
```

**定位路径**:
- `lume/Lume_3D/src/gltf/gltf2_loader.cpp` - GLTF加载器
- `lume/Lume_3D/src/gltf/gltf2_importer.cpp` - GLTF导入器
- `lume/LumeEngine/src/io/file_manager.cpp` - 文件管理

---

### Q6: 动画不播放

**症状**: 模型加载成功但动画不播放。

**可能原因**:
1. 动画组件未正确添加
2. 动画系统未初始化
3. 动画轨道数据缺失

**排查代码**:
```javascript
// 检查动画是否存在
console.log('Animations:', scene.animations);

// 检查动画组件
const animComponent = node.getComponent('Animation');
console.log('Animation component:', animComponent);

// 手动播放动画
if (animation) {
    animation.play();
    console.log('Animation playing:', animation.isPlaying);
}
```

**定位路径**:
- `lume/Lume_3D/src/ecs/systems/animation_system.cpp` - 动画系统
- `kits/js/src/AnimationJS.cpp` - JS动画绑定

---

### Q7: 内存泄漏

**症状**: 应用运行一段时间后内存持续增长。

**可能原因**:
1. Scene未正确销毁
2. 资源未释放
3. JS对象循环引用

**排查步骤**:

1. **检查Scene销毁**:
```javascript
// 确保调用destroy
scene.destroy();
```

2. **检查Native层日志**:
```
W/AGP: leaking Node
W/AGP: leaking Mesh
```

3. **使用内存分析工具**:
```bash
# 查看进程内存
ps -A -o PID,NAME,RSS,VSZ | grep app_name

# 或查看详细内存映射
cat /proc/$(pidof app_name)/maps | grep graphic
```

**定位路径**:
- `kits/js/src/native_module_export.cpp:52-68` - 析构器中的泄漏检测
- `lume/LumeEngine/src/plugin_registry.cpp` - 插件生命周期管理

---

## 性能问题

### Q8: 帧率低（卡顿）

**症状**: 3D场景渲染帧率低于预期。

**可能原因**:
1. 模型面数过多
2. 过度绘制
3. 未使用GPU实例化

**排查步骤**:

1. **检查GPU时间**:
```cpp
// 启用性能追踪
#define CORE_PERF_ENABLED 1

// 查看渲染时间
WIDGET_LOGD("Render time: %llu us", renderTime);
```

2. **优化建议**:
```javascript
// 减少渲染物体数量
scene.renderMode = RenderMode.SIMPLE;

// 降低材质复杂度
material.metallic = 0.0;  // 避免复杂PBR计算
```

**定位路径**:
- `lume/LumeRender/src/renderer.cpp` - 渲染器主循环
- `lume/LumeEngine/src/perf/performance_data_manager.cpp` - 性能数据

---

### Q9: Shader编译卡顿

**症状**: 首次渲染时出现明显卡顿。

**原因**: Shader运行时编译导致。

**解决**: 使用预编译Shader缓存

```gn
# 在BUILD.gn中启用shader缓存
defines = [
  "CORE_EMBEDDED_ASSETS_ENABLED=2",
]
```

---

## 调试技巧

### 启用详细日志

```gn
# lume/LumeEngine/BUILD.gn
config("lume_engine_api") {
  defines = [
    "CORE_LOG_NO_DEBUG=0",
    "CORE_LOG_DEBUG=1",
  ]
}
```

### 使用hilog过滤

```bash
# 查看AGP相关日志
hilog -p graphic_3d

# 查看错误级别以上
hilog -l E

# 实时查看
hilog -g
```

### Native调试

```bash
# 附加调试器
debuggerd $(pidof app_name)

# 或查看崩溃日志
cat /data/log/faultlog/temp/
```

---

## 相关文档

- [架构设计 →](02_Architecture.md)
- [N-API接口 →](03_NAPI_Reference.md)
- [安全风险 →](06_Security.md)
- [GN构建 →](05_GN_Build.md)
