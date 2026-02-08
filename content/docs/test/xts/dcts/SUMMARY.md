# DCTS Wiki 导航

## 快速导航

- [首页 / README](README.md)
- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [模块详解](04_Modules.md)
- [构建系统](05_Build_System.md)
- [安全评审](06_Security_Review.md)

---

## 新人学习路线

### 路线 A：快速入门（30 分钟）

```
1. 阅读 [01_Overview.md](01_Overview.md) → 了解项目定位
2. 阅读 [02_Directory_Structure.md](02_Directory_Structure.md) → 熟悉目录结构
3. 浏览 [04_Modules.md](04_Modules.md) → 了解测试覆盖范围
```

### 路线 B：深入开发（2 小时）

```
1. 完成路线 A
2. 阅读 [03_Architecture.md](03_Architecture.md) → 理解架构设计
3. 阅读 [05_Build_System.md](05_Build_System.md) → 掌握构建系统
4. 阅读 [appendix/Test_Frameworks.md](appendix/Test_Frameworks.md) → 了解测试框架
5. 参考 [06_Security_Review.md](06_Security_Review.md) → 安全开发注意事项
```

## 安全研究路线

### 路线 A：攻击面分析（30 分钟）

```
1. 阅读 [01_Overview.md](01_Overview.md) → 了解项目边界和依赖
2. 阅读 [02_Directory_Structure.md](02_Directory_Structure.md) → 定位关键代码位置
3. 阅读 [06_AttackSurface.md](06_AttackSurface.md) → 识别所有攻击入口
```

### 路线 B：深度安全审计（2 小时）

```
1. 完成路线 A
2. 阅读 [06_Security_Review.md](06_Security_Review.md) → 了解已识别风险和修复建议
3. 阅读 [04_Modules.md](04_Modules.md) → 理解各模块的安全机制
4. 阅读 [03_Architecture.md](03_Architecture.md) → 理解信任边界
5. 阅读 [wiki/_work/NOTES.md](wiki/_work/NOTES.md) → 查看完整代码证据
```

## 模块索引

| 模块 | 路径 | 说明 |
|------|------|------|
| ability | `ability/` | 分布式能力框架测试 (DMS) |
| communication | `communication/` | 软总线通信测试 |
| distributeddatamgr | `distributeddatamgr/` | 分布式数据管理测试 |
| distributedhardware | `distributedhardware/` | 分布式硬件设备测试 |
| filemanagement | `filemanagement/` | 文件管理测试 |
| multimedia | `multimedia/` | 多媒体会话测试 |
| testtools | `testtools/` | 测试工具 |
| common | `common/` | 共享内存工具 |

---

## 附录

- [测试框架说明](appendix/Test_Frameworks.md)
- [攻击面分析](06_AttackSurface.md) - **NEW**: 详细攻击面清单和利用路径
- [调用链图示](appendix/Callgraphs.md) - *待完善*
- [API 调用示例](appendix/API_Examples.md) - *待完善*

## 反馈与贡献

如发现文档错误或遗漏，请提交 Issue 或 PR 更新。
