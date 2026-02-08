# graphics_effect 问题排查指南

## 概述

本章节提供 graphics_effect 常见构建、运行、调试问题的排查路径和解决方案。

---

## 构建问题

### 问题 1: 编译失败 - 找不到头文件

**错误信息**:
```
fatal error: 'xxx.h' file not found
```

**排查路径**:

1. 检查 GN 配置
   ```bash
   # 确认 include_dirs 配置
   grep -n "include_dirs" BUILD.gn
   ```

2. 验证头文件存在
   ```bash
   ls -la include/ | grep xxx.h
   ```

3. 清理并重新构建
   ```bash
   rm -rf out/
   ./build.sh --product-name <product> --build-target graphics_effect:graphics_effect_core
   ```

**证据来源**: `BUILD.gn:54` - `include_dirs = [ "include" ]`

---

### 问题 2: 链接失败 - 找不到依赖库

**错误信息**:
```
ld: cannot find -lgraphic_2d: No such file or directory
```

**排查路径**:

1. 检查外部依赖声明
   ```bash
   grep -n "external_deps" BUILD.gn
   ```

2. 确认依赖库已构建
   ```bash
   # 先构建依赖
   ./build.sh --product-name <product> --build-target graphic_2d:2d_graphics
   ```

3. 检查依赖路径
   ```bash
   ls out/<product>/libs/ | grep graphic_2d
   ```

**证据来源**: `BUILD.gn:142-146` - external_deps 配置

---

### 问题 3: CFI 编译失败

**错误信息**:
```
error: unknown argument '-cfi'
```

**排查路径**:

1. 检查 GCC/Clang 版本
   ```bash
   clang --version
   ```

2. 确认 sanitizer 配置
   ```bash
   grep -n "sanitize" BUILD.gn
   ```

3. 降级构建（非 CFI 模式）
   ```bash
   # 修改 BUILD.gn 或使用不同 toolchain
   ```

**证据来源**: `BUILD.gn:24-28` - sanitize 配置

---

### 问题 4: Skia 版本不匹配

**错误信息**:
```
error: 'USE_M133_SKIA' macro redefined
```

**排查路径**:

1. 检查 feature 配置
   ```bash
   grep -n "graphics_effect_feature_upgrade_skia" config.gni
   ```

2. 清理构建缓存
   ```bash
   rm -rf out/
   ```

3. 同步 Skia 依赖
   ```bash
   # 确保 Skia 版本一致
   ```

**证据来源**: `config.gni:17` - feature 定义

---

## 运行时问题

### 问题 1: 渲染效果异常

**现象**: 模糊/光效未正确显示

**排查路径**:

1. 检查效果参数
   ```cpp
   // 确认参数值有效
   effect->SetParam("radius", 10.0f);  // 确认范围
   ```

2. 验证图像数据
   ```cpp
   // 检查图像是否有效
   if (!image || image->IsEmpty()) {
       GE_LOGE("Invalid image");
   }
   ```

3. 检查渲染管线
   ```bash
   # 查看日志
   hilog | grep GE_
   ```

**证据来源**: `include/ge_visual_effect.h:48-72` - 参数设置

---

### 问题 2: 内存占用过高

**现象**: 渲染大图像时内存激增

**排查路径**:

1. 检查图像尺寸
   ```cpp
   auto dims = image->GetDimensions();
   GE_LOGI("Image size: %{public}u x %{public}u", dims.width, dims.height);
   ```

2. 启用缓存监控
   ```cpp
   // 检查缓存使用
   ```

3. 分块渲染
   ```cpp
   // 对超大图像分块处理
   ```

**证据来源**: `04_Build.md` - 内存问题相关

---

### 问题 3: Shader 编译失败

**现象**: 效果不显示，无报错

**排查路径**:

1. 启用详细日志
   ```bash
   # 设置日志级别
   ```

2. 检查 Shader 代码
   ```cpp
   // 查看 RuntimeEffect 编译结果
   auto effect = Drawing::RuntimeEffect::CreateForSkshaders(
       shaderCode, &errorText);
   if (!effect) {
       GE_LOGE("Shader compilation failed: %{public}s", errorText.c_str());
   }
   ```

3. 验证 Shader 语法
   ```bash
   # 使用 Skia skslc 工具验证
   ```

---

## 调试方法

### 日志输出

graphics_effect 使用 `GE_LOG_*` 系列宏进行日志输出：

| 宏 | 级别 | 用途 |
|---|------|------|
| `GE_LOGE` | ERROR | 错误日志 |
| `GE_LOGW` | WARNING | 警告日志 |
| `GE_LOGI` | INFO | 信息日志 |
| `GE_LOGD` | DEBUG | 调试日志 |

**启用日志**:
```bash
# 通过 hilog 查看
hilog | grep -E "GE_|GraphicsEffect"
```

**证据来源**: `include/ge_log.h`

### 性能追踪

使用 hitrace 进行性能分析：

```bash
# 启动追踪
hitrace --trace_begin graphics_effect

# 执行渲染操作

# 结束追踪
hitrace --trace_dump
```

### 核心转储分析

```bash
# 启用 core dump
ulimit -c unlimited

# 发生崩溃后
gdb ./program core
# 查看堆栈
bt full
```

---

## 常见问题 FAQ

### Q1: 如何添加新的动视效？

**答案**:

1. 在 `include/` 创建头文件，继承 `GEVisualEffect`
2. 在 `src/` 实现对应的 Shader Filter
3. 注册效果类型（如果需要）
4. 添加到 `BUILD.gn` 的 sources 列表

**参考**: `include/ge_grey_shader_filter.h` - 简单效果示例

---

### Q2: 如何调试 Shader 效果？

**答案**:

1. 使用 `Drawing::RuntimeEffect::CreateForSkshaders()` 单独验证 Shader
2. 将 Shader 代码输出到日志
3. 使用 RenderDoc 进行 GPU 调试

---

### Q3: 为什么效果在某些设备上不工作？

**答案**:

1. 检查 GPU 支持情况
2. 验证 Shader 版本兼容性
3. 检查内存限制
4. 查看设备特定的日志

---

### Q4: 如何降级到旧版 Skia？

**答案**:

1. 禁用 feature
   ```bash
   # 不启用 graphics_effect_feature_upgrade_skia
   ```

2. 确保 Skia 版本一致
   ```bash
   # 同步 third_party/skia
   ```

---

## 诊断工具

### 1. 单元测试

```bash
./build.sh --product-name <product> --build-target graphics_effect:GraphicsEffectTest
./bin/GraphicsEffectTest
```

### 2. Fuzz 测试

```bash
./build.sh --product-name <product> --build-target graphics_effect:fuzztest
./bin/fuzztest
```

### 3. 内存检测

```bash
# 使用 ASan 构建
./build.sh ... --enable-asan
```

---

## 相关文档

- [项目概述](01_Overview.md) - 定位和核心能力
- [架构说明](02_Architecture.md) - 详细架构设计
- [内部 API](03_InnerAPIs.md) - API 参考
- [构建指南](04_Build.md) - 编译配置
- [安全评审](05_Security.md) - 安全风险
