# SecurityGuard 调试与排错指南

**文档版本**: 3.1.0  
**最后更新**: 2026-02-06

---

## 1. 日志系统

### 1.1 日志标签与过滤

SecurityGuard 使用 OpenHarmony HiLog 系统输出日志。

**日志标签**:
| 标签 | 服务 | 域值 |
|------|------|------|
| `SG_Service` | SecurityGuard 服务层 | 0xD002F07 |
| `S_COLLCTOR` | SecurityCollector 采集器层 | 0xD002F07 |

**日志宏定义**:
```cpp
// 服务层宏
SGLOGD(fmt, ...)  // DEBUG 级别
SGLOGI(fmt, ...)  // INFO 级别
SGLOGW(fmt, ...)  // WARN 级别
SGLOGE(fmt, ...) // ERROR 级别
SGLOGF(fmt, ...) // FATAL 级别

// 采集器层宏
LOGD(fmt, ...)
LOGI(fmt, ...)
LOGW(fmt, ...)
LOGE(fmt, ...)
LOGF(fmt, ...)
```

### 1.2 日志查看命令

**基础命令**:
```bash
# 查看所有 SecurityGuard 日志
hilog | grep -E "SG_Service|S_COLLCTOR"

# 实时跟踪日志
hilog -g

# 按标签过滤
hilog -t SG_Service
hilog -t S_COLLCTOR

# 按域值过滤
hilog -T 0xD002F07
```

**按级别过滤**:
```bash
# 只显示 ERROR 及以上级别
hilog -l error

# 显示 DEBUG 及以上级别
hilog -l debug

# 组合过滤: 标签 + 级别
hilog -t SG_Service -l error

# 组合过滤: 域值 + 级别
hilog -T 0xD002F07 -l info
```

**导出日志**:
```bash
# 导出到文件
hilog > /data/security_guard.log

# 带时间戳导出
hilog -v time > /data/sg_log_$(date +%Y%m%d_%H%M%S).log

# 导出特定标签
hilog -t SG_Service > /data/sg_service.log
```

### 1.3 日志含义解析

**关键日志条目**:
| 日志内容 | 含义 | 排查方向 |
|----------|------|----------|
| `[NapiGetModelResult]` | 收到 JS 层模型查询请求 | 检查输入参数 |
| `[RequestSecurityModelResult]` | 发起 IPC 调用 | 检查 SA 服务状态 |
| `[ResponseSecurityModelResult]` | 收到模型结果 | 检查结果解析 |
| `[HasPermission]` | 权限校验 | 检查权限配置 |
| `[CheckRiskContent]` | 内容校验 | 检查输入格式 |

---

## 2. 错误码速查

### 2.1 JS 层错误码

**错误码定义位置**: `frameworks/js/napi/security_guard_napi.cpp:75-81`

| 错误码 | 常量名 | 含义 | 处理建议 |
|--------|--------|------|----------|
| 0 | `JS_ERR_SUCCESS` | 操作成功 | 正常流程 |
| 201 | `JS_ERR_NO_PERMISSION` | 无权限 | 动态申请权限 |
| 202 | `JS_ERR_NO_SYSTEMCALL` | 非系统应用 | 需要系统应用签名 |
| 401 | `JS_ERR_BAD_PARAM` | 参数错误 | 检查参数格式和范围 |
| 801 | `JS_ERR_API_SUPPORT_ERROR` | API 不支持 | 检查系统版本 |
| 21200001 | `JS_ERR_SYS_ERR` | 系统错误 | 查看系统日志 |

### 2.2 内部错误码

**错误码定义位置**: `frameworks/common/constants/include/security_guard_define.h`

