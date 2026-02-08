# SecurityGuard 快速开始指南

**文档版本**: 3.1.0  
**最后更新**: 2026-02-06  
**预计时间**: 5 分钟

---

## 1. 前置条件

### 1.1 开发环境要求

| 要求 | 说明 |
|------|------|
| OpenHarmony SDK | 3.1+ |
| DevEco Studio | 4.0+ |
| Node.js | 14.0+ (用于打包) |
| Python | 3.8+ (用于 GN 构建) |

### 1.2 系统能力依赖

在 `module.json5` 中声明系统能力：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.QUERY_SECURITY_EVENT"
      },
      {
        "name": "ohos.permission.COLLECT_SECURITY_EVENT"
      }
    ],
    "abilities": [
      {
        "skills": [
          {
            "entities": [
              "entity.system.home"
            ],
            "actions": [
              "action.system.home"
            ]
          }
        ]
      }
    ]
  }
}
```

### 1.3 APL 等级要求

部分敏感 API 需要更高的 APL 等级：

| API | 所需 APL |
|-----|----------|
| getModelResult | system_basic |
| querySecurityEvent | system_basic |
| updatePolicyFile | system_core |

> **说明**: APL (Ability Privilege Level) 在 `module.json5` 的 `apl` 字段中设置。

---

## 2. 安装与集成

### 2.1 导入模块

```javascript
// 方式 1: 命名导入
import securityGuard from '@ohos.security.securityGuard';

// 方式 2: 默认导入
import sg from '@ohos.security.securityGuard';
```

### 2.2 版本检查

```javascript
import securityGuard from '@ohos.security.securityGuard';

// 检查 API 是否可用
if (securityGuard) {
    console.log('SecurityGuard 模块已加载');
} else {
    console.error('SecurityGuard 模块不可用');
}
```

---

## 3. 快速示例

### 3.1 示例 1: 设备安全检查

完整代码：

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 设备安全检查示例
 * 检测设备是否越狱、设备完整性、是否为模拟器
 */
async function deviceSecurityCheck() {
    console.log('=== 设备安全检查 ===');

    // 定义要检查的模型
    const models = [
        'SecurityGuard_JailbreakCheck',   // 越狱检测
        'SecurityGuard_IntegrityCheck',   // 完整性检测
        'SecurityGuard_SimulatorCheck'    // 模拟器检测
    ];

    const results = [];

    for (const modelName of models) {
        try {
            console.log(`检查: ${modelName}...`);
            const result = await securityGuard.getModelResult(modelName);
            console.log(`  结果: ${JSON.stringify(result)}`);
            results.push({
                model: modelName,
                ...result
            });
        } catch (error) {
            console.error(`  错误: [${error.code}] ${error.message}`);
            results.push({
                model: modelName,
                error: error.code
            });
        }
    }

    return results;
}

// 运行示例
deviceSecurityCheck()
    .then(results => {
        console.log('\n=== 检查完成 ===');
        console.log('结果:', JSON.stringify(results, null, 2));
    })
    .catch(error => {
        console.error('检查失败:', error);
    });
```

### 3.2 示例 2: 安全事件查询

完整代码：

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 安全事件查询示例
 * 查询特定类型的历史安全事件
 */
function querySecurityEvents() {
    console.log('=== 查询安全事件 ===');

    // 定义查询条件
    const rulers = [
        {
            eventId: 1011015005,  // 文件事件
            beginTime: '20240101000000',  // 2024-01-01 00:00:00
            endTime: '20241231235959'     // 2024-12-31 23:59:59
        }
    ];

    // 创建查询回调
    const querier = {
        onQuery: (events) => {
            console.log(`收到 ${events.length} 个事件:`);
            events.forEach((event, index) => {
                console.log(`  [${index + 1}] 事件ID: ${event.eventId}`);
                console.log(`      时间: ${event.timestamp}`);
                console.log(`      内容: ${event.content}`);
            });
        },
        onComplete: () => {
            console.log('\n=== 查询完成 ===');
        },
        onError: (error) => {
            console.error('查询失败:', error.code, error.message);
        }
    };

    // 执行查询
    securityGuard.querySecurityEvent(rulers, querier);
}

