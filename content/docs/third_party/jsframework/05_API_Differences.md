# 05_API_Differences - API 差异

## 5.1 API 变更总览

### 新增 API 分类

jsframework OH 适配版本相比上游 Weex 0.30.0 新增了大量 OH 特有的 API，主要分为以下几类：

| 类别         | 数量    | 说明                     |
| ------------ | ------- | ------------------------ |
| **系统模块** | 13 个   | 完全新增的 OH 系统模块   |
| **组件**     | 50+ 个  | OH 特有组件和扩展组件    |
| **API 方法** | 100+ 个 | 新增的组件方法和系统 API |

### 差异概览

| 维度           | 上游 Weex       | OH 适配版本         |
| -------------- | --------------- | ------------------- |
| **系统模块**   | ~8 个基础模块   | 13 个 + OH 特化模块 |
| **组件数量**   | ~30 个          | 50+ 个              |
| **动画系统**   | basic animation | ohos.animator       |
| **设备支持**   | Android/iOS     | 标准设备 + 穿戴设备 |
| **字节码格式** | JS Bundle       | JS Bundle + .abc    |

---

## 5.2 系统模块差异

### 模块对比表

| 模块名                 | Weex | OH 适配 | 差异     |
| ---------------------- | ---- | ------- | -------- |
| `system.router`        | ✅   | ✅      | OH 增强  |
| `system.app`           | ✅   | ✅      | OH 增强  |
| `system.prompt`        | ✅   | ✅      | OH 增强  |
| `system.configuration` | ❌   | ✅      | **新增** |
| `system.device`        | ✅   | ✅      | OH 增强  |
| `system.grid`          | ❌   | ✅      | **新增** |
| `system.mediaquery`    | ✅   | ✅      | OH 增强  |
| `system.resource`      | ❌   | ✅      | **新增** |
| `timer`                | ✅   | ✅      | 相同     |
| `animation`            | ✅   | ✅      | 相同     |
| `ohos.animator`        | ❌   | ✅      | **新增** |
| `digitalCrown`         | ❌   | ✅      | **新增** |

### 新增模块详解

#### 1. system.configuration

**功能**: 获取系统配置信息

**API 列表**:

| API         | 参数 | 返回值 | 说明                 |
| ----------- | ---- | ------ | -------------------- |
| `getLocale` | 无   | string | 获取当前系统语言区域 |

**使用示例**:

```javascript
import configuration from "@ohos.configuration";

const locale = configuration.getLocale();
console.log("Current locale:", locale);
```

#### 2. system.grid

**功能**: 获取系统布局信息

**API 列表**:

| API                   | 参数 | 返回值   | 说明             |
| --------------------- | ---- | -------- | ---------------- |
| `getSystemLayoutInfo` | 无   | GridInfo | 获取网格布局信息 |

**使用示例**:

```javascript
import grid from "@ohos.grid";

const layoutInfo = grid.getSystemLayoutInfo();
console.log("Grid columns:", layoutInfo.columns);
console.log("Grid rows:", layoutInfo.rows);
```

#### 3. system.resource

**功能**: 读取资源文件

**API 列表**:

| API        | 参数               | 返回值 | 说明             |
| ---------- | ------------------ | ------ | ---------------- |
| `readText` | resource: Resource | string | 读取资源文件内容 |

**使用示例**:

```javascript
import resource from "@ohos.resource";

const content = resource.readText($r("app.string.test"));
console.log("Resource content:", content);
```

#### 4. ohos.animator

**功能**: OH 动画器，提供更强大的动画能力

**API 列表**:

| API              | 参数                     | 返回值   | 说明                  |
| ---------------- | ------------------------ | -------- | --------------------- |
| `createAnimator` | options: AnimatorOptions | Animator | 创建动画器实例        |
| `create`         | 同上                     | Animator | createAnimator 的别名 |

**AnimatorOptions**:

| 属性         | 类型   | 说明              |
| ------------ | ------ | ----------------- |
| `duration`   | number | 动画持续时间 (ms) |
| `easing`     | string | 缓动函数          |
| `delay`      | number | 延迟时间 (ms)     |
| `iterations` | number | 迭代次数          |
| `fill`       | string | 填充模式          |
| `direction`  | string | 动画方向          |

**Animator 实例方法**:

| 方法      | 参数                                 | 说明     |
| --------- | ------------------------------------ | -------- |
| `play`    | 无                                   | 开始动画 |
| `pause`   | 无                                   | 暂停动画 |
| `stop`    | 无                                   | 停止动画 |
| `reset`   | 无                                   | 重置动画 |
| `finish`  | 无                                   | 完成动画 |
| `onframe` | callback: (progress: number) => void | 帧回调   |