| 错误码 | 常量名 | 含义 | 处理建议 |
|--------|--------|------|----------|
| 0 | `SUCCESS` | 成功 | - |
| 1 | `FAILED` | 通用失败 | 检查日志详情 |
| 2 | `NO_PERMISSION` | 无权限 | 申请权限 |
| 3 | `NO_SYSTEMCALL` | 非系统调用 | 检查应用类型 |
| 4 | `STREAM_ERROR` | 流错误 | 检查文件描述符 |
| 5 | `FILE_ERR` | 文件错误 | 检查文件路径 |
| 6 | `BAD_PARAM` | 参数错误 | 检查参数有效性 |
| 7 | `JSON_ERR` | JSON 解析错误 | 检查 JSON 格式 |
| 8 | `NULL_OBJECT` | 空对象 | 检查对象是否初始化 |
| 9 | `TIME_OUT` | 超时 | 检查网络或重试 |
| 10 | `NOT_FOUND` | 未找到 | 检查配置或数据 |
| 14-17 | `DB_*_ERR` | 数据库错误 | 检查数据库完整性 |
| 1005-1008 | `FILTER_*_LIMIT` | 过滤器/客户端超限 | 减少数量 |
| 801 | `API_SUPPORT_ERROR` | API 不支持 | 升级系统版本 |

### 2.3 错误码速查表

**按场景分类**:

| 场景 | 常见错误码 | 排查步骤 |
|------|------------|----------|
| 权限相关 | 2, 201, 202 | 1. 检查 module.json5 权限声明<br>2. 动态申请权限<br>3. 检查 APL 等级 |
| 参数相关 | 6, 401, 7 | 1. 检查参数类型<br>2. 检查参数范围<br>3. 检查 JSON 格式 |
| 服务相关 | 1, 9, 10 | 1. 检查 SA 服务状态<br>2. 查看日志<br>3. 重试操作 |
| 数据库相关 | 14, 15, 16, 17 | 1. 检查存储空间<br>2. 检查数据库文件<br>3. 重启服务 |
| 资源加载 | 5 | 1. 检查文件路径<br>2. 检查文件权限<br>3. 检查库兼容性 |

---

## 3. 常见问题与解决方案

### 3.1 模块导入问题

**问题**: 导入 SecurityGuard 模块失败

**错误表现**:
```javascript
import securityGuard from '@ohos.security.securityGuard';
// 运行时: securityGuard is undefined
```

**排查步骤**:
```bash
# 1. 检查模块是否在设备上
ls /system/lib/module/security/
# 应看到: securityguard_napi.z.so

# 2. 检查系统能力
hilog | grep "SystemCapability.Security.SecurityGuard"

# 3. 检查系统版本
param get const.version
# 应 >= 3.1.0
```

**解决方案**:
1. 确认设备系统版本 >= 3.1.0
2. 确认模块已正确安装
3. 重启设备

### 3.2 权限问题

**问题**: API 调用返回错误码 201

**错误表现**:
```javascript
const result = await securityGuard.getModelResult('SecurityGuard_JailbreakCheck');
// 抛出: { code: 201, message: "check permission fail" }
```

**排查步骤**:
```bash
# 1. 检查应用权限声明
cat config.json | grep -A 10 "requestPermissions"

# 2. 检查权限是否已授予
hilitool perm check <package_name> ohos.permission.QUERY_SECURITY_EVENT

# 3. 检查 APL 等级
cat module.json5 | grep '"apl"'
```

**解决方案**:
```json
// 1. 在 module.json5 中声明权限
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.QUERY_SECURITY_EVENT"
      },
      {
        "name": "ohos.permission.COLLECT_SECURITY_EVENT"
      }
    ]
  }
}

// 2. 动态申请权限（可选）
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';

async function requestPermissions() {
  const atManager = abilityAccessCtrl.createAtManager();
  const bundleInfo = await bundleManager.getBundleInfoForSelf(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
  );
  
  atManager.requestPermissionsFromUser(
    bundleInfo.appInfo.accessTokenId,
    ['ohos.permission.QUERY_SECURITY_EVENT']
  );
}
```

### 3.3 参数校验问题

**问题**: API 调用返回错误码 401

**错误表现**:
```javascript
// 参数类型错误
await securityGuard.getModelResult(3001000000);
// 抛出: { code: 401, message: "Parameter error" }
```

