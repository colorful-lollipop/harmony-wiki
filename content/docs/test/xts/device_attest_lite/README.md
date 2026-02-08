# device_attest_lite Wiki

> OpenHarmony 设备认证模块工程文档

## 文档覆盖范围

本文档覆盖 OpenHarmony XTS 子系统中的 `device_attest_lite` 模块，用于设备认证和生态设备统计。

### 适用系统

| 系统类型 | 内核 | 支持状态 |
|---------|------|---------|
| Mini System | LiteOS-M | ✅ 支持 |
| Small System | LiteOS-A / Linux | ✅ 支持 |

## 文档结构

| 文档 | 描述 |
|------|------|
| [SUMMARY](SUMMARY.md) | 全站导航与阅读路线 |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、关键概念 |
| [02_Architecture](02_Architecture.md) | 组件架构、数据流、线程模型 |
| [03_JSI_API](03_JSI_API.md) | JS 接口 (JSI) API 清单 |
| [04_InnerAPI](04_InnerAPI.md) | Inner API 与模块依赖 |
| [05_Build](05_Build.md) | GN Targets 与编译产物 |
| [06_Security](06_Security.md) | 安全风险评审 |
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |

## 关键术语

- **manuKey**: 从 OpenHarmony 兼容性平台获取的制造商密钥
- **productId**: 产品标识符
- **productKey**: 产品密钥
- **token**: 设备凭证，每个设备唯一
- **AttestResult**: 设备认证结果

## 代码证据原则

本文档所有关键结论均可追溯到代码证据：
- 文件路径 (必要时刻含行号)
- 关键符号名 (函数/类/宏/target)
- 最小必要代码片段

## 更新方式

当代码变更时：
1. 更新对应模块的文档章节
2. 确保 API 清单与代码一致
3. 更新编译产物说明
4. 更新安全风险评估

## 相关链接

- **代码仓库**: https://gitee.com/openharmony/xts_device_attest_lite
- **OpenHarmony 兼容性平台**: https://compatibility.openharmony.cn
- **bundle.json**: `/Volumes/lexar/code/d/work/oh/test/xts/device_attest_lite/bundle.json`

---

*文档生成时间: 2026-02-06*
*负责 Agent: Sisyphus*
