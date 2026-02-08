# 全站导航 (Site Navigation)

本文档提供 certificate_manager Wiki 的完整导航，包含新人学习路线和安全研究路线。

---

## 快速导航

### 📊 文档统计
| 类别 | 数量 |
|------|------|
| **核心文档** | 8 篇 |
| **N-API 接口** | 32 个 |
| **C API 函数** | 36 个 |
| **IPC 方法** | 27 个 |
| **存储类型** | 5 种 |

---

## 🎯 推荐阅读路线

### 路线 A：新人学习路线（Newcomer Track）

**目标**：在 30 分钟内快速掌握项目基础

| 步骤 | 文档 | 预计时间 | 关键收获 |
|------|------|----------|----------|
| 1️⃣ | [01_Overview.md](01_Overview.md) | 5 分钟 | 理解项目定位、能力边界、依赖关系 |
| 2️⃣ | [02_Architecture.md](02_Architecture.md) | 10 分钟 | 掌握三层架构、证书生命周期、数据流 |
| 3️⃣ | [03_CodeMap.md](03_CodeMap.md) | 5 分钟 | 定位核心代码位置、了解目录职责 |
| 4️⃣ | [04_Interface.md](04_Interface.md) | 按需查阅 | 学习 API 使用方式、参数和返回值 |
| 🔧 | [07_Build.md](07_Build.md) | 按需查阅 | 理解构建系统、产物和依赖 |

