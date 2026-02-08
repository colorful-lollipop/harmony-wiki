# WuKong 命令行接口

## 命令概述

| 命令 | 说明 | 缩写 |
|------|------|------|
| `version` | 获取版本信息 | `-v`, `--version` |
| `help` | 获取帮助信息 | - |
| `appinfo` | 查询应用 bundleName 和 ability 名 | - |
| `special` | 专项测试 | - |
| `exec` | 随机测试 | - |
| `focus` | 专注测试 | - |

## 全局选项

| 选项 | 说明 | 备注 |
|------|------|------|
| `--track` | 启用跟踪级别日志 | 详细日志 |
| `--debug` | 启用调试级别日志 | 最详细日志 |

```bash
# 示例: 启用调试日志
wukong exec --debug -c 10
```

## version 命令

获取 WuKong 版本信息。

```bash
wukong version
# 或
wukong -v
# 或
wukong --version
```

**输出示例:**
```
WuKong version 3.2.0.0
```

## help 命令

获取 WuKong 帮助信息。

```bash
wukong help
```

**输出示例:**
```
Usage: wukong [COMMAND] [OPTIONS]

Commands:
  version   Get wukong version information
  help      Get wukong help information
  appinfo   Query support pulling up the application bundleName and mainAbility name
  special   wukong special test
  exec      wukong randomly tests
  focus     wukong focus tests
```

## appinfo 命令

查询支持拉起应用的 bundleName 和对应的 mainAbility 名。

```bash
wukong appinfo
```

**输出示例:**
```
com.example.app/MainAbility
com.ohos.settings/MainAbility
...
```

## special 命令

专项测试，支持特定场景的定向测试。

### 选项

| 选项 | 功能 | 必选 | 默认值 | 备注 |
|------|------|------|--------|------|
| `-h`, `--help` | 获取帮助信息 | 否 | - | 专项测试帮助 |
| `-k`, `--spec_insomnia` | 休眠唤醒专项测试 | 否 | - | - |
| `-c`, `--count` | 设置执行次数 | 否 | 10 | 单位次 |
| `-i`, `--interval` | 设置执行间隔 | 否 | 1500 | 单位 ms |
| `-S`, `--swap` | 滑动测试 | 否 | - | - |
| `-s`, `--start [x,y]` | 设置滑动起点坐标 | 否 | - | - |
| `-e`, `--end [x,y]` | 设置滑动终点坐标 | 否 | - | - |
| `-b`, `--bilateral` | 设置往返滑动 | 否 | false | 默认不往返 |
| `-t`, `--touch [x,y]` | 点击测试 | 否 | - | - |
| `-T`, `--time` | 设置测试总时间 | 否 | 10 | 单位分钟 |
| `-C`, `--component` | 控件顺序遍历测试 | 否 | - | 需设置应用名 |
| `-r`, `--record` | 录制 | 否 | - | 需指定录制文件 |
| `-R`, `--replay` | 回放 | 否 | - | 需指定回放文件 |
| `-p`, `--screenshot` | 控件测试截图 | 否 | - | - |

### 使用示例

```bash
# 控件顺序遍历测试
wukong special -C com.example.app -p

# 休眠唤醒专项测试
wukong special -k -c 5

# 滑动测试
wukong special -S -s 100,200 -e 400,500 -b

# 点击测试
wukong special -t 300,400 -c 20

# 录制
wukong special -r /data/record.txt

# 回放
wukong special -R /data/record.txt
```

## exec 命令

随机测试，根据配置的权重随机生成测试事件。

### 选项

