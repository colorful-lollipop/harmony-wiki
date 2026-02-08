# 示例：安全事件监控

**场景**：实时订阅和监控系统的安全事件

**适用**：安全审计应用、实时监控面板、合规检测工具

---

## 完整代码

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 安全事件监控器
 * 提供实时安全事件订阅、历史查询和统计分析功能
 */
class SecurityEventMonitor {
    constructor() {
        // 缓存配置
        this.maxCachedEvents = 1000;
        this.eventCache = [];

        // 事件统计
        this.stats = {
            totalReceived: 0,
            lastEventTime: null,
            eventsByType: {},
            eventsBySource: {}
        };

        // 订阅状态
        this.isSubscribed = false;
        this.subscriptionInfo = null;
        this.subscriptionCallback = null;
    }

    /**
     * 订阅安全事件
     * @param {Object} eventInfo - 订阅配置
     * @param {Function} callback - 事件回调
     * @returns {Promise<boolean>} 订阅是否成功
     */
    async subscribe(eventInfo, callback) {
        if (this.isSubscribed) {
            console.warn('[Monitor] 已有订阅，请先取消');
            return false;
        }

        console.log('[Monitor] 开始订阅安全事件:', eventInfo);

        // 设置回调
        this.subscriptionCallback = (event) => {
            // 缓存事件
            this.cacheEvent(event);

            // 更新统计
            this.updateStats(event);

            // 调用用户回调
            if (callback) {
                callback(event);
            }
        };

        // 执行订阅
        try {
            securityGuard.on('securityEventOccur', eventInfo, this.subscriptionCallback);

            this.isSubscribed = true;
            this.subscriptionInfo = eventInfo;

            console.log('[Monitor] 订阅成功');
            return true;
        } catch (error) {
            console.error('[Monitor] 订阅失败:', error);
            this.subscriptionCallback = null;
            return false;
        }
    }

    /**
     * 取消订阅
     * @returns {boolean} 取消是否成功
     */
    unsubscribe() {
        if (!this.isSubscribed || !this.subscriptionInfo) {
            console.warn('[Monitor] 没有活跃的订阅');
            return false;
        }

        try {
            securityGuard.off('securityEventOccur', this.subscriptionInfo, this.subscriptionCallback);

            this.isSubscribed = false;
            this.subscriptionInfo = null;
            this.subscriptionCallback = null;

            console.log('[Monitor] 已取消订阅');
            return true;
        } catch (error) {
            console.error('[Monitor] 取消订阅失败:', error);
            return false;
        }
    }

    /**
     * 查询历史事件
     * @param {Object} query - 查询条件
     * @returns {Promise<Array>} 查询结果
     */
    async queryHistory(query = {}) {
        const rulers = [];

        // 构建查询条件
        if (query.eventId) {
            rulers.push({
                eventId: query.eventId
            });
        }

        if (query.beginTime) {
            rulers.push({
                eventId: query.eventId || 0,
                beginTime: query.beginTime
            });
        }

        if (query.endTime) {
            rulers.push({
                eventId: query.eventId || 0,
                endTime: query.endTime
            });
        }

        // 如果没有指定条件，查询所有事件
        if (rulers.length === 0) {
            rulers.push({ eventId: 0 });
        }

        console.log('[Monitor] 查询历史事件:', rulers);

        return new Promise((resolve, reject) => {
            const querier = {
                onQuery: (events) => {
                    console.log(`[Monitor] 查询到 ${events.length} 个事件`);
                    resolve(events);
                },
                onComplete: () => {
                    console.log('[Monitor] 查询完成');
                },
                onError: (error) => {
                    console.error('[Monitor] 查询失败:', error);
                    reject(error);
                }
            };

            securityGuard.querySecurityEvent(rulers, querier);
        });
    }

    /**
     * 缓存事件
     * @param {Object} event - 安全事件
     */
    cacheEvent(event) {
        this.eventCache.unshift({
            ...event,
            receivedAt: new Date().toISOString()
        });

        // 限制缓存大小
        if (this.eventCache.length > this.maxCachedEvents) {
            this.eventCache.pop();
        }
    }