**使用示例**:

```javascript
import animator from "@ohos.animator";

const anim = animator.createAnimator({
  duration: 1000,
  easing: "ease-in-out",
  iterations: 1,
});

anim.onframe = (progress) => {
  console.log("Progress:", progress);
};

anim.play();
```

#### 5. digitalCrown

**功能**: 数字表冠支持（穿戴设备专用）

**API 列表**:

| API                          | 参数                    | 说明             |
| ---------------------------- | ----------------------- | ---------------- |
| `setMonitorForCrownEvents`   | callback: CrownCallback | 开始监听表冠事件 |
| `clearMonitorForCrownEvents` | 无                      | 停止监听         |

**CrownCallback**:

| 事件类型 | 说明         |
| -------- | ------------ |
| `change` | 表冠旋转事件 |
| `click`  | 表冠点击事件 |

**使用示例**:

```javascript
import digitalCrown from "@ohos.digitalcrown";

digitalCrown.setMonitorForCrownEvents({
  change: (event) => {
    console.log("Crown rotated:", event.angle);
  },
  click: () => {
    console.log("Crown clicked");
  },
});
```

---

## 5.3 组件 API 差异

### 组件对比表

#### 基础组件

| 组件     | Weex | OH 适配 | 差异                |
| -------- | ---- | ------- | ------------------- |
| `div`    | ✅   | ✅      | 相同                |
| `text`   | ✅   | ✅      | 相同                |
| `image`  | ✅   | ✅      | 相同                |
| `button` | ✅   | ✅      | OH 新增 setProgress |
| `input`  | ✅   | ✅      | OH 新增方法         |
| `switch` | ✅   | ✅      | 相同                |

#### 容器组件

| 组件             | Weex | OH 适配 | 差异            |
| ---------------- | ---- | ------- | --------------- |
| `list`           | ✅   | ✅      | OH 新增链式动画 |
| `stack`          | ✅   | ✅      | 相同            |
| `flex`           | ✅   | ✅      | 相同            |
| `grid-container` | ❌   | ✅      | **新增**        |
| `tabs`           | ✅   | ✅      | OH 增强         |
| `swiper`         | ✅   | ✅      | OH 新增方法     |

#### OH 特有组件

| 组件           | Weex | OH 适配 | 功能说明     |
| -------------- | ---- | ------- | ------------ |
| `xcomponent`   | ❌   | ✅      | 原生组件渲染 |
| `camera`       | ❌   | ✅      | 相机预览     |
| `web`          | ❌   | ✅      | WebView      |
| `canvas`       | ❌   | ✅      | 画布绘制     |
| `video`        | ✅   | ✅      | OH 增强      |
| `dialog`       | ✅   | ✅      | OH 增强      |
| `picker`       | ✅   | ✅      | OH 增强      |
| `panel`        | ❌   | ✅      | **新增**     |
| `menu`         | ❌   | ✅      | **新增**     |
| `calendar`     | ❌   | ✅      | **新增**     |
| `chart`        | ❌   | ✅      | **新增**     |
| `colorpicker`  | ❌   | ✅      | **新增**     |
| `badge`        | ❌   | ✅      | **新增**     |
| `clock`        | ❌   | ✅      | **新增**     |
| `rating`       | ❌   | ✅      | **新增**     |
| `select`       | ❌   | ✅      | **新增**     |
| `stepper`      | ❌   | ✅      | **新增**     |
| `toolbar`      | ❌   | ✅      | **新增**     |
| `toolbar-item` | ❌   | ✅      | **新增**     |

---

## 5.4 XComponent 详解

### 概述

XComponent 是 OH 特有的组件，用于渲染原生内容，如地图、游戏引擎、相机预览等。

### 属性

| 属性        | 类型   | 说明                               |
| ----------- | ------ | ---------------------------------- |
| `id`        | string | 组件唯一标识                       |
| `type`      | string | 原生类型 ('surface' / 'component') |
| `surfaceId` | string | 渲染表面 ID                        |

### 方法

| 方法                       | 参数                                    | 返回值            | 说明           |
| -------------------------- | --------------------------------------- | ----------------- | -------------- |
| `getXComponentContext`     | 无                                      | XComponentContext | 获取组件上下文 |
| `getXComponentSurfaceId`   | 无                                      | string            | 获取表面 ID    |
| `setXComponentSurfaceSize` | size: { width: number, height: number } | void              | 设置表面大小   |

### 使用示例

