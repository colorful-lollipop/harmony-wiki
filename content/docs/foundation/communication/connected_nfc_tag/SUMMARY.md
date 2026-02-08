# Connected NFC Tag Wiki - 文档导航

## 新人阅读路线

```
1. 阅读顺序建议：
   00_Overview.md → 01_Architecture.md → 02_NAPI.md → 04_Build.md → 05_Security.md
   
2. 如需深入了解内部实现：
   03_InnerAPI.md → 附录调用链
   
3. 如需排查构建/运行问题：
   04_Build.md → 常见问题
```

---

## 文档清单

### 核心文档

| 文档 | 描述 | 适用范围 |
|------|------|----------|
| [README](README.md) | 文档说明、更新方式、生成信息 | 所有读者 |
| [00_Overview](00_Overview.md) | 项目定位、核心能力、运行环境 | 新人入门 |
| [01_Architecture](01_Architecture.md) | 组件架构、数据流、线程模型 | 架构师、开发者 |
| [02_NAPI](02_NAPI.md) | JS API 接口文档 | 应用开发者 |
| [03_InnerAPI](03_InnerAPI.md) | 内部模块接口 | 系统开发者 |
| [04_Build](04_Build.md) | GN 构建系统、编译产物 | 构建工程师 |
| [05_Security](05_Security.md) | 安全风险评估 | 安全工程师 |

### 附录

| 文档 | 描述 |
|------|------|
| [附录：调用链](appendix/Callgraphs.md) | 关键调用链路图示 |
| [附录：配置开关](appendix/Config_Flags.md) | 编译/运行时开关说明 |

---

## 快速索引

### N-API 快速查找

| 功能 | API 名称 | 文件位置 |
|------|---------|----------|
| 初始化 | `init` / `initialize` | `02_NAPI.md#初始化` |
| 读 NDEF | `readNdefTag` / `read` | `02_NAPI.md#读写napi` |
| 写 NDEF | `writeNdefTag` / `write` | `02_NAPI.md#读写napi` |
| 事件订阅 | `on` / `off` | `02_NAPI.md#事件监听` |

### GN Target 快速查找

| 产物 | Target | 构建位置 |
|------|--------|----------|
| JS 绑定 | `connectedtag` | `frameworks/js/napi/BUILD.gn` |
| 服务 | `nfc_tag_service` | `services/BUILD.gn` |
| 内部 API | `nfc_tag_inner_kits` | `interfaces/inner_api/BUILD.gn` |
| HDI 适配 | `nfc_tag_hdi_adapter` | `services/src/hdi/BUILD.gn` |

### 错误码快速查找

| 错误码 | 含义 | 处理建议 |
|--------|------|----------|
| `NFC_SUCCESS` | 成功 | 无需处理 |
| `NFC_GRANT_FAILED` | 权限不足 | 检查权限声明 |
| `NFC_INVALID_PARAMETER` | 参数错误 | 检查输入参数 |
| `NFC_CALLBACK_REGISTERED` | 回调已注册 | 先取消再注册 |
| `NFC_TOO_MANY_CALLBACK` | 回调过多 | 释放不需要的回调 |

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始版本 |
