# 安全风险评审

## 评估概述

### 评估范围

本文档对 `app_samples` 仓库中的所有 ArkTS 示例进行安全风险评估。

| 评估对象 | 说明 |
|---------|------|
| **评估范围** | code/ArkTS-Sta/ 下的 8 个示例 |
| **技术栈** | ArkTS 1.2 + Stage 模型 + API 20 |
| **分析深度** | 静态代码分析 + 架构审查 |

### 风险等级说明

| 等级 | 说明 |
|-----|------|
| **高** | 可被直接利用，可能导致权限提升或数据泄露 |
| **中** | 需要特定条件触发，可能影响用户体验 |
| **低** | 风险较低或仅影响调试场景 |

## 攻击面分析

### 已识别攻击面

```
┌─────────────────────────────────────────────────────┐
│                   外部输入                           │
│  • 用户界面输入 (TextInput, 按钮点击)               │
│  • 相机拍照结果 (startAbilityForResult)             │
│  • 文件选择 (Rawfile, zlib解压)                    │
└─────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│                   信任边界                          │
│  • OpenHarmony SDK API                             │
│  • 系统能力 (相机、文件系统)                        │
│  • 应用沙箱                                         │
└─────────────────────────────────────────────────────┘
```

### 攻击面清单

| 攻击面 | 类型 | 示例 | 信任边界 |
|-------|------|------|---------|
| 用户输入 | UI 交互 | TextInput 输入 | 应用沙箱内 |
| 相机结果 | IPC | startAbilityForResult 返回 | 系统框架 |
| 文件资源 | 本地 | Rawfile 解压 | 应用沙箱内 |
| 系统 API | 接口 | hilog 日志 | 系统框架 |

## 风险评估结果

### 风险汇总

| 风险 ID | 风险描述 | 等级 | 影响范围 | 状态 |
|--------|---------|------|---------|------|
| S-001 | 大文件复制无进度取消机制 | 低 | FilesSample | 建议优化 |
| S-002 | 错误信息通过 hilog 输出 | 低 | 所有示例 | 已知限制 |
| S-003 | 无输入长度限制 | 低 | CameraSample | 建议优化 |
| S-004 | 资源路径硬编码 | 低 | 所有示例 | 需评估 |

### 详细风险分析

#### S-001: 大文件复制无进度取消机制

**证据**: FilesSample/README.md - "大文件复制" 功能

```
路径: code/ArkTS-Sta/FilesSample/entry/src/main/ets/pages/BigFileCopy/
```

**描述**: 文件复制过程中，用户无法取消操作，可能导致：
- 长时间占用系统资源
- 复制大量数据后才发现问题

**触发条件**:
1. 复制超大文件（>100MB）
2. 存储空间不足

**影响**: 资源浪费，用户体验下降

**修复建议**:
```typescript
// 添加取消标志
let isCancelled = false;

async function copyFile() {
  while (!isCancelled && bytesRead > 0) {
    // 复制逻辑
  }
}

function cancelCopy() {
  isCancelled = true;
}
```

#### S-002: 错误信息通过 hilog 输出

**证据**: 所有示例中的错误处理模式

```typescript
// 典型错误处理代码
} catch (err) {
  hilog.error(0x0000, 'testTag', `failed: ${err.code}, ${err.message}`);
}
```

**描述**: 调试日志可能泄露敏感信息（如文件路径、错误详情）

**触发条件**:
1. 设备已 root
2. 使用日志工具读取 hilog

**影响**: 信息泄露（路径、参数等）

**修复建议**:
- 生产环境关闭 debug 日志
- 敏感信息脱敏处理

#### S-003: 无输入长度限制

**证据**: CameraSample - 评论输入框

```typescript
TextInput()
  .onChange((textInComment: string) => {
    this.commentContent = textInComment;  // 无长度限制
  })
```

**触发条件**: 用户输入超长文本

**影响**:
- 内存占用增加
- UI 渲染性能下降

**修复建议**:
```typescript
TextInput()
  .maxLength(1000)  // 添加长度限制
```