```javascript
// 模板
<xcomponent
  id="mapComponent"
  type="surface"
  @load="onXComponentLoad"
/>

// 脚本
onXComponentLoad() {
  const xcomponent = this.$element('mapComponent')
  const context = xcomponent.getXComponentContext()
  const surfaceId = xcomponent.getXComponentSurfaceId()

  // 使用 native 接口初始化地图/游戏引擎
}
```

---

## 5.5 Camera 组件详解

### 概述

Camera 组件用于相机预览和拍照录像功能。

### 属性

| 属性       | 类型   | 说明    |
| ---------- | ------ | ------- |
| `id`       | string | 组件 ID |
| `deviceId` | string | 设备 ID |

### 方法

| 方法            | 参数                      | 返回值 | 说明     |
| --------------- | ------------------------- | ------ | -------- |
| `takePhoto`     | 无                        | void   | 拍照     |
| `startRecorder` | options?: RecorderOptions | void   | 开始录像 |
| `closeRecorder` | 无                        | void   | 结束录像 |

### 使用示例

```javascript
// 模板
<camera id="camera" deviceId="0" />;

// 脚本
const camera = this.$element("camera");

// 拍照
camera.takePhoto();

// 开始录像
camera.startRecorder({
  duration: 60000, // 60秒
  bitRate: 2000000,
});

// 停止录像
camera.closeRecorder();
```

---

## 5.6 Web 组件详解

### 概述

Web 组件基于 WebView，提供网页加载能力。

### 属性

| 属性  | 类型   | 说明     |
| ----- | ------ | -------- |
| `src` | string | 网页地址 |
| `id`  | string | 组件 ID  |

### 方法

| 方法                         | 参数 | 返回值               | 说明           |
| ---------------------------- | ---- | -------------------- | -------------- |
| `reload`                     | 无   | void                 | 重新加载网页   |
| `createIntersectionObserver` | 无   | IntersectionObserver | 创建交叉观察者 |

### 使用示例

```javascript
// 模板
<web id="web" src="https://www.example.com" />;

// 脚本
const web = this.$element("web");

// 刷新页面
web.reload();

// 创建交叉观察
web.createIntersectionObserver();
```

---

## 5.7 Canvas 组件详解

### 概述

Canvas 组件用于 2D/3D 画布绘制。

### 方法

| 方法         | 参数                            | 返回值                   | 说明           |
| ------------ | ------------------------------- | ------------------------ | -------------- |
| `getContext` | type: '2d' \| 'webgl'           | CanvasRenderingContext2D | 获取渲染上下文 |
| `toDataURL`  | type?: string, quality?: number | string                   | 导出图片       |

### 2D 上下文方法

| 方法        | 参数                       | 说明     |
| ----------- | -------------------------- | -------- |
| `fillRect`  | x, y, width, height        | 填充矩形 |
| `clearRect` | x, y, width, height        | 清除矩形 |
| `fillText`  | text, x, y                 | 绘制文本 |
| `drawImage` | image, x, y, width, height | 绘制图片 |

### 使用示例

```javascript
// 模板
<canvas id="canvas" style="width: 300px; height: 300px;" />;

// 脚本
const canvas = this.$element("canvas");
const ctx = canvas.getContext("2d");

// 绘制
ctx.fillStyle = "#ff0000";
ctx.fillRect(0, 0, 100, 100);

// 导出
const dataUrl = canvas.toDataURL();
```

---

## 5.8 API 使用限制

### 设备限制

| API/组件       | 限制设备  | 说明               |
| -------------- | --------- | ------------------ |
| `digitalCrown` | 穿戴设备  | 仅在穿戴设备上可用 |
| `camera`       | 手机/平板 | 需要相机权限       |
| `xcomponent`   | 标准设备  | 需要配置表面大小   |

### 权限要求

| 组件/模块         | 所需权限               |
| ----------------- | ---------------------- |
| `camera`          | ohos.permission.CAMERA |
| `system.resource` | 无需权限               |
| `ohos.animator`   | 无需权限               |

---

## 5.9 升级注意事项

### API 兼容性

| 场景           | 建议                        |
| -------------- | --------------------------- |
| 从 Weex 迁移   | 参考本节差异，更新 API 调用 |
| 从旧版 OH 升级 | 检查 API 签名变更           |
| 新功能开发     | 使用推荐的 OH 特有 API      |

### 常见迁移问题

| 问题                   | 解决方案                      |
| ---------------------- | ----------------------------- |
| `animation` 模块不可用 | 使用 `ohos.animator` 替代     |
| 缺少组件               | 检查是否需要导入额外模块      |
| 权限错误               | 在 config.json 中声明所需权限 |

---

## 相关文档

- [01_Overview.md](./01_Overview.md) - 库概览
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用方式
- [06_Security.md](./06_Security.md) - 安全风险分析
