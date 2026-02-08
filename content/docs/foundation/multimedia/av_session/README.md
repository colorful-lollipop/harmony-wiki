# AVSession 组件 Wiki

## 项目概述

AVSession（Audio/Video Session）是 OpenHarmony 多媒体子系统的核心组件，提供统一的媒体控制能力。用户可以通过系统播控中心对本端和组网内的远端音视频应用的播放行为进行控制，展示相关播放信息。

**源码路径**: `/foundation/multimedia/av_session`

**主要能力**:
- 统一的本地和分布式媒体播放控制
- 全局播控入口，展示媒体信息
- 分布式设备信息展示和远程控制
- 精简的 JS 接口供开发者快速构建媒体应用

---

## Wiki 覆盖范围

| 文档 | 状态 | 说明 |
|------|------|------|
| [README](README.md) | ✅ | 本文档 |
| [SUMMARY](SUMMARY.md) | ✅ | 全站导航 |
| [概览](00_Overview.md) | ✅ | 项目定位、核心能力、运行环境 |
| [架构](01_Architecture.md) | ✅ | 组件图、数据流、线程模型 |
| [N-API](02_NAPI.md) | ✅ | JS API 接口文档 |
| [Inner API](03_InnerAPI.md) | ✅ | Native 接口文档 |
| [构建](04_Build.md) | ✅ | GN Targets 与编译产物 |
| [安全](05_Security.md) | ✅ | 风险评审与修复建议 |

---

## 快速开始

### 获取所有会话
```javascript
import AVSessionManager from '@ohos.multimedia.avsession';

AVSessionManager.getAllSessionDescriptors().then((descriptors) => {
    console.log('Active sessions:', descriptors);
});
```

### 创建会话
```javascript
let session = await AVSessionManager.createAVSession(
    'mySession', 
    AVSessionManager.AVSESSION_TYPE_AUDIO, 
    elementName
);
```

---

## 相关链接

- [官方 API 文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-avsession.md)
- [开发指导](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/media/avsession-overview.md)
- [约束和限制](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/media/avsession-overview.md)
- [OpenHarmony 多媒体仓库](https://gitee.com/openharmony/multimedia_av_session)

---

## 更新说明

**文档版本**: 1.0  
**生成时间**: 2026-02-06  
**代码版本**: 基于 OpenHarmony master 分支  
**生成工具**: Sisyphus Wiki Agent
