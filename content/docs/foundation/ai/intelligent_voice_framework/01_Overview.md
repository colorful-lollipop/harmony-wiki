# 项目概览

> **目的**: 帮助开发者快速理解 Intelligent Voice Framework 的定位、边界和核心能力  
> **适用范围**: 所有参与智能语音框架开发、维护、调试的工程师  
> **最后更新**: 2026-02-06

---

## 1. 项目定位

### 1.1 在 OpenHarmony 中的位置

```
OpenHarmony 系统
├── 应用层
├── 框架层
│   └── AI 子系统
│       └── Intelligent Voice Framework  ← 本文档対象
├── 系统服务层
└── 内核层
```

Intelligent Voice Framework 属于 OpenHarmony AI 子系统的一部分，提供**语音注册**和**语音唤醒**两大核心能力。

**证据来源**:
- `bundle.json`: `"subsystem": "ai"`
- `bundle.json`: `"name": "@ohos/intelligent_voice_framework"`

### 1.2 与其他模块的关系

| 依赖模块 | 依赖类型 | 说明 |
|---------|---------|------|
| `audio_framework` | 外部依赖 | 音频采集和播放 |
| `ipc` | 外部依赖 | 进程间通信 |
| `safwk` | 外部依赖 | 系统能力框架 |
| `hdf_core` | 外部依赖 | HDI 驱动框架 |
| `huks` | 外部依赖 | 密钥管理 |
| `ability_runtime` | 外部依赖 | 能力管理 |

---

## 2. 核心能力

### 2.1 语音注册（Voice Enrollment）

将用户说出的唤醒词转换为**声学模型**和**声纹特征**，用于后续唤醒验证。

**关键特征**:
- 支持多轮注册（多次录入提高准确率）
- 实时反馈注册质量
- 支持不同语言和地区

**证据来源**:
- `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts`: `EnrollIntelligentVoiceEngine` 接口
- `frameworks/js/napi/enroll_intell_voice_engine_napi.h`: 注册引擎 N-API 实现

### 2.2 语音唤醒（Voice Wakeup）

检测当前说话者是否为已注册用户，是则唤醒系统。

**关键特征**:
- DSP 级别的唤醒检测（低功耗）
- 支持多种唤醒词
- 灵敏度和唤醒词可配置

**证据来源**:
- `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts`: `WakeupIntelligentVoiceEngine` 接口
- `frameworks/js/napi/wakeup_intell_voice_engine_napi.h`: 唤醒引擎 N-API 实现

### 2.3 系统事件感知

监听系统事件以优化语音服务策略。

| 监听事件 | 用途 |
|---------|------|
| `usual.event.BOOT_COMPLETED` | 开机完成后初始化 |
| `usual.event.POWER_SAVE_MODE_CHANGED` | 省电模式切换 |

**证据来源**:
- `sa_profile/intell_voice_service.json`: SA 启动配置

---

## 3. 基本概念

### 3.1 核心术语

| 术语 | 定义 | 代码位置 |
|------|------|---------|
| **Voice Enrollment** | 将唤醒词转换为声学模型和声纹特征的过程 | `EnrollIntelligentVoiceEngine` |
| **Voice Wakeup** | 验证说话者身份并唤醒系统的过程 | `WakeupIntelligentVoiceEngine` |
| **DSP Chip** | 实现数字信号处理的专用芯片 | HDI 驱动层 |
| **Wakeup Phrase** | 用于唤醒系统的特定词语 | `WakeupIntelligentVoiceEngineDescriptor.wakeupPhrase` |
| **VPR (Voice Print Recognition)** | 声纹识别技术 | 引擎内部实现 |

### 3.2 引擎类型

```typescript
enum IntelligentVoiceEngineType {
    ENROLL_ENGINE_TYPE = 0,   // 注册引擎
    WAKEUP_ENGINE_TYPE = 1,   // 唤醒引擎
    UPDATE_ENGINE_TYPE = 2     // 更新引擎
}
```

**证据来源**: `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts:343-368`

### 3.3 灵敏度等级

```typescript
enum SensibilityType {
    LOW_SENSIBILITY = 1,      // 低灵敏度
    MIDDLE_SENSIBILITY = 2,   // 中灵敏度
    HIGH_SENSIBILITY = 3      // 高灵敏度
}
```

**证据来源**: `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts:691-716`

---

## 4. 运行环境

### 4.1 系统要求

| 要求 | 说明 |
|------|------|
| **系统类型** | standard（标准系统） |
| **最低 API** | 10（动态）/ 22（静态） |
| **系统应用** | 仅系统应用可用（`@systemapi`） |

### 4.2 权限要求

所有 API 需要以下权限：

| 权限 | 用途 | 必需性 |
|------|------|--------|
| `ohos.permission.MANAGE_INTELLIGENT_VOICE` | 管理智能语音 | **必需** |
| `ohos.permission.MICROPHONE` | 访问麦克风 | 动态 API 22+ |

**错误码**:

| 错误码 | 含义 |
|--------|------|
| 201 | Permission denied - 权限被拒绝 |
| 202 | Not system application - 非系统应用 |

**证据来源**:
- `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts`: API 权限声明
- `utils/intell_voice_util.cpp`: 权限验证实现

---

## 5. 当前限制

| 限制 | 说明 | 状态 |
|------|------|------|
| 唤醒词数量 | 仅支持一个唤醒词 | 当前限制 |
| 语言支持 | 主要支持中文 | 需扩展 |

**证据来源**: `README.md`: "Currently, the intelligent voice framework supports the enrollment and wakeup of only one wakeup word."

---

## 6. 相关文档

| 文档 | 说明 |
|------|------|
| [架构设计](./02_Architecture.md) | 组件图、数据流、线程模型 |
| [目录结构](./03_Directory_Structure.md) | 模块职责和代码组织 |
| [N-API 参考](./04_NAPI_Reference.md) | API 详细用法 |
| [安全评审](./08_Security_Review.md) | 安全机制和风险分析 |

---

## 7. 快速开始示例

### 7.1 获取管理器

```typescript
import intelligentVoice from '@ohos.ai.intelligentVoice';

// 获取智能语音管理器
const manager = intelligentVoice.getIntelligentVoiceManager();
```

### 7.2 创建注册引擎

```typescript
// 创建注册引擎
const engine = await intelligentVoice.createEnrollIntelligentVoiceEngine({
    wakeupPhrase: '小艺小艺'  // 设置唤醒词
});

// 初始化引擎
await engine.init({
    language: 'zh',  // 中文
    region: 'CN'     // 中国大陆
});

// 开始注册
const result = await engine.enrollForResult(true);  // isLast=true

// 提交注册数据
await engine.commit();

// 释放引擎
await engine.release();
```

### 7.3 创建唤醒引擎

```typescript
// 创建唤醒引擎
const wakeupEngine = await intelligentVoice.createWakeupIntelligentVoiceEngine({
    needReconfirm: false,
    wakeupPhrase: '小艺小艺'
});

// 订阅唤醒事件
wakeupEngine.on('wakeupIntelligentVoiceEvent', (event) => {
    console.info('唤醒事件:', event.eventId, event.isSuccess, event.context);
});

// 设置应用信息
await wakeupEngine.setWakeupHapInfo({
    bundleName: 'com.example.app',
    abilityName: 'EntryAbility'
});
```