// 运行示例
querySecurityEvents();
```

### 3.3 示例 3: 安全事件订阅

完整代码：

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 安全事件订阅示例
 * 实时接收系统安全事件通知
 */
function subscribeSecurityEvents() {
    console.log('=== 订阅安全事件 ===');

    // 定义要订阅的事件
    const eventInfo = {
        eventId: 1011015006  // 进程事件
    };

    // 创建订阅回调
    function onEventReceived(event) {
        console.log('\n收到安全事件:');
        console.log(`  事件ID: ${event.eventId}`);
        console.log(`  时间: ${event.timestamp || new Date().toISOString()}`);
        console.log(`  内容: ${event.content || 'N/A'}`);
    }

    // 订阅事件
    securityGuard.on('securityEventOccur', eventInfo, onEventReceived);
    console.log('已订阅安全事件，收到事件时会打印日志');

    // 10秒后取消订阅
    setTimeout(() => {
        console.log('\n取消订阅...');
        securityGuard.off('securityEventOccur', eventInfo, onEventReceived);
        console.log('已取消订阅');
    }, 10000);
}

// 运行示例
subscribeSecurityEvents();
```

### 3.4 示例 4: 安全事件上报

完整代码：

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 安全事件上报示例
 * 向系统上报自定义安全事件
 */
function reportSecurityEvent() {
    console.log('=== 上报安全事件 ===');

    // 构建事件信息
    const eventInfo = {
        eventId: 1011015001,  // 账户事件
        version: '1.0',
        content: JSON.stringify({
            eventType: 'login_attempt',
            userId: 'user123',
            status: 'success',
            ip: '192.168.1.100',
            timestamp: new Date().toISOString()
        })
    };

    // 上报事件
    const success = securityGuard.reportSecurityEvent(eventInfo);

    if (success) {
        console.log('事件上报成功');
    } else {
        console.error('事件上报失败');
    }

    return success;
}

// 运行示例
reportSecurityEvent();
```

---

## 4. 完整应用示例

### 4.1 完整的设备安全监控应用

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * SecurityGuard 完整示例
 * 集成设备检查、事件查询和事件订阅
 */
class SecurityMonitor {
    constructor() {
        this.isMonitoring = false;
        this.subscriptionHandle = null;
        this.eventCache = [];
    }

    /**
     * 初始化监控
     */
    async init() {
        console.log('[Monitor] 初始化安全监控...');

        // 检查模块可用性
        if (!securityGuard) {
            throw new Error('SecurityGuard 模块不可用');
        }

        // 执行初始设备检查
        const deviceStatus = await this.checkDeviceSecurity();
        console.log('[Monitor] 设备状态:', JSON.stringify(deviceStatus));

        // 启动事件订阅
        await this.startEventSubscription();

        this.isMonitoring = true;
        console.log('[Monitor] 安全监控已启动');
    }

    /**
     * 检查设备安全状态
     */
    async checkDeviceSecurity() {
        const models = [
            'SecurityGuard_JailbreakCheck',
            'SecurityGuard_IntegrityCheck',
            'SecurityGuard_SimulatorCheck'
        ];

        const results = {};

        for (const model of models) {
            try {
                const result = await securityGuard.getModelResult(model);
                results[model] = {
                    success: true,
                    data: result
                };
            } catch (error) {
                results[model] = {
                    success: false,
                    error: error.code,
                    message: error.message
                };
            }
        }

        return results;
    }

    /**
     * 启动安全事件订阅
     */
    async startEventSubscription() {
        const eventInfo = {
            eventId: 1011015007  // 网络事件
        };

        const callback = (event) => {
            console.log('[Monitor] 收到安全事件:', event.eventId);
            this.eventCache.push({
                event,
                timestamp: new Date()
            });

            // 保留最近100个事件
            if (this.eventCache.length > 100) {
                this.eventCache.shift();
            }
        };

        securityGuard.on('securityEventOccur', eventInfo, callback);
        this.subscriptionHandle = { eventInfo, callback };
        console.log('[Monitor] 已订阅安全事件');
    }

    /**
     * 停止监控
     */
    stop() {
        if (this.subscriptionHandle) {
            const { eventInfo, callback } = this.subscriptionHandle;
            securityGuard.off('securityEventOccur', eventInfo, callback);
            console.log('[Monitor] 已取消事件订阅');
        }
        this.isMonitoring = false;
        console.log('[Monitor] 安全监控已停止');
    }

    /**
     * 获取缓存的事件
     */
    getCachedEvents() {
        return [...this.eventCache];
    }
}

// 使用示例
async function main() {
    const monitor = new SecurityMonitor();

    try {
        await monitor.init();

        // 模拟运行一段时间
        await new Promise(resolve => setTimeout(resolve, 30000));

        // 获取缓存的事件
        const events = monitor.getCachedEvents();
        console.log(`[Main] 收到 ${events.length} 个安全事件`);

    } catch (error) {
        console.error('[Main] 错误:', error);
    } finally {
        monitor.stop();
    }
}

// 运行
main();
```

