# 常见问题

## 构建问题

### 问题 1: 编译失败 - 找不到头文件

**现象**:
```
error: 'napi.h' file not found
#include "napi.h"
```

**原因**: 外部依赖未正确配置

**解决**:
1. 确认 `external_deps` 包含 `napi:ace_napi`
2. 确认 `deps` 包含必要的 utils 库
3. 检查 `include_dirs` 配置

**参考配置** (`interfaces/kits/js/BUILD.gn:38-134`):
```gn
ohos_shared_library("fileio") {
    include_dirs = [
        "src/common",
        "src/common/file_helper",
        "src/common/napi",
        "${utils_path}/common/include",
    ]
    
    deps = [
        "${utils_path}/filemgmt_libhilog:filemgmt_libhilog",
    ]
    
    external_deps = [
        "hilog:libhilog",
        "napi:ace_napi",
        "libuv:uv",
    ]
}
```

### 问题 2: 链接错误 - 未定义符号

**现象**:
```
error: undefined reference to 'NVal::CreateObject'
```

**原因**: 缺少 LibN 库链接

**解决**:
```gn
deps += [
    "${file_api_path}/utils/filemgmt_libn:filemgmt_libn",
]
```

### 问题 3: HyperAIO 编译失败

**现象**:
```
error: 'io_uring.h' file not found
```

**原因**: HyperAIO 需要 liburing 且默认关闭

**解决**:
1. 检查 feature flag
   ```bash
   gn args out --list | grep file_api_feature_hyperaio
   ```
2. 启用 HyperAIO（如需要）
   ```bash
   gn gen out --args='file_api_feature_hyperaio=true'
   ```
3. 如不需要，确保 BUILD.gn 不依赖 HyperAio

## 运行时问题

### 问题 4: 模块加载失败

**现象**:
```
Error: cannot find module '@ohos.file.fs'
```

**排查步骤**:
1. 检查产物是否存在
   ```bash
   adb shell ls -la /system/lib/module/file/libfs.z.so
   ```
2. 检查依赖库
   ```bash
   adb shell ldd /system/lib/module/file/libfs.z.so
   ```
3. 检查文件权限
   ```bash
   adb shell ls -laZ /system/lib/module/file/libfs.z.so
   ```
4. 查看详细日志
   ```bash
   adb logcat | grep -i "file.fs\|dlopen"
   ```

### 问题 5: Permission denied

**现象**:
```
Error: EACCES: permission denied
```

**原因**: 访问权限不足

**解决**:
1. 检查访问路径是否在应用沙箱内
2. 检查是否需要申请权限
   ```javascript
   // 需要权限的操作
   import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
   
   let atManager = abilityAccessCtrl.createAtManager();
   atManager.requestPermissionsFromUser(context, ['ohos.permission.FILE_ACCESS_MANAGER']);
   ```
3. 检查是否为系统应用（部分功能仅限系统应用）

### 问题 6: 文件描述符泄漏

**现象**: 应用运行一段时间后出现 `EMFILE: too many open files`

**原因**: 文件未正确关闭

**解决**:
1. 确保使用 `close()` 或 `closeSync()`
   ```javascript
   // 不好的做法
   let file = fs.openSync('test.txt', fs.OpenMode.READ_ONLY);
   // 忘记关闭
   
   // 好的做法
   let file = fs.openSync('test.txt', fs.OpenMode.READ_ONLY);
   try {
       // 使用 file
   } finally {
       fs.closeSync(file);
   }
   ```
2. 使用 RAII 模式
   ```javascript
   // 使用 with 模式（如支持）
   ```

### 问题 7: 异步操作不回调

**现象**: Promise 永远 pending，Callback 不被调用

**原因**: 多种可能

**排查步骤**:
1. 检查参数是否正确
   ```javascript
   // 确保回调是函数类型
   fs.open('file.txt', mode, 'invalid')  // 错误：第三个参数应该是函数
   ```
2. 检查是否有未捕获的异常
   ```javascript
   process.on('unhandledRejection', (reason, promise) => {
       console.log('Unhandled Rejection:', reason);
   });
   ```
