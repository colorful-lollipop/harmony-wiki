# 对外 N-API 接口

## 概述

graphic_2d 通过 N-API 提供对 Native 绘图能力的 JavaScript/ArkTS 接口访问。

### 模块清单

| JS 命名空间 | C++ 模块文件 | 导出类 | 绑定文件数 |
|-------------|--------------|--------|-----------|
| `graphics.drawing` | `drawing_module.cpp` | 22+ | 40+ |
| `uieffect` | `native_module_ohos_ui_effect.cpp` | 3 | 6 |
| `effectKit` | `native_module_ohos_effect.cpp` | 2 | 5 |
| `graphics.colorSpaceManager` | `color_space_manager_module.cpp` | 2 | 8 |
| `animation.windowAnimationManager` | `rs_window_animation_module.cpp` | 1 | 6 |
| `graphics.hdrCapability` | `hdr_capability_module.cpp` | 1 | 3 |
| `webgl` | `module.cpp` | 20+ | 35+ |

---

## graphics.drawing 模块

### Canvas 类

**JS 类名**: `Canvas`  
**C++ 实现**: `js_canvas.cpp`  
**静态方法**: `__createTransfer__`

**实例方法**:

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `clear(color)` | color: number | void | 清空画布 |
| `drawArc(...)` | 多种重载 | void | 绘制圆弧 |
| `drawRect(rect, paint)` | rect, paint | void | 绘制矩形 |
| `drawCircle(center, radius, paint)` | center, radius, paint | void | 绘制圆形 |
| `drawPath(path, paint)` | path, paint | void | 绘制路径 |
| `drawImage(image, x, y)` | image, x, y | void | 绘制图像 |
| `drawColor(color, blendMode?)` | color, blendMode | void | 填充颜色 |
| `drawOval(...)` | oval, paint | void | 绘制椭圆 |
| `drawPoint(point, paint)` | point, paint | void | 绘制点 |
| `drawPoints(points, paint)` | points, paint | void | 绘制多点 |
| `drawTextBlob(textBlob, x, y, paint)` | textBlob, x, y, paint | void | 绘制文本 |
| `attachPen(pen)` | pen | void | 绑定画笔 |
| `attachBrush(brush)` | brush | void | 绑定画刷 |
| `detachPen()` | - | void | 解除画笔 |
| `detachBrush()` | - | void | 解除画刷 |
| `save()` | - | number | 保存状态 |
| `restore()` | - | void | 恢复状态 |
| `restoreToCount(count)` | count | void | 恢复到指定状态 |
| `clipRect(...)` | rect, clipOp | void | 裁剪矩形 |
| `clipPath(path)` | path | void | 裁剪路径 |
| `clipRegion(region)` | region | void | 裁剪区域 |
| `setMatrix(matrix)` | matrix | void | 设置矩阵 |
| `resetMatrix()` | - | void | 重置矩阵 |
| `translate(dx, dy)` | dx, dy | void | 平移 |
| `rotate(degrees, px, py)` | degrees, px, py | void | 旋转 |
| `scale(sx, sy)` | sx, sy | void | 缩放 |
| `skew(sx, sy)` | sx, sy | void | 倾斜 |
| `getSaveCount()` | - | number | 获取保存计数 |
| `getWidth()` | - | number | 获取画布宽度 |
| `getHeight()` | - | number | 获取画布高度 |
| `isClipEmpty()` | - | boolean | 裁剪区是否为空 |
| `quickRejectRect(...)` | rect, paint | boolean | 快速拒绝检测 |
| `drawPixelMapMesh(...)` | pixelMap, vertexCount, texTuples, meshColors | void | 绘制像素图网格 |
| `drawRegion(...)` | region, paint | void | 绘制区域 |
| `drawShadow(...)` | path, z, lightPos, lightRadius, ambientShadowColor, spotShadowColor | void | 绘制阴影 |
| `drawBackground(...)` | brush | void | 绘制背景 |
| `drawRoundRect(...)` | roundRect, paint | void | 绘制圆角矩形 |
| `drawNestedRoundRect(...)` | outer, inner, paint | void | 绘制嵌套圆角矩形 |

**证据来源**: `interfaces/kits/napi/graphic/drawing/js_canvas.cpp`

---

### Font 类

**JS 类名**: `Font`  
**C++ 实现**: `js_font.cpp`  
**静态方法**: `createFont`

**实例方法**:

