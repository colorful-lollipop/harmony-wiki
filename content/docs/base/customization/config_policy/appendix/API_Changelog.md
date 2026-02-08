# API 变更历史

## 版本 3.2（当前版本）

### 新增 API

| API 名称 | 类型 | 说明 |
|----------|------|------|
| `getOneCfgFileSync` | 同步 | 同步获取最高优先级配置文件 |
| `getCfgFilesSync` | 同步 | 同步获取所有配置文件 |
| `getCfgDirListSync` | 同步 | 同步获取配置目录列表 |

### FollowXMode 枚举

| 枚举值 | 版本 | 说明 |
|--------|------|------|
| `FOLLOWX_MODE_DEFAULT` | 1.0 | 默认 Follow 规则 |
| `FOLLOWX_MODE_NO_RULE_FOLLOWED` | 1.0 | 不使用 Follow 规则 |
| `FOLLOWX_MODE_SIM_DEFAULT` | 1.0 | 默认 SIM 卡 |
| `FOLLOWX_MODE_SIM_1` | 1.0 | SIM 卡 1 |
| `FOLLOWX_MODE_SIM_2` | 1.0 | SIM 卡 2 |
| `FOLLOWX_MODE_USER_DEFINED` | 1.0 | 用户自定义 |

---

## 历史版本

### 版本 2.0

- 初始 N-API 接口发布
- 支持异步 API 调用模式
- 添加 HiSysEvent 监控

### 版本 1.0

- 仅支持 C++ 内部 API
- 无 JS/N-API 接口