3. 检查系统资源是否耗尽
4. 查看系统日志
   ```bash
   adb logcat -s FileApi
   ```

## 性能问题

### 问题 8: 文件读取慢

**原因**: 使用同步 API 或缓冲区太小

**优化建议**:
1. 使用异步 API
   ```javascript
   // 同步 - 阻塞
   let len = fs.readSync(fd, buffer);
   
   // 异步 - 非阻塞
   let result = await fs.read(fd, buffer);
   ```
2. 使用合适的缓冲区大小（通常 4KB-64KB）
3. 批量读取代替多次小读取

### 问题 9: 内存使用高

**原因**: 大文件一次性读入内存

**解决**:
1. 使用流式读取
   ```javascript
   let stream = fs.createStreamSync('large_file.bin', 'r');
   let buf = new ArrayBuffer(4096);
   while (stream.readSync(buf) > 0) {
       // 处理数据
   }
   stream.closeSync();
   ```
2. 使用 ReaderIterator
   ```javascript
   let iterator = fs.readLinesSync('large_file.txt');
   for (let line of iterator) {
       // 逐行处理
   }
   ```

## 兼容性问题

### 问题 10: API 行为不一致

**现象**: 同一 API 在不同版本表现不同

**解决**:
1. 检查 API 版本兼容性
2. 使用条件编译
   ```cpp
   #ifdef WEARABLE_PRODUCT
       // 穿戴设备特殊处理
   #else
       // 标准处理
   #endif
   ```
3. 查阅版本变更日志

### 问题 11: 模拟器与真机差异

**现象**: 模拟器上正常，真机失败

**常见原因**:
| 差异 | 模拟器 | 真机 |
|------|--------|------|
| 权限 | 较宽松 | 严格沙箱 |
| 文件系统 | 宿主文件系统 | 实际文件系统 |
| 性能 | 较慢 | 正常 |

**解决**:
1. 在真机上测试
2. 注意路径处理差异
3. 检查平台特定代码

## 调试技巧

### 启用详细日志

```cpp
// 在代码中设置
FileApiDebug::isLogEnabled = true;
FileApiDebug::isTraceEnhanced = true;
```

或通过系统参数：
```bash
# 设置调试参数
param set param.key.fileapi.debug.log true
param set param.key.fileapi.debug.trace true
```

### 查看 Native 日志

```bash
# 查看 File API 相关日志
adb logcat -s FileApi:D

# 查看 LibN 日志
adb logcat -s LibN:D

# 查看所有日志
adb logcat | grep -i "file\|fs"
```

### 使用调试工具

1. **strace** - 跟踪系统调用
   ```bash
   adb shell strace -p $(pidof your_app) 2>&1 | grep -E "open|read|write"
   ```

2. **lsof** - 查看打开的文件
   ```bash
   adb shell lsof -p $(pidof your_app)
   ```

3. **procfs** - 查看进程信息
   ```bash
   adb shell cat /proc/$(pidof your_app)/fd/
   ```

## 错误代码速查

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| `ENOENT` | 文件不存在 | 路径错误、文件被删除 |
| `EACCES` | 权限不足 | 沙箱限制、未授权 |
| `EEXIST` | 文件已存在 | 创建已存在文件 |
| `ENOTDIR` | 不是目录 | 对文件使用目录操作 |
| `EISDIR` | 是目录 | 对目录使用文件操作 |
| `EINVAL` | 无效参数 | 参数类型/范围错误 |
| `EBUSY` | 资源忙 | 文件被占用 |
| `EMFILE` | 打开文件过多 | FD 泄漏 |
| `ENOSPC` | 空间不足 | 磁盘已满 |
| `ELOOP` | 符号链接循环 | 路径中有循环链接 |

## 获取帮助

### 内部资源

1. 查看本文档相关章节
2. 检查 `wiki/_work/NOTES.md` 中的已知问题
3. 查看代码注释

### 外部资源

1. [OpenHarmony 文档](https://gitee.com/openharmony/docs)
2. [N-API 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/napi-guidelines.md)
3. [File API 接口文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-file-fs.md)
