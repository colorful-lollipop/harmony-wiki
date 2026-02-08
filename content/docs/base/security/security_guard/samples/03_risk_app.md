# 示例：风控应用集成

**场景**：将设备安全检测集成到风控决策系统中

**适用**：金融应用、支付系统、敏感数据访问控制

---

## 完整代码

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 设备安全检查工具类
 * 从 01_device_check.md 复制而来
 */
class DeviceSecurityChecker {
    constructor() {
        this.SUPPORTED_MODELS = {
            JAILBREAK: 'SecurityGuard_JailbreakCheck',
            INTEGRITY: 'SecurityGuard_IntegrityCheck',
            SIMULATOR: 'SecurityGuard_SimulatorCheck',
            RISK_FACTOR: 'SecurityGuard_RiskFactorCheck',
            WIFI_CHECK: 'SecurityGuard_WifiCheck'
        };
    }

    /**
     * 执行完整设备安全检查
     * @returns {Promise<Object>} 安全检查结果
     */
    async checkAll() {
        const results = {
            timestamp: new Date().toISOString(),
            checks: {},
            overall: 'UNKNOWN',
            recommendations: []
        };

        for (const [name, modelId] of Object.entries(this.SUPPORTED_MODELS)) {
            try {
                const result = await this.checkSingleModel(modelId);
                results.checks[name] = {
                    modelId,
                    success: true,
                    riskLevel: result.riskLevel,
                    riskType: result.riskType,
                    confidence: result.confidence
                };
            } catch (error) {
                results.checks[name] = {
                    modelId,
                    success: false,
                    error: error.message
                };
                results.recommendations.push(`[${name}] 检查失败: ${error.message}`);
            }
        }

        results.overall = this.calculateOverallRisk(results.checks);
        return results;
    }

    /**
     * 检查单个模型
     * @param {string} modelId - 模型标识符
     * @returns {Promise<Object>} 检查结果
     */
    async checkSingleModel(modelId) {
        const result = await securityGuard.getModelResult({ modelName: modelId });
        return result;
    }

    /**
     * 计算整体风险等级
     * @param {Object} checks - 检查结果
     * @returns {string} 整体风险等级
     */
    calculateOverallRisk(checks) {
        const riskLevels = Object.values(checks)
            .filter(c => c.success)
            .map(c => c.riskLevel);

        if (riskLevels.some(level => level === 4)) return 'CRITICAL';
        if (riskLevels.some(level => level === 3)) return 'HIGH';
        if (riskLevels.some(level => level === 2)) return 'MEDIUM';
        return 'LOW';
    }
}

/**
 * 风控决策引擎
 * 基于设备安全状态进行风险评估和决策
 */
class RiskDecisionEngine {
    constructor(config = {}) {
        // 决策配置
        this.config = {
            // 是否在风险设备上强制要求二次验证
            requireSecondaryOnRisk: config.requireSecondaryOnRisk ?? true,
            // 是否允许在风险设备上进行敏感操作
            allowSensitiveOnRisk: config.allowSensitiveOnRisk ?? false,
            // 风险阈值
            riskThreshold: config.riskThreshold ?? 2,
            // 缓存时间 (毫秒)
            cacheDuration: config.cacheDuration ?? 5 * 60 * 1000, // 5分钟
            // 二次验证超时 (毫秒)
            secondaryTimeout: config.secondaryTimeout ?? 30 * 1000 // 30秒
        };

        // 决策缓存
        this.decisionCache = new Map();

        // 设备检查器
        this.deviceChecker = new DeviceSecurityChecker();

        // 决策历史
        this.decisionHistory = [];
    }

    /**
     * 评估操作风险
     * @param {string} operationId - 操作标识
     * @param {Object} context - 操作上下文
     * @returns {Promise<Object>} 风险评估结果
     */
    async evaluateRisk(operationId, context = {}) {
        const cacheKey = `${operationId}_${JSON.stringify(context)}`;

        // 检查缓存
        if (this.decisionCache.has(cacheKey)) {
            const cached = this.decisionCache.get(cacheKey);
            if (Date.now() - cached.timestamp < this.config.cacheDuration) {
                console.log(`[RiskEngine] 命中缓存: ${operationId}`);
                return cached.decision;
            }
        }

        // 获取设备安全状态
        const deviceStatus = await this.deviceChecker.quickCheck();

        // 构建风险特征
        const features = this.extractFeatures(operationId, context, deviceStatus);

        // 计算风险分数
        const riskScore = this.calculateRiskScore(features);

        // 生成决策
        const decision = this.makeDecision(operationId, riskScore, features);

        // 添加决策到历史
        this.addToHistory(decision);

        // 缓存决策结果
        this.decisionCache.set(cacheKey, {
            timestamp: Date.now(),
            decision
        });

        return decision;
    }

