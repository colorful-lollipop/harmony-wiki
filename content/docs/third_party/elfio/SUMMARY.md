# 阅读路线建议

## 快速定位

根据您的需求，选择以下阅读路径：

### 场景 1：了解 ELFIO 在 OH 中的整体情况
**推荐阅读顺序**：
1. `README.md` - 库概览
2. `03_Build_Integration.md` - 构建适配
3. `04_Usage_in_OH.md` - 依赖关系和使用

### 场景 2：需要修改或适配 ELFIO
**推荐阅读顺序**：
1. `README.md` - 基础信息
2. `03_Build_Integration.md` - 构建配置
3. `05_API_Differences.md` - API 差异（如有）

### 场景 3：排查 ELFIO 相关问题
**推荐阅读顺序**：
1. `04_Usage_in_OH.md` - 确认使用方式
2. `02_Patches.md` - 检查是否有相关 Patch
3. `03_Build_Integration.md` - 构建配置

### 场景 4：准备升级 ELFIO 版本
**推荐阅读顺序**：
1. `02_Patches.md` - 确认 OH 特有修改
2. `03_Build_Integration.md` - 构建配置差异
3. `06_Security.md` - 安全风险评估

## 文档优先级

| 优先级 | 文档 | 必读程度 |
|--------|------|----------|
| ⭐⭐⭐ | `README.md` | 所有用户必读 |
| ⭐⭐⭐ | `03_Build_Integration.md` | 开发者必读 |
| ⭐⭐ | `04_Usage_in_OH.md` | 集成者建议阅读 |
| ⭐⭐ | `02_Patches.md` | 维护者必读 |
| ⭐ | `05_API_Differences.md` | 按需阅读 |
| ⭐ | `06_Security.md` | 安全相关按需阅读 |

## 关键信息速查

### 基础信息
- **版本**：Release_3.12
- **许可证**：MIT
- **构建类型**：共享库（ohos_shared_library）
- **主要依赖者**：hapsigner, irtoc, libbpf, code_signature

### 适配层
- **Patch**：无（干净集成）
- **C 包装器**：`c_wrapper/` 目录
- **构建配置**：`elfio_public_config`

## 常见问题

**Q: ELFIO 在 OH 中做什么？**
A: 提供 ELF 文件解析和生成能力，主要用于签名工具、eBPF 程序处理等场景。

**Q: 为什么没有 Patch？**
A: ELFIO 是纯头文件库，跨平台兼容性好，OH 通过 C 包装器和 BUILD.gn 完成适配。

**Q: 如何在 OH 中使用？**
A: 在 BUILD.gn 中添加 `external_deps = [ "elfio:elfio" ]`

**Q: 支持 C 语言接口吗？**
A: 支持，OH 提供了完整的 C 包装器在 `c_wrapper/` 目录。
