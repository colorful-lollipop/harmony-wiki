# 常见问题

## 概述

本文档收录 request_cangjie_wrapper 使用过程中的常见问题及解决方案。

## 构建问题

### Q1: 编译时提示找不到 cangjie_ark_interop

**问题描述**:
```
error: cannot find module 'cangjie_ark_interop'
```

**原因**: 依赖组件未先编译

**解决方案**:
```bash
# 先编译依赖组件
hb build -p cangjie_ark_interop
hb build -p arkui_cangjie_wrapper
hb build -p hiviewdfx_cangjie_wrapper
hb build -p ability_cangjie_wrapper
hb build -p request

# 再编译本模块
hb build -p request_cangjie_wrapper
```

**相关文件**: `ohos/request/BUILD.gn:31-34`

---

### Q2: Windows/Mac 平台编译失败

**问题描述**: 在 Windows 或 macOS 上编译时使用了错误的源码

**原因**: 平台判断条件 `is_mingw || is_mac` 导致使用了 mock 实现

**解决方案**:
1. 确认是否需要在非标准平台构建
2. 如需全功能，请使用 Linux 标准设备环境
3. 仅测试场景可直接使用 mock

**相关文件**: `ohos/request/BUILD.gn:20-28`

---

### Q3: SDK 复制目标执行失败

**问题描述**: `copy_sdk_request_cangjie_libs` 目标执行失败

**原因**: 源目标未编译完成

**解决方案**:
```bash
# 确保模块先编译
hb build -p request_cangjie_wrapper --build-only-gn

# 然后执行复制
hb build -p request_cangjie_wrapper
```

**相关文件**: `BUILD.gn:19-21`

---

### Q4: 编译产物大小异常

**问题描述**: 编译产物的 ROM/RAM 占用与预期不符

**原因**: 可能是 Debug 版本或未优化

**解决方案**: 使用 Release 配置编译
```bash
hb build -p request_cangjie_wrapper --build-target release
```

**预期值**: ROM 400KB, RAM 332KB

---

## 运行时问题

### Q5: 下载任务创建失败

**问题描述**: 调用 `createDownload()` 抛出异常

**常见原因**:
1. savePath 指定的文件已存在
2. URL 格式无效
3. 缺少必要权限

**排查步骤**:
```cangjie
try {
    let task = RequestAgent.createDownload(config)
} catch (e: BusinessException) {
    // 检查错误码和消息
    print("错误码: ${e.code}")
    print("错误消息: ${e.message}")
}
```

**权限检查**:
```bash
# 确认应用已申请权限
# 在 config.json 中检查
"requestPermissions": [
    {"name": "ohos.permission.INTERNET"},
    {"name": "ohos.permission.WRITE_MEDIA"},
    {"name": "ohos.permission.READ_MEDIA"}
]
```

---

### Q6: 任务状态不更新

**问题描述**: 订阅了进度事件但回调未被触发

**可能原因**:
1. 任务未正确启动
2. 回调注册时机错误
3. 网络请求未成功

**排查步骤**:
```cangjie
// 1. 确认任务创建成功
let task = RequestAgent.createDownload(config)

// 2. 在启动前注册回调
task.onProgress { progress ->
    print("进度: ${progress.progress}%")
}

// 3. 启动任务
task.start()
```

---

### Q7: 文件上传部分失败

**问题描述**: 多文件上传时部分文件失败但任务仍显示成功

**原因**: 多文件上传策略为任务维度判断

**解决方案**: 使用单文件上传或手动检查
```cangjie
// 检查单个文件状态
task.onComplete { result ->
    // 手动验证所有文件
}
```

**相关文档**: `README.md:72`

---

### Q8: 断点续传不生效

**问题描述**: 暂停后恢复任务，但从头开始下载

**可能原因**:
1. 服务器不支持 Range 请求
2. savePath 文件被删除
3. 临时文件被清理

**解决方案**:
```cangjie
// 确认服务器支持 Range 请求
// 不要手动删除 savePath 文件
// 保持任务连续性
task.pause()
task.resume()  // 续传
```

---

### Q9: 回调函数执行线程

**问题描述**: 回调中执行 UI 操作导致崩溃

**原因**: 回调可能不在主线程执行

**解决方案**: 使用 UI 上下文调度
```cangjie
import { uiAbility } from '@ohos.app.ability.ui_ability'

task.onProgress { progress ->
    // 在主线程执行 UI 操作
    uiAbility.context.runScopedTask {
        // UI 更新
    }
}
```

---

### Q10: 内存占用过高

**问题描述**: 大文件下载时内存占用持续增长

**可能原因**:
1. 未正确处理响应流
2. 回调中创建了临时对象
3. 缓存未及时释放

**优化建议**:
```cangjie
// 及时移除不需要的任务
task.onComplete { _ ->
    RequestAgent.remove(task.taskId)
}

// 避免在回调中创建大对象
```

---