| 方法 | 说明 |
|------|------|
| `setSize(size)` | 设置字号 |
| `getSize()` | 获取字号 |
| `setTypeface(typeface)` | 设置字体 |
| `getTypeface()` | 获取字体 |
| `setScaleX(scale)` | 设置水平缩放 |
| `setSkewX(skew)` | 设置水平倾斜 |
| `setEdging(edging)` | 设置边缘处理 |
| `setHinting(hinting)` | 设置字形微调 |
| `enableSubpixel()` | 启用次像素 |
| `enableEmbolden()` | 启用加粗 |
| `isBaselineSnap()` | 是否基线对齐 |
| `getMetrics()` | 获取度量信息 |
| `measureText(text)` | 测量文本宽度 |
| `measureSingleCharacter(char)` | 测量单字符 |
| `countText(text)` | 字符计数 |
| `textToGlyphs(text)` | 转换为字形 |
| `createPathForGlyph(glyph, direction)` | 创建字形路径 |
| `getBounds()` | 获取边界 |
| `getTextPath(...)` | 获取文本路径 |

---

### Path 类

**JS 类名**: `Path`  
**C++ 实现**: `js_path.cpp`

**移动/直线操作**:

| 方法 | 说明 |
|------|------|
| `moveTo(x, y)` | 移动到 |
| `lineTo(x, y)` | 直线到 |
| `rMoveTo(dx, dy)` | 相对移动 |
| `rLineTo(dx, dy)` | 相对直线 |

**曲线操作**:

| 方法 | 说明 |
|------|------|
| `quadTo(x1, y1, x2, y2)` | 二次贝塞尔曲线 |
| `conicTo(x1, y1, x2, y2, w)` | 圆锥曲线 |
| `cubicTo(x1, y1, x2, y2, x3, y3)` | 三次贝塞尔曲线 |
| `arcTo(...)` | 圆弧到 |
| `rQuadTo(...)` | 相对二次贝塞尔 |
| `rCubicTo(...)` | 相对三次贝塞尔 |

**形状添加**:

| 方法 | 说明 |
|------|------|
| `addPolygon(points, close)` | 添加多边形 |
| `addOval(oval, startAngle, sweepAngle)` | 添加椭圆弧 |
| `addCircle(x, y, radius, dir)` | 添加圆形 |
| `addArc(arc, startAngle, sweepAngle)` | 添加圆弧 |
| `addRect(rect, dir)` | 添加矩形 |
| `addRoundRect(roundRect, dir)` | 添加圆角矩形 |
| `addPath(path)` | 添加路径 |

**路径操作**:

| 方法 | 说明 |
|------|------|
| `transform(matrix)` | 变换 |
| `contains(x, y)` | 是否包含点 |
| `set(otherPath)` | 设置路径 |
| `setFillType(fillType)` | 设置填充类型 |
| `getFillType()` | 获取填充类型 |
| `setLastPoint(x, y)` | 设置最后点 |
| `getBounds()` | 获取边界 |
| `close()` | 闭合路径 |
| `offset(dx, dy)` | 偏移 |
| `rewind()` | 重置 |
| `reset()` | 重置 |
| `op(path, op)` | 路径运算 |
| `getLength()` | 获取长度 |
| `getPositionAndTangent(fraction)` | 获取位置和切线 |
| `getSegment(start, stop, exact)` | 获取片段 |
| `approximate(precision)` | 近似 |
| `interpolate(...)` | 插值 |

---

### 其他 Drawing 类

| 类名 | 关键方法 | 说明 |
|------|----------|------|
| **Brush** | `setColor`, `setAntiAlias`, `setBlendMode`, `setShaderEffect`, `setColorFilter` | 画刷 |
| **Pen** | `setColor`, `setStrokeWidth`, `setAntiAlias`, `setBlendMode` | 画笔 |
| **Matrix** | `setValues`, `getValues`, `preTranslate`, `preScale`, `rotate`, `postTranslate` | 矩阵 |
| **ColorFilter** | `createBlendModeColorFilter`, `createComposeColorFilter` | 颜色滤镜 |
| **ImageFilter** | `createBlurImageFilter`, `createColorFilterImageFilter` | 图像滤镜 |
| **MaskFilter** | `createBlurMaskFilter` | 遮罩滤镜 |
| **PathEffect** | `createDashPathEffect`, `createCornerPathEffect` | 路径效果 |
| **ShaderEffect** | `createColorShader`, `createLinearGradient`, `createRadialGradient` | 着色器 |
| **ShadowLayer** | `createShadowLayer` | 阴影层 |
| **RoundRect** | 构造函数(rect, radii) | 圆角矩形 |
| **Lattice** | `createLattice` | 网格 |
| **Region** | `setRect`, `setPath`, `contains`, `op` | 区域 |
| **SamplingOptions** | 构造函数(filterMode) | 采样选项 |
| **Typeface** | `makeFromFile`, `getFamilyName`, `getUniqueID` | 字体 |
| **TextBlob** | `makeFromString`, `bounds` | 文本块 |
| **PathIterator** | `next`, `hasNext` | 路径迭代器 |

