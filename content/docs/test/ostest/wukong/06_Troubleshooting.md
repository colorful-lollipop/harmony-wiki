# WuKong 故障排查

## 常见问题

### Q1: 提示 "Not a development mode state"

**错误信息**:
```
Not a development mode state, please check device mode.
```

**原因**: 设备未开启开发者模式

**解决方案**:
```bash
# 开启开发者模式
hdc_std shell param set const.security.developermode.state 1
hdc_std shell reboot
```

**证据**: `shell_command/src/wukong_main.cpp:132-135`

---

### Q2: 提示 "error: wukong has running, allow one program run"

**错误信息**:
```
error: wukong has running, allow one program run.
```

**原因**: WuKong 已在运行（信号量检测）

**解决方案**:
```bash
# 检查运行实例
hdc_std shell pidof wukong

# 如果需要强制终止
hdc_std shell kill -9 <pid>

# 或者清理信号量
hdc_std shell rm -f /dev/shm/wukong_*
```

**证据**: `shell_command/src/wukong_main.cpp:161-163`

---

### Q3: 编译失败 - 找不到依赖

**错误信息**:
```
error: dependency not found: xxx
```

**原因**: OpenHarmony SDK 或依赖组件未正确配置

**解决方案**:
1. 确保在正确配置的 OpenHarmony 开发环境中编译
2. 检查 `bundle.json` 中的依赖组件是否存在

**证据**: `bundle.json:21-41`

---

### Q4: 设备推送失败

**错误信息**:
```
hdc_std: command not found
# 或
Failed to push file: ...
```

**原因**: hdc_std 工具未配置或设备未连接

**解决方案**:
```bash
# 检查设备连接
hdc_std list targets

# 如果设备未识别，检查 USB 调试
```

---

### Q5: 随机测试事件不执行

**现象**: `wukong exec` 运行但无事件注入

**原因**: 可能参数配置问题

**解决方案**:
1. 检查所有事件比例总和是否合理
2. 查看日志级别是否太低

```bash
# 使用调试日志
wukong exec --debug -c 10
```

**证据**: `shell_command/src/wukong_main.cpp:144-151`

---

### Q6: 应用拉起失败

**现象**: `-a/--appswitch` 参数不生效

**原因**:
- BundleName 不存在
- 应用未安装

**解决方案**:
```bash
# 先查询可用应用
wukong appinfo

# 使用正确的 bundle 名称
wukong exec -b com.example.app -c 10
```

---

### Q7: 滑动/点击坐标无效

**现象**: 输入事件注入后无响应

**原因**: 坐标超出屏幕范围

**解决方案**:
```bash
# 获取屏幕分辨率
hdc_std shell wm size

# 使用有效坐标
wukong special -t 300,400  # 确保在屏幕范围内
```

---

### Q8: 报告生成失败

**现象**: 测试完成后无报告输出

**原因**: 存储路径不可写

**解决方案**:
```bash
# 检查存储权限
hdc_std shell ls -la /data/

# 确保有写权限
hdc_std shell mount -o rw,remount /
```

---

## 日志调试

### 日志级别

| 级别 | 选项 | 说明 |
|------|------|------|
| INFO | 默认 | 基本运行信息 |
| TRACK | `--track` | 详细跟踪信息 |
| DEBUG | `--debug` | 调试信息 |

### 启用调试日志

```bash
# 跟踪级别
wukong exec --track -c 10

# 调试级别
wukong exec --debug -c 10
```

**证据**: `shell_command/src/wukong_main.cpp:144-151`

### 日志输出位置

```cpp
// 证据: BUILD.gn:135-137
#define LOG_TAG "WuKong"
#define LOG_DOMAIN 0xD003200
```

日志通过 OpenHarmony `hilog` 系统输出，可通过以下命令查看：

```bash
hdc_std shell hilog | grep WuKong
```

---

## 调试技巧

### 1. 复现随机问题

使用相同的随机种子可以复现问题：

```bash
# 首次运行，记录种子
wukong exec -s 12345 -c 100

# 复现相同序列
wukong exec -s 12345 -c 100
```

**证据**: `README.md:119` - "配置相同随机种子，会生成相同随机事件序列"

### 2. 最小化测试

先使用最小配置确认功能正常：

```bash
# 最小化测试
wukong exec -c 1

# 单事件类型测试
wukong exec -t 1 -c 10
```

### 3. 录制回放调试

```bash
# 录制测试序列
wukong special -r /data/record.txt -c 10

# 回放调试
wukong special -R /data/record.txt
```

### 4. 检查组件树

```bash
# 启用组件遍历测试
wukong special -C com.example.app -c 10
```

---

## 性能问题

### 内存占用

| 阶段 | 预期内存 | 说明 |
|------|----------|------|
| 启动 | ~10MB | 基础加载 |
| 运行 | ~50-100MB | 根据测试复杂度 |
| 报告生成 | 可能临时增加 | 大报告场景 |

### 优化建议

1. **减少并发**: 单实例运行设计
2. **限制时间**: 使用 `-T` 参数限制测试时长
3. **清理缓存**: 定期重启 WuKong

---

## 相关文档

- [命令行接口](02_CommandLine.md)
- [模块详情](03_Module_Details.md)
- [安全评审](05_Security_Review.md)
