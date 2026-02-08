# Wiki 导航

## 新人学习路线

```
1. 先读: 01_Overview.md (10分钟)
   理解 init 模块定位、能力边界、启动三阶段

2. 次读: 02_Architecture.md (15分钟)
   理解组件划分、数据流、线程模型

3. 根据角色选择:
   ├── 前端/JS 开发者 → 03_NAPI.md
   ├── 框架/系统开发 → 04_InnerAPI.md
   └── 构建/集成 → 05_Build.md

4. 代码导航: 03_CodeMap.md (按需)
   快速定位核心代码文件

5. 问题排查: 07_Troubleshooting.md (按需)
```

---

## 安全研究员路线

```
1. 首先读: 05_AttackSurface.md (15分钟)
   理解外部输入入口、敏感操作、信任边界

2. 深入分析: 06_SecurityReview.md (30分钟)
   了解具体风险点、证据链、修复建议

3. 代码验证: 03_CodeMap.md + 源码
   验证风险描述的准确性

4. 架构理解: 02_Architecture.md
   理解整体安全架构
```

---

## 全站索引

### 快速入门
- [首页](README.md)
- [概览](01_Overview.md)
- [架构](02_Architecture.md)

### 代码导航
- [代码地图](03_CodeMap.md)

### 接口参考
- [N-API 对外接口](03_NAPI.md)
- [Inner API 内部接口](04_InnerAPI.md)

### 工程参考
- [构建与编译](05_Build.md)

### 安全研究（新增）
- [攻击面分析](05_AttackSurface.md)
- [安全风险评估](06_SecurityReview.md)

### 运维支持
- [问题定位](07_Troubleshooting.md)

---

## 安全研究员检查清单

- [ ] 阅读 05_AttackSurface.md 理解攻击面
- [ ] 阅读 06_SecurityReview.md 了解具体风险
- [ ] 验证 R1-R8 风险描述的准确性
- [ ] 检查 trust boundary 图的正确性
- [ ] 评估风险等级评定的合理性
- [ ] 验证修复建议的可行性

---

## 相关链接

- **代码仓库**: `//base/startup/init`
- **官方文档**: [OpenHarmony Startup Subsystem](https://gitee.com/openharmony/docs/blob/master/en/readme/startup.md)
- **组件配置**: `bundle.json`
- **构建配置**: `begetd.gni`