---

## uieffect 模块

### Filter 类

**JS 类名**: `Filter`  
**C++ 实现**: `filter_napi.cpp`

**常量**: `TileMode` (CLAMP, REPEAT, MIRROR, DECAL)

**实例方法**:

| 方法 | 参数 | 说明 |
|------|------|------|
| `blur(radius, tileMode)` | radius, tileMode | 高斯模糊 |
| `pixelStretch(pixels, tileMode)` | pixels, tileMode | 像素拉伸 |
| `waterRipple(...)` | 参数组 | 水波纹 |
| `flyInFlyOutEffect(...)` | 参数组 | 飞入飞出 |
| `colorGradient(...)` | 参数组 | 颜色渐变 |
| `distort(...)` | 参数组 | 扭曲 |
| `radiusGradientBlur(radius)` | radius | 径向渐变模糊 |
| `displacementDistort(...)` | 参数组 | 置换扭曲 |
| `edgeLight(...)` | 参数组 | 边缘光 |
| `directionLight(...)` | 参数组 | 方向光 |
| `bezierWarp(...)` | 参数组 | 贝塞尔扭曲 |
| `maskDispersion(...)` | 参数组 | 遮罩色散 |
| `hdrBrightnessRatio(ratio)` | ratio | HDR 亮度比 |
| `contentLight(...)` | 参数组 | 内容光 |
| `maskTransition(...)` | 参数组 | 遮罩过渡 |
| `variableRadiusBlur(...)` | 参数组 | 可变半径模糊 |
| `frostedGlass(blurRadius, saturation)` | blurRadius, saturation | 磨砂玻璃 |
| `frostedGlassBlur(radius)` | radius | 磨砂模糊 |

### VisualEffect 类

**JS 类名**: `VisualEffect`  
**C++ 实现**: `effect_napi.cpp`

**静态方法**: `createEffect`, `createBrightnessBlender`, `createHdrBrightnessBlender`, `createShadowBlender`

**实例方法**:
- `backgroundColorBlender(color)`
- `borderLight(...)`
- `colorGradient(...)`
- `liquidMaterial(...)`
- `frostedGlass(...)`

### Mask 类

**JS 类名**: `Mask`  
**C++ 实现**: `mask_napi.cpp`

**静态方法**:
- `createRippleMask`
- `createRadialGradientMask`
- `createPixelMapMask`
- `createWaveGradientMask`
- `createUseEffectMask`

---

## effectKit 模块

### ColorPicker 类

**JS 类名**: `ColorPicker`  
**C++ 实现**: `color_picker_napi.cpp`

**常量**:
- `PictureComplexityDegree`: UNKNOWN, PURE, MODERATE_COMPLEXITY, VERY_FLOWERY
- `PictureShadeDegree`: UNKNOWN, EXTREMELY_LIGHT, VERY_LIGHT, LIGHT, MODERATE_SHADE, DARK, EXTREMELY_DARK

**静态方法**: `createColorPicker(pixelMap)`

**实例方法**:

| 方法 | 说明 |
|------|------|
| `getMainColor()` | 获取主色 |
| `getMainColorSync()` | 同步获取主色 |
| `getLargestProportionColor()` | 获取最大比例色 |
| `getHighestSaturationColor()` | 获取最高饱和度色 |
| `getAverageColor()` | 获取平均色 |
| `isBlackOrWhiteOrGrayColor(color)` | 判断黑白灰色 |
| `getMorandiBackgroundColor()` | 获取莫兰迪背景色 |
| `getMorandiShadowColor()` | 获取莫兰迪阴影色 |
| `getDeepenImmersionColor()` | 获取加深沉浸色 |
| `getImmersiveBackgroundColor()` | 获取沉浸背景色 |
| `getImmersiveForegroundColor()` | 获取沉浸前景色 |
| `getComplexityDegree()` | 获取复杂度 |
| `getShadeDegree()` | 获取色度 |
| `getReverseColor(color)` | 获取反转色 |
| `getTopProportionColors()` | 获取前 N 比例色 |
| `getTopProportionColorsAndPercentage()` | 获取比例色和百分比 |
| `getAlphaZeroTransparentProportion()` | 获取透明像素比例 |

