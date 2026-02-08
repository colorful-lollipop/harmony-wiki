# 常见问题与调试

## 1. 构建问题

### 1.1 SDK 版本不匹配

**问题描述**:
```
ERROR: SDK version mismatch.
Required: API 14 (5.0.2)
Found: API xx
```

**解决方案**:
1. 确认 DevEco Studio 版本为 5.0.2 Release
2. 检查 SDK Manager 中的 API 14 是否安装
3. 修改 `build-profile.json5`:

```json5
{
  "products": [
    {
      "compileSdkVersion": 23,
      "compatibleSdkVersion": 23
    }
  ]
}
```

**证据位置**: `build-profile.json5:23-26`

### 1.2 模块依赖缺失

**问题描述**:
```
ERROR: Module not found: @ohos/utils
```

**解决方案**:
1. 检查 `build-profile.json5` 模块配置
2. 确认 utils 模块路径正确
3. 清理并重新构建:

```bash
hvigor clean
hvigor assembleHap
```

**证据位置**: `build-profile.json5:44-46`

### 1.3 签名配置错误

**问题描述**:
```
ERROR: Signing config not found: default
```

**解决方案**:
1. 检查 `signature/` 目录下的签名文件
2. 确认 `build-profile.json5` 中的 `signingConfig` 配置正确
3. 使用默认签名或配置自定义签名

**证据位置**: `build-profile.json5:22`

## 2. 运行问题

### 2.1 应用启动崩溃

**问题描述**: 应用启动后闪退

**排查步骤**:

1. **检查日志**:
```bash
hdc_std shell hilog | grep -E "MainAbility|note"
```

2. **常见原因**:
   - `AppStorage` 初始化失败
   - `RdbStore` 数据库初始化失败
   - 页面路由配置错误

3. **检查要点**:
   - 确认 `main_pages.json` 配置正确
   - 确认页面文件路径正确

**证据位置**: `product/default/src/main/resources/base/profile/main_pages.json`

### 2.2 数据库操作失败

**问题描述**:
```
ERROR: RdbStore operation failed
```

**排查步骤**:

1. **检查数据库配置**:
   - 确认 `RdbStoreUtil.createRdbStore()` 调用成功
   - 检查数据库表是否创建成功

2. **检查权限**:
   - 确认应用有数据库操作权限

3. **日志分析**:
```typescript
// 在 RdbStoreUtil 中添加日志
LogUtil.info(TAG, "createRdbStore success");
```

**证据位置**: `common/utils/src/main/ets/default/baseUtil/RdbStoreUtil.ets`

### 2.3 跨设备流转失败

**问题描述**: 设备间流转无响应

**排查步骤**:

1. **检查网络**:
   - 确认两台设备在同一网络
   - 检查防火墙设置

2. **检查流转参数**:
```typescript
// 确认参数完整
onContinue(wantParam) {
  wantParam["ContinueNote"] = continueNote;
  wantParam["ContinueSection"] = continueSection;
  // ...
}
```

3. **日志分析**:
```bash
hdc_std shell hilog | grep -E "onContinue|continue"
```

**证据位置**: `MainAbility.ts:163-214`

## 3. 调试方法

### 3.1 日志打印

**使用 LogUtil**:

```typescript
import { LogUtil } from '@ohos/utils/src/main/ets/default/baseUtil/LogUtil';

LogUtil.info(TAG, "debug message");
LogUtil.error(TAG, "error message: " + JSON.stringify(data));
```

**日志级别**:

| 级别 | 方法 | 用途 |
|------|------|------|
| INFO | `LogUtil.info()` | 普通信息 |
| ERROR | `LogUtil.error()` | 错误信息 |
| DEBUG | `LogUtil.debug()` | 调试信息 |
| WARN | `LogUtil.warn()` | 警告信息 |

**证据位置**: `common/utils/src/main/ets/default/baseUtil/LogUtil.ets`

### 3.2 查看日志

**获取全部日志**:
```bash
hdc_std shell hilog > hilog.log
```

**过滤应用日志**:
```bash
hdc_std shell hilog | grep -E "MainAbility|Note"
```

**过滤错误日志**:
```bash
hdc_std shell hilog | grep -E "ERROR|error"
```

### 3.3 断点调试

1. 在 DevEco Studio 中打开项目
2. 在代码行号处点击设置断点
3. 点击调试按钮开始调试
4. 使用调试控制台查看变量值

### 3.4 页面调试

**启用页面调试**:

1. 在 DevEco Studio 中打开 `DevTools`
2. 进入页面调试模式
3. 查看组件树和属性

## 4. 常见错误码

### 4.1 relationalStore 错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 401 | 参数错误 | 检查数据库参数 |
| 14800000 | 通用错误 | 检查日志详情 |
| 14800001 | 数据库不存在 | 检查数据库初始化 |

### 4.2 文件操作错误码

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 13900001 | 文件不存在 | 检查文件路径 |
| 13900002 | 权限不足 | 检查文件权限 |

## 5. 性能问题

### 5.1 启动慢

**排查方法**:
1. 检查 `onCreate()` 中的初始化逻辑
2. 优化数据库初始化（异步）
3. 减少同步操作

**优化建议**:
```typescript
// 异步初始化数据库
createRdbStore(context) {
  relationalStore.getRdbStore(context, dbName)
    .then((store) => {
      // 初始化完成后再加载数据
    });
}
```

### 5.2 页面卡顿

**排查方法**:
1. 检查组件状态更新频率
2. 优化列表渲染（使用 LazyForEach）
3. 减少不必要的重渲染

**优化建议**:
```typescript
// 使用 LazyForEach 优化长列表
LazyForEach(this.dataSource, (item: NoteData) => {
  NoteListComp({ note: item })
})
```

## 6. 相关跳转

| 目标 | 链接 |
|------|------|
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 构建配置 | [04_GN_Targets.md](04_GN_Targets.md) |
| 安全评估 | [06_Security_Review.md](06_Security_Review.md) |