**正确用法**:
```javascript
// 模型名称是字符串
await securityGuard.getModelResult('SecurityGuard_JailbreakCheck');

// 事件 ID 是数字
const success = securityGuard.reportSecurityEvent({
    eventId: 1011015005,  // 数字类型
    version: '1.0',       // 字符串类型
    content: JSON.stringify({ type: 'test' })  // 字符串类型
});
```

### 3.4 服务未启动问题

**问题**: API 调用无响应或超时

**错误表现**:
```javascript
// Promise 从未 resolve
await securityGuard.getModelResult('SecurityGuard_JailbreakCheck');
// 等待 30 秒后超时
```

**排查步骤**:
```bash
# 1. 检查 SA 服务状态
sm -s
# 应看到:
# 3523: RiskAnalysisManager
# 3524: DataCollectManager
# 3525: SecurityCollectorManager

# 2. 检查服务是否已启动
hidumper -s 3523
hidumper -s 3524
hidumper -s 3525

# 3. 查看服务启动日志
hilog | grep -E "RiskAnalysisManager|DataCollectManager|SecurityCollectorManager"
```

**解决方案**:
```javascript
// 设置超时
function withTimeout(promise, timeoutMs = 10000) {
    return Promise.race([
        promise,
        new Promise((_, reject) =>
            setTimeout(() => reject(new Error('Service timeout')), timeoutMs)
        )
    ]);
}

// 使用
try {
    const result = await withTimeout(
        securityGuard.getModelResult('SecurityGuard_JailbreakCheck'),
        15000  // 15秒超时
    );
} catch (error) {
    if (error.message === 'Service timeout') {
        console.error('SA 服务响应超时');
    }
}
```

### 3.5 内存不足问题

**问题**: 返回错误码 1004

**错误表现**:
```javascript
const result = await securityGuard.getModelResult('SecurityGuard_JailbreakCheck');
// 返回: { code: 1004, message: "Out of memory" }
```

**排查步骤**:
```bash
# 1. 检查设备内存
cat /proc/meminfo

# 2. 检查 SecurityGuard 内存使用
cat /proc/<sg_pid>/status | grep VmRSS

# 3. 查看内存分配日志
hilog | grep -E "memory|Memory|alloc"
```

**解决方案**:
1. 释放不需要的资源
2. 减少缓存的事件数量
3. 重启设备

---

## 4. SA 服务调试

### 4.1 服务状态检查

```bash
# 查看所有 SA 服务状态
sm -s

# 查看特定服务状态
hidumper -s 3523  # RiskAnalysisManager
hidumper -s 3524  # DataCollectManager
hidumper -s 3525  # SecurityCollectorManager
```

### 4.2 服务重启

```bash
# 停止 SA 服务
killall sg_collect_service
killall sg_classify_service
killall security_collector_service

# 启动 SA 服务
start sg_collect_service
start sg_classify_service
start security_collector_service
```

### 4.3 服务日志级别调整

```bash
# 查看当前日志级别
hilog -b

# 设置 SecurityGuard 日志级别为 DEBUG
hilog -b D -T 0xD002F07

# 设置所有日志为 ERROR
hilog -b E
```

---

## 5. 性能监控

### 5.1 API 调用耗时

```javascript
// 包装 API 调用以测量耗时
async function measureTime(apiCall, apiName) {
    const startTime = Date.now();
    try {
        const result = await apiCall();
        const duration = Date.now() - startTime;
        console.log(`[${apiName}] 耗时: ${duration}ms`);
        return result;
    } catch (error) {
        const duration = Date.now() - startTime;
        console.error(`[${apiName}] 失败: ${duration}ms, 错误:`, error);
        throw error;
    }
}

// 使用
const result = await measureTime(
    () => securityGuard.getModelResult('SecurityGuard_JailbreakCheck'),
    'getModelResult'
);
```

### 5.2 内存使用监控

