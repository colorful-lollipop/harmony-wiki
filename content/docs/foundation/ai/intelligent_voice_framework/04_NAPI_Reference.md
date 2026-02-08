# N-API 参考

> **目的**: 提供 Intelligent Voice Framework 所有对外 N-API 的详细参考文档  
> **适用范围**: JS/TS 应用开发者、接口调用方  
> **最后更新**: 2026-02-06  
> **系统要求**: 仅系统应用可用（`@systemapi`）

---

## 1. 模块注册

### 1.1 模块信息

| 属性 | 值 |
|------|-----|
| **模块名** | `@ohos.ai.intelligentVoice` |
| **模块路径** | `foundation/ai/intelligent_voice_framework/frameworks/js` |
| **N-API 入口** | `intell_voice_manager_napi.cpp` |

### 1.2 导出函数

```typescript
// 获取智能语音管理器
function getIntelligentVoiceManager(): IntelligentVoiceManager;

// 获取唤醒管理器
function getWakeupManager(): WakeupManager;

// 创建注册引擎
function createEnrollIntelligentVoiceEngine(
    descriptor: EnrollIntelligentVoiceEngineDescriptor
): Promise<EnrollIntelligentVoiceEngine>;

// 创建唤醒引擎
function createWakeupIntelligentVoiceEngine(
    descriptor: WakeupIntelligentVoiceEngineDescriptor
): Promise<WakeupIntelligentVoiceEngine>;
```

**证据来源**: `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts`

---

## 2. IntelligentVoiceManager 接口

### 2.1 接口定义

```typescript
interface IntelligentVoiceManager {
    // 查询能力信息
    getCapabilityInfo(): Array<IntelligentVoiceEngineType>;

    // 订阅服务状态变更事件
    on(type: 'serviceChange', callback: Callback<ServiceChangeType>): void;
    onServiceChange(callback: Callback<ServiceChangeType>): void;

    // 取消订阅服务状态变更事件
    off(type: 'serviceChange', callback?: Callback<ServiceChangeType>): void;
    offServiceChange(callback?: Callback<ServiceChangeType>): void;
}
```

### 2.2 API 详情

#### getCapabilityInfo()

**签名**:
```typescript
getCapabilityInfo(): Array<IntelligentVoiceEngineType>
```

**描述**: 获取支持的引擎类型列表

**权限**: `ohos.permission.MANAGE_INTELLIGENT_VOICE`

**返回**: 支持的引擎类型数组

**错误码**:
| 错误码 | 含义 |
|--------|------|
| 201 | Permission denied |
| 202 | Not system application |

**C++ 实现位置**: `frameworks/js/napi/intell_voice_manager_napi.cpp`

---

#### on(type, callback) / onServiceChange()

**签名**:
```typescript
on(type: 'serviceChange', callback: Callback<ServiceChangeType>): void
onServiceChange(callback: Callback<ServiceChangeType>): void
```

**描述**: 订阅智能语音服务状态变更事件

**权限**: `ohos.permission.MANAGE_INTELLIGENT_VOICE`

**参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| type | string | 是 | 固定值 `'serviceChange'` |
| callback | Callback | 是 | 服务状态变更回调 |

**事件类型**:
```typescript
enum ServiceChangeType {
    SERVICE_UNAVAILABLE = 0  // 服务不可用
}
```

---

#### off(type, callback) / offServiceChange()

**签名**:
```typescript
off(type: 'serviceChange', callback?: Callback<ServiceChangeType>): void
offServiceChange(callback?: Callback<ServiceChangeType>): void
```

**描述**: 取消订阅服务状态变更事件

---

## 3. EnrollIntelligentVoiceEngine 接口

### 3.1 接口定义

```typescript
interface EnrollIntelligentVoiceEngine {
    // 查询支持的地区
    getSupportedRegions(): Promise<Array<string>>;

    // 初始化引擎
    init(config: EnrollEngineConfig): Promise<void>;

    // 执行注册
    enrollForResult(isLast: boolean): Promise<EnrollCallbackInfo>;

    // 停止注册
    stop(): Promise<void>;

    // 提交注册
    commit(): Promise<void>;

    // 设置唤醒应用信息
    setWakeupHapInfo(info: WakeupHapInfo): Promise<void>;

    // 设置灵敏度
    setSensibility(sensibility: SensibilityType): Promise<void>;

    // 设置参数
    setParameter(key: string, value: string): Promise<void>;

    // 获取参数
    getParameter(key: string): Promise<string>;

    // 评估唤醒词
    evaluateForResult(word: string): Promise<EvaluationResult>;

    // 释放引擎
    release(): Promise<void>;
}
```

### 3.2 API 详情

#### createEnrollIntelligentVoiceEngine()

**签名**:
```typescript
function createEnrollIntelligentVoiceEngine(
    descriptor: EnrollIntelligentVoiceEngineDescriptor
): Promise<EnrollIntelligentVoiceEngine>
```

**描述**: 创建注册引擎实例

