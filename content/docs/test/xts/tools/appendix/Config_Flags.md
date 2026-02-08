# Config_Flags.md

## 目的

本文档列出 XTS Tools 的关键宏与 feature flags，帮助开发者理解构建与行为控制。

## 适用范围

- 仅涉及 tools/ 仓库内的宏与开关
- 不涉及 acts 测试用例宏

---

## 测试用例级别宏

| 宏名称 | 值 | 含义 | 证据 |
|--------|-----|------|------|
| Level0 | Smoke | 验证关键功能基本可运行性 | README.md:73-78 |
| Level1 | Basic | 验证关键功能基本可测性 | README.md:80-85 |
| Level2 | Major | 验证关键功能基本功能与错误处理 | README.md:87-92 |
| Level3 | Regular | 验证所有关键功能与 DFX 属性 | README.md:94-99 |
| Level4 | Rare | 验证极端异常与非常规输入 | README.md:101-106 |

---

## 测试用例粒度宏

| 宏名称 | 值 | 含义 | 证据 |
|--------|-----|------|------|
| SmallTest | 小规模测试 | 模块/类/函数级别，本地 PC 执行 | README.md:136-141 |
| MediumTest | 中等规模测试 | 模块/子系统功能，单设备执行 | README.md:129-134 |
| LargeTest | 大规模测试 | 服务功能/全场景，接近真实设备 | README.md:122-127 |

---

## 测试用例类型宏

| 宏名称 | 值 | 含义 | 证据 |
|--------|-----|------|------|
| Function | 功能测试 | 测试服务与平台功能正确性 | README.md:154-158 |
| Performance | 性能测试 | 测试处理能力（QPS、FPS 等） | README.md:160-164 |
| Power | 功耗测试 | 测试单位时间功耗 | README.md:166-170 |
| Reliability | 可靠性测试 | 测试稳定性/压力/故障注入 | README.md:172-174 |
| Security | 安全测试 | 测试安全威胁防御与隐私保护 | README.md:175-178 |
| Global | 国际化测试 | 测试多语言/本地化能力 | README.md:180-183 |
| Compatibility | 兼容性测试 | 测试数据/系统/硬件兼容性 | README.md:185-188 |
| User | 用户测试 | 测试用户体验（主观评价） | README.md:190-193 |
| Standard | 标准测试 | 测试行业标准与协议合规性 | README.md:195-198 |
| Safety | 安全特性测试 | 测试安全属性（避免危害） | README.md:200-203 |
| Resilience | 韧性测试 | 测试攻击承受与恢复能力 | README.md:205-208 |

---

## 编译相关宏

| 宏名称 | 值 | 含义 | 证据 |
|--------|-----|------|------|
| -Wno-error | 编译选项 | 将警告不视为错误 | README.md:326、458 |
| UNITY_INCLUDE_CONFIG_H | HCTest define | Unity 框架配置 | `lite/hctest/BUILD.gn:34` |
| GTEST_HAS_CLONE=0 | HCPPTest define | Googletest 配置 | `lite/hcpptest/BUILD.gn:81` |
| ohos_kernel_type | "liteos_a" / "linux" | 条件编译内核类型 | `lite/checksum/BUILD.gn:14` |

---

## GN 目标相关宏

| 宏名称 | 值 | 含义 | 证据 |
|--------|-----|------|------|
| board_name | liteos_m / liteos_a | 区分 Mini/Small 系统 | README.md:336-343、466-473 |
| suite_name | "acts" | 测试套件名称 | README.md:319、444 |
| ohos_kernel_type | "liteos_a" / "linux" | 内核类型（条件编译） | `lite/checksum/BUILD.gn:14` |
| board_toolchain_type | "iccarm" 等 | 工具链类型（条件编译） | `lite/hctest/BUILD.gn:38` |

---

## 运行时相关配置

| 配置名称 | 值 | 含义 | 证据 |
|---------|-----|------|------|
| NFS 挂载路径 | 192.168.1.10:/nfs | NFS 共享目录路径 | README.md:498-504 |
| 挂载点 | /nfs | 开发板挂载目录 | README.md:498-504 |
| 测试结果格式 | "xx Tests xx Failures xx Ignored" | 串口日志输出格式 | README.md:368-369 |

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [05_GN_Targets.md](05_GN_Targets.md)
- [08_Common_Troubleshooting.md](08_Common_Troubleshooting.md)
