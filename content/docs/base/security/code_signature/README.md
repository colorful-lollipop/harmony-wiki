# Code Signature Wiki - 文档说明

## 覆盖范围

本文档覆盖 OpenHarmony `base/security/code_signature` 子系统的完整技术细节，包括：

- **项目定位与核心能力**：代码签名组件的定位、核心功能、运行环境
- **目录结构与模块职责**：各模块职责边界与依赖关系
- **架构说明**：组件图、数据流、线程模型、关键时序
- **Inner API 接口**：Inner API 清单、参数、返回值、错误码
- **GN Targets 与编译产物**：Build 配置、产物清单、加载关系
- **安全风险评审**：攻击面分析、信任边界、安全风险与修复建议

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── index.md              # 项目首页
├── 01_Overview.md        # 项目概览
├── 02_API_Reference.md   # Inner API 参考
├── 03_Architecture.md    # 架构说明
├── 04_Build_System.md     # GN 构建系统
├── 05_Security_Review.md # 安全风险评审
└── appendix/
    ├── Callgraphs.md      # 关键调用链
    └── Config_Flags.md    # 配置开关
```

## 更新方式

本文档基于代码自动生成，如有以下情况需要更新：

1. **新增 API**：在 `interfaces/inner_api/` 添加新的头文件后
2. **修改 BUILD.gn**：添加或修改 targets 后
3. **修改服务配置**：修改 `.cfg` 或 SA profile 后
4. **安全相关变更**：添加新的安全检查或权限校验后

更新时，请同步更新对应的文档章节，确保：
- API 清单与代码一致
- 错误码与 `errcode.h` 一致
- GN targets 与 `BUILD.gn` 一致

## 生成时间

本文档生成时间：2024-XX-XX
