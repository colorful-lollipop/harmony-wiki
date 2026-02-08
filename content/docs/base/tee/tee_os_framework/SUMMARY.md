# 文档导航

## 核心文档

| 文档 | 描述 | 阅读优先级 |
|------|------|-----------|
| [README](README.md) | 文档说明与使用指南 | ⭐ 必读 |
| [00_Overview](00_Overview.md) | 项目定位、边界、核心能力 | ⭐ 必读 |
| [01_Architecture](01_Architecture.md) | 组件图、数据流、线程模型 | ⭐ 必读 |
| [02_Module_Detail](02_Module_Detail.md) | 各模块职责与接口 | 🔧 开发参考 |
| [03_Native_API](03_Native_API.md) | GP 标准 API 与 Native 接口 | 🔧 API 参考 |
| [04_Build_System](04_Build_System.md) | 构建配置与编译产物 | 🔧 构建参考 |
| [05_Security_Review](05_Security_Review.md) | 安全风险评审 | ⚠️ 安全相关 |

## 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图示 |

## 快速跳转

### 按功能查找

- **TA 开发** → `03_Native_API.md` → GP API 部分
- **服务开发** → `02_Module_Detail.md` → Services 部分
- **驱动开发** → `02_Module_Detail.md` → Drivers 部分
- **构建编译** → `04_Build_System.md`
- **安全审计** → `05_Security_Review.md`

### 按场景查找

| 场景 | 文档章节 |
|------|----------|
| 理解 TEE 架构 | `01_Architecture.md` |
| 新增 TA 模块 | `03_Native_API.md` TA Entry Points |
| 新增驱动 | `02_Module_Detail.md` → Driver Manager |
| 定位安全风险 | `05_Security_Review.md` |
| 编译产物路径 | `04_Build_System.md` → 产物清单 |

## 阅读路线图

```
新人入门
├── 第1步: 阅读 00_Overview.md 了解项目定位
├── 第2步: 阅读 01_Architecture.md 理解整体架构
├── 第3步: 按需阅读 02_Module_Detail.md 模块详情
└── 第4步: 开发时查阅 03_Native_API.md API 参考

安全审计
├── 必读 05_Security_Review.md 了解攻击面
└── 结合 01_Architecture.md 理解信任边界
```

## 文档约定

- **代码路径**：使用 `path:line` 格式引用代码位置
- **API 引用**：Native API 使用 `module.function()` 格式
- **TODO**：标记为 `TODO(需确认)` 表示证据不足待验证
- **警告**：⚠️ 标记安全相关重要信息