    /**
     * 提取风险特征
     * @param {string} operationId - 操作标识
     * @param {Object} context - 操作上下文
     * @param {Object} deviceStatus - 设备状态
     * @returns {Object} 特征向量
     */
    extractFeatures(operationId, context, deviceStatus) {
        return {
            // 操作特征
            operationId,
            operationType: context.operationType || 'UNKNOWN',
            operationAmount: context.amount || 0,

            // 设备特征
            deviceRiskLevel: deviceStatus.maxRiskLevel,
            deviceStatus: deviceStatus.status,

            // 环境特征
            networkType: context.networkType || 'UNKNOWN',
            isTrustedNetwork: this.isTrustedNetwork(context.networkType),
            hourOfDay: new Date().getHours(),

            // 用户特征
            userId: context.userId,
            isNewUser: context.isNewUser || false,
            hasVerified: context.hasVerified || false
        };
    }

    /**
     * 计算风险分数
     * @param {Object} features - 特征向量
     * @returns {number} 风险分数 (0-10)
     */
    calculateRiskScore(features) {
        let score = 0;

        // 设备风险 (0-4 分)
        score += features.deviceRiskLevel;

        // 操作类型风险 (0-2 分)
        const operationRiskMap = {
            'LOGIN': 1,
            'PAYMENT': 2,
            'TRANSFER': 2,
            'PASSWORD_CHANGE': 2,
            'DATA_ACCESS': 1,
            'VIEW': 0
        };
        score += operationRiskMap[features.operationType] || 0;

        // 金额风险 (0-2 分)
        if (features.operationAmount > 10000) score += 2;
        else if (features.operationAmount > 1000) score += 1;

        // 环境风险 (0-1 分)
        if (!features.isTrustedNetwork && features.networkType !== 'UNKNOWN') {
            score += 1;
        }

        // 时间风险 (0-1 分)
        const hour = features.hourOfDay;
        if (hour < 6 || hour > 23) score += 1;

        // 用户风险 (0-2 分)
        if (features.isNewUser) score += 1;
        if (!features.hasVerified) score += 1;

        return Math.min(score, 10); // 最高10分
    }

    /**
     * 生成决策
     * @param {string} operationId - 操作标识
     * @param {number} riskScore - 风险分数
     * @param {Object} features - 特征向量
     * @returns {Object} 决策结果
     */
    makeDecision(operationId, riskScore, features) {
        const decision = {
            operationId,
            riskScore,
            level: this.getRiskLevel(riskScore),
            timestamp: new Date().toISOString(),
            allowed: true,
            requireSecondary: false,
            reason: [],
            actions: []
        };

        // 风险等级判定
        if (riskScore >= 8) {
            // 高风险
            decision.level = 'CRITICAL';
            decision.allowed = false;
            decision.reason.push('风险分数过高，操作被拒绝');
            decision.actions.push('建议联系客服');
        } else if (riskScore >= this.config.riskThreshold) {
            // 中风险
            decision.level = 'HIGH';
            if (this.config.requireSecondaryOnRisk) {
                decision.requireSecondary = true;
                decision.allowed = true;
                decision.reason.push('检测到风险，需要二次验证');
                decision.actions.push('请进行生物识别验证');
            } else {
                decision.allowed = this.config.allowSensitiveOnRisk;
                decision.reason.push('检测到风险，操作可能受限');
                decision.actions.push('建议在安全环境下操作');
            }
        } else if (riskScore >= 4) {
            // 中等风险
            decision.level = 'MEDIUM';
            decision.allowed = true;
            decision.reason.push('存在一定风险');
            decision.actions.push('建议确认操作环境安全');
        } else {
            // 低风险
            decision.level = 'LOW';
            decision.allowed = true;
            decision.reason.push('设备安全状态良好');
        }

        // 设备状态检查
        if (features.deviceStatus === 'UNSAFE') {
            decision.reason.push('设备安全状态异常');
            if (decision.allowed) {
                decision.actions.push('请在安全的设备上操作');
            }
        }

        return decision;
    }

