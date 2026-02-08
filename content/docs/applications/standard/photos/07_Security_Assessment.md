# 安全风险评审

## 1. 评审概述

### 1.1 评审范围

| 范围 | 说明 |
|------|------|
| **代码扫描** | `applications/standard/photos/` |
| **排除范围** | 测试目录（test/, tests/ 等） |
| **语言** | ArkTS / TypeScript |

### 1.2 评审方法

- 代码静态分析
- 架构安全性评估
- 权限配置审查
- 输入验证分析

---

## 2. 攻击面分析

### 2.1 外部输入点

| 输入类型 | 来源 | 处理模块 |
|----------|------|----------|
| **Want 参数** | 外部应用 `startAbility` | `MainAbility.ts` |
| **URI 参数** | 外部应用传入 | `parseWantParameter()` |
| **媒体文件路径** | 用户选择/相机 | `MediaLibrary` |
| **用户操作** | UI 交互 | 各页面组件 |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    可信区域                               │  │
│  │  • 系统 API 调用                                          │  │
│  │  • 内部模块调用 (@ohos/common, @ohos/timeline)            │  │
│  │  • 已签名的系统服务                                        │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ▲                                  │
│                              │                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    边界区域                               │  │
│  │  • Want 参数解析 [MainAbility.ts:92-166]                  │  │
│  │  • 外部传入 URI                                           │  │
│  │  • 第三方回调数据                                         │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 权限机制

### 3.1 已声明权限

| 权限 | 敏感度 | 用途 | 触发组件 |
|------|--------|------|----------|
| `ohos.permission.READ_IMAGEVIDEO` | **高** | 读取媒体文件 | FormAbility |
| `ohos.permission.WRITE_IMAGEVIDEO` | **高** | 写入媒体文件 | FormAbility |
| `ohos.permission.MEDIA_LOCATION` | **中** | 访问位置信息 | FormAbility |
| `ohos.permission.PROXY_AUTHORIZATION_URI` | **低** | 代理授权 | MainAbility |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | **中** | 后台启动 | MainAbility |
| `ohos.permission.GET_BUNDLE_INFO` | **低** | 获取包信息 | MainAbility |

> **证据**: `module.json5:21-55`

### 3.2 权限使用评估

| 权限 | 必要性 | 评估结论 |
|------|--------|----------|
| READ_IMAGEVIDEO | ✅ 必要 | FA 卡片浏览需要 |
| WRITE_IMAGEVIDEO | ⚠️ 可选 | 仅编辑功能需要 |
| MEDIA_LOCATION | ⚠️ 可选 | 仅位置标签需要 |
| START_ABILITIES_FROM_BACKGROUND | ⚠️ 可选 | 特定场景需要 |
| GET_BUNDLE_INFO | ⚠️ 可选 | 第三方调用溯源 |

---

## 4. 安全风险分析

### 4.1 已识别风险

#### 风险 1：Want 参数注入

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-001 |
| **风险类型** | 输入验证不足 |
| **严重程度** | **中** |

**描述**:
`parseWantParameter()` 函数直接解析外部传入的 Want 参数，未进行严格的类型校验和范围限制。

**证据**:
- `MainAbility.ts:92-166` - `parseWantParameter()` 函数
- `MainAbility.ts:118` - `mFilterMediaType = wantParam?.filterMediaType as string`
- `MainAbility.ts:125` - `mMaxSelectCount = wantParam?.maxSelectCount as number`

**触发条件**:
```typescript
// 恶意应用可传入异常参数
let maliciousWant = {
  bundleName: "com.ohos.photos",
  abilityName: "com.ohos.photos.MainAbility",
  parameters: {
    uri: "multipleselect",
    maxSelectCount: 99999999,  // 异常大数值
    filterMediaType: "<script>alert('xss')</script>"  // 异常类型
  }
};
```

**潜在影响**:
1. 拒绝服务（异常内存占用）
2. UI 异常（异常类型导致渲染问题）

**修复建议**:
```typescript
// 添加参数校验
parseWantParameter(isOnNewWant: boolean, want: Want): void {
  // 校验 maxSelectCount 范围
  let maxSelect = wantParam?.maxSelectCount as number;
  mMaxSelectCount = (maxSelect > 0 && maxSelect <= 100) ? maxSelect : Constants.NUMBER_9;

  // 校验 filterMediaType
  let filterType = wantParam?.filterMediaType as string;
  if (![AlbumDefine.FILTER_MEDIA_TYPE_ALL,
        AlbumDefine.FILTER_MEDIA_TYPE_IMAGE,
        AlbumDefine.FILTER_MEDIA_TYPE_VIDEO].includes(filterType)) {
    mFilterMediaType = AlbumDefine.FILTER_MEDIA_TYPE_ALL;
  }
}
```