#### S-004: 资源路径硬编码

**证据**: FilesSample - 沙箱路径硬编码

```typescript
// 硬编码路径
const outputPath = '/data/app/el2/100/base/com.samples.filessample/haps/entry/files/';
```

**描述**: 硬编码路径不易维护，可能在不同设备上失效

**触发条件**:
1. 不同 bundleName
2. 不同设备类型

**修复建议**:
```typescript
// 使用 Context 获取路径
const context = getContext(this);
const filesDir = context.filesDir;
```

## 信任边界说明

### 组件信任边界

```
┌─────────────────────────────────────────────────────┐
│  不可信区域                                         │
│  • 用户输入                                         │
│  • 外部文件                                        │
└───────────────────────┬─────────────────────────────┘
                        │ 数据验证
                        ▼
┌─────────────────────────────────────────────────────┐
│  信任边界（应用沙箱）                                │
│  • 已验证的用户数据                                  │
│  • SDK API 调用                                    │
│  • 本示例代码                                      │
└───────────────────────┬─────────────────────────────┘
                        │ 系统调用
                        ▼
┌─────────────────────────────────────────────────────┐
│  可信区域（系统框架）                                │
│  • OpenHarmony SDK                                 │
│  • 系统能力                                         │
└─────────────────────────────────────────────────────┘
```

### 数据流安全

| 数据流 | 来源 | 处理 | 风险等级 |
|-------|------|------|---------|
| 用户输入 → UI | TextInput | 显示 | 低 |
| 相机结果 → Image | startAbilityForResult | 渲染 | 低 |
| Rawfile → 沙箱 | zlib | 复制 | 低 |

## 安全最佳实践

### 1. 输入验证

```typescript
// 输入长度限制
TextInput()
  .maxLength(100)

// 输入格式验证
function validateInput(input: string): boolean {
  return input.length <= 100 && !containsSpecialChars(input);
}
```

### 2. 错误处理

```typescript
try {
  // 业务逻辑
} catch (err) {
  // 日志脱敏
  const code = (err as BusinessError).code;
  const message = (err as BusinessError).message?.replace(/\d+/g, '[NUM]');
  hilog.error(0x0000, 'Tag', `error: ${code}, ${message}`);
}
```

### 3. 文件操作安全

```typescript
import { context } from '@kit.BasicServicesKit';

// 使用上下文获取路径
const filesDir = context.filesDir;

// 路径白名单检查
function isValidPath(path: string): boolean {
  return path.startsWith(filesDir);
}
```

### 4. 资源释放

```typescript
// 确保资源正确释放
let fd: number = -1;
try {
  fd = fs.openSync(path, fs.OpenMode.READ);
  // 操作文件
} finally {
  if (fd >= 0) {
    fs.closeSync(fd);
  }
}
```

## 检查清单

### 代码审查清单

- [ ] 用户输入是否有长度限制
- [ ] 错误信息是否包含敏感数据
- [ ] 文件路径是否硬编码
- [ ] 资源是否正确释放
- [ ] 异步操作是否有取消机制

### 安全配置

| 配置项 | 推荐值 |
|-------|-------|
| 日志级别 | Debug: 开, Release: 关 |
| 输入长度 | <= 1000 字符 |
| 文件操作 | 沙箱目录内 |

## 结论

### 整体评估

`app_samples` 仓库示例**整体风险较低**：

| 评估维度 | 评级 | 说明 |
|---------|------|------|
| 代码质量 | 中 | 遵循基本规范，个别优化空间 |
| 安全实践 | 中 | 有错误处理，缺少输入验证 |
| 攻击面 | 低 | 无外部网络、无敏感权限 |
| 可利用性 | 低 | 需要用户主动操作 |

### 改进建议

1. **输入验证**: 添加 TextInput maxLength 限制
2. **日志脱敏**: 错误信息去除敏感数据
3. **异步取消**: 大文件操作支持取消
4. **路径动态化**: 使用 context 获取路径

---
*评估日期: 2024年*
*评估方法: 静态代码分析 + 架构审查*
