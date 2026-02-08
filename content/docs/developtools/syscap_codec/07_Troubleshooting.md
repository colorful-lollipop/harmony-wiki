# 07_Troubleshooting - 常见问题

## 目的

本文档整理 `syscap_codec` 常见的构建、运行和调试问题及解决方案。

## 适用范围

- 遇到问题的开发者
- 需要调试工具的工程师

## 构建问题

### 问题1: 编译失败 - 找不到 cJSON 头文件

**现象**:
```
error: 'cJSON.h' file not found
#include "cJSON.h"
         ^~~~~~~~~
```

**原因**: cJSON 组件未正确引入

**解决方案**:
1. 确认 `bundle.json` 中已声明依赖:
   ```json
   "deps": {
     "components": [
       "cJSON"
     ]
   }
   ```

2. 对于 Lite 系统，检查是否正确使用了内部 cJSON:
   ```gn
   if (defined(ohos_lite)) {
     deps += [ "//build/lite/config/component/cJSON:cjson_static" ]
   } else {
     external_deps += [ "cJSON:cjson_static" ]
   }
   ```

**参考**: `BUILD.gn:59-63`

---

### 问题2: 链接失败 - 未定义的符号

**现象**:
```
undefined reference to `EncodeOsSyscap'
```

**原因**: 动态库导出符号配置不正确

**解决方案**:
1. 检查 `libsyscap_interface_shared.versionscript` 是否包含该符号
2. 确认函数实现存在于 `syscap_interface.c`
3. 重新编译动态库

**参考**: `libsyscap_interface_shared.versionscript`

---

### 问题3: Windows 交叉编译失败

**现象**:
```
error: realpath is not available on Windows
```

**原因**: Windows 环境下缺少 POSIX 函数

**解决方案**:
1. 确保使用 MinGW 工具链
2. 检查是否正确定义了 `_POSIX_` 宏:
   ```gn
   if (is_mingw) {
     defines += [ "_POSIX_" ]
   }
   ```

**参考**: `BUILD.gn:47-49`

---

### 问题4: Mac 上无法编译 Linux 版本

**现象**:
编译成功但生成的二进制无法在 Linux 运行

**原因**: Ubuntu 只能编译 Windows/Linux 版本，Mac 版本需要在 macOS 上编译

**解决方案**:
- 在 macOS 开发机上编译 macOS 版本
- 使用 CI/CD 系统进行交叉编译

**参考**: `README_ZH.md:72`

## 运行问题

### 问题5: 运行时崩溃 - 找不到 pcid.sc

**现象**:
```
ERROR: GetFileContext failed, input file : /system/etc/pcid.sc
```

**原因**: PCID 文件不存在或路径错误

**解决方案**:
1. 确认设备上存在 `/system/etc/pcid.sc`
2. 检查文件权限:
   ```bash
   ls -la /system/etc/pcid.sc
   ```
3. 重新生成 PCID:
   ```bash
   syscap_tool -P -e -i SystemCapability.json -o /system/etc/
   ```

**参考**: `interfaces/inner_api/syscap_interface.c:61`

---

### 问题6: N-API 接口返回空字符串

**现象**:
JS 调用 `querySystemCapabilities()` 返回空字符串

**原因**: 
1. PCID 文件读取失败
2. 内存分配失败

**排查步骤**:
1. 检查设备日志:
   ```bash
   hilog | grep syscap
   ```
2. 确认 PCID 文件存在且可读
3. 检查内存使用情况

**参考**: `napi/napi_query_syscap.cpp:177-228`

---

### 问题7: 兼容性检查失败

**现象**:
```
Fail! The pcid does not meet the rpcid
Missing: SystemCapability.xxx.xxx
```

**原因**: 设备不支持应用所需的系统能力

**解决方案**:
1. 检查应用 manifest 中声明的 syscap
2. 确认设备是否支持这些能力
3. 更新设备固件或调整应用需求

**参考**: `src/syscap_tool.c:761-765`

---

### 问题8: JSON 解析失败

**现象**:
```
ERROR: cJSON_Parse failed, context buffer is:
```

**原因**: 输入 JSON 格式不正确

**排查步骤**:
1. 检查 JSON 语法:
   ```bash
   python3 -m json.tool input.json
   ```
2. 确认 JSON 包含必需的字段:
   - PCID: `api_version`, `system_type`, `manufacturer_id`, `syscap`
   - RPCID: `api_version`, `syscap`
3. 检查文件编码（应为 UTF-8）

**参考**: `src/create_pcid.c:259`, `src/syscap_tool.c:157`

## 调试方法

### 启用详细日志

代码中已定义 `PRINT_ERR` 宏用于输出错误信息:

```c
// include/context_tool.h:24-28
#define PRINT_ERR(...) \
    do { \
        printf("ERROR: [%s: %d] -> ", __FILE__, __LINE__); \
        printf(__VA_ARGS__); \
    } while (0)
