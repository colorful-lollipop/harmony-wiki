# OpenHarmony Settings 应用 Wiki

> 生成时间：2026-02-06 00:11:23
> 项目：@ohos/settings (Settings 应用)
> 版本：3.1
> 子系统：applications
> 系统类型：standard

---

## 文档覆盖范围

本文档覆盖 OpenHarmony Standard Settings 应用的以下方面：

### ✅ 已覆盖
- 项目概览与定位
- 目录结构与模块职责
- 对外 API（N-API / ANI / CJ FFI）
- 内部架构与 Inner API
- GN 构建系统与 Targets
- 编译产物与安装路径
- 安全风险评审（基于代码证据）
- 常见构建/运行/调试问题

### ⚠️ 部分覆盖
- UI 组件设计（common/component）
- 具体功能实现细节（product/phone 页面）

### ❌ 未覆盖
- 测试相关内容（test/、*_test.*）
- 第三方库依赖（external_deps 的详细实现）

---

## 如何更新本文档

### 随代码更新

当代码库更新时，应同步更新对应的 Wiki 文档：

1. **N-API 变更**：
   - 更新 `wiki/04_NAPI_API.md`
   - 检查是否有新增/修改的导出 API
   - 更新 API 清单表

2. **架构变更**：
   - 更新 `wiki/03_Architecture.md`
   - 检查组件依赖关系是否变化
   - 更新 Mermaid 图

3. **构建系统变更**：
   - 更新 `wiki/06_GN_Targets.md`
   - 检查是否有新增/修改的 targets
   - 更新产物清单

4. **安全相关变更**：
   - 更新 `wiki/08_Security_Audit.md`
   - 检查是否有新增的安全风险点
   - 更新修复建议

### 更新流程

1. 阅读代码变更
2. 更新对应的 Wiki 文档
3. 在文档顶部添加更新日志（如有重大变更）
4. 更新 `wiki/SUMMARY.md` 中的文档状态
5. 提交变更到 Git

---

## 文档生成方法

本文档通过以下方式生成：

1. **Phase 0**：初始化工作区，创建 NOTES.md 和 PLAN.md
2. **Phase 1**：全局扫描，收集代码证据
   - N-API 模块分析
   - IPC/SA 接口分析
   - GN 构建系统分析
   - 权限与安全机制分析
   - 目录结构与模块职责分析
3. **Phase 2**：搭建 Wiki 骨架
4. **Phase 3**：对外 API / N-API 深入梳理
5. **Phase 4**：内部架构与 Inner API 分析
6. **Phase 5**：GN Targets 与编译产物梳理
7. **Phase 6**：安全风险评审（基于证据）
8. **Phase 7**：一致性校验与润色

详细工作记录见 `wiki/_work/PLAN.md`。

---

## 文档结构

```
wiki/
├── README.md (本文件)
├── SUMMARY.md (导航)
├── 00_Overview.md (概览)
├── 01_Positioning.md (项目定位)
├── 02_Directory_Structure.md (目录结构)
├── 03_Architecture.md (架构)
├── 04_NAPI_API.md (对外 API)
├── 05_Inner_API.md (内部 API)
├── 06_GN_Targets.md (GN Targets)
├── 07_Build_Artifacts.md (编译产物)
├── 08_Security_Audit.md (安全评审)
├── 09_FAQ.md (常见问题)
└── appendix/
    ├── Callgraphs.md (调用链图) [可选]
    └── Config_Flags.md (配置标志) [可选]
```

---

## 维护者

- 生成工具：OpenCode Wiki Agent
- 维护方式：随代码更新手动维护
- 联系方式：通过 GitHub Issues

---

## 许可证

本文档采用 Apache License 2.0 许可证，与项目保持一致。

---

## 参考资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [N-API 开发指南](https://docs.openharmony.cn/application-dev/haps/napi-guidelines-overview)
- [ANI 开发指南](https://docs.openharmony.cn/application-dev/aps/aps-ani-overview)
- [GN 构建系统](https://docs.openharmony.cn/application-dev/quick-start/start-overview-0000001473876304)

---

**最后更新**：2026-02-06 00:11:23
