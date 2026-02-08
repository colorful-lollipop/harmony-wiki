# Connected NFC Tag Wiki

## 文档覆盖范围

本文档描述 OpenHarmony `connected_nfc_tag` 组件的完整工程文档，包括：

### 覆盖内容

| 分类 | 描述 |
|------|------|
| **项目概览** | 组件定位、核心能力、运行环境 |
| **架构说明** | 组件图、数据流、线程模型、关键时序 |
| **对外 API** | N-API 接口定义、参数、返回值、错误码 |
| **内部 API** | 模块接口、依赖方向、稳定性标注 |
| **GN 构建** | targets 列表、依赖、产物、开关 |
| **编译产物** | `.so`/`.a`/可执行文件、安装路径 |
| **安全评审** | 攻击面、信任边界、风险点、修复建议 |

### 未覆盖内容

| 分类 | 描述 |
|------|------|
| **测试代码** | `test/` 目录下所有内容不作为业务证据 |
| **第三方依赖** | HDI 驱动实现细节（由 `drivers_interface_connected_nfc_tag` 提供） |
| **运行时配置** | 系统 NFC 服务配置（如 `nfc_tag_service.cfg` 详细参数） |

## 更新方式

当代码发生以下变更时，需同步更新本文档：

1. **新增/删除/修改 N-API 接口**：更新 `02_NAPI.md`
2. **修改 GN 构建配置**：更新 `04_Build.md`
3. **修改权限模型**：更新 `05_Security.md`
4. **修改模块依赖**：更新 `03_InnerAPI.md` 和 `01_Architecture.md`

## 生成信息

| 属性 | 值 |
|------|------|
| 生成时间 | 2026-02-06 |
| 代码版本 | OpenHarmony 3.1+ |
| 组件路径 | `//foundation/communication/connected_nfc_tag` |
| SA ID | 1148 |

## 相关链接

- [OpenHarmony NFC 官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-connectedTag.md)
- [组件源码](https://gitee.com/openharmony/communication_connected_nfc_tag)