**权限**: `ohos.permission.MANAGE_INTELLIGENT_VOICE`

**参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| descriptor | EnrollIntelligentVoiceEngineDescriptor | 是 | 引擎描述符 |

**EnrollIntelligentVoiceEngineDescriptor**:
```typescript
interface EnrollIntelligentVoiceEngineDescriptor {
    wakeupPhrase: string;  // 唤醒词
}
```

**返回**: 注册引擎实例

**错误码**:
| 错误码 | 含义 |
|--------|------|
| 201 | Permission denied |
| 202 | Not system application |
| 401 | Parameter error |
| 22700101 | No memory |
| 22700102 | Invalid parameter |

**C++ 实现位置**: `frameworks/js/napi/enroll_intell_voice_engine_napi.cpp`

---

#### init()

**签名**:
```typescript
init(config: EnrollEngineConfig): Promise<void>
```

**描述**: 初始化注册引擎

**权限**: `ohos.permission.MANAGE_INTELLIGENT_VOICE`

**参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| config | EnrollEngineConfig | 是 | 引擎配置 |

**EnrollEngineConfig**:
```typescript
interface EnrollEngineConfig {
    language: string;  // 语言，如 'zh'
    region: string;    // 地区，如 'CN'
}
```

---

#### enrollForResult()

**签名**:
```typescript
enrollForResult(isLast: boolean): Promise<EnrollCallbackInfo>
```

**描述**: 执行一次注册

**权限**: 
- API 10-21: `ohos.permission.MANAGE_INTELLIGENT_VOICE`
- API 22+: `ohos.permission.MANAGE_INTELLIGENT_VOICE` + `ohos.permission.MICROPHONE`

**参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| isLast | boolean | 是 | 是否为最后一次注册 |

**返回**: `EnrollCallbackInfo`
```typescript
interface EnrollCallbackInfo {
    result: EnrollResult;   // 注册结果
    context: string;        // 上下文信息
}
```

**EnrollResult**:
| 值 | 含义 |
|---|------|
| 0 (SUCCESS) | 成功 |
| -1 (VPR_TRAIN_FAILED) | 声纹训练失败 |
| -2 (WAKEUP_PHRASE_NOT_MATCH) | 唤醒词不匹配 |
| -3 (TOO_NOISY) | 环境太嘈杂 |
| -4 (TOO_LOUD) | 音量太大 |
| -5 (INTERVAL_LARGE) | 间隔太大 |
| -6 (DIFFERENT_PERSON) | 不同人 |
| -100 (UNKNOWN_ERROR) | 未知错误 |

---

#### commit()

**签名**:
```typescript
commit(): Promise<void>
```

**描述**: 提交注册数据

**权限**: `ohos.permission.MANAGE_INTELLIGENT_VOICE`

**错误码**:
| 错误码 | 含义 |
|--------|------|
| 22700104 | 提交注册失败 |

---

#### setWakeupHapInfo()

**签名**:
```typescript
setWakeupHapInfo(info: WakeupHapInfo): Promise<void>
```

**描述**: 设置唤醒应用信息

**参数**:
```typescript
interface WakeupHapInfo {
    bundleName: string;   // Bundle 名称
    abilityName: string;  // Ability 名称
}
```

---

#### setSensibility()

**签名**:
```typescript
setSensibility(sensibility: SensibilityType): Promise<void>
```

**描述**: 设置灵敏度

**参数**:
```typescript
enum SensibilityType {
    LOW_SENSIBILITY = 1,      // 低灵敏度
    MIDDLE_SENSIBILITY = 2,   // 中灵敏度
    HIGH_SENSIBILITY = 3      // 高灵敏度
}
```

---

#### evaluateForResult()

**签名**:
```typescript
evaluateForResult(word: string): Promise<EvaluationResult>
```

**描述**: 评估唤醒词质量

**返回**:
```typescript
interface EvaluationResult {
    score: int;                      // 评估分数
    resultCode: EvaluationResultCode; // 评估结果码
}
```

**EvaluationResultCode**:
| 值 | 含义 |
|---|------|
| 0 (UNKNOWN) | 未知 |
| 1 (PASS) | 通过 |
| 2 (WORD_EMPTY) | 唤醒词为空 |
| 3 (CHINESE_ONLY) | 仅支持中文 |
| 4 (INVALID_LENGTH) | 无效长度 |
| 5 (UNUSUAL_WORD) | 不常见词语 |
| 6 (CONSECUTIVE_SAME_WORD) | 连续重复词 |
| 7 (TOO_FEW_PHONEMES) | 音素太少 |
| 8 (TOO_MANY_PHONEMES) | 音素太多 |
| 9 (COMMON_INSTRUCTION) | 包含常用指令 |
| 10 (COMMON_SPOKEN_LANGUAGE) | 包含常用口语 |
| 11 (SENSITIVE_WORD) | 包含敏感词 |
| 12 (NO_INITIAL_CONSONANT) | 无首辅音 |
| 13 (REPEATED_PHONEME) | 包含重复音素 |

---

