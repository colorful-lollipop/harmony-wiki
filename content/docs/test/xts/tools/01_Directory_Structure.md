# 01_Directory_Structure.md

## 目的

本文档说明 XTS Tools 仓库的目录结构与各模块职责，帮助开发者快速定位代码与工具。

## 适用范围

- 仅包含非测试目录
- 忽略 test/tests/unittest/fuzz/fuzztest 等测试相关目录（不引用其作为业务证据）

---

## 顶层目录树（不含测试）

```
tools/
├── README.md（中文：README_zh.md）
├── LICENSE
├── OAT.xml
├── bundle.json
├── ci/                  # 持续集成相关（待补充）
├── config/               # 配置文件（待补充）
├── figures/              # 文档插图资源
├── lite/                # 轻量级系统测试框架与工具
├── others/              # 其他工具与示例
├── sample/              # 样例与演示
├── standard_check/      # 标准系统检查工具
├── xts-project-tools/   # XTS 项目工具链
└── build/               # 构建脚本与 GN 配置
```

---

## 目录职责

- README.md/README_zh.md
  - 项目说明、系统类型、测试框架指南
  - 证据：README.md:1-649

- LICENSE
  - 开源许可证（待补充具体内容）

- OAT.xml
  - 开源审计追踪（待补充）

- bundle.json
  - OpenHarmony 包描述文件（待补充）

- ci/
  - 持续集成相关配置与脚本（待补充细节）

- config/
  - 构建与工具配置（待补充）

- figures/
  - 文档插图（icon-note.gif、icon-tip.gif 等）
  - 证据：glob 返回 figures/icon-*.gif

- lite/
  - 轻量级系统测试框架与工具
  - 包含：hctest（C）、hcpptest（C++）、checksum、others/query 等
  - 证据：README.md:43-54；glob 发现 lite/ 下多个 BUILD.gn

- others/
  - 其他工具与示例
  - 包含：query（标准系统查询工具）
  - 证据：glob 发现 others/query/BUILD.gn

- sample/
  - 样例与演示（AppSampleD/E、ServerSampleD/E）
  - 包含：Java/JS/C++ 示例与配置
  - 证据：glob 返回 sample/ 下大量 .md/.png/.sql/.json5/.ts 等文件

- standard_check/
  - 标准系统检查工具
  - 包含：hvigor 检查（check_hvigor.py）
  - 证据：README.md 未详述，glob 发现 standard_check/check_hvigor.py

- xts-project-tools/
  - XTS 项目工具链
  - 包含：format-tc-doc、hvigor-update 等
  - 证据：glob 返回 xts-project-tools/ 下多个脚本与 README

- build/
  - 构建脚本与 GN 配置
  - 包含：suite.py、suite.gni、utils.py、test_package_select.py、judgePart.py 等
  - 证据：README.md:18 提及测试用例开发框架相关 tools；glob 返回 build/ 下多个 Python 与 .gni

---

## 详细子目录（待补充证据）

### lite/

- hctest/ —— C 测试框架（Mini 系统）
  - BUILD.gn（证据：glob）
- hcpptest/ —— C++ 测试框架（Small/Standard 系统）
  - BUILD.gn（证据：glob）
- checksum/ —— 校验和工具
  - BUILD.gn（证据：glob）
- others/query/ —— 查询工具（标准系统）
  - BUILD.gn、QueryMainStandard.cpp（证据：glob、bash 查询）

### others/

- query/ —— 标准系统查询工具
  - BUILD.gn（证据：glob）

### sample/

- AppSampleD/ —— 应用示例 D
  - hvigorw、oh-package.json5、build-profile.json5、AppScope 等（证据：glob）
- AppSampleE/ —— 应用示例 E（待补充）
- ServerSampleD/ —— 服务器示例 D（Java/Spring 示例）
  - README.md、img/、java/ 等（证据：glob）
- ServerSampleE/ —— 服务器示例 E（待补充）

### standard_check/

- check_hvigor.py —— HVIGR 检查脚本（证据：glob）
- hvigor-wrapper.js、hvigorw.bat 等（证据：glob）

### xts-project-tools/

- format-tc-doc/ —— 测试用例文档格式化
  - format_tc_doc.py、README.md 等（证据：glob）
- hvigor-update/hvigor5/ —— HVIGR 5 更新管理
  - batch-update.sh、hvigor-update.sh、README.md 等（证据：glob）

### build/

- suite.py、suite.gni —— 测试套件管理与 GN 模板（证据：glob）
- utils.py、test_package_select.py、judgePart.py —— 构建辅助脚本（证据：glob）
- copy_hypium_static.sh —— 复制 Hypium 静态库（证据：glob）

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [02_Architecture.md](02_Architecture.md)
- [05_GN_Targets.md](05_GN_Targets.md)
