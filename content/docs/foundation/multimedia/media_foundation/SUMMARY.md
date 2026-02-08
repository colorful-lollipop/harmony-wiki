# HiStreamer Wiki 文档导航

## 文档索引

| 文档 | 说明 | 阅读顺序 |
|------|------|----------|
| [README](README.md) | 文档说明、更新方式 | **必读** |
| [00_Overview](00_Overview.md) | 项目概览、核心能力 | **必读** |
| [01_Architecture](01_Architecture.md) | 三层架构设计 | 推荐 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块职责 | 推荐 |
| [03_C_API](03_C_API.md) | C API 接口文档 | API使用者必读 |
| [04_Pipeline_Framework](04_Pipeline_Framework.md) | Pipeline 框架详解 | 开发者必读 |
| [05_Plugin_System](05_Plugin_System.md) | 插件系统架构 | 插件开发者必读 |
| [06_GN_Build](06_GN_Build.md) | 构建配置与 Targets | 构建相关必读 |
| [07_Build_Artifacts](07_Build_Artifacts.md) | 编译产物说明 | 部署相关必读 |
| [08_Security_Review](08_Security_Review.md) | **安全风险评审（增强版）** | **安全相关必读** |
| [09_Troubleshooting](09_Troubleshooting.md) | 常见问题与定位 | 故障排查 |

## 新人学习路线

### 路线 A：快速上手（10分钟）

```
README → 00_Overview（理解项目定位）
```

适合：只想了解 HiStreamer 是什么、能做什么

### 路线 B：API 使用者（30分钟）

```
README → 00_Overview → 03_C_API
```

适合：使用 C API 进行媒体开发的开发者

**关键文档**：
- 03_C_API：OH_AVBuffer、OH_AVFormat API 详解
- 03_C_API：错误码对照表

### 路线 C：Pipeline 开发者（1小时）

```
README → 00_Overview → 01_Architecture → 04_Pipeline_Framework
```

适合：需要理解 Pipeline 数据流、Filter 状态机的开发者

**关键文档**：
- 01_Architecture：三层架构图、数据流图
- 04_Pipeline_Framework：Filter 生命周期、状态机
- 02_Directory_Structure：代码导航速查

### 路线 D：插件开发者（1-2小时）

```
README → 00_Overview → 01_Architecture → 05_Plugin_System → 02_Directory_Structure
```

适合：需要开发自定义 Source/Demuxer/Codec/Sink 插件

**关键文档**：
- 05_Plugin_System：插件接口、实现模式、注册机制
- 02_Directory_Structure：插件目录导航
- 06_GN_Build：插件 Feature 开关

### 路线 E：构建/部署工程师（30分钟）

```
README → 00_Overview → 06_GN_Build → 07_Build_Artifacts
```

适合：需要定制编译、产物部署的工程师

**关键文档**：
- 06_GN_Build：Feature 开关详解
- 07_Build_Artifacts：产物清单、安装路径

---

## 安全研究路线

### 路线 F：安全审计（2小时）

```
README → 00_Overview → 01_Architecture → 08_Security_Review
```

适合：安全研究员、渗透测试工程师

**关键文档**：
- 08_Security_Review：**完整的攻击面分析、风险清单、修复建议**
- 01_Architecture：信任边界图
- 按代码证据索引：`_work/NOTES.md`

**快速检查清单**（08_Security_Review 第5节）：
- [ ] 文件路径是否经过规范化处理？
- [ ] AVBuffer capacity 是否有上限？
- [ ] Buffer index 是否经过边界验证？
- [ ] 插件加载是否有签名验证？
- [ ] HTTP 响应头是否有长度限制？
- [ ] 错误路径是否正确释放资源？

---

## 快速跳转

### 按功能分类

| 功能 | 相关文档 |
|------|----------|
| 播放/录制 | 00_Overview, 01_Architecture, 04_Pipeline_Framework |
| 编解码 | 01_Architecture, 05_Plugin_System |
| 文件格式 | 01_Architecture, 05_Plugin_System |
| 构建编译 | 06_GN_Build, 07_Build_Artifacts |
| 安全相关 | **08_Security_Review** |
| 问题排查 | 09_Troubleshooting |

### 按代码路径分类

| 目标 | 路径 |
|-----|------|
| N-API 入口 | 03_C_API |
| Pipeline 核心 | 04_Pipeline_Framework |
| 插件系统 | 05_Plugin_System |
| 构建配置 | 06_GN_Build |
| 安全分析 | 08_Security_Review |
| 代码证据 | `_work/NOTES.md` |
