# Wi-Fi Aware Wiki 导航

## 新人阅读路线

**推荐顺序**: `index` → `architecture` → `api` → `build` → `security`

## 文档列表

### 入门
- [README](README.md) - 项目说明与导航
- [SUMMARY](SUMMARY.md) - 本导航页
- [index](index.md) - 项目概览与核心能力

### 架构与 API
- [architecture](architecture.md) - 模块架构、层次、数据流
- [api](api.md) - C API 参考（9个函数）
- [hal](hal.md) - HAL 接口定义（11个函数）

### 构建与安全
- [build](build.md) - GN Targets 与编译产物
- [security](security.md) - 安全风险评审

## 快速链接

### 关键文件
- 框架实现: `frameworks/source/wifiaware.c:1`
- 对外 API: `interfaces/kits/wifiaware.h:1`
- HAL 接口: `hals/hal_wifiaware.h:1`
- 构建配置: `BUILD.gn:1`

### 关键常量
- 成功码: `WIFIAWARE_SUCCESS` (`interfaces/kits/wifiaware.h:70`)
- 失败码: `WIFIAWARE_FAIL` (`interfaces/kits/wifiaware.h:75`)
- 默认信道: `WIFIAWARE_DEFAULT_CHANNEL` (`interfaces/kits/wifiaware.h:81`)

### 关键函数
- 初始化: `InitNAN()` (`interfaces/kits/wifiaware.h:129`)
- 订阅服务: `SubscribeService()` (`interfaces/kits/wifiaware.h:158`)
- 发送数据: `SendData()` (`interfaces/kits/wifiaware.h:180`)
