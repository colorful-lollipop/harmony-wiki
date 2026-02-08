# API 参考

## 概述

本文档汇总 `app_samples` 仓库示例中使用的 OpenHarmony SDK API。

> **说明**: 本仓库为纯 ArkTS 示例，**不涉及 Native N-API**。所有 API 均为系统内置 ArkTS API。

## API 分类

### Ability 能力

| API | 用途 | 示例 |
|-----|------|------|
| [startAbilityForResult](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-ability-kit/js-apis-inner-application-uiAbilityContext.md#startabilityforresult) | 启动 Ability 并获取结果 | CameraSample |

### UI 组件

| API | 用途 | 示例 |
|-----|------|------|
| [Image](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-basic-components-image.md) | 图片展示 | CameraSample, FilesSample |
| [TextInput](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-basic-components-textinput.md) | 文本输入 | CameraSample |
| [ForEach](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-generators-foreach.md) | 列表渲染 | CameraSample |
| [Column/Row](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-container-column.md) | 布局容器 | ComponentSample |
| [List](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-container-list.md) | 列表组件 | ComponentSample |
| [Scroll](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-container-scroll.md) | 滚动容器 | ComponentSample |
| [Flex](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-container-flex.md) | 弹性布局 | ComponentSample |
| [CalendarView](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-container-calendarsignin.md) | 日历组件 | ComponentSample |

### 数据管理

| API | 用途 | 示例 |
|-----|------|------|
| [LazyForEach](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-generators-lazyforeach.md) | 懒加载数据源 | ComponentSample |
| [IDataSource](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-data-source.md) | 数据源接口 | ComponentSample |

### 文件操作

| API | 用途 | 示例 |
|-----|------|------|
| [fs](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-core-file-kit/js-apis-file-fs.md) | 文件系统操作 | FilesSample |
| [RawFileDescriptor](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-core-file-kit/js-apis/rawfile.md) | Rawfile 描述符 | FilesSample |
| [zlib.compressFile](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-core-file-kit/js-apis-zlib.md#zlibcompressfile) | 文件压缩 | FilesSample |
| [zlib.decompressFile](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-core-file-kit/js-apis-zlib.md#zlibdecompressfile) | 文件解压 | FilesSample |

### 工具类

| API | 用途 | 示例 |
|-----|------|------|
| [hilog](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-hilog-kit/js-apis-hilog.md) | 日志输出 | 所有示例 |
| [BusinessError](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-basic-framework-kit/js-apis-error.md) | 错误处理 | 所有示例 |
| [promptAction](https://docs.openharmony.cn/pages/v5.1/zh-cn/application-dev/reference/apis-arkui/arkui-ts/ts-universal-attributes-modal.md) | 弹窗提示 | FilesSample |

## 常用 API 调用模式

### 1. 启动 Ability 获取结果

```typescript
import { common, Want } from '@kit.AbilityKit';

// 启动相机并获取照片
context.startAbilityForResult(want,
  (err: BusinessError, result: common.AbilityResult) => {
    if (err.code) {
      hilog.error(0x0000, 'Tag', `failed: ${err.code}`);
      return;
    }
    if (result.resultCode === 0) {
      const uri: string = result.want?.parameters?.['KEY_RESULT'] as string;
    }
  });
```

### 2. 懒加载数据

```typescript
import { IDataSource, LazyDataSource } from '@kit.ArkUI';

// 实现 IDataSource 接口
class MyDataSource implements IDataSource {
  // ... required methods
}

// 使用懒加载
const dataSource = new LazyDataSource(dataList);
List() {
  LazyForEach(dataSource, (item: Item) => {
    // ...
  })
}
```

### 3. 文件复制

```typescript
import { rawfile, context } from '@kit.BasicServicesKit';

// 获取 Rawfile 描述符
const fileDescriptor = rawfile.getRawFileDescriptor('large_image.jpg');

// 创建文件流并复制
const srcFile = fs.openSync(fileDescriptor.fd, fs.OpenMode.READ);
const destFile = fs.openSync(destPath, fs.OpenMode.CREAT | fs.OpenMode.WRITE);

// 使用 buffer 复制
const buffer = new ArrayBuffer(bufferSize);
while (fs.readSync(srcFile.fd, buffer) > 0) {
  fs.writeSync(destFile.fd, buffer);
}
```

### 4. 错误处理

```typescript
try {
  // 业务代码
} catch (err) {
  const error = err as BusinessError;
  hilog.error(0x0000, 'Tag', `error: ${error.code}, ${error.message}`);
}
```

## API 版本兼容性

| API 级别 | 说明 |
|---------|------|
| API 20 | 当前示例使用的目标 SDK 版本 |
| ArkTS 1.2 | 当前示例使用的 ArkTS 语言版本 |

> **注意**: API 兼容性以 [OpenHarmony 官方文档](https://docs.openharmony.cn) 为准。
