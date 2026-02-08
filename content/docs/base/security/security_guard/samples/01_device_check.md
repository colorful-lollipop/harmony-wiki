# 示例：设备安全检查

**场景**：检测设备是否越狱、设备完整性、是否为模拟器

**适用**：需要了解设备安全状态的应用

---

## 完整代码

```javascript
import securityGuard from '@ohos.security.securityGuard';

/**
 * 设备安全检查工具类
 */
class DeviceSecurityChecker {
    constructor() {
        // 支持的检测模型
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
        console.log('[DeviceChecker] 开始设备安全检查...');

        const results = {
            timestamp: new Date().toISOString(),
            checks: {},
            overall: 'UNKNOWN',
            recommendations: []
        };

        // 执行各项检查
        for (const [name, modelId] of Object.entries(this.SUPPORTED_MODELS)) {
            try {
                const result = await this.checkSingleModel(modelId);
                results.checks[name] = {
                    modelId,
                    success: true,
                    riskLevel: result.riskLevel,
                    riskType: result.riskType,
                    details: result
                };
            } catch (error) {
                results.checks[name] = {
                    modelId,
                    success: false,
                    error: error.code,
                    message: error.message
                };
            }
        }

        // 计算总体风险等级
        results.overall = this.calculateOverallRisk(results.checks);

        // 生成建议
        results.recommendations = this.generateRecommendations(results.checks);

        return results;
    }

    /**
     * 检查单个安全模型
     * @param {string} modelName - 模型名称
     * @returns {Promise<Object>} 检测结果
     */
    async checkSingleModel(modelName) {
        console.log(`[DeviceChecker] 检查: ${modelName}`);

        const startTime = Date.now();
        const result = await securityGuard.getModelResult(modelName);
        const duration = Date.now() - startTime;

        console.log(`[DeviceChecker] ${modelName}: riskLevel=${result.riskLevel}, 耗时=${duration}ms`);

        return result;
    }

    /**
     * 计算总体风险等级
     * @param {Object} checks - 检查结果
     * @returns {string} 总体风险等级
     */
    calculateOverallRisk(checks) {
        let maxRisk = 0;
        let hasError = false;

        for (const check of Object.values(checks)) {
            if (!check.success) {
                hasError = true;
                continue;
            }
            maxRisk = Math.max(maxRisk, check.riskLevel || 0);
        }

        if (hasError) return 'UNKNOWN';
        if (maxRisk >= 3) return 'HIGH';
        if (maxRisk >= 1) return 'MEDIUM';
        return 'LOW';
    }

    /**
     * 生成安全建议
     * @param {Object} checks - 检查结果
     * @returns {Array<string>} 建议列表
     */
    generateRecommendations(checks) {
        const recommendations = [];

        // 越狱检测建议
        if (!checks.JAILBREAK?.success) {
            recommendations.push('无法执行越狱检测，请检查权限配置');
        } else if (checks.JAILBREAK?.riskLevel >= 3) {
            recommendations.push('⚠️ 检测到越狱风险，建议在安全环境使用');
        }

        // 完整性检测建议
        if (!checks.INTEGRITY?.success) {
            recommendations.push('无法执行完整性检测，请检查权限配置');
        } else if (checks.INTEGRITY?.riskLevel >= 3) {
            recommendations.push('⚠️ 设备完整性可能受损，请谨慎处理敏感数据');
        }

        // 模拟器检测建议
        if (!checks.SIMULATOR?.success) {
            recommendations.push('无法执行模拟器检测，请检查权限配置');
        } else if (checks.SIMULATOR?.riskLevel >= 3) {
            recommendations.push('⚠️ 检测到模拟器环境，部分功能可能受限');
        }

        // 风险因子检测
        if (checks.RISK_FACTOR?.success && checks.RISK_FACTOR?.riskLevel >= 3) {
            recommendations.push('⚠️ 存在多个风险因子，建议进行全面安全评估');
        }

        // WiFi 检测
        if (checks.WIFI_CHECK?.success && checks.WIFI_CHECK?.riskLevel >= 3) {
            recommendations.push('⚠️ WiFi 环境存在安全风险');
        }

        if (recommendations.length === 0) {
            recommendations.push('✅ 设备安全状态良好');
        }

        return recommendations;
    }

    /**
     * 快速风险评估（简化版）
     * @returns {Promise<Object>} 风险评估结果
     */
    async quickCheck() {
        // 只检查最关键的三个模型
        const results = await Promise.all([
            this.checkSingleModel(this.SUPPORTED_MODELS.JAILBREAK),
            this.checkSingleModel(this.SUPPORTED_MODELS.INTEGRITY),
            this.checkSingleModel(this.SUPPORTED_MODELS.SIMULATOR)
        ]);

        // 计算最大风险等级
        const maxRisk = Math.max(
            results[0]?.riskLevel || 0,
            results[1]?.riskLevel || 0,
            results[2]?.riskLevel || 0
        );

        return {
            timestamp: new Date().toISOString(),
            maxRiskLevel: maxRisk,
            status: maxRisk >= 3 ? 'UNSAFE' : maxRisk >= 1 ? 'WARNING' : 'SAFE',
            results
        };
    }
}

// ==================== 使用示例 ====================

async function main() {
    const checker = new DeviceSecurityChecker();

    // 方法1: 完整检查
    console.log('=== 完整安全检查 ===');
    const fullResult = await checker.checkAll();
    console.log('检查结果:', JSON.stringify(fullResult, null, 2));

    // 方法2: 快速检查
    console.log('\n=== 快速风险评估 ===');
    const quickResult = await checker.quickCheck();
    console.log('快速结果:', JSON.stringify(quickResult, null, 2));

    // 方法3: 单项检查
    console.log('\n=== 单项检查 ===');
    const singleResult = await checker.checkSingleModel(
        checker.SUPPORTED_MODELS.JAILBREAK
    );
    console.log('越狱检测结果:', JSON.stringify(singleResult, null, 2));
}

export { DeviceSecurityChecker };
export default DeviceSecurityChecker;
```