## 调试问题

### Q11: 如何打印日志

**问题描述**: 需要追踪代码执行流程

**解决方案**: 使用 hilog
```cangjie
import { hilog } from '@ohos.hiviewdfx.hilog'

hilog.info(0x0001, "RequestAgent", "创建下载任务: %{public}s", url)
```

**日志级别**:
- DEBUG: 0x0000
- INFO: 0x0001
- WARN: 0x0002
- ERROR: 0x0003

---

### Q12: 抓取网络包

**问题描述**: 需要分析网络请求细节

**解决方案**:
1. 使用 Charles/Fiddler 代理
2. 配置设备网络代理
3. 安装 CA 证书（需 root）

**注意**: 只能抓取 HTTP，HTTPS 需要解密配置

---

### Q13: 任务调试

**问题描述**: 需要查看任务内部状态

**解决方案**:
```cangjie
// 查询任务当前状态
let task = RequestAgent.query(taskId)

// 查看任务配置
if (task != null) {
    print("任务ID: ${task.taskId}")
    print("任务状态: ${task.status}")
    print("进度: ${task.progress?.progress}%")
}
```

---

### Q14: 异常堆栈获取

**问题描述**: 捕获异常但无详细堆栈

**解决方案**:
```cangjie
try {
    RequestAgent.createDownload(config)
} catch (e: BusinessException) {
    // 打印完整堆栈
    e.printStackTrace()
}
```

---

### Q15: 性能分析

**问题描述**: 需要分析任务执行性能

**工具推荐**:
1. DevEco Studio Profiler
2. hdc shell perf
3. systrace

**分析方法**:
```bash
# 录制性能数据
hdc shell perf record -a -g -- sleep 30

# 分析结果
hdc shell perf report
```

---

## API 使用问题

### Q16: 与 ArkTS API 差异

**问题描述**: 从 ArkTS 迁移到 Cangjie 发现 API 不同

**已知差异**:
| 功能 | ArkTS | Cangjie |
|------|-------|---------|
| 速率限制 | 支持 | 不支持 |
| 失败原因订阅 | 支持 | 不支持 |
| 等待原因订阅 | 支持 | 不支持 |

**相关文档**: `README.md:73-76`

---

### Q17: 返回值类型不匹配

**问题描述**: API 返回值类型与预期不符

**解决方案**: 检查 API 签名
```cangjie
// 确认方法签名
RequestAgent.createDownload(config): RequestTask
RequestAgent.query(taskId): RequestTask?
```

---

### Q18: 事件监听重复触发

**问题描述**: 同一个事件被触发多次

**原因**: 未正确移除监听器

**解决方案**:
```cangjie
// 保存回调引用
let progressCallback = task.onProgress { _ ->
    // 处理
}

// 任务完成时移除
task.onComplete { _ ->
    task.off(progressCallback)
}
```

---

## 平台兼容性

### Q19: 标准设备 vs 轻量设备

**问题描述**: 模块在轻量设备上不可用

**原因**: 配置仅支持 standard 系统类型

**说明**:
- 本模块仅支持标准设备 (standard)
- 轻量设备需使用其他上传下载方案

**相关配置**: `bundle.json:20-21`

---

### Q20: 版本兼容性

**问题描述**: 不同 OpenHarmony 版本 API 差异

**兼容性矩阵**:
| 本模块版本 | OpenHarmony 版本 |
|-----------|------------------|
| 6.1 | 4.1+ |
| 6.0 | 4.0 |
| 5.0 | 3.2+ |

---

## 性能相关

### Q21: 批量下载优化

**问题描述**: 多个下载任务性能不佳

**优化建议**:
```cangjie
// 限制并发数
let maxConcurrent = 3
let semaphore = Semaphore(maxConcurrent)

// 使用信号量控制
for (url in urls) {
    semaphore.acquire()
    RequestAgent.createDownload(config).onComplete { _ ->
        semaphore.release()
    }
}
```

---

### Q22: 大文件上传内存优化

**问题描述**: 大文件上传 OOM

**解决方案**:
```cangjie
// 使用流式上传（如果支持）
// 避免一次加载整个文件
// 分块上传
```

---

## 错误码速查

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| 0 | 成功 | - |
| 1 | 未知错误 | 异常未捕获 |
| 文件已存在 | savePath 冲突 | 文件已存在 |
| 网络错误 | 连接失败 | 网络不可用 |
| 超时 | 请求超时 | 网络慢/服务器慢 |

**证据来源**: `error.cj` 文件定义

## 反馈渠道

### 问题反馈

如遇本文档未收录的问题，请通过以下渠道：

1. **Issue**: OpenHarmony Issues
2. **邮件列表**: dev@openharmony.io
3. **Gitee**: 项目 Issue 跟踪

### 贡献指南

欢迎贡献 FAQ 补充：
1. 提交 Issue 说明问题
2. 提供解决方案
3. 经过验证后合入文档
