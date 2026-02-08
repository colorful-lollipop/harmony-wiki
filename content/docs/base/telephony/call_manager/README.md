# Call Manager 模块 Wiki

**文档版本**: 1.0
**生成时间**: 2026-02-06
**最后更新**: 2026-02-06

---

## 文档覆盖范围

本 Wiki 旨在为 OpenHarmony `telephony_call_manager` 模块提供全面的技术文档覆盖，帮助开发者快速理解模块架构、API 使用、构建系统和安全考量。

### 已覆盖内容

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目定位与边界 | ✅ 完成 | 模块职责、核心能力 |
| 目录结构 | ✅ 完成 | 源码目录归类 |
| 架构说明 | 🔄 进行中 | 组件图、数据流 |
| 对外 API | 🔄 进行中 | N-API 和 JS API |
| GN 构建 | 🔄 进行中 | targets 和产物 |
| 安全评审 | ⏳ 待开始 | 风险分析 |

### 未覆盖内容

| 类别 | 说明 |
|------|------|
| 测试代码 | 根据规范，测试相关内容不纳入 Wiki |
| 运行时细节 | 需要进一步代码分析 |

---

## 使用指南

### 阅读顺序建议

**新人入门**:
1. [概览](00_Overview.md) → 2. [架构](01_Architecture.md) → 3. [API 参考](02_API_Reference.md)

**开发者**:
1. [API 参考](02_API_Reference.md) → 2. [构建系统](03_Build_System.md) → 3. [安全评审](04_Security_Review.md)

**问题排查**:
1. [常见问题](05_Troubleshooting.md) → 相关 API 章节

---

## 文档更新机制

### 更新触发条件

本 Wiki 应在以下情况下更新：

| 触发条件 | 响应行动 |
|----------|----------|
| 新增 N-API 接口 | 更新 `02_API_Reference.md` |
| 新增 GN target | 更新 `03_Build_System.md` |
| 安全相关修改 | 更新 `04_Security_Review.md` |
| 架构重构 | 更新 `01_Architecture.md` |

### 更新方式

```bash
# 1. 克隆仓库
git clone <telephony_call_manager_repo>

# 2. 编辑 wiki/ 目录下的相应文件
# 3. 提交修改
git commit -m "docs: update wiki for ..."

# 4. 提交 PR
```

---

## 快速链接

- [API 清单](02_API_Reference.md#api-清单)
- [权限要求](02_API_Reference.md#权限要求)
- [错误码说明](02_API_Reference.md#错误码)
- [构建产物](03_Build_System.md#编译产物)
- [安全风险](04_Security_Review.md)

---

## 参考链接

- OpenHarmony Telephony 子系统: https://gitee.com/openharmony/docs
- telephony_core_service: https://gitee.com/openharmony/telephony_core_service
- telephony_cellular_call: https://gitee.com/openharmony/telephony_cellular_call
