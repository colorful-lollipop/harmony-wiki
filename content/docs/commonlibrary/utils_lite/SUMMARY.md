# Wiki 导航目录

> 本文档提供 utils_lite Wiki 的完整导航，包含新人学习路线和安全研究路线。

---

## 📘 新人学习路线

**目标**: 快速理解项目定位、掌握 API 使用、了解基本架构

**预计时间**: 30-45 分钟

### Step 1: 项目概览 (5 分钟)

1. **[README](README.md)** - 文档说明、覆盖范围、更新方式
2. **[00_Overview](00_Overview.md)** - 项目定位、核心能力、运行环境
   - 了解 utils_lite 解决什么问题
   - 明确不同平台支持的功能差异
   - 掌握基本概念（JSI、KAL、HAL）

### Step 2: 架构理解 (10 分钟)

3. **[02_Architecture](02_Architecture.md)** - 架构设计、组件图、数据流
   - 理解分层架构（JSI → C → HAL/KAL）
   - 掌握数据流和调用关系
   - 了解线程模型和时序

4. **[01_Directory_Structure](01_Directory_Structure.md)** - 目录结构与模块职责
   - 定位核心代码位置
   - 理解各模块的职责划分
   - 建立代码导航地图

### Step 3: API 掌握 (15 分钟)

5. **[04_Inner_API](04_Inner_API.md)** - 内部 C/C++ API 与模块依赖
   - 学习 C API 使用方法
   - 了解内部接口契约
   - 掌握错误处理机制

6. **[03_NAPI_Reference](03_NAPI_Reference.md)** - N-API/JSI 接口完整参考
   - 学习 JS API 使用方法
   - 查看完整参数说明
   - 理解同步/异步调用

### Step 4: 构建与故障 (10 分钟)

7. **[05_GN_Build](05_GN_Build.md)** - GN Targets 与构建配置
   - 了解 Feature 开关
   - 掌握构建流程

8. **[06_Build_Artifacts](06_Build_Artifacts.md)** - 编译产物与加载关系
   - 理解产物类型和位置
   - 了解库的加载关系

9. **[08_Troubleshooting](08_Troubleshooting.md)** - 常见问题与排查
   - 解决常见构建问题
   - 处理运行时错误

---

## 🛡️ 安全研究路线

**目标**: 快速识别攻击面、理解安全边界、评估潜在风险

**预计时间**: 45-60 分钟

### Step 1: 攻击面识别 (10 分钟)

1. **[README](README.md)** - 文档说明、覆盖范围
2. **[05_AttackSurface](05_AttackSurface.md)** - 攻击面分析
   - 识别所有外部输入入口
   - 定位敏感操作和权限点
   - 理解信任边界和隔离机制

### Step 2: 架构与数据流 (10 分钟)

3. **[02_Architecture](02_Architecture.md)** - 架构设计、组件图、数据流
   - 理解数据流和信任边界
   - 分析跨层调用路径
   - 定位安全关键点

4. **[04_Inner_API](04_Inner_API.md)** - 内部 C/C++ API 与模块依赖
   - 了解底层 API 实现
   - 分析权限检查机制
   - 理解资源管理

### Step 3: 输入验证分析 (10 分钟)

5. **[03_NAPI_Reference](03_NAPI_Reference.md)** - N-API/JSI 接口完整参考
   - 查看 JS API 参数说明
   - 分析输入验证机制
   - 定位参数解析代码

### Step 4: 安全风险评估 (15 分钟)

6. **[07_Security_Review](07_Security_Review.md)** - 安全风险评审
   - 评估已识别的风险
   - 分析触发路径和影响
   - 理解修复建议

### Step 5: 构建与配置 (10 分钟)

7. **[05_GN_Build](05_GN_Build.md)** - GN Targets 与构建配置
   - 了解安全相关 Feature 开关
   - 分析编译选项

8. **[06_Build_Artifacts](06_Build_Artifacts.md)** - 编译产物与加载关系
   - 理解安全加固机制
   - 分析库加载安全性

---

## 📚 完整目录

### 核心文档

```
Wiki/
├── README.md                          # 文档说明
├── SUMMARY.md                         # 本导航文件
├── 00_Overview.md                     # 项目概述
├── 01_Directory_Structure.md          # 目录结构
├── 02_Architecture.md                # 架构说明
├── 03_NAPI_Reference.md              # N-API 参考
├── 04_Inner_API.md                  # 内部 API
├── 05_GN_Build.md                   # GN 构建
├── 06_Build_Artifacts.md            # 编译产物
├── 05_AttackSurface.md             # 攻击面分析 ⭐ 新增
├── 07_Security_Review.md            # 安全风险评审
└── 08_Troubleshooting.md            # 故障排查
```

### 附录文档