---

## 5. 常见问题

### Q1: 模块导入失败

**问题**: `import securityGuard from '@ohos.security.securityGuard'` 报错

**解决方案**:
1. 检查 `module.json5` 中是否正确声明了模块依赖
2. 确保应用已获取必要的权限
3. 确认设备系统版本支持 (3.1+)

### Q2: API 调用返回错误 201

**问题**: 收到错误 `{ code: 201, message: "check permission fail" }`

**解决方案**:
```javascript
// 动态申请权限
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

async function requestPermission() {
    const atManager = abilityAccessCtrl.createAtManager();
    const bundleInfo = await bundleManager.getBundleInfoForSelf(
        bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );
    
    const tokenId = bundleInfo.appInfo.accessTokenId;
    
    await atManager.requestPermissionsFromUser(
        tokenId,
        ['ohos.permission.QUERY_SECURITY_EVENT']
    );
}
```

### Q3: Promise 无响应

**问题**: `getModelResult()` 调用后 Promise 从未 resolve

**解决方案**:
1. 检查 SA 服务是否已启动
2. 查看日志确认调用是否到达服务端
3. 设置超时时间：

```javascript
function withTimeout(promise, timeoutMs) {
    return Promise.race([
        promise,
        new Promise((_, reject) =>
            setTimeout(() => reject(new Error('Timeout')), timeoutMs)
        )
    ]);
}

// 使用
try {
    const result = await withTimeout(
        securityGuard.getModelResult('SecurityGuard_JailbreakCheck'),
        10000  // 10秒超时
    );
} catch (error) {
    console.error('调用超时:', error);
}
```

### Q4: 事件订阅收不到回调

**问题**: `on()` 注册后收不到事件

**解决方案**:
1. 确认订阅的事件类型正确
2. 检查回调函数是否正确定义
3. 确认是否有事件触发

```javascript
// 正确的事件订阅
securityGuard.on('securityEventOccur', { eventId: 1011015005 }, (event) => {
    console.log('收到事件:', event);
});

// 取消订阅
securityGuard.off('securityEventOccur', { eventId: 1011015005 });
```

---

## 6. 下一步

| 主题 | 文档 |
|------|------|
| API 详细参考 | [02_NAPI_Reference.md](../02_NAPI_Reference.md) |
| 架构详解 | [03_Architecture.md](../03_Architecture.md) |
| 调试指南 | [08_Debugging.md](./08_Debugging.md) |
| 代码示例 | [samples/](./samples/) |

---

## 7. 相关文档

- [API 参考](../02_NAPI_Reference.md)
- [架构文档](../03_Architecture.md)
- [构建配置](../04_Build.md)
- [安全评审](../06_Security_Review.md)
