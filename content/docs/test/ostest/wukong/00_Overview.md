# WuKong 项目概览

## 项目定位

**WuKong** 是 OpenHarmony 平台的稳定性测试自动化工具，通过模拟用户行为对系统及应用进行压力测试。

> 证据：`README.md:4` - "OpenHarmony stability testing automation tool simulates disorderly user behavior to stress test the stability of OpenHarmony systems and applications."

## 核心能力

| 能力 | 说明 | 代码位置 |
|------|------|----------|
| 命令行解析 | 支持命令行获取参数并解析 | `shell_command/` |
| 运行环境管理 | 根据命令行初始化整体运行环境 | `common/` |
| 系统接口管理 | 检查并获取指定的 mgr，注册回调函数 | `common/` |
| 随机事件生成 | 通过 random 函数生成随机序列 | `input_factory/` |
| 事件注入 | 根据支持的事件类型向系统注入事件 | `input_factory/` |
| 异常捕获/报告生成 | 通过 DFX 子系统获取异常信息并生成报告 | `report/` |

> 证据：`README.md:9-15`

## 子模块职责

```
wukong/
├── common/              # 应用控制、随机事件注入、多模事件注入
├── component_event/     # 定义 ability/page/Component 树结构
├── input_factory/      # 屏幕点击、滑动、拖拽、键盘等事件注入
├── report/             # 异常信息收集、统计、显示
├── shell_command/      # 命令行解析和执行
└── test_flow/          # 测试流程控制（随机/专项/专注测试）
```

> 证据：`README.md:19-32`

## 运行环境

| 要求 | 说明 |
|------|------|
| 系统版本 | OpenHarmony 3.2 及以上 |
| 硬件架构 | 支持的设备架构（通过 BUILD.gn 配置） |
| 开发者模式 | 必须开启开发者模式才能运行 |

### 开发者模式检查

```cpp
// 证据: shell_command/src/wukong_main.cpp:132
if (!OHOS::system::GetBoolParameter("const.security.developermode.state", true)) {
    std::cout << "Not a development mode state, please check device mode." << std::endl;
    return 0;
}
```

## 依赖组件

| 组件分类 | 组件名称 |
|----------|----------|
| 能力框架 | ability_base, ability_runtime |
| 无障碍 | accessibility |
| 包管理 | bundle_framework |
| 图形 | graphic_2d, libpng |
| 输入 | input |
| 窗口 | window_manager |
| IPC | ipc, samgr |
| 系统 | init, hilog, hisysevent, hidumper |
| 工具 | c_utils, image_framework |

> 证据：`bundle.json:21-41` 和 `BUILD.gn:100-129`

## 版本信息

| 版本 | 发布内容 |
|------|----------|
| 3.2.0.0 | 初始预置版本，支持应用拉起、随机事件、专项测试、日志打印、白黑名单 |

> 证据：`README.md:145-151`

## 约束

1. **版本约束**: WuKong 在 3.2 系统版本后开始预置使用
2. **编译方式**: 3.2 之前版本需自行编译后推送至设备

### 编译命令

```bash
./build.sh --product-name rk3568 --build-target wukong
```

### 推送命令

```bash
hdc_std shell mount -o rw,remount /
hdc_std file send wukong /
hdc_std shell chmod a+x /wukong
hdc_std shell mv /wukong /bin/
```

> 证据：`README.md:38-52`

## 相关文档

- [架构说明](01_Architecture.md)
- [命令行接口](02_CommandLine.md)
- [模块详情](03_Module_Details.md)