    /**
     * 更新事件统计
     * @param {Object} event - 安全事件
     */
    updateStats(event) {
        this.stats.totalReceived++;
        this.stats.lastEventTime = new Date().toISOString();

        // 按事件类型统计
        const eventType = String(event.eventId);
        this.stats.eventsByType[eventType] = (this.stats.eventsByType[eventType] || 0) + 1;

        // 按来源统计
        const source = event.source || 'unknown';
        this.stats.eventsBySource[source] = (this.stats.eventsBySource[source] || 0) + 1;
    }

    /**
     * 获取缓存的事件
     * @param {number} limit - 最大返回数量
     * @returns {Array} 缓存的事件列表
     */
    getCachedEvents(limit = 100) {
        return this.eventCache.slice(0, limit);
    }

    /**
     * 获取统计信息
     * @returns {Object} 统计信息
     */
    getStats() {
        return {
            ...this.stats,
            cacheSize: this.eventCache.length,
            isSubscribed: this.isSubscribed
        };
    }

    /**
     * 导出事件日志
     * @param {string} format - 导出格式 ('json' | 'csv')
     * @returns {string} 导出内容
     */
    exportLog(format = 'json') {
        if (format === 'json') {
            return JSON.stringify({
                exportTime: new Date().toISOString(),
                stats: this.stats,
                events: this.eventCache
            }, null, 2);
        }

        if (format === 'csv') {
            const headers = ['eventId', 'version', 'content', 'timestamp', 'receivedAt'];
            const rows = this.eventCache.map(e =>
                headers.map(h => e[h] || '').join(',')
            );
            return [headers.join(','), ...rows].join('\n');
        }

        return '';
    }

    /**
     * 清除缓存和统计
     */
    clear() {
        this.eventCache = [];
        this.stats = {
            totalReceived: 0,
            lastEventTime: null,
            eventsByType: {},
            eventsBySource: {}
        };
        console.log('[Monitor] 已清除缓存和统计');
    }
}

// ==================== 使用示例 ====================

async function main() {
    const monitor = new SecurityEventMonitor();

    // 1. 订阅实时事件
    console.log('=== 实时事件订阅 ===');
    const eventInfo = {
        eventId: 1011015006  // 进程事件
    };

    await monitor.subscribe(eventInfo, (event) => {
        console.log(`[实时] 收到事件: ID=${event.eventId}, 内容=${event.content?.substring(0, 50)}...`);
    });

    // 2. 查询历史事件
    console.log('\n=== 查询历史事件 ===');
    const history = await monitor.queryHistory({
        beginTime: '20240101000000',
        eventId: 1011015006
    });
    console.log(`历史事件数量: ${history.length}`);

    // 3. 查看统计
    console.log('\n=== 统计信息 ===');
    const stats = monitor.getStats();
    console.log('统计:', JSON.stringify(stats, null, 2));

    // 4. 导出日志
    console.log('\n=== 导出日志 ===');
    const jsonLog = monitor.exportLog('json');
    console.log('日志长度:', jsonLog.length, '字符');

    // 5. 取消订阅
    console.log('\n=== 取消订阅 ===');
    monitor.unsubscribe();

    // 6. 清理
    monitor.clear();
}

export { SecurityEventMonitor };
export default SecurityEventMonitor;
```

---

## 常用事件类型

### 系统事件 ID

| 事件 ID | 名称 | 说明 |
|---------|------|------|
| 1011015000 | PASTEBOARD | 剪贴板事件 |
| 1011015001 | ACCOUNT | 账户事件 |
| 1011015002 | WINDOW | 窗口事件 |
| 1011015003 | VOLUMN | 音量事件 |
| 1011015004 | PRINTER | 打印机事件 |
| 1011015005 | FILE | 文件事件 |
| 1011015006 | PROCESS | 进程事件 |
| 1011015007 | NETWORK | 网络事件 |
| 1011015008 | FILE_GUARD | 文件守护事件 |
| 1011015009 | CAMERA | 相机事件 |
| 1011015010 | APPLICATION | 应用事件 |
| 1011015011 | MOUSE | 鼠标事件 |
| 1011015012 | KEYBOARD | 键盘事件 |

---

## 事件数据结构

### 事件对象字段

| 字段 | 类型 | 说明 |
|------|------|------|
| eventId | number | 事件唯一标识 |
| version | string | 事件版本 |
| content | string | 事件内容 (JSON 字符串) |
| timestamp | string | 事件时间戳 |
| source | number | 数据来源 (0=用户层, 1=内核层, 2=模型层, 3=HiView) |
| userId | number | 用户 ID |

### 事件内容示例

```json
{
  "eventId": 1011015005,
  "version": "1.0",
  "content": "{\"filePath\": \"/data/app/files/test.txt\", \"operation\": \"read\", \"result\": \"success\"}",
  "timestamp": "20240115103000",
  "source": 0,
  "userId": 100
}
```

---

## 高级用法

### 事件聚合分析

```javascript
class EventAnalyzer {
    constructor(monitor) {
        this.monitor = monitor;
    }

