# testfwk_cangjie_wrapper 工程 Wiki

**项目**: OpenHarmony Cangjie 语言 UI 测试框架包装器  
**版本**: 6.1  
**生成时间**: 2026-02-06  
**文档语言**: 中文  

---

## 文档范围

本 Wiki 提供 `testfwk_testfwk_cangjie_wrapper` 项目的完整工程分析，包括：

- **项目概览与架构设计**
- **对外 API 参考（Cangjie 层）**
- **内部架构与模块依赖**
- **GN 构建系统与编译产物**
- **安全风险评审**
- **问题排查指南**

---

## 文档结构

```
wiki/
├── README.md              # 本文档 - 项目说明与导航
├── SUMMARY.md             # 全站导航与阅读顺序
├── 00_Overview.md         # 项目概览
├── 10_Architecture.md     # 架构设计
├── 20_API_Reference.md    # API 参考手册
├── 30_Build_System.md     # 构建系统
├── 40_Security.md         # 安全分析
├── 50_Troubleshooting.md  # 问题排查
├── appendix/
│   └── Callgraphs.md      # 关键调用链
└── _work/                 # 工作区（不包含在文档中）
    ├── NOTES.md
    └── PLAN.md
```

---

## 快速开始

### 新人阅读路线

1. **[项目概览](00_Overview.md)** - 了解项目定位、边界和能力
2. **[架构设计](10_Architecture.md)** - 理解分层架构和数据流
3. **[API 参考](20_API_Reference.md)** - 掌握对外接口使用方法
4. **[构建系统](30_Build_System.md)** - 了解如何编译和集成
5. **[安全分析](40_Security.md)** - 理解安全模型和注意事项

### 关键信息速查

| 项目 | 内容 |
|------|------|
| **编程语言** | Cangjie (仓颉) |
| **项目类型** | UI 测试框架包装器 |
| **目标平台** | OpenHarmony Standard |
| **API Level** | 22+ |
| **SystemCapability** | SystemCapability.Test.UiTest |
| **核心类** | Driver, On, Component, UiWindow |
| **构建系统** | GN (Generate Ninja) |
| **主要产物** | libohos.ui_test.so, libkit.TestKit.so |

---

## 更新维护

### 文档生成方式

本文档基于代码仓库静态分析自动生成，关键结论均引用代码位置：
- 文件路径 (如 `ohos/ui_test/ui_test_api.cj`)
- 行号 (如 `:115` 表示第 115 行)
- 符号名 (如 `Driver.create`)

### 如何更新文档

1. 代码变更后，重新运行 Wiki Agent 分析
2. 更新 `wiki/_work/NOTES.md` 记录新发现
3. 同步修改相关 `.md` 文档
4. 更新 `SUMMARY.md` 导航链接（如新增页面）

### 未覆盖内容

- **测试代码** (`test/` 目录) - 按需求排除
- **原生实现** - 本仓库仅为 Cangjie 包装层，底层实现在 `arkxtest`
- **使用教程** - 请参考官方开发指南

---

## 相关资源

- **官方 API 文档**: [cj-apis-ui_test.md](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/TestKit/cj-apis-ui_test.md)
- **开发指南**: [cj-arkxtest-guidelines.md](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/application-test/cj-arkxtest-guidelines.md)
- **依赖项目**:
  - [arkxtest](https://gitcode.com/openharmony/testfwk_arkxtest) - UI 测试底层实现
  - [ability_cangjie_wrapper](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper) - Ability 框架绑定
  - [cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) - Cangjie-ArkTS 互操作

---

*文档由 OpenHarmony 工程 Wiki Agent 生成*  
*基于 commit: b388b58e20e6ba488f8e8620ac3f9da064bf6ac9*
