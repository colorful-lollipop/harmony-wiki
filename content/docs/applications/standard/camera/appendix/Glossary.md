# 附录 A: 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| ArkTS | Ark TypeScript | OpenHarmony 应用开发语言，TypeScript 的超集 |
| ArkUI | Ark UI | OpenHarmony 声明式 UI 开发框架 |
| Ability | Ability | OpenHarmony 应用组件，包括 UIAbility 和 ExtensionAbility |
| UIAbility | UI Ability | 带 UI 界面的 Ability，是用户交互的入口 |
| ExtensionAbility | Extension Ability | 扩展 Ability，用于特定场景（如卡片、选择器等） |
| HAP | HarmonyOS Ability Package | OpenHarmony 应用安装包格式 |
| HAR | HarmonyOS Archive | OpenHarmony 静态共享包格式 |
| HSP | HarmonyOS Shared Package | OpenHarmony 动态共享包格式 |
| Entry | Entry Module | 应用入口模块，可独立安装运行 |
| Feature | Feature Module | 特性模块，用于功能拆分 |
| HAR | HAR Module | 静态共享模块，编译时打包到宿主 |
| Want | Want | OpenHarmony 中用于组件间通信的意图对象 |
| AbilityStage | Ability Stage | Ability 运行期的容器，管理 Ability 生命周期 |
| WindowStage | Window Stage | 窗口阶段，管理应用窗口生命周期 |
| XComponent | XComponent | 用于嵌入原生界面的组件，相机预览使用 |
| CaptureSession | Capture Session | 相机捕获会话，管理输入输出流 |
| CameraInput | Camera Input | 相机输入，对应物理相机设备 |
| PreviewOutput | Preview Output | 预览输出，将相机数据渲染到界面 |
| PhotoOutput | Photo Output | 拍照输出，捕获静态图片 |
| VideoOutput | Video Output | 录像输出，捕获视频流 |
| ImageReceiver | Image Receiver | 图片接收器，接收拍照数据 |
| AVRecorder | Audio Video Recorder | 音视频录制器 |
| Redux | Redux | 状态管理架构，Camera 应用使用类 Redux 方案 |
| Store | Store | 状态存储，保存应用全局状态 |
| Action | Action | 动作，描述状态变更的意图 |
| Reducer | Reducer | 纯函数，根据 Action 计算新状态 |
| EventBus | Event Bus | 事件总线，用于组件间解耦通信 |
| Worker | Worker | Web Worker，用于多线程处理 |
| Preference | Preference | 轻量级键值存储 |
| RDB | Relational Database | 关系型数据库 |
| EXIF | EXIF | 可交换图像文件格式，包含照片元数据（如位置） |
| Permission | Permission | 权限，控制应用对系统能力的访问 |
| user_grant | User Grant | 用户授权权限，需要运行时弹窗申请 |
| system_grant | System Grant | 系统授权权限，安装时自动授予 |
| Syscap | System Capability | 系统能力，表示设备支持的功能 |
| hvigor | hvigor | OpenHarmony 构建工具，类似 Gradle |
| ohpm | OpenHarmony Package Manager | OpenHarmony 包管理器 |
| API Version | API Version | OpenHarmony API 版本号，如 API 23 |
| SDK | SDK | 软件开发工具包 |
| Compile SDK | Compile SDK | 编译时使用的 SDK 版本 |
| Compatible SDK | Compatible SDK | 兼容的最低 SDK 版本 |
| Bundle | Bundle | 应用包，包含多个模块 |
| Bundle Name | Bundle Name | 应用唯一标识，如 com.ohos.camera |
| Module | Module | 模块，应用的功能单元 |
| ets | eTS | Extended TypeScript，ArkTS 文件扩展名 |
| ts | TypeScript | TypeScript 文件扩展名 |
| FA | Feature Ability | 旧版 Ability 类型（已废弃） |
| Stage | Stage Model | OpenHarmony 推荐的应用模型 |
