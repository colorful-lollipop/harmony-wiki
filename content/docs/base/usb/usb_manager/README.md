# OpenHarmony USB Manager Wiki

本 Wiki 文档覆盖 OpenHarmony USB Manager 子系统的完整技术细节，基于代码证据生成。

## 覆盖范围

### 已覆盖
- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ N-API 接口清单（JS API → C++ 实现映射）
- ✅ GN 构建配置与编译产物
- ✅ IPC/SA 架构设计
- ✅ System Ability (SA ID: 4201) 配置
- ✅ 安全事件与权限管理
- ✅ 攻击面分析（外部输入点和敏感操作）
- ✅ 安全风险评估（详细漏洞分析）
- ✅ 代码地图（快速定位指南）

### 未覆盖
- ❌ 具体实现代码细节（请查阅源码）
- ❌ 性能测试数据
- ❌ 历史版本变更记录

## 文档结构

\`\`\`
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航 + 双路线（新人/安全）
├── index.md               # 首页（快速索引）
│
├── 01_Overview.md          # 项目概览
├── 02_Architecture.md      # 架构与数据流
├── 03_CodeMap.md          # 目录结构与代码地图
├── 04_Interface.md         # 对外接口文档
│
├── 05_AttackSurface.md     # 攻击面分析
├── 06_SecurityReview.md    # 安全风险评估
│
├── 07_Build.md            # 构建与产物
├── 08_Internals.md        # 内部实现细节
│
├── _work/                 # 工作区
│   ├── ASSESSMENT.md       # 项目评估结果
│   ├── NOTES.md            # 代码证据汇总
│   └── PLAN.md            # 任务进度追踪
│
└── appendix/               # 附录
    └── callgraphs.md       # 关键调用链
\`\`\`

## 受众与阅读路线

### 新人学习路线

**目标**：快速理解项目、学习 API 使用、掌握架构设计

**推荐顺序**：
1. [项目概览](01_Overview.md) - 理解项目定位和功能边界
2. [对外接口文档](04_Interface.md) - 学习 JS API 使用方法
3. [架构与数据流](02_Architecture.md) - 理解三层架构和数据流向
4. [目录结构与代码地图](03_CodeMap.md) - 快速定位核心代码
5. [构建与产物](07_Build.md) - 了解编译配置（可选）
6. [内部实现细节](08_Internals.md) - 深入核心类设计（可选）

**预期效果**：
- ✅ 5 分钟理解项目定位
- ✅ 15 分钟找到核心代码位置
- ✅ 30 分钟理解基本架构
- ✅ 2 小时完成 API 学习和代码定位

### 安全研究路线

**目标**：快速识别攻击面、分析安全风险、定位漏洞点

**推荐顺序**：
1. [项目概览](01_Overview.md) - 了解功能边界和权限要求
2. [攻击面分析](05_AttackSurface.md) - 识别所有外部输入点和敏感操作（**优先级最高**）
3. [架构与数据流](02_Architecture.md) - 理解信任域划分和权限检查机制
4. [安全风险评估](06_SecurityReview.md) - 分析具体漏洞点、触发路径、修复建议
5. [对外接口文档](04_Interface.md) - 研究 API 参数和输入验证逻辑
6. [内部实现细节](08_Internals.md) - 查看关键函数实现和并发控制
7. [构建与产物](07_Build.md) - 了解 Feature 开关和安全编译选项（可选）

**预期效果**：
- ✅ 30 分钟识别所有攻击面
- ✅ 1 小时理解信任边界
- ✅ 3 小时完成风险评估
- ✅ 5 小时定位关键漏洞点

## 核心内容概览

### 快速索引

#### N-API 接口

| 模块 | JS API | 说明 |
|------|--------|------|
| usb | `getDevices()` | 获取 USB 设备列表 |
| usb | `requestRight()` | 请求设备权限 |
| usb | `bulkTransfer()` | 批量传输 |
| usbmanager | `setCurrentFunctions()` | 设置 USB 功能 |
| usbmanager | `setPortRoles()` | 设置端口角色 |
| serial | `open()` | 打开串口 |
| serial | `read()/write()` | 串口读写 |

#### 构建产物

| 产物 | 类型 | 路径 |
|------|------|------|
| libusbservice.z.so | System Ability | system/lib64/ |
| libusb.z.so | N-API | system/lib64/module/ |
| libusbmanager.z.so | N-API | system/lib64/module/ |
| libserial.z.so | N-API | system/lib64/module/usbmanager/ |

#### SA 配置

- **SA ID**: 4201
- **进程**: usb_service
- **产物**: libusbservice.z.so
- **自动重启**: 是

## 更新方式

当代码发生以下变更时，需要同步更新 Wiki：

1. 新增/删除 N-API 接口
2. 修改 SA ID 或 IPC 接口
3. 变更 Feature Flags 配置
4. 新增/删除构建产物
5. 修改权限校验逻辑
6. 修改架构设计或模块依赖

## 质量标准

本 Wiki 遵循以下质量标准：

- ✅ **证据优先**：所有技术结论都有代码证据支撑（文件路径、行号、代码片段）
- ✅ **受众导向**：明确区分新人学习路线和安全研究路线
- ✅ **链接有效**：所有内部链接正确且可访问
- ✅ **术语统一**：技术术语使用一致
- ✅ **代码片段**：关键代码有语法高亮和上下文说明

## 生成信息

- **代码版本**: OpenHarmony USB Manager v3.1.0
- **生成时间**: 2026-02-07
- **证据来源**:
  - `interfaces/kits/js/napi/` (N-API 实现)
  - `interfaces/innerkits/` (IPC 接口）
  - `services/native/` (服务实现）
  - `services/BUILD.gn` (构建配置）
  - `sa_profile/4201.json` (SA 配置）
  - `bundle.json` (组件元信息）

## 相关链接

- [项目 README](../README.md) - 项目说明和示例
- [完整导航](SUMMARY.md) - 全站导航和阅读路线
- [项目评估](./_work/ASSESSMENT.md) - Phase 0 项目评估结果
- [代码证据](./_work/NOTES.md) - 代码证据汇总

---

**最后更新**: 2026-02-07