    /**
     * 分析事件频率
     * @param {string} eventType - 事件类型
     * @param {number} windowMs - 时间窗口 (毫秒)
     * @returns {number} 频率 (事件数/秒)
     */
    calculateFrequency(eventType, windowMs = 60000) {
        const now = Date.now();
        const events = this.monitor.getCachedEvents(10000);

        const recentEvents = events.filter(e => {
            const eventTime = new Date(e.receivedAt).getTime();
            return e.eventId === eventType && (now - eventTime) < windowMs;
        });

        return recentEvents.length / (windowMs / 1000);
    }

    /**
     * 检测异常事件模式
     * @returns {Array} 检测到的异常
     */
    detectAnomalies() {
        const anomalies = [];
        const events = this.monitor.getCachedEvents();
        const stats = this.monitor.getStats();

        // 检测高频事件
        const frequencyThreshold = 10; // 每秒10个
        for (const [eventId, count] of Object.entries(stats.eventsByType)) {
            const frequency = this.calculateFrequency(parseInt(eventId), 1000);
            if (frequency > frequencyThreshold) {
                anomalies.push({
                    type: 'HIGH_FREQUENCY',
                    eventId,
                    frequency,
                    severity: 'WARNING',
                    message: `事件 ${eventId} 频率异常: ${frequency.toFixed(2)}/s`
                });
            }
        }

        return anomalies;
    }

    /**
     * 生成安全报告
     * @returns {Object} 安全报告
     */
    generateReport() {
        const stats = this.monitor.getStats();
        const anomalies = this.detectAnomalies();

        return {
            reportTime: new Date().toISOString(),
            summary: {
                totalEvents: stats.totalReceived,
                eventTypes: Object.keys(stats.eventsByType).length,
                uniqueSources: Object.keys(stats.eventsBySource).length,
                lastEventTime: stats.lastEventTime
            },
            topEventTypes: Object.entries(stats.eventsByType)
                .sort((a, b) => b[1] - a[1])
                .slice(0, 5)
                .map(([id, count]) => ({ eventId: id, count })),
            anomalies,
            riskLevel: anomalies.length > 5 ? 'HIGH' : anomalies.length > 0 ? 'MEDIUM' : 'LOW'
        };
    }
}

// 使用
const monitor = new SecurityEventMonitor();
const analyzer = new EventAnalyzer(monitor);

// 订阅事件
await monitor.subscribe({ eventId: 1011015007 }); // 网络事件

// 生成报告
const report = analyzer.generateReport();
console.log('安全报告:', JSON.stringify(report, null, 2));
```

---

## 常见问题

### Q1: 收不到事件

**排查步骤**:
```javascript
// 1. 检查订阅状态
console.log('订阅状态:', monitor.isSubscribed);

// 2. 检查缓存
console.log('缓存事件:', monitor.getCachedEvents());

// 3. 检查统计
console.log('统计:', monitor.getStats());

// 4. 查看日志
// hilog | grep "SecurityGuard|SecurityCollector"
```

### Q2: 事件内容解析失败

**原因**: content 字段是 JSON 字符串

**解决方案**:
```javascript
function parseEventContent(event) {
    try {
        return {
            ...event,
            parsedContent: JSON.parse(event.content || '{}')
        };
    } catch (e) {
        return {
            ...event,
            parsedContent: null,
            parseError: e.message
        };
    }
}

// 使用
const events = monitor.getCachedEvents().map(parseEventContent);
```

---

## 相关文档

- [API 参考](../../02_NAPI_Reference.md)
- [快速开始](../../07_QuickStart.md)
- [设备安全检查示例](./01_device_check.md)
- [调试指南](../../08_Debugging.md)
