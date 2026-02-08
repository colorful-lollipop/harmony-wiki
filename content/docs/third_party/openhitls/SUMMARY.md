# 阅读路线建议

根据你的角色和目的，选择以下阅读路线：

---

## < 15 分钟快速了解

只想快速了解这个库？阅读以下文档的**摘要部分**即可：

1. [README.md](./README.md) - 快速概览
2. [01_Overview.md](./01_Overview.md) - 库简介部分
3. [02_Patches.md](./02_Patches.md) - 一句话结论：无 Patch

---

## 应用开发者路线 (30 分钟)

**目标**：在应用中使用 openHiTLS 的国密能力

阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 了解库的基本概念
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解使用方式和 NDK 暴露
3. **实践**：参考 curl 的 `lib/vtls/openhitls.c` 学习调用方式

---

## 系统开发者路线 (45 分钟)

**目标**：在系统模块中集成 openHiTLS

阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 了解组件结构
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解 BUILD.gn 配置
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解依赖关系
4. **实践**：参考 curl/BUILD.gn 中的集成方式

---

## 维护者/升级者路线 (60 分钟)

**目标**：升级版本、维护 BUILD.gn、跟踪安全更新

阅读顺序：
1. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整评估报告
2. [02_Patches.md](./02_Patches.md) - Patch 策略（无 Patch）
3. [03_Build_Integration.md](./03_Build_Integration.md) - 深入 BUILD.gn
4. [06_Security.md](./06_Security.md) - 安全跟踪策略

---

## 安全审计者路线 (40 分钟)

**目标**：评估安全风险、合规性检查

阅读顺序：
1. [01_Overview.md](./01_Overview.md) - 算法支持清单
2. [02_Patches.md](./02_Patches.md) - 确认无 Patch 修改
3. [06_Security.md](./06_Security.md) - 完整安全分析
4. **检查**：确认 OAT.xml 合规配置

---

## 文档地图

```
wiki/
├── README.md ...................... 入口导航
├── SUMMARY.md ..................... 本文件 - 阅读路线
├── 01_Overview.md ................. 库简介、OH 定位
├── 02_Patches.md .................. Patch 分析（核心）
├── 03_Build_Integration.md ........ BUILD.gn 详解
├── 04_Usage_in_OH.md .............. 依赖关系、使用场景
├── 05_API_Differences.md .......... API 差异
├── 06_Security.md ................. 安全分析
└── _work/
    ├── ASSESSMENT.md .............. 评估报告
    └── NOTES.md ................... 过程记录
```

---

## 关键结论速查

| 问题 | 答案 |
|-----|------|
| 有 Patch 吗？ | 无，原生支持 OH |
| 主要用途？ | curl 国密 HTTPS |
| 几个组件？ | 5 个：bsl/crypto/pki/tls/auth |
| NDK 可用？ | 是，标记 llndk/ndk |
| 版本号？ | 0.2.1 (原始) / 4.0 (OH) |
| 许可证？ | Mulan PSL v2 |