    /**
     * 执行二次验证
     * @param {string} operationId - 操作标识
     * @param {string} verifyType - 验证类型
     * @returns {Promise<boolean>} 验证是否成功
     */
    async performSecondaryVerification(operationId, verifyType = 'BIOMETRIC') {
        console.log(`[RiskEngine] 执行二次验证: ${operationId}, 类型=${verifyType}`);

        // 模拟验证过程
        return new Promise((resolve) => {
            // 实际应用中，这里会调用生物识别或其他验证 API
            setTimeout(() => {
                const success = Math.random() > 0.1; // 90% 成功率模拟
                console.log(`[RiskEngine] 二次验证${success ? '成功' : '失败'}`);
                resolve(success);
            }, this.config.secondaryTimeout);
        });
    }

    /**
     * 完整风控流程
     * @param {string} operationId - 操作标识
     * @param {Object} context - 操作上下文
     * @returns {Promise<Object>} 风控结果
     */
    async executeRiskControl(operationId, context) {
        const result = {
            operationId,
            timestamp: new Date().toISOString(),
            steps: []
        };

        // 步骤1: 风险评估
        console.log('[RiskControl] 步骤1: 风险评估');
        const riskDecision = await this.evaluateRisk(operationId, context);
        result.steps.push({
            step: 'RISK_EVALUATION',
            decision: riskDecision
        });

        // 步骤2: 如果需要二次验证
        if (riskDecision.requireSecondary) {
            console.log('[RiskControl] 步骤2: 二次验证');
            const verifySuccess = await this.performSecondaryVerification(operationId);
            result.steps.push({
                step: 'SECONDARY_VERIFICATION',
                success: verifySuccess
            });

            if (!verifySuccess) {
                return {
                    ...result,
                    finalDecision: 'BLOCKED',
                    reason: '二次验证失败'
                };
            }
        }

        // 步骤3: 执行操作
        console.log('[RiskControl] 步骤3: 执行操作');
        result.steps.push({
            step: 'OPERATION_EXECUTION',
            allowed: riskDecision.allowed
        });

        // 最终决策
        result.finalDecision = riskDecision.allowed ? 'ALLOWED' : 'BLOCKED';
        result.riskScore = riskDecision.riskScore;
        result.riskLevel = riskDecision.level;
        result.recommendations = riskDecision.actions;

        return result;
    }

    /**
     * 判断是否为可信网络
     * @param {string} networkType - 网络类型
     * @returns {boolean}
     */
    isTrustedNetwork(networkType) {
        const trustedNetworks = ['WIFI_TRUSTED', 'ETHERNET'];
        return trustedNetworks.includes(networkType);
    }

    /**
     * 获取风险等级标签
     * @param {number} score - 风险分数
     * @returns {string}
     */
    getRiskLevel(score) {
        if (score >= 8) return 'CRITICAL';
        if (score >= 5) return 'HIGH';
        if (score >= 3) return 'MEDIUM';
        if (score >= 1) return 'LOW';
        return 'SAFE';
    }

    /**
     * 添加决策到历史
     * @param {Object} decision - 决策结果
     */
    addToHistory(decision) {
        this.decisionHistory.unshift(decision);
        // 保留最近100条
        if (this.decisionHistory.length > 100) {
            this.decisionHistory.pop();
        }
    }

    /**
     * 获取决策历史
     * @param {number} limit - 最大数量
     * @returns {Array}
     */
    getHistory(limit = 10) {
        return this.decisionHistory.slice(0, limit);
    }

    /**
     * 清空历史
     */
    clearHistory() {
        this.decisionHistory = [];
        this.decisionCache.clear();
        console.log('[RiskEngine] 已清空历史记录');
    }
}

// ==================== 使用示例 ====================

