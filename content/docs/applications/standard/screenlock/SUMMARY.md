# Wiki 导航

> OpenHarmony 锁屏应用 - 完整文档导航

---

## 📖 核心文档

### [README](README.md)
文档首页，包含项目基本信息和阅读建议。

### [00. 项目概览](00_Overview.md)
- 项目定位与边界
- 核心能力清单
- 运行环境要求
- 关键概念解释

### [01. 项目边界](01_Project_Boundary.md)
- 功能边界
- 职责范围
- 与系统服务的关系
- 外部依赖

### [02. 目录结构](02_Directory_Structure.md)
- 完整目录树
- 13个模块职责说明
- 文件命名规范
- 模块依赖关系

### [03. 架构设计](03_Architecture.md)
- 整体架构图（Mermaid）
- 组件关系图
- 数据流图
- 线程模型
- 关键时序图

---

## 🔌 API文档

### [04. 对外API](04_External_API.md)
- 系统服务使用清单
- @ohos模块调用统计
- 权限与API对应关系
- 关键调用链

### [05. 内部API](05_Internal_API.md)
- Manager类接口
- EventBus接口
- ViewModel接口
- 模块间通信接口

---

## ⚙️ 构建与产物

### [06. 构建系统](06_Build_System.md)
- Hvigor构建配置
- build-profile.json5解析
- 模块构建配置
- 构建流程

### [07. 编译产物](07_Artifacts.md)
- 产物清单
- .hap包结构
- 安装路径
- 运行时加载关系

---

## 🔒 安全与运维

### [08. 安全风险评审](08_Security.md)
- 攻击面分析
- 信任边界
- 21个权限风险分析
- 可被利用点（基于证据）
- 修复建议

### [09. 常见问题与调试](09_Troubleshooting.md)
- 构建问题
- 运行问题
- 调试方法
- 日志分析

---

## 📎 附录

### [附录A: 调用链](appendix/Callgraphs.md)
- 锁屏启动调用链
- 解锁流程调用链
- 通知显示调用链
- 事件传播调用链

### [附录B: 配置标志](appendix/Config_Flags.md)
- 编译期配置
- 功能开关
- 调试标志

---

## 🎯 推荐阅读顺序

### 新人入门路线
```
README → 00_Overview → 02_Directory_Structure → 03_Architecture → 08_Security
```

### 开发参考路线
```
02_Directory_Structure → 03_Architecture → 04_External_API → 05_Internal_API → appendix/Callgraphs
```

### 安全审计路线
```
01_Project_Boundary → 04_External_API → 08_Security → appendix/Callgraphs
```

---

## 🔗 外部链接

- [OpenHarmony官方文档](https://docs.openharmony.cn/)
- [ArkTS语言指南](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/quick-start/arkts-get-started.md)
- [ArkUI开发文档](https://docs.openharmony.cn/pages/v4.1/zh-cn/application-dev/ui/arkui-overview.md)

---

*最后更新: 2026-02-06*
