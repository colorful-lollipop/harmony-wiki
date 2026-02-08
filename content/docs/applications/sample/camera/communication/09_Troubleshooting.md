# 常见问题与调试

> 常见构建/运行/调试问题与定位路径

---

## 目的

本文档提供 `@ohos/camera_sample_communication` 项目的常见问题和调试指南，帮助开发者快速定位和解决问题。

## 适用范围

本文档适用于：
- 遇到构建问题的开发者
- 遇到运行时问题的工程师
- 需要调试代码的开发人员

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 代码组织方式
- [编译产物](07_Build_Artifacts.md) - 输出产物和部署

---

## 构建问题

### 问题 1：编译错误 - 找不到头文件

**症状**:
```
fatal error: 'utils/includes.h' file not found
```

**原因**: wpa_cli 模块依赖第三方 wpa_supplicant 头文件，但头文件路径不正确。

**证据**: `wpa_cli/BUILD.gn:20-23`

**解决方案**:

1. 确认 wpa_supplicant 已构建：
   ```bash
   hb build -f //third_party/wpa_supplicant/wpa_supplicant-2.9
   ```

2. 检查头文件路径是否正确：
   ```bash
   ls //third_party/wpa_supplicant/wpa_supplicant-2.9/src/utils/includes.h
   ```

3. 如果头文件不存在，检查 wpa_supplicant 源码目录结构。

---

### 问题 2：链接错误 - 找不到库

**症状**:
```
/usr/bin/ld: cannot find -lwpa_client
```

**原因**: libwpa_client.so 未构建或路径不正确。

**证据**: `wpa_cli/BUILD.gn:34-37`

**解决方案**:

1. 确认 wpa_supplicant 已构建：
   ```bash
   hb build -f //third_party/wpa_supplicant/wpa_supplicant-2.9:wpa_supplicant
   ```

2. 检查库文件是否存在：
   ```bash
   find out -name "libwpa_client.so"
   ```

3. 检查 ldflags 配置：
   ```gn
   ldflags = [
      "-L${out_dir}",  # 确认此路径正确
      "-lwpa_client"
   ]
   ```

---

### 问题 3：构建失败 - GN 语法错误

**症状**:
```
ERROR at //applications/sample/camera/communication/hostapd/BUILD.gn:24:5
```

**原因**: BUILD.gn 文件语法错误。

**解决方案**:

1. 检查 GN 语法是否正确：
   - 确保所有括号匹配
   - 确保字符串使用双引号
   - 确保列表使用方括号

2. 使用 GN 格式化工具：
   ```bash
   gn format hostapd/BUILD.gn
   ```

3. 参考 GN 文档：https://gn.googlesource.com/gn/

---

## 运行时问题

### 问题 1：动态库找不到

**症状**:
```
error while loading shared libraries: libwpa.so: cannot open shared object file
```

**原因**: 动态库未安装或路径不正确。

**证据**:
- hostapd: `hostapd/src/hostapd_sample.c:30`
- wpa_supplicant: `wpa_supplicant/src/wpa_sample.c:30`

**解决方案**:

1. 确认库文件已安装：
   ```bash
   ls /usr/lib/libwpa.so
   ls /usr/lib/libwpa_client.so
   ```

2. 如果库文件不存在，安装 wpa_supplicant：
   ```bash
   # 从源码安装
   hb build -f //third_party/wpa_supplicant/wpa_supplicant-2.9
   ```

3. 设置库路径：
   ```bash
   export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
   ```

4. 或者在代码中修改库路径：
   ```c
   void *handleLibWpa = dlopen("/system/lib/libwpa.so", RTLD_NOW | RTLD_LOCAL);
   ```

---

### 问题 2：配置文件找不到

**症状**:
```
[HostapdSample]run ap_main failed, ret:-1
```

**原因**: 配置文件路径不正确或文件不存在。

**解决方案**:

1. 确认配置文件存在：
   ```bash
   ls /etc/hostapd.conf
   ls /etc/wpa_supplicant.conf
   ```

2. 如果配置文件不存在，复制配置文件：
   ```bash
   cp out/ohos-arm-release/etc/hostapd.conf /etc/
   cp out/ohos-arm-release/etc/wpa_supplicant.conf /etc/
   ```

3. 使用绝对路径启动：
   ```bash
   hostapd /etc/hostapd.conf
   wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
   ```

---

### 问题 3：WiFi 接口不存在

**症状**:
```
[HostapdSample]run ap_main failed, ret:-2
```

**原因**: WiFi 网络接口不存在或驱动未加载。

**解决方案**:

1. 检查网络接口：
   ```bash
   ifconfig -a
   ```

2. 检查 WiFi 驱动状态：
   ```bash
   dmesg | grep -i wifi
   ```

3. 如果驱动未加载，加载 WiFi 驱动：
   ```bash
   insmod /usr/lib/xxx_wifi.ko
   ```

4. 修改配置文件中的接口名称：
   ```
   interface=wlan0  # 改为实际的接口名称
   ```

---

### 问题 4：wpa_cli 连接失败

**症状**:
```
[WpaCliSample]connect to wpa failed, err = FAIL.
```

**原因**: wpa_supplicant 未运行或控制接口不可用。

**证据**: `wpa_cli/src/wpa_cli_sample.c:187-197`

**解决方案**:

1. 确认 wpa_supplicant 正在运行：
   ```bash
   ps -A | grep wpa_supplicant
   ```

2. 如果 wpa_supplicant 未运行，启动它：
   ```bash
   wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
   ```