| 选项 | 功能 | 必选 | 默认值 | 备注 |
|------|------|------|--------|------|
| `-h`, `--help` | 获取帮助信息 | 否 | - | 随机测试帮助 |
| `-c`, `--count` | 设置执行次数 | 否 | 10 | 与 -T 冲突 |
| `-i`, `--interval` | 设置执行间隔 | 否 | 1500 | 单位 ms |
| `-s`, `--seed` | 设置随机种子 | 否 | - | 相同种子生成相同序列 |
| `-b`, `--bundle` | 设置允许应用名单 | 否 | 所有应用 | 与 -p 冲突 |
| `-p`, `--prohibit` | 设置禁止应用名单 | 否 | 无 | 与 -b 冲突 |
| `-d`, `--page` | 设置禁止页面名单 | 否 | system 页面 | - |
| `-e`, `--allow` | 设置允许 ability 页面 | 否 | - | 需配合 -b |
| `-E`, `--block` | 设置禁止 ability 页面 | 否 | - | 需配合 -b |
| `-a`, `--appswitch` | 应用拉起测试比例 | 否 | 10% | 0-1 浮点数 |
| `-t`, `--touch` | 触摸测试比例 | 否 | 10% | 0-1 浮点数 |
| `-S`, `--swap` | 滑动测试比例 | 否 | 3% | 0-1 浮点数 |
| `-m`, `--mouse` | 鼠标测试比例 | 否 | 1% | 0-1 浮点数 |
| `-k`, `--keyboard` | 键盘测试比例 | 否 | 2% | 0-1 浮点数 |
| `-H`, `--hardkey` | 硬键测试比例 | 否 | 2% | 0-1 浮点数 |
| `-r`, `--rotate` | 旋转测试比例 | 否 | 2% | 0-1 浮点数 |
| `-C`, `--component` | 控件测试比例 | 否 | 70% | 0-1 浮点数 |
| `-I`, `--screenshot` | 控件测试截图 | 否 | - | - |
| `-T`, `--time` | 设置测试总时间 | 否 | 10 | 单位分钟，与 -c 冲突 |
| `-U`, `-uri` | 应用拉起页面 uri | 否 | - | - |
| `-x`, `-uriType` | 应用拉起页面 uriType | 否 | - | - |

### 使用示例

```bash
# 基础随机测试
wukong exec -s 10 -i 1000 -a 0.28 -t 0.72 -c 100

# 指定允许应用
wukong exec -b com.example.app,com.example.app2 -c 50

# 指定禁止应用
wukong exec -p com.ohos.systemui -c 100

# 指定禁止页面
wukong exec -d pages/index,pages/settings -c 50

# 允许/禁止特定 ability
wukong exec -b com.ohos.settings -e com.ohos.settings.MainAbility -E com.ohos.settings.AppInfoAbility

# 隐式启动
wukong exec -b com.example.app -U uri -x uriType

# 指定测试时间
wukong exec -T 30 -s 100

# 完整配置示例
wukong exec -s 1000 -i 500 -a 0.1 -t 0.3 -S 0.1 -k 0.1 -H 0.1 -C 0.2 -c 200
```

> **说明**: 配置相同的随机种子（-s），会生成相同的随机事件序列，便于复现问题。

### 权重分配逻辑

| 事件类型 | 默认比例 | 范围 | 说明 |
|----------|----------|------|------|
| component | 70% | 0-1 | 控件操作（主要测试） |
| touch | 10% | 0-1 | 触摸事件 |
| appswitch | 10% | 0-1 | 应用切换 |
| keyboard | 2% | 0-1 | 键盘输入 |
| hardkey | 2% | 0-1 | 硬件按键 |
| rotate | 2% | 0-1 | 屏幕旋转 |
| swap | 3% | 0-1 | 滑动操作 |
| mouse | 1% | 0-1 | 鼠标事件 |

## focus 命令

专注测试，针对特定类型组件进行深度测试。

### 选项

| 选项 | 功能 | 必选 | 默认值 | 备注 |
|------|------|------|--------|------|
| `-n`, `--numberfocus` | 每个控件注入次数 | 否 | 1 | - |
| `-f`, `--focustypes` | 需要专注的控件类型 | 否 | - | 英文逗号分隔 |

其余参数继承自 `exec` 命令。

### 使用示例

```bash
# 专注测试 Button 组件
wukong focus -f Button -n 5 -c 50

# 专注测试多种组件
wukong focus -f Button,Text,Image -n 3 -c 100

# 带截图的专注测试
wukong focus -f Button -n 5 -I -c 50
```

## 退出码

| 退出码 | 说明 |
|--------|------|
| 0 | 正常退出 |
| 1 | 日志启动失败 |
| 其他 | 异常退出 |

```cpp
// 证据: shell_command/src/wukong_main.cpp:153-154
if (!WuKonglogger->Start()) {
    return 1;
}
```

## 相关文档

- [架构说明](01_Architecture.md)
- [模块详情](03_Module_Details.md)
- [故障排查](06_Troubleshooting.md)