---

## graphics.colorSpaceManager 模块

### ColorSpaceManager

**JS 模块**: `graphics.colorSpaceManager`  
**C++ 实现**: `js_color_space_manager.cpp`

**常量 (ColorSpace 枚举)**:
- `ADAPTIVE_WIDE_GAMUT`
- `BT2020_HLG`
- `BT2020_PQ`
- `DCI_P3`
- `DISPLAY_P3`
- `SRGB`

**静态方法**: `create(colorSpace)`

**ColorSpace 对象方法**:
- `getColorSpaceName()` - 获取色彩空间名
- `getWhitePoint()` - 获取白点
- `getGamma()` - 获取伽马值
- `getColorSpacePrimaries()` - 获取原色

---

## animation.windowAnimationManager 模块

**JS 模块**: `animation.windowAnimationManager`  
**C++ 实现**: `rs_window_animation_manager.cpp`

**方法**:
- `setController(controller)` - 设置控制器
- `minimizeWindowWithAnimation(...)` - 最小化窗口动画

---

## graphics.hdrCapability 模块

**JS 模块**: `graphics.hdrCapability`  
**C++ 实现**: `hdr_capability_module.cpp`

**方法**:
- `getSupportedHDRTypes()` - 获取支持的 HDR 类型
- `isCapabilitySupported()` - 检查 HDR 能力

---

## webgl 模块

### WebGLRenderingContext

**JS 类名**: `WebGLRenderingContext` / `WebGL2RenderingContext`  
**C++ 实现**: `webgl_rendering_context.cpp`, `webgl2_rendering_context.cpp`

**WebGL API 完全覆盖** (30+ 对象类型):

| 对象类型 | 说明 |
|----------|------|
| `WebGLBuffer` | Buffer 对象 |
| `WebGLFramebuffer` | Framebuffer 对象 |
| `WebGLProgram` | Program 对象 |
| `WebGLShader` | Shader 对象 |
| `WebGLTexture` | Texture 对象 |
| `WebGLRenderbuffer` | Renderbuffer 对象 |
| `WebGLUniformLocation` | Uniform 位置 |
| `WebGLActiveInfo` | 活动信息 |
| `WebGLVertexArrayObject` | VAO 对象 |
| `WebGLSampler` | Sampler 对象 |
| `WebGLQuery` | Query 对象 |
| `WebGLSync` | Sync 对象 |

**关键方法类别**:
- Shader: `createShader`, `compileShader`, `getShaderParameter`
- Program: `createProgram`, `linkProgram`, `getProgramParameter`
- Buffer: `bindBuffer`, `bufferData`, `bufferSubData`
- Texture: `bindTexture`, `texImage2D`, `texParameter`
- Framebuffer: `bindFramebuffer`, `framebufferTexture2D`
- Drawing: `drawArrays`, `drawElements`, `clear`
- Pixels: `readPixels`, `clearColor`
- Uniforms: `uniform1f`, `uniform1i`, `uniformMatrix4fv`

---

## N-API 注册模式

### 注册方式 1: NAPI_MODULE 宏

```cpp
// WebGL 使用
NAPI_MODULE(libwebglnapi, Export)
```

### 注册方式 2: napi_module_register

```cpp
// 大多数模块使用 __attribute__((constructor)) 自动注册
napi_module_register(&g_module);
napi_module_register(&_module);
```

### 属性定义宏

| 宏 | 用途 |
|-----|------|
| `DECLARE_NAPI_FUNCTION` | 实例方法 |
| `DECLARE_NAPI_STATIC_FUNCTION` | 静态方法 |
| `DECLARE_NAPI_PROPERTY` | 属性 |
| `DECLARE_NAPI_STATIC_PROPERTY` | 静态属性 |

---

## 相关文档

- [架构说明](03_Architecture.md) - 绘图引擎架构
- [目录结构](02_Directory_Structure.md) - N-API 目录位置
- [构建文档](06_Build.md) - N-API 构建配置