3. 检查控制接口：
   ```bash
   netstat -ul | grep wpa
   ```

4. 确认接口名称正确：
   ```bash
   ifconfig wlan0
   ```

---

## 调试技巧

### 查看日志

**系统日志**:
```bash
dmesg | grep -i wpa
dmesg | grep -i wifi
```

**应用日志**:
```bash
hilog | grep -i wpa
hilog | grep -i wifi
```

**实时日志**:
```bash
hilog | grep -E "wpa|wifi"
```

---

### 跟踪进程

**查看进程状态**:
```bash
ps -A | grep -E "wpa|hostapd"
```

**查看进程详细信息**:
```bash
ps -ef | grep wpa_supplicant
```

**查看进程打开的文件**:
```bash
lsof -p <pid>
```

---

### 网络调试

**查看网络接口**:
```bash
ifconfig -a
```

**查看网络连接**:
```bash
netstat -r
```

**捕获网络数据包**:
```bash
tcpdump -i wlan0 -n
```

**扫描 WiFi 网络**:
```bash
iwlist wlan0 scan
```

---

### GDB 调试

**编译调试版本**:
```bash
hb build -f //applications/sample/camera/communication/wpa_cli --gn-args='is_debug=true'
```

**使用 GDB 调试**:
```bash
gdb --args wpa_cli
```

**GDB 常用命令**:
```
(gdb) break main        # 在 main 函数设置断点
(gdb) run              # 运行程序
(gdb) next             # 单步执行
(gdb) print variable   # 打印变量值
(gdb) backtrace        # 查看调用栈
(gdb) continue         # 继续执行
```

---

### 内存调试

**使用 Valgrind 检查内存泄漏**:
```bash
valgrind --leak-check=full --show-leak-kinds=all ./wpa_cli
```

**使用 AddressSanitizer**:
```bash
hb build -f //applications/sample/camera/communication/wpa_cli --gn-args='use_asan=true'
```

---

## 常见错误消息

### 错误消息：dlopen libwpa failed

**含义**: 动态加载 libwpa.so 失败

**可能原因**:
1. 库文件不存在
2. 库路径不正确
3. 库文件权限不正确

**解决方案**:
1. 检查库文件是否存在：`ls /usr/lib/libwpa.so`
2. 检查库路径是否正确
3. 检查库文件权限：`ls -l /usr/lib/libwpa.so`

**证据**: `hostapd/src/hostapd_sample.c:32`

---

### 错误消息：dlsym ap_main failed

**含义**: 查找 ap_main 符号失败

**可能原因**:
1. libwpa.so 中没有 ap_main 符号
2. libwpa.so 版本不匹配

**解决方案**:
1. 检查 libwpa.so 中的符号：`nm -D /usr/lib/libwpa.so | grep ap_main`
2. 确认 wpa_supplicant 版本正确

**证据**: `hostapd/src/hostapd_sample.c:39`

---

### 错误消息：send ctrl request failed

**含义**: 发送控制命令失败

**可能原因**:
1. wpa_supplicant 未运行
2. 控制接口不可用
3. 命令格式错误

**解决方案**:
1. 确认 wpa_supplicant 正在运行：`ps -A | grep wpa_supplicant`
2. 检查命令格式是否正确
3. 使用 PING 命令测试连接

**证据**: `wpa_cli/src/wpa_cli_sample.c:136`

---

### 错误消息：connect to wpa failed

**含义**: 连接 wpa_supplicant 失败

**可能原因**:
1. wpa_supplicant 未运行
2. 控制接口不可用
3. 接口名称错误

**解决方案**:
1. 确认 wpa_supplicant 正在运行：`ps -A | grep wpa_supplicant`
2. 检查控制接口：`netstat -ul | grep wpa`
3. 确认接口名称正确：`ifconfig wlan0`

**证据**: `wpa_cli/src/wpa_cli_sample.c:196`

---

## 性能问题

### 问题：扫描速度慢

**症状**: WiFi 扫描需要很长时间

**解决方案**:

1. 检查扫描超时设置：
   ```bash
   wpa_cli SCAN_INTERVAL 5
   ```

2. 减少扫描频率：
   ```c
   // 修改 TestScan() 函数
   ```

3. 使用后台扫描模式

---

### 问题：连接超时

**症状**: WiFi 连接超时

**解决方案**:

1. 检查信号强度：
   ```bash
   iwconfig wlan0
   ```

2. 增加连接超时时间：
   ```
   # 在配置文件中添加
   bgscan="simple:30:-70:30"
   ```

3. 检查路由器设置

---

## 开发建议

### 代码风格

- 遵循 OpenHarmony C 代码风格指南
- 使用 4 空格缩进
- 每行不超过 100 字符

### 错误处理

- 所有函数调用都应检查返回值
- 使用有意义的错误消息
- 在错误发生时清理资源

### 日志

- 使用 hilog 替代 printf
- 使用适当的日志级别
- 避免在日志中输出敏感信息

### 测试

- 编写单元测试
- 进行集成测试
- 进行性能测试

---

## 获取帮助

### 文档资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [wpa_supplicant 官方文档](https://w1.fi/wpa_supplicant/)
- [GN 构建系统文档](https://gn.googlesource.com/gn/)

### 社区资源

- [OpenHarmony Gitee](https://gitee.com/openharmony)
- [OpenHarmony 论坛](https://developer.huawei.com/consumer/cn/forum/)

### 报告问题

如果遇到文档中未涵盖的问题，请通过以下方式报告：

1. 搜索 Issue 列表
2. 创建新的 Issue
3. 提供详细的错误信息和复现步骤