## 4. WakeupIntelligentVoiceEngine 接口

### 4.1 接口定义

```typescript
interface WakeupIntelligentVoiceEngine {
    getSupportedRegions(): Promise<Array<string>>;
    setWakeupHapInfo(info: WakeupHapInfo): Promise<void>;
    setSensibility(sensibility: SensibilityType): Promise<void>;
    setParameter(key: string, value: string): Promise<void>;
    getParameter(key: string): Promise<string>;
    on(type: 'wakeupIntelligentVoiceEvent', callback: Callback<WakeupIntelligentVoiceEngineCallbackInfo>): void;
    off(type: 'wakeupIntelligentVoiceEvent', callback?: Callback<WakeupIntelligentVoiceEngineCallbackInfo>): void;
    startCapturer(): Promise<void>;
    stopCapturer(): Promise<void>;
    getPcm(): Promise<void>;
    read(): Promise<void>;
    release(): Promise<void>;
}
```

### 4.2 API 详情

#### createWakeupIntelligentVoiceEngine()

**签名**:
```typescript
function createWakeupIntelligentVoiceEngine(
    descriptor: WakeupIntelligentVoiceEngineDescriptor
): Promise<WakeupIntelligentVoiceEngine>
```

**描述**: 创建唤醒引擎实例

**参数**:
```typescript
interface WakeupIntelligentVoiceEngineDescriptor {
    needReconfirm: boolean;   // 是否需要重新确认
    wakeupPhrase: string;     // 唤醒词
}
```

---

#### on(type, callback)

**签名**:
```typescript
on(type: 'wakeupIntelligentVoiceEvent', 
   callback: Callback<WakeupIntelligentVoiceEngineCallbackInfo>): void
```

**描述**: 订阅唤醒事件

**事件类型**:
```typescript
enum WakeupIntelligentVoiceEventType {
    INTELLIGENT_VOICE_EVENT_WAKEUP_NONE = 0,           // 无唤醒
    INTELLIGENT_VOICE_EVENT_RECOGNIZE_COMPLETE = 1,     // 识别完成
    INTELLIGENT_VOICE_EVENT_HEADSET_RECOGNIZE_COMPLETE = 2 // 耳机识别完成
}
```

**回调信息**:
```typescript
interface WakeupIntelligentVoiceEngineCallbackInfo {
    eventId: WakeupIntelligentVoiceEventType;  // 事件ID
    isSuccess: boolean;                        // 是否成功
    context: string;                           // 上下文信息
}
```

---

## 5. WakeupManager 接口

### 5.1 接口定义

```typescript
interface WakeupManager {
    setParameter(key: string, value: string): Promise<void>;
    getParameter(key: string): Promise<string>;
    getUploadFiles(maxCount: int): Promise<Array<UploadFile>>;
    getWakeupSourceFiles(): Promise<Array<WakeupSourceFile>>;
    enrollWithWakeupFilesForResult(
        wakeupFiles: Array<WakeupSourceFile>,
        wakeupInfo: string
    ): Promise<EnrollResult>;
    clearUserData(): Promise<void>;
}
```

### 5.2 API 详情

#### getUploadFiles()

**签名**:
```typescript
getUploadFiles(maxCount: int): Promise<Array<UploadFile>>
```

**描述**: 获取待上传的文件列表

**参数**: `maxCount` 范围 (0, 100]

**返回**:
```typescript
interface UploadFile {
    type: UploadFileType;       // 文件类型
    filesDescription: string;   // 文件描述
    filesContent: Array<ArrayBuffer>; // 文件内容
}

enum UploadFileType {
    ENROLL_FILE = 0,  // 注册文件
    WAKEUP_FILE = 1  // 唤醒文件
}
```

---

#### getWakeupSourceFiles()

**签名**:
```typescript
getWakeupSourceFiles(): Promise<Array<WakeupSourceFile>>
```

**描述**: 获取唤醒源文件

**返回**:
```typescript
interface WakeupSourceFile {
    filePath: string;      // 文件路径
    fileContent: ArrayBuffer; // 文件内容
}
```

---

## 6. 通用错误码

| 错误码 | 名称 | 含义 |
|--------|------|------|
| 201 | PERMISSION_DENIED | 权限被拒绝 |
| 202 | NOT_SYSTEM_APPLICATION | 非系统应用 |
| 22700101 | NO_MEMORY | 内存不足 |
| 22700102 | INVALID_PARAM | 无效参数 |
| 22700103 | INIT_FAILED | 初始化失败 |
| 22700104 | COMMIT_ENROLL_FAILED | 提交注册失败 |
| 22700105 | START_CAPTURER_FAILED | 启动录音失败 |
| 22700106 | READ_FAILED | 读取失败 |
| 22700107 | SYSTEM_ERROR | 系统错误 |

**证据来源**: `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts:790-847`

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Overview.md) | 权限要求 |
| [架构设计](./02_Architecture.md) | 调用链 |
| [安全评审](./08_Security_Review.md) | 权限验证 |