```javascript
// 获取应用内存使用
import resourceManager from '@ohos.resourceManager';

async function getMemoryInfo() {
    const memInfo = await resourceManager.getAppMemoryInfo();
    return {
        appMemory: memInfo.appMemory,           // 应用内存 (MB)
        pid: memInfo.pid,                       // 进程 ID
        processName: memInfo.processName        // 进程名
    };
}

// 定期检查内存
setInterval(async () => {
    const mem = await getMemoryInfo();
    console.log(`内存使用: ${mem.appMemory}MB`);
}, 60000);  // 每分钟
```

### 5.3 事件队列监控

```javascript
// 监控事件队列长度
let eventQueueSize = 0;

const monitor = {
    onQuery: (events) => {
        eventQueueSize = events.length;
        console.log(`事件队列: ${events.length}`);
    }
};

// 在查询时监控
securityGuard.querySecurityEvent(rules, {
    ...monitor,
    onComplete: () => {
        console.log(`查询完成, 最终队列: ${eventQueueSize}`);
    }
});
```

---

## 6. 调试技巧

### 6.1 开启详细日志

```javascript
// 开发环境开启详细日志
const isDebug = __DEBUG__;  // 或通过配置控制

if (isDebug) {
    // 监听 N-API 调用
    const originalGetModelResult = securityGuard.getModelResult;
    securityGuard.getModelResult = async function(modelName) {
        console.log(`[DEBUG] 调用 getModelResult: ${modelName}`);
        const startTime = Date.now();
        try {
            const result = await originalGetModelResult.call(this, modelName);
            console.log(`[DEBUG] getModelResult 完成: ${Date.now() - startTime}ms`);
            return result;
        } catch (error) {
            console.error(`[DEBUG] getModelResult 失败:`, error);
            throw error;
        }
    };
}
```

### 6.2 异常捕获

```javascript
// 全局异常处理器
window.onerror = function(msg, url, line, column, error) {
    console.error(`[全局错误] ${msg} at ${url}:${line}:${column}`);
    console.error('堆栈:', error?.stack);
    return false;
};

// Promise 异常捕获
process.on('unhandledRejection', (reason, promise) => {
    console.error('[未捕获的 Promise 拒绝]:', reason);
});

// async/await 异常
try {
    await securityGuard.getModelResult('SecurityGuard_JailbreakCheck');
} catch (error) {
    console.error('[API 错误]:', {
        code: error.code,
        message: error.message,
        stack: error.stack
    });
}
```

### 6.3 网络调试

```bash
# 查看 IPC 通信
hilog | grep -E "IPC|Proxy|Stub"

# 查看配置加载
hilog | grep -E "Config|Load|Parse"

# 查看数据库操作
hilog | grep -E "SQL|DB|Database"
```

---

## 7. 常用调试命令速查

### 7.1 日志命令

```bash
# 查看所有 SecurityGuard 日志
hilog | grep -E "SG_Service|S_COLLCTOR|0xD002F07"

# 实时跟踪
hilog -g | grep SG_Service

# 导出日志
hilog > /data/sg_debug_$(date +%Y%m%d_%H%M%S).log
```

### 7.2 服务命令

```bash
# 查看 SA 服务
sm -s | grep security

# 查看服务状态
hidumper -s 3523
hidumper -s 3524
hidumper -s 3525

# 重启服务
killall sg_collect_service
start sg_collect_service
```

### 7.3 权限命令

```bash
# 查看应用权限
hilitool perm list <package_name>

# 检查权限状态
hilitool perm check <package_name> ohos.permission.QUERY_SECURITY_EVENT
```

### 7.4 文件命令

```bash
# 查看配置文件
cat /system/etc/security_guard_event.json
cat /data/service/el1/public/security_guard/security_guard_event.json

# 查看库文件
ls -la /system/lib/module/security/
ls -la /system/lib/ | grep sg
```

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [快速开始](./07_QuickStart.md) | 5分钟上手教程 |
| [API 参考](./02_NAPI_Reference.md) | 完整 API 文档 |
| [架构详解](./03_Architecture.md) | 系统架构说明 |
| [构建配置](./04_Build.md) | 构建与编译 |
| [安全评审](./06_Security_Review.md) | 安全风险分析 |