```

**使用方式**:
1. 查看标准错误输出
2. 在设备上使用 `hilog` 查看日志

---

### 命令行调试

**查看帮助**:
```bash
./syscap_tool --help
```

**验证 PCID 文件**:
```bash
# 解码 PCID
./syscap_tool -Pdi /system/etc/pcid.sc -o /tmp/
cat /tmp/pcid.json
```

**验证 RPCID 文件**:
```bash
# 解码 RPCID
./syscap_tool -Rdi rpcid.sc -o /tmp/
cat /tmp/rpcid.json
```

**比较兼容性**:
```bash
# 文件比较
./syscap_tool -C pcid.txt rpcid.txt

# 字符串比较
./syscap_tool -sC "pcidstring" "rpcidstring"
```

---

### 使用 GDB 调试

**调试命令行工具**:
```bash
gdb ./syscap_tool
(gdb) set args -Pdi input.json -o /tmp/
(gdb) break main
(gdb) run
```

**常用断点位置**:
- `src/main.c:71` - 程序入口
- `src/syscap_tool.c:145` - RPCID编码
- `src/create_pcid.c:247` - PCID创建
- `interfaces/inner_api/syscap_interface.c:81` - 内部API

---

### 内存问题检测

**使用 AddressSanitizer**:
```gn
# 在 BUILD.gn 中添加
config("asan") {
  cflags = [ "-fsanitize=address", "-fno-omit-frame-pointer" ]
  ldflags = [ "-fsanitize=address" ]
}

ohos_executable("syscap_tool_bin") {
  configs += [ ":asan" ]
  # ...
}
```

## 性能优化

### 减少内存分配

**问题**: 频繁的大内存分配影响性能

**优化建议**:
1. 使用栈缓冲区代替堆分配（小数据）
2. 复用缓冲区而不是重复分配
3. 预计算所需内存大小

### 文件读取优化

**当前实现**: `src/context_tool.c:40-93`

**优化建议**:
1. 使用内存映射（mmap）代替 fread（大文件）
2. 添加文件缓存机制

## 常见问题速查表

| 问题 | 快速检查 | 解决方案 |
|------|----------|----------|
| 编译失败 | 检查依赖 | 确认 bundle.json 和 BUILD.gn |
| 链接失败 | 检查符号导出 | 检查 versionscript 文件 |
| 运行时崩溃 | 检查 pcid.sc | 确认文件存在且可读 |
| 返回空值 | 检查日志 | 使用 hilog 查看错误 |
| JSON解析失败 | 验证JSON格式 | 使用 json.tool 检查 |
| 兼容性检查失败 | 检查 syscap 列表 | 对比设备和应用需求 |

## 相关跳转

- [项目概览](00_Overview.md) - 运行环境说明
- [架构说明](02_Architecture.md) - 数据流说明
- [安全风险分析](06_Security_Analysis.md) - 安全问题排查