---

## 运行结果示例

### 完整检查结果

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "checks": {
    "JAILBREAK": {
      "modelId": "SecurityGuard_JailbreakCheck",
      "success": true,
      "riskLevel": 0,
      "riskType": "safe",
      "details": {
        "riskLevel": 0,
        "riskType": "safe",
        "confidence": 0.95
      }
    },
    "INTEGRITY": {
      "modelId": "SecurityGuard_IntegrityCheck",
      "success": true,
      "riskLevel": 0,
      "riskType": "safe",
      "details": {
        "riskLevel": 0,
        "riskType": "safe",
        "confidence": 0.98
      }
    },
    "SIMULATOR": {
      "modelId": "SecurityGuard_SimulatorCheck",
      "success": true,
      "riskLevel": 0,
      "riskType": "safe",
      "details": {
        "riskLevel": 0,
        "riskType": "safe",
        "confidence": 0.99
      }
    }
  },
  "overall": "LOW",
  "recommendations": [
    "✅ 设备安全状态良好"
  ]
}
```

### 风险等级说明

| 等级 | 值 | 含义 | 建议 |
|------|-----|------|------|
| SAFE | 0 | 安全 | 正常操作 |
| LOW | 1 | 低风险 | 正常使用 |
| MEDIUM | 2-3 | 中风险 | 谨慎操作 |
| HIGH | 4-5 | 高风险 | 停止敏感操作 |

---

## 常见问题

### Q1: 检查返回 UNKNOWN

**原因**: 权限未配置或服务未启动

**解决方案**:
```javascript
// 检查权限
if (!hasPermission('QUERY_SECURITY_EVENT')) {
    await requestPermission('QUERY_SECURITY_EVENT');
}

// 检查服务
try {
    await checker.checkSingleModel('SecurityGuard_JailbreakCheck');
} catch (error) {
    if (error.code === 21200001) {
        console.error('SA 服务未启动，请等待服务注册');
    }
}
```

### Q2: 耗时过长

**原因**: 网络模型首次加载

**解决方案**:
```javascript
// 预加载模型
async function preloadModels() {
    const models = [
        'SecurityGuard_JailbreakCheck',
        'SecurityGuard_IntegrityCheck',
        'SecurityGuard_SimulatorCheck'
    ];
    
    for (const model of models) {
        await securityGuard.getModelResult(model);
    }
}
```

---

## 相关文档

- [API 参考](../../02_NAPI_Reference.md)
- [快速开始](../../07_QuickStart.md)
- [事件监控示例](./02_event_monitoring.md)
