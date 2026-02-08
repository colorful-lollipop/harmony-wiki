# Telephony Data Storage Wiki - 文档说明

## 文档覆盖范围

本 Wiki 旨在为 OpenHarmony Telephony Data Storage（电话数据存储）模块提供完整的工程文档，帮助新人快速理解项目架构、API 接口、构建系统和安全风险。

### 涵盖内容

| 分类 | 状态 | 说明 |
|------|------|------|
| 项目定位与边界 | ✅ | 模块职责、核心能力、运行环境 |
| 目录结构 | ✅ | 模块职责划分（不含测试） |
| 架构说明 | ✅ | 组件图、数据流、线程模型 |
| 对外 API | ✅ | DataShare URIs、数据结构、权限 |
| 内部 API | ✅ | 模块接口、依赖方向、稳定性标注 |
| GN 构建 | ✅ | targets 列表、依赖、产物 |
| 编译产物 | ✅ | .so/.hap/配置文件及安装路径 |
| 安全评审 | ✅ | 攻击面、风险点、修复建议 |

### 未涵盖内容

| 分类 | 原因 |
|------|------|
| 测试代码 | 按规范忽略测试相关内容 |
| 第三方依赖内部实现 | 仅描述接口，不深入实现细节 |
| 运行时配置细节 | 超出代码证据范围 |

---

## 文档更新方式

### 何时需要更新 Wiki

当代码发生以下变更时，应同步更新 Wiki：

1. **新增模块**：添加新的数据 Ability 或服务
2. **API 变更**：新增、修改或删除 DataShare URI
3. **权限变更**：修改权限要求或添加新权限
4. **构建变更**：新增 BUILD.gn targets 或变更依赖关系
5. **安全机制变更**：修改权限检查逻辑或认证流程

### 更新流程

```bash
# 1. 检出代码仓库
git clone https://gitee.com/openharmony/telephony_data.git

# 2. 进入工作区
cd telephony_data/wiki/_work

# 3. 更新 NOTES.md 记录变更
vim NOTES.md

# 4. 编辑对应的 Wiki 文档
vim ../01_Overview.md  # 或其他需要更新的文档

# 5. 验证链接和术语一致性
# 检查 SUMMARY.md 导航链接

# 6. 提交变更
git add .
git commit -m "docs: update wiki for XXX change"
git push
```

### 质量检查清单

更新后请确认：

- [ ] 所有链接指向正确的文档
- [ ] 术语使用一致（DataShare vs DataAbility）
- [ ] 关键结论有代码证据（路径+符号）
- [ ] API 表格包含完整字段
- [ ] 安全风险点有修复建议

---

## 文档生成信息

| 项目 | 值 |
|------|-----|
| 生成时间 | 2024-02-06 |
| 生成工具 | OpenHarmony Wiki Generator Agent |
| 代码版本 | telephony_data @ HEAD |
| 目标读者 | OpenHarmony 开发者、安全评审人员 |
| 默认语言 | 中文（除非另有说明） |

---

## 相关链接

- [OpenHarmony Telephony 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/telephony.md)
- [DataShare 框架文档](https://developer.harmonyos.com)
- [ telephony_data 源码](https://gitee.com/openharmony/telephony_data)

---

*最后更新: 2024-02-06*