async function main() {
    // 创建风控引擎
    const engine = new RiskDecisionEngine({
        riskThreshold: 3,
        requireSecondaryOnRisk: true,
        allowSensitiveOnRisk: false
    });

    // 场景1: 用户登录
    console.log('=== 场景1: 用户登录 ===');
    const loginResult = await engine.executeRiskControl('USER_LOGIN', {
        operationType: 'LOGIN',
        userId: 'user123',
        isNewUser: false,
        networkType: 'WIFI_TRUSTED'
    });
    console.log('登录风控结果:', JSON.stringify(loginResult, null, 2));

    // 场景2: 大额转账
    console.log('\n=== 场景2: 大额转账 ===');
    const transferResult = await engine.executeRiskControl('BIG_TRANSFER', {
        operationType: 'TRANSFER',
        userId: 'user456',
        amount: 50000,
        networkType: 'CELLULAR',
        hourOfDay: new Date().getHours()
    });
    console.log('转账风控结果:', JSON.stringify(transferResult, null, 2));

    // 场景3: 查看敏感数据
    console.log('\n=== 场景3: 查看敏感数据 ===');
    const viewResult = await engine.executeRiskControl('VIEW_SENSITIVE_DATA', {
        operationType: 'DATA_ACCESS',
        userId: 'user789',
        hasVerified: true,
        networkType: 'UNKNOWN'
    });
    console.log('数据访问风控结果:', JSON.stringify(viewResult, null, 2));

    // 查看决策历史
    console.log('\n=== 决策历史 ===');
    const history = engine.getHistory(5);
    console.log('最近决策:', JSON.stringify(history, null, 2));
}

export { RiskDecisionEngine };
export default RiskDecisionEngine;
```

---

## 风险评分模型

### 评分因素

| 因素 | 权重 | 说明 |
|------|------|------|
| 设备风险 | 40% | 设备越狱、root、模拟器等 |
| 操作类型 | 20% | 不同操作有不同的风险权重 |
| 操作金额 | 20% | 金额越大风险越高 |
| 网络环境 | 10% | 可信网络 vs 公共网络 |
| 时间因素 | 5% | 深夜操作风险更高 |
| 用户状态 | 5% | 新用户、已验证等 |

### 风险阈值设置

| 阈值 | 含义 | 建议动作 |
|------|------|----------|
| 0-2 | 低风险 | 正常放行 |
| 3-4 | 中等风险 | 提示用户 |
| 5-7 | 高风险 | 二次验证 |
| 8-10 | 极高风险 | 直接拒绝 |

---

## 集成最佳实践

### 1. 分层风控

```javascript
// 第一层: 设备检查 (快速)
const deviceOk = await checkDevice();
// 失败: 阻止所有操作

// 第二层: 风险评估 (中等)
const risk = await engine.evaluateRisk(operation, context);
// 根据风险级别决定后续流程

// 第三层: 行为分析 (可选, 复杂)
const behavior = analyzeUserBehavior(userId);
// 用于进一步细分风险
```

### 2. 缓存策略

```javascript
class DecisionCache {
    constructor(ttlMs = 300000) { // 5分钟
        this.cache = new Map();
        this.ttlMs = ttlMs;
    }

    get(key) {
        const item = this.cache.get(key);
        if (!item) return null;
        if (Date.now() - item.time > this.ttlMs) {
            this.cache.delete(key);
            return null;
        }
        return item.decision;
    }

    set(key, decision) {
        this.cache.set(key, {
            time: Date.now(),
            decision
        });
    }
}
```

### 3. 降级策略

```javascript
async function riskControlWithFallback(operation, context) {
    try {
        // 正常风控流程
        return await engine.executeRiskControl(operation, context);
    } catch (error) {
        // 风控服务异常时的降级策略
        console.error('[RiskControl] 服务异常，使用降级策略');

        // 降级: 限制高风险操作
        if (isHighRiskOperation(operation)) {
            return {
                finalDecision: 'BLOCKED',
                reason: '风控服务异常，高风险操作被临时禁止'
            };
        }

        // 放行低风险操作，但记录
        return {
            finalDecision: 'ALLOWED',
            reason: '风控服务异常，已放行',
            logged: true
        };
    }
}
```

---

## 常见问题

### Q1: 风控判断不一致

**原因**: 缓存或特征提取不一致

**解决方案**:
```javascript
// 1. 禁用缓存进行调试
const debugEngine = new RiskDecisionEngine({ cacheDuration: 0 });

// 2. 输出完整特征
const features = debugEngine.extractFeatures(opId, ctx, deviceStatus);
console.log('特征:', JSON.stringify(features, null, 2));
```

### Q2: 误拦截过多

**解决方案**: 调整阈值
```javascript
// 放宽阈值
const engine = new RiskDecisionEngine({
    riskThreshold: 4,  // 从3提升到4
    requireSecondaryOnRisk: false,
    allowSensitiveOnRisk: true
});
```

---

## 相关文档

- [API 参考](../../02_NAPI_Reference.md)
- [快速开始](../../07_QuickStart.md)
- [设备安全检查示例](./01_device_check.md)
- [事件监控示例](./02_event_monitoring.md)
- [调试指南](../../08_Debugging.md)