---

#### 风险 2：路径遍历（低风险）

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-002 |
| **风险类型** | 路径遍历 |
| **严重程度** | **低** |

**描述**:
虽然 ArkTS 应用不直接操作文件系统，但 `UserFileManagerAccess` 接收的 URI 可能指向敏感路径。

**证据**:
- `MainAbility.ts:68` - `UserFileManagerAccess.getInstance().onCreate()`

**触发条件**:
```typescript
// 传入恶意构造的 URI
wantParam?.albumUri = "../../../etc/passwd";
```

**潜在影响**:
OpenHarmony 媒体库 API 本身有沙箱保护，路径遍历风险较低。

**修复建议**:
- 依赖 `MediaLibrary` API 的内置校验
- 勿直接拼接 URI 路径

---

#### 风险 3：第三方回调数据泄露

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-003 |
| **风险类型** | 信息泄露 |
| **严重程度** | **低** |

**描述**:
`select-item-list` 返回结果包含选中文件的完整路径，可能泄露文件位置信息。

**证据**:
- `MainAbility.ts:113-130` - 单选模式参数处理
- `README_zh.md:147-154` - 返回结果示例

**潜在影响**:
- 第三方应用获取文件实际路径
- 可能用于进一步攻击

**修复建议**:
- 考虑使用 Content URI 替代文件路径
- 评估是否需要限制返回信息粒度

---

#### 风险 4：越权访问（UI 扩展）

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-004 |
| **风险类型** | 权限越权 |
| **严重程度** | **中** |

**描述**:
`DeleteUIExtensionAbility`、`SaveUIExtensionAbility` 等 UI 扩展未声明明确的权限校验机制。

**证据**:
- `module.json5:156-167` - UI Extension 配置

**触发条件**:
任何应用可通过指定 `type: sysDialog/common` 调用这些对话框。

**潜在影响**:
- 恶意应用诱导用户删除/保存文件

**修复建议**:
- 添加调用方签名验证
- 限制可调用该对话框的应用白名单

---

#### 风险 5：日志泄露敏感信息

| 属性 | 值 |
|------|-----|
| **风险 ID** | SEC-005 |
| **风险类型** | 信息泄露 |
| **严重程度** | **低** |

**描述**:
日志输出可能包含敏感信息（如文件路径、用户数据）。

**证据**:
- `MainAbility.ts:74` - `Log.info(TAG, 'Application onCreate end')`
- `MainAbility.ts:93` - `Log.info(TAG, \`Application isOnNewWant=${isOnNewWant}, want=${JSON.stringify(want)}\`)`

**触发条件**:
通过 `hilog` 查看日志时可能看到敏感信息。

**修复建议**:
```typescript
// 敏感信息脱敏
Log.info(TAG, `want param: ${JSON.stringify(want).slice(0, 100)}...`);
```

---

### 4.2 未发现风险

| 检查项 | 状态 | 说明 |
|--------|------|------|
| SQL 注入 | ✅ 无风险 | ArkTS 使用参数化查询 |
| 原生代码漏洞 | ✅ 无风险 | 无 N-API 纯 TS 实现 |
| 动态代码加载 | ✅ 无风险 | 无 eval/Function |
| 不安全的加密 | ✅ 无风险 | 无加密操作 |
| 硬编码密钥 | ✅ 未发现 | 代码检查未发现 |

---

## 5. 安全最佳实践

### 5.1 当前已采用

| 实践 | 实现方式 | 证据 |
|------|----------|------|
| **参数类型校验** | `as` 类型断言 | `MainAbility.ts` |
| **单例窗口** | `launchType: singleton` | `module.json5:66` |
| **权限声明** | `requestPermissions` | `module.json5:21-55` |
| **签名打包** | SHA256withECDSA | `build-profile.json5` |

### 5.2 建议采用

| 建议 | 优先级 | 说明 |
|------|--------|------|
| 参数范围校验 | 高 | `maxSelectCount` 等 |
| 调用方验证 | 中 | UI Extension 白名单 |
| 日志脱敏 | 低 | 敏感信息脱敏 |
| 返回数据最小化 | 低 | Content URI 替代路径 |

---

## 6. 安全相关配置

### 6.1 签名配置

| 配置 | 值 | 说明 |
|------|-----|------|
| 算法 | SHA256withECDSA | 强签名算法 |
| 密钥别名 | OpenHarmony Application Release | 专用密钥 |

> **证据**: `build-profile.json5:18`

### 6.2 权限配置

所有权限使用系统标准权限，无自定义权限。

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [03_External_API.md](03_External_API.md) | 外部调用接口 |
| [08_Troubleshooting.md](08_Troubleshooting.md) | 问题排查 |