```
appendix/
├── Callgraphs.md                    # 调用链图谱
└── Config_Flags.md                  # 配置开关
```

---

## 🔍 按功能索引

### 文件操作相关

- 文件系统 C API：`04_Inner_API.md#file-模块`
- 文件操作 JS API：`03_NAPI_Reference.md#filekit-接口`
- 文件操作安全：`07_Security_Review.md#风险-1-路径遍历漏洞`

### 键值存储相关

- KV 存储 JS API：`03_NAPI_Reference.md#kvstorekit-接口`
- KV 存储安全：`07_Security_Review.md#风险-2-kv-store-key-注入`

### 定时器相关

- 定时器 JS API：`03_NAPI_Reference.md#定时器-api`
- 定时器 C API：`04_Inner_API.md#kal_timer-模块`
- 定时器安全：`07_Security_Review.md#风险-5-定时器回调处理`

### 设备信息相关

- 设备信息 JS API：`03_NAPI_Reference.md#deviceinfokit-接口`
- 设备信息安全：`07_Security_Review.md#风险-3-设备信息泄露`

### 构建相关

- GN 配置：`05_GN_Build.md`
- 产物说明：`06_Build_Artifacts.md`
- Feature 开关：`appendix/Config_Flags.md`

---

## 🎯 按角色索引

### 应用开发者

1. `03_NAPI_Reference.md` - 查找 JS API 使用方法
2. `06_Build_Artifacts.md` - 了解如何链接库
3. `08_Troubleshooting.md` - 解决使用问题

**阅读顺序**: 新人学习路线

### 系统开发者

1. `05_GN_Build.md` - 修改构建配置
2. `02_Architecture.md` - 理解模块依赖
3. `04_Inner_API.md` - 使用内部 C API

**阅读顺序**: 新人学习路线

### 安全研究员

1. `05_AttackSurface.md` - 攻击面分析 ⭐
2. `07_Security_Review.md` - 安全风险分析
3. `03_NAPI_Reference.md` - API 安全考量
4. `appendix/Config_Flags.md` - 安全相关配置

**阅读顺序**: 安全研究路线

---

## 📋 版本历史

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 1.0 | 2026-02-06 | 初始版本 |
| 2.0 | 2026-02-06 | 添加双路线导航（新人/安全） |
| 2.1 | 2026-02-06 | 新增攻击面分析文档 |

---

## 📝 更新日志

- 2026-02-06：创建完整 Wiki 结构
- 2026-02-06：添加双路线导航（新人学习路线 + 安全研究路线）
- 2026-02-06：新增 05_AttackSurface.md 攻击面分析文档

## 完整目录

```
Wiki/
├── README.md                          # 文档说明
├── SUMMARY.md                          # 本导航文件
├── 00_Overview.md                      # 项目概述
├── 01_Directory_Structure.md           # 目录结构
├── 02_Architecture.md                  # 架构说明
├── 03_NAPI_Reference.md                # N-API 参考
├── 04_Inner_API.md                    # 内部 API
├── 05_GN_Build.md                     # GN 构建
├── 06_Build_Artifacts.md              # 编译产物
├── 07_Security_Review.md              # 安全评审
├── 08_Troubleshooting.md              # 故障排查
└── appendix/
    ├── Callgraphs.md                  # 调用链图谱
    └── Config_Flags.md                # 配置开关
```

## 按功能索引

### 文件操作相关

- 文件系统 API：`03_NAPI_Reference.md#文件操作-api`
- 内部实现：`04_Inner_API.md#file-模块`

### 键值存储相关

- KV 存储 API：`03_NAPI_Reference.md#kv-存储-api`
- 内部实现：`04_Inner_API.md#kv_store-模块`

### 定时器相关

- 定时器 API：`03_NAPI_Reference.md#定时器-api`
- KAL 实现：`04_Inner_API.md#kal_timer-模块`
- Timer Task：`04_Inner_API.md#timer_task-模块`

### 构建相关

- GN 配置：`05_GN_Build.md`
- 产物说明：`06_Build_Artifacts.md`

## 按角色索引

### 应用开发者

1. `03_NAPI_Reference.md` - 查找 JS API 使用方法
2. `06_Build_Artifacts.md` - 了解如何链接库
3. `08_Troubleshooting.md` - 解决使用问题

### 系统开发者

1. `05_GN_Build.md` - 修改构建配置
2. `02_Architecture.md` - 理解模块依赖
3. `04_Inner_API.md` - 使用内部 C API

### 安全工程师

1. `07_Security_Review.md` - 安全风险分析
2. `03_NAPI_Reference.md` - API 安全考量
3. `appendix/Config_Flags.md` - 安全相关配置

## 版本历史

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 1.0 | 2026-02-06 | 初始版本 |

## 更新日志

- 2026-02-06：创建完整 Wiki 结构
