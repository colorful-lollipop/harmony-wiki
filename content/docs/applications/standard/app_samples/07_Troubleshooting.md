# 常见问题

## 构建问题

### Q1: SDK 版本不匹配

**问题描述**:
```
error: compatibleSdkVersion "6.0.0(20)" does not match installed SDK.
```

**原因**: 安装的 DevEco Studio SDK 版本与项目要求的版本不一致。

**解决方案**:

1. 检查 DevEco Studio 版本
   - 需要版本: 6.0.0.43 或更高
   - 菜单: Help > About

2. 检查 SDK 版本
   - 菜单: File > Project Structure > SDK
   - 确认安装的 SDK 版本为 6.0.0(20)

3. 修改 build-profile.json5（如果需要降级）

```json5
{
  "products": [{
    "compatibleSdkVersion": "6.0.0(20)",  // 修改为已安装版本
    "targetSdkVersion": "6.0.0(20)"
  }]
}
```

### Q2: ArkTS 版本不兼容

**问题描述**:
```
error: ArkTS version '1.2' is not compatible with current SDK.
```

**原因**: 代码使用了当前 SDK 不支持的 ArkTS 语法。

**解决方案**:

1. 升级 DevEco Studio 到最新版本
2. 或修改语法为兼容版本

**常见不兼容语法**:

| 语法 | ArkTS 1.0 | ArkTS 1.1+ |
|-----|----------|-----------|
| @Prop 继承 | 不支持 | 支持 |
| 泛型约束 | 有限制 | 完全支持 |

### Q3: 签名配置缺失

**问题描述**:
```
error: No signing config found for release build.
```

**原因**: release 构建需要签名配置。

**解决方案**:

1. 生成签名证书
   - 菜单: Build > Generate Key and CSR
   - 填写证书信息

2. 配置签名
   - 菜单: File > Project Structure > Signing
   - 启用 "Automatically generate signature"

### Q4: 资源文件缺失

**问题描述**:
```
error: resource not found: [resource_name]
```

**原因**: 引用的资源文件不存在。

**解决方案**:

1. 检查 resources 目录结构
2. 确认资源文件名大小写
3. 验证 $r() 引用格式

```typescript
// 错误示例
Image($r('app.media.icon'))  // icon.png 不存在

// 正确示例  
Image($r('app.media.icon'))  // 确认文件名为 icon.png
```

### Q5: hvigor 构建失败

**问题描述**:
```
hvigor: ERROR: build Failed
```

**解决方案**:

1. 清理构建缓存

```bash
./hvigorw clean
rm -rf build/ entry/build/
```

2. 检查 Java 版本

```bash
java -version
# 需要 Java 17+
```

3. 查看详细日志

```bash
./hvigorw assembleDebug --stacktrace
```

## 运行问题

### Q6: 设备连接失败

**问题描述**:
```
error: no device found
```

**解决方案**:

1. 启用开发者模式
   - 设置 > 关于 > 连续点击版本号
   - 设置 > 系统 > 开发者选项 > 开启 USB 调试

2. 检查 USB 连接
   - 更换 USB 数据线
   - 选择 "传输文件" 模式

3. 授权设备

```bash
hdc list targets
hdc connect [device_ip]
```

### Q7: 安装失败

**问题描述**:
```
error: install failed due to different signature
```

**原因**: 应用签名与已安装版本不一致。

**解决方案**:

1. 卸载已安装版本

```bash
hdc shell bm uninstall -n com.example.entry
```

2. 或使用 debug 签名重新构建

### Q8: 启动崩溃

**问题描述**:
```
error: app start failed
```

**解决方案**:

1. 检查日志

```bash
hdc shell hilog | grep -E "ERROR|FATAL"
```

2. 常见原因
   - module.json5 配置错误
   - 页面路由未声明
   - Ability 配置缺失

### Q9: 页面跳转失败

**问题描述**:
```
error: page not found: pages/Example/Example
```

**原因**: 页面未在 main_pages.json 中声明。

**解决方案**:

检查 `entry/src/main/resources/profile/main_pages.json`:

```json5
{
  "src": [
    {
      "name": "Index",
      "srcEntry": "ets/pages/Index.ets"
    },
    {
      "name": "Example",  // 添加缺失页面
      "srcEntry": "ets/pages/Example/Example.ets"
    }
  ]
}
```

### Q10: 图片资源不显示

**问题描述**:
Image 组件显示空白或报错。

**解决方案**:

1. 检查资源路径

```typescript
// 正确: 使用 $r() 引用
Image($r('app.media.image_name'))

// 错误: 直接使用路径
Image('resources/base/media/image_name.png')
```

2. 确认资源位置
   - base/media/ 存放默认分辨率
   - dark/media/ 存放深色主题

3. 检查图片格式
   - 支持: PNG, JPG, SVG
   - 确认文件扩展名正确

## 调试问题

### Q11: hilog 日志不显示

**问题描述**:
使用 hilog 输出的日志看不到。

**解决方案**:

1. 确认日志标签

```typescript
// 使用自定义标签
hilog.error(0x0000, 'MyTag', 'message');

// 过滤日志
hilog | grep 'MyTag'
```

2. 检查日志级别

```bash
hilog | grep -E "Error|Warn"
```

### Q12: 断点调试无效

**问题描述**:
IDE 断点无法触发。

**解决方案**:

1. 使用 debug 构建

```bash
./hvigorw assembleDebug
```

2. 确认调试配置
   - Run > Debug Configurations
   - 选择正确的设备

3. 检查代码是否压缩
   - release 构建可能优化掉断点

### Q13: 性能分析

**问题描述**:
应用运行卡顿。

**解决方案**:

1. 使用性能分析器
   - DevEco Studio > Profiler
   - 选择 CPU / Memory 分析

2. 检查性能瓶颈
   - 大量 List 渲染（使用 LazyForEach）
   - 大图片加载
   - 复杂动画

3. 优化建议
   - 使用懒加载
   - 图片压缩
   - 减少嵌套层级

## 兼容性问题

### Q14: 不同设备显示异常

**问题描述**:
页面在平板上显示错乱。

**解决方案**:

1. 使用响应式布局

```typescript
Column() {
  if (isTablet()) {
    // 平板布局
  } else {
    // 手机布局
  }
}
```

2. 使用自适应能力

```typescript
// 使用 Grid 替代固定列数
Grid() {
  // 根据屏幕宽度调整列数
}
```

### Q15: 深色主题不生效

**问题描述**:
开启深色模式后界面无变化。

**解决方案**:

1. 配置深色资源

```
resources/
├── base/element/color.json
└── dark/element/color.json  // 深色主题
```

2. 检查颜色引用

```typescript
// 使用资源引用（自动适配主题）
Text('Hello')
  .backgroundColor($r('app.color.background'))
```

## 最佳实践

### 开发环境检查清单

- [ ] DevEco Studio 版本 >= 6.0.0.43
- [ ] SDK 版本 = 6.0.0(20)
- [ ] Java 版本 = 17+
- [ ] Node.js 版本 >= 18
- [ ] hvigor 配置正确

### 构建前检查

- [ ] 资源文件存在且命名正确
- [ ] 页面路由已声明
- [ ] module.json5 配置完整
- [ ] 签名配置（release 构建）

### 提交前检查

- [ ] 代码编译通过
- [ ] 无 lint 错误
- [ ] 资源文件无缺失
- [ ] README 文档更新
