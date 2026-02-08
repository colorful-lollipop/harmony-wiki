# 安全风险评审

## 评审概述

本评审基于代码静态分析，对安全隐私中心模块进行安全风险评估。评审范围包括：输入校验、权限模型、数据存储、页面跳转等方面。

**评审时间**：2026-02-06
**评审范围**：`entry/src/main/ets/` 目录下的业务代码（不含测试）
**评审方法**：静态代码分析 + 威胁建模

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        不可信区域                                │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              用户操作（点击、输入）                        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ▲                                   │
│                              │ 信任边界                          │
│                              ▼                                   │
├─────────────────────────────────────────────────────────────────┤
│                        可信区域（应用沙箱内）                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │  页面渲染     │  │  业务逻辑     │  │  本地数据存储        │   │
│  │  (ArkUI)     │  │  (Model)     │  │  (RDB)              │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              系统 API 调用（权限控制范围内）                │    │
│  │  geoLocationManager / bundleManager / osAccount         │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流

| 数据源 | 处理方式 | 信任级别 |
|-------|---------|---------|
| 用户点击 | 路由到 ViewModel 处理 | 低（需校验） |
| BMS 返回 | 转换为 MenuInfo | 中（系统可信） |
| RDB 缓存 | 直接使用 | 中（沙箱内） |
| 系统事件 | 通过回调处理 | 高（系统可信） |

## 攻击面分析

### 1. 页面路由攻击面

**风险点**：router.pushUrl 可能被恶意利用跳转未授权页面

**涉及代码**：
```typescript
// AutoMenuModel.ets:121-143
router.pushUrl({
  url: menuInfo.dstAbilityName  // 外部可控
}, router.RouterMode.Single);
```

**风险等级**：🟡 中

**缓解措施**：
- `RouterMode.Single` 防止页面栈混乱
- 跳转目标由 BMS 配置控制，非用户直接输入

**建议**：增加 URL 白名单校验

### 2. 外部 Intent 注入

**风险点**：UiExtensionPage 参数可能包含恶意 Intent

**涉及代码**：
```typescript
// UiExtensionPage.ets:25-27
@State routerParma: Record<string, string> = router.getParams() as Record<string, string>;
@State dstBundleName: string = this.routerParma['dstBundleName'];
@State dstAbilityName: string = this.routerParma['dstAbilityName'];

// UiExtensionPage.ets:31-36
UIExtensionComponent({
  bundleName: this.dstBundleName,
  abilityName: this.dstAbilityName,
  parameters: {
    'ability.want.params.uiExtensionType': 'sys/commonUI',
  }
})
```

**风险等级**：🟡 中

**缓解措施**：
- uiExtensionType 固定为 `sys/commonUI`
- 系统对 UIExtension 有隔离机制

**建议**：增加 bundleName/abilityName 有效性校验

### 3. 资源路径遍历

**风险点**：资源名称解析可能存在路径遍历

**涉及代码**：
```typescript
// AutoMenuModel.ets:206-211
let titleResList = res.split(':');
if (titleResList.length >= 2) {
  let title = titleResList[1];
  return await resourceManager.getStringByName(title);
}
```

**风险等级**：🟢 低

**分析**：资源名称由 BMS 配置管理，非用户输入

### 4. 数据存储安全

**涉及代码**：
```typescript
// AutoMenuModel.ets:147-157
private async _refreshDb(context: Context, menuInfoList: MenuInfo[]) {
  let predicates = new relationalStore.RdbPredicates(
    DataShareConstants.ANTO_MENU_TABLE_V2.tableName
  );
  // ...
}
```

**风险等级**：🟢 低

**分析**：
- RDB 在应用沙箱内，其他应用无法访问
- 使用参数化查询防止 SQL 注入

### 5. 权限滥用风险

**涉及代码**：
```typescript
// module.json5:51-77
"requestPermissions": [
  { "name": "ohos.permission.MANAGE_SECURE_SETTINGS" },
  { "name": "ohos.permission.CONTROL_LOCATION_SWITCH" },
  // ...
]
```

**风险等级**：🟡 中

**分析**：
- 声明了 7 个权限，其中 5 个高敏感权限
- 权限使用受系统管控，恶意使用会被检测

**建议**：
- 定期审计权限使用情况
- 遵循最小权限原则

## 可被利用点清单

| 编号 | 风险类型 | 可利用路径 | 影响 | 等级 | 修复建议 |
|-----|---------|-----------|------|------|---------|
| S01 | 页面路由 | 恶意构造 MenuInfo 跳转 | 未授权页面访问 | 🟡 | 增加 URL 白名单校验 |
| S02 | 参数注入 | UiExtension 参数注入 | UI 劫持 | 🟡 | 增加参数有效性校验 |
| S03 | 权限声明 | 过多高敏感权限 | 权限滥用风险 | 🟡 | 定期审计权限使用 |
| S04 | 资源解析 | 异常资源名称 | 资源加载失败 | 🟢 | 增加异常处理 |
| S05 | 数据泄露 | 缓存敏感数据 | 信息泄露 | 🟢 | 敏感数据加密存储 |

## 安全控制措施

### 1. 权限控制

| 权限 | 用途 | 系统保护级别 |
|-----|------|-------------|
| ohos.permission.MANAGE_SECURE_SETTINGS | 管理安全设置 | 仅系统应用可申请 |
| ohos.permission.CONTROL_LOCATION_SWITCH | 控制位置开关 | 受限权限 |

### 2. 沙箱隔离

- RDB 数据仅限本应用访问
- 页面栈管理防止越权访问
- UIExtension 组件有独立隔离

### 3. 输入校验

| 校验点 | 校验方式 |
|-------|---------|
| bundleName | 系统 API 返回，可信 |
| abilityName | BMS 配置，非用户输入 |
| 资源名称 | 参数化查询 |

### 4. 日志脱敏

```typescript
// Logger.ets
// 日志应避免记录敏感信息
Logger.info(TAG, `enable location, result: ${JSON.stringify(res)}`)
// 注意：生产环境应考虑日志脱敏
```

## 局限性说明

1. **静态分析局限**：本评审基于静态代码分析，未进行运行时动态分析
2. **测试范围局限**：未覆盖边界条件和异常场景的渗透测试
3. **依赖分析局限**：系统 API 的安全性依赖 OpenHarmony 框架本身
4. **时间局限**：评审时间为 2026-02-06，代码变更后需重新评审

## 安全加固建议

### 短期（高优先级）

1. **页面跳转校验**：对 `dstAbilityName` 增加白名单校验
2. **参数校验**：对 UiExtension 参数增加有效性验证
3. **日志脱敏**：审查 Logger 调用，避免敏感信息泄露

### 中期（优先级）

1. **权限最小化**：审查权限使用，移除未使用的权限声明
2. **安全测试**：引入安全测试用例，覆盖边界条件
3. **依赖扫描**：定期扫描依赖库的安全漏洞

### 长期（规划）

1. **安全审计机制**：建立代码变更的安全评审流程
2. **安全监控**：增加运行时安全监控能力
3. **应急响应**：建立安全漏洞应急响应机制

## 返回导航

- [SUMMARY.md](./SUMMARY.md) → 文档导航
- [07_Build_Outputs.md](./07_Build_Outputs.md) → 编译产物
- [09_FAQ.md](./09_FAQ.md) → 常见问题
