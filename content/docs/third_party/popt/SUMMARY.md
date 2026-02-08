# 阅读路线建议

## 根据角色的阅读建议

### 如果你是... 系统开发者 (想集成 popt)
**阅读顺序**:
1. [README.md](./README.md) - 快速了解
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖示例
3. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建配置

**关键信息**:
- 依赖方式: `external_deps = ["popt:popt_static"]`
- 头文件路径: `//third_party/popt/src/popt.h`
- 参考示例: `third_party/gptfdisk`

---

### 如果你是... 维护者 (需升级版本)
**阅读顺序**:
1. [README.md](./README.md) - 确认当前状态
2. [02_Patches.md](./02_Patches.md) - 确认无 Patch 需迁移
3. [03_Build_Integration.md](./03_Build_Integration.md) - 检查 BUILD.gn 兼容性
4. [06_Security.md](./06_Security.md) - 检查安全公告

**关键信息**:
- 本库 **无 Patch**，升级只需替换源码
- 需关注 `config.h` 是否需要更新
- 需验证 gptfdisk 功能是否正常

---

### 如果你是... 安全审计人员
**阅读顺序**:
1. [README.md](./README.md) - 了解库用途
2. [02_Patches.md](./02_Patches.md) - 确认无安全 Patch
3. [06_Security.md](./06_Security.md) - 完整安全分析
4. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解使用范围

**关键信息**:
- 使用范围有限 (仅 gptfdisk)
- 输入来源受控 (命令行参数)
- 无已知高危 CVE

---

### 如果你是... 新接触该库的开发者
**阅读顺序**:
1. [README.md](./README.md) - 概览
2. [01_Overview.md](./01_Overview.md) - 库功能介绍
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看实际使用示例
4. 阅读上游文档: `popt.pdf`

**关键信息**:
- popt 是 getopt 的增强替代
- 支持自动帮助生成
- 有完整的 API 文档

---

## 文档依赖关系

```
README.md
    ├── 01_Overview.md (基础信息)
    ├── 02_Patches.md (适配分析)
    ├── 03_Build_Integration.md (构建细节)
    ├── 04_Usage_in_OH.md (使用场景)
    └── 06_Security.md (安全分析)
```

## 按主题查找

| 主题 | 相关文档 |
|------|---------|
| 如何集成 | 04_Usage_in_OH.md |
| 构建配置 | 03_Build_Integration.md |
| 版本升级 | 02_Patches.md, 06_Security.md |
| 安全评估 | 06_Security.md |
| API 使用 | 01_Overview.md, 04_Usage_in_OH.md |
| 许可信息 | 01_Overview.md |