**💡 新人提示**：
- 先读概览了解全貌，再深入感兴趣的章节
- 接口文档可以按需查阅，不必一次性读完
- 遇到问题先查阅 [常见问题](#faq-section)

---

### 路线 B：安全研究路线（Security Research Track）

**目标**：在 45 分钟内完成攻击面分析和安全评估

| 步骤 | 文档 | 预计时间 | 关键收获 |
|------|------|----------|----------|
| 1️⃣ | [01_Overview.md](01_Overview.md) | 5 分钟 | 理解信任边界、敏感操作、运行环境 |
| 2️⃣ | [05_AttackSurface.md](05_AttackSurface.md) | 15 分钟 | 识别所有外部输入入口、信任边界、敏感操作清单 |
| 3️⃣ | [06_SecurityReview.md](06_SecurityReview.md) | 20 分钟 | 分析 5 类安全风险、查看具体代码证据 |
| 4️⃣ | [08_Internals.md](08_Internals.md) | 按需查阅 | 深入理解权限控制、输入验证实现 |
| 5️⃣ | [04_Interface.md](04_Interface.md) | 5 分钟 | 检查输入验证机制和错误处理 |

**💡 安全研究员提示**：
- 攻击面分析提供完整的入口点映射
- 安全评估包含具体的代码位置和行号
- 建议按顺序阅读：概览 → 攻击面 → 安全评估
- 查阅 [代码证据汇总](_work/NOTES.md) 获取更多细节

---

## 📚 完整文档列表

### 核心文档（P0 - 必读）

| 文档 | 优先级 | 受众 | 阅读时间 | 核心内容 |
|------|--------|------|----------|----------|
| [01_Overview.md](01_Overview.md) | P0 | 所有人 | 5 分钟 | 项目定位、能力边界、快速开始 |
| [02_Architecture.md](02_Architecture.md) | P0 | 所有人 | 15 分钟 | 三层架构、数据流、时序图 |
| [03_CodeMap.md](03_CodeMap.md) | P0 | 所有人 | 10 分钟 | 目录结构、代码地图、导航图 |
| [04_Interface.md](04_Interface.md) | P0 | 所有人 | 按需 | N-API/C API/IPC 完整清单 |
| [05_AttackSurface.md](05_AttackSurface.md) | P0 | 安全研究员 | 15 分钟 | 输入清单、攻击面、信任边界 |
| [06_SecurityReview.md](06_SecurityReview.md) | P0 | 安全研究员 | 30 分钟 | 5 类风险分析、代码证据 |

### 进阶文档（P1 - 按需）

| 文档 | 优先级 | 受众 | 阅读时间 | 核心内容 |
|------|--------|------|----------|----------|
| [07_Build.md](07_Build.md) | P1 | 开发者 | 10 分钟 | GN targets、编译产物、依赖组件 |
| [08_Internals.md](08_Internals.md) | P1 | 深入用户 | 20 分钟 | 核心类、资源生命周期、HUKS 集成 |

---

## 🔍 按主题查找

### 功能导向导航

#### 证书安装
- [01_Overview.md](01_Overview.md) → 证书生命周期管理
- [04_Interface.md](04_Interface.md) → installPublicCertificate, installPrivateCertificate
- [06_SecurityReview.md](06_SecurityReview.md) → 输入验证缺陷

#### 证书查询
- [04_Interface.md](04_Interface.md) → getAllPublicCertificates, getPublicCertificate
- [02_Architecture.md](02_Architecture.md) → 查询数据流

#### 授权管理
- [04_Interface.md](04_Interface.md) → grantPublicCertificate, getAuthorizedAppList
- [06_SecurityReview.md](06_SecurityReview.md) → 权限与鉴权风险

#### 密码学操作
- [04_Interface.md](04_Interface.md) → init, update, finish, abort
- [08_Internals.md](08_Internals.md) → Session 管理

#### 对话框 UI
- [04_Interface.md](04_Interface.md) → security.certManagerDialog 模块

#### 系统集成
- [02_Architecture.md](02_Architecture.md) → HUKS 集成、事件监听
- [07_Build.md](07_Build.md) → SystemAbility 注册

---

## 📝 工作文档

| 文档 | 用途 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目自我评估结果、文档策略决策 |
| [_work/NOTES.md](_work/NOTES.md) | 代码证据汇总、关键发现、待确认项 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度追踪、完成状态 |

---

## <a name="faq-section"></a> ❓ 常见问题（FAQ）

### Q1: 证书管理支持哪些证书类型？
**A**:
- **CA 证书**：只含公钥，用于验签或验证对端身份
- **业务证书**：含公钥和私钥，用于业务场景的签名和验签
- **用户信任证书**：用户安装的 CA 证书
- **系统信任证书**：系统预安装的 CA 证书

### Q2: 证书存储在哪里？
**A**:
- 系统预安装证书：`/etc/security/certificates`
- 应用证书：`/data/service/el1/public/cert_manager_service/certificates`
- 用户信任证书：`/data/service/el1/public/cert_manager_service/certificates/user_open/`
- 私钥存储：HUKS 模块（硬件通用密钥库）

### Q3: 如何集成证书管理功能？
**A**:
```typescript
import { certificateManager } from '@kit.DeviceCertificateKit';

// 安装证书
await certificateManager.installPrivateCertificate(
    keystore,
    keystorePwd,
    alias
);

// 获取证书列表
const certList = await certificateManager.getPrivateCertificate();
```

详见 [04_Interface.md](04_Interface.md)

### Q4: 需要什么权限？
**A**:
```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.ACCESS_CERT_MANAGER"
      }
    ]
  }
}
```

详见 [01_Overview.md](01_Overview.md#权限要求)

### Q5: 如何进行安全测试？
**A**:
- 查看 [05_AttackSurface.md](05_AttackSurface.md) 了解所有攻击面
- 阅读 [06_SecurityReview.md](06_SecurityReview.md) 分析潜在漏洞
- 参考 [test/fuzz_test/](../../test/fuzz_test/) 了解模糊测试覆盖

---

## 📞 获取帮助

### 文档问题
如果发现文档错误、遗漏或不一致处，请：
1. 检查 [_work/NOTES.md](_work/NOTES.md) 获取最新的代码证据
2. 确认代码路径和行号是否正确
3. 联系维护者更新文档

### API 使用问题
- 查阅官方文档：http://docs.openharmony.cn/
- 参考 Gitee 仓库：https://gitee.com/openharmony/security_certificate_manager
- 华为开发者文档：https://developer.huawei.com/consumer/en/doc/harmonyos-guides/

---

*最后更新：2026-02-07*
