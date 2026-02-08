# MediaLibrary 常见问题与排查

## 构建问题

### Q1: GN 构建失败

**问题描述**: 执行 `hb build` 时 GN 配置错误

**常见错误**:
```
gn gen out/default --check
ERROR: Can't find //build/ohos.gni
```

**排查步骤**:

1. **检查环境**
```bash
# 确认 OpenHarmony SDK
echo $OHOS_SDK_ROOT

# 检查 hbw 工具
which hb
hb --version
```

2. **检查路径配置**
```bash
# 确认 media_library.gni 存在
ls -la foundation/multimedia/media_library/media_library.gni

# 检查相对路径
pwd
```

3. **解决方案**
```bash
# 重新同步代码
git pull

# 清理并重新构建
hb clean -p multimedia/media_library
hb build -p multimedia/media_library
```

---

### Q2: 依赖缺失

**问题描述**: 编译时提示头文件找不到

**错误示例**:
```
fatal error: 'media_asset.h' file not found
```

**排查步骤**:

1. **检查 include_dirs**
```bash
# 查看 BUILD.gn 中的 include_dirs
grep -r "include_dirs" interfaces/kits/js/BUILD.gn
```

2. **检查依赖声明**
```bash
# 查看 deps
grep -r "deps" interfaces/kits/js/BUILD.gn
```

3. **解决方案**
```bash
# 同步依赖模块
hb build -p multimedia/media_framework

# 检查子模块更新
git submodule update --init --recursive
```

---

## 运行时问题

### Q3: 权限被拒绝

**问题描述**: 应用调用 MediaLibrary API 返回权限错误

**错误码**: `14000003` (EC_PERMISSION_DENIED)

**排查步骤**:

1. **检查权限声明**
```json
// module.json5
"requestPermissions": [
  {
    "name": "ohos.permission.READ_IMAGEVIDEO"
  }
]
```

2. **动态权限申请**
```typescript
// 在代码中申请权限
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
let atManager = abilityAccessCtrl.createAtManager();
atManager.requestPermissionsFromUser(this.context, [
  'ohos.permission.READ_IMAGEVIDEO'
]);
```

3. **验证权限状态**
```cpp
// 代码中验证
AccessTokenID token = IPCSkeleton::GetSelfTokenID();
int result = AccessTokenKit::VerifyAccessToken(token, PERM_READ_IMAGEVIDEO);
if (result != PERMISSION_GRANTED) {
    // 权限未授予
}
```

---

### Q4: N-API 返回 null

**问题描述**: N-API 调用返回 undefined 或 null

**排查步骤**:

1. **检查参数**
```typescript
// 确认参数类型正确
let asset = await mediaLibrary.getMediaAssets({
    // 正确的参数
    sourcePath: 'Pictures/'
});
```

2. **检查异步处理**
```cpp
// N-API 中 Promise 处理
napi_value promise;
napi_create_promise(env, &context->deferred, &promise);

// 确保在 complete 回调中 resolve
napi_resolve_promise(env, context->deferred, result);
```

3. **错误日志**
```bash
# 查看日志
hilog | grep MediaLibrary
```

---

### Q5: IPC 调用超时

**问题描述**: 服务调用返回超时

**错误码**: `1500001` (IPC timeout)

**排查步骤**:

1. **检查 SA 状态**
```bash
# 查看 SA 列表
hidumper -sa

# 查看 MediaLibrary SA
hidumper -sa MediaLibrary
```

2. **检查服务日志**
```bash
# 查看服务日志
hilog | grep MediaAssetsService
```

3. **重启服务**
```bash
# 重启 MediaLibrary SA
killall media_library_service
start media_library_service
```

---

## 数据库问题

### Q6: 查询无结果

**问题描述**: 查询 API 返回空结果集

**排查步骤**:

1. **检查文件是否存在**
```typescript
// 使用文件 API 确认文件
let file = await fs.open('/storage/media/local/files/test.jpg');
```

2. **检查数据库同步**
```cpp
// 检查是否需要刷新
MediaLibraryManager::Refresh();
```

3. **检查查询条件**
```typescript
// 确认查询条件正确
let assets = await mediaLibrary.getMediaAssets({
    sourcePath: 'Pictures/',  // 正确路径
    mediaType: photoType     // 正确类型
});
```

---

### Q7: 数据库损坏

**问题描述**: 数据库操作返回错误

**错误码**: `1002` (E_DATABASE_ERROR)

**排查步骤**:

1. **检查数据库文件**
```bash
ls -la /data/storage/el2/database/media_library/
```

2. **恢复数据库**
```cpp
// 使用备份恢复
MediaLibraryManager::RestoreFromBackup();
```

3. **重建数据库**
```bash
# 清除数据后重建
rm -rf /data/storage/el2/database/media_library/*
reboot
```

---

## 调试技巧

### 日志级别

| 级别 | 说明 | 启用方式 |
|-----|------|---------|
| **ERROR** | 错误 | 默认开启 |
| **WARN** | 警告 | 默认开启 |
| **INFO** | 信息 | 默认开启 |
| **DEBUG** | 调试 | 需要修改代码 |

### 日志查看

```bash
# 查看所有 MediaLibrary 日志
hilog | grep -E "MediaLibrary|MediaAsset|MediaAlbum"

# 查看特定级别
hilog | grep "MediaLibrary.*E"
hilog | grep "MediaLibrary.*W"
hilog | grep "MediaLibrary.*I"
```

### 调试断点

```cpp
// 添加调试日志
MEDIA_INFO_LOG("Enter GetMediaAssets");
MEDIA_INFO_LOG("Selection: %{public}s", selection.c_str());
MEDIA_INFO_LOG("Result count: %{public}d", result.size());
```

---

## 性能问题

### Q8: 查询慢

**问题描述**: 大量媒体文件时查询性能差

**优化建议**:

1. **使用分页**
```typescript
// 限制每页数量
let assets = await mediaLibrary.getMediaAssets({
    selection: 'date_taken > 0',
    selectionArgs: [],
    limit: 100,
    offset: 0
});
```

2. **优化索引**
```sql
-- 检查索引
PRAGMA index_list(Photos);
PRAGMA index_info(index_name);
```

3. **预加载缩略图**
```cpp
// 异步加载缩略图
ThumbnailManager::Preload(uris);
```

---

## 相关文档

| 文档 | 描述 |
|-----|------|
| [03_N-API_Reference](03_N-API_Reference.md) | N-API 接口 |
| [05_Build_System](05_Build_System.md) | 构建系统 |
| [06_Security_Review](06_Security_Review.md) | 安全评审 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 配置项 |
