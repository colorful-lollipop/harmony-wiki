# 故障排查

## 概述

本章节提供 font_manager 常见问题的诊断和解决方案。

## 常见错误码

### 权限相关错误

| 错误码 | 常量 | 含义 | 排查方法 |
|--------|------|------|----------|
| 201 | ERR_NO_PERMISSION | 无 UPDATE_FONT 权限 | 检查应用是否声明并获取权限 |
| 202 | ERR_NOT_SYSTEM_APP | 非系统应用 | 字体安装仅限系统应用 |

### 参数相关错误

| 错误码 | 常量 | 含义 | 排查方法 |
|--------|------|------|----------|
| 401 | ERR_INVALID_PARAM | 参数无效 | 检查参数类型和格式 |
| 31100101 | ERR_FILE_NOT_EXISTS | 字体文件不存在 | 检查文件路径是否正确 |
| 31100102 | ERR_FILE_VERIFY_FAIL | 字体格式不支持 | 检查是否为有效字体文件 |

### 安装相关错误

| 错误码 | 常量 | 含义 | 排查方法 |
|--------|------|------|----------|
| 31100103 | ERR_COPY_FAIL | 文件复制失败 | 检查存储空间和权限 |
| 31100104 | ERR_INSTALLED_ALRADY | 字体已安装 | 检查是否重复安装 |
| 31100105 | ERR_MAX_FILE_COUNT | 超过最大数量 | 检查已安装字体数量 |
| 31100106 | ERR_INSTALL_FAIL | 安装失败 | 查看系统日志 |

### 卸载相关错误

| 错误码 | 常量 | 含义 | 排查方法 |
|--------|------|------|----------|
| 31100107 | ERR_UNINSTALL_FILE_NOT_EXISTS | 字体不存在 | 检查字体名称 |
| 31100108 | ERR_UNINSTALL_REMOVE_FAIL | 文件删除失败 | 检查文件权限 |
| 31100109 | ERR_UNINSTALL_FAIL | 卸载失败 | 查看系统日志 |

### 系统错误

| 错误码 | 常量 | 含义 | 排查方法 |
|--------|------|------|----------|
| 31100110 | ERR_SYSTEM_ERROR | 系统错误 | 查看系统日志 |
| 31100111 | ERR_DATA_MIGRATIONING | 正在迁移中 | 等待迁移完成 |

## 调试方法

### 1. 查看日志

```bash
# 查看 font_manager 相关日志
hilog | grep -E "FONT_MSG|FontManager"

# 实时查看日志
hilog -v brief | grep "FONT_MSG"
```

### 2. 检查 SA 状态

```bash
# 查看 SA 是否运行
dumpsys --dump-service-list | grep font

# 查看 SA 详细信息
dumpsys font_manager_server
```

### 3. 检查字体安装

```bash
# 查看已安装字体
ls -la /data/service/el1/0/for-all-app/fonts/

# 查看字体配置
cat /data/service/el1/0/for-all-app/fonts/install_fontconfig.json
```

### 4. 检查权限

```bash
# 查看应用权限
# 在代码中验证 permission 声明
```

## 常见问题

### Q1: 字体安装返回 201 权限错误

**问题**: 调用 `installFont` 返回错误码 201

**原因**: 应用没有 `ohos.permission.UPDATE_FONT` 权限

**解决方案**:

1. 在应用配置文件中声明权限：

```json
// module.json5
"requestPermissions": [
    {
        "name": "ohos.permission.UPDATE_FONT",
        "usedScene": {
            "abilities": ["EntryAbility"],
            "when": "inuse"
        }
    }
]
```

2. 动态申请权限（如果需要）：

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

let atManager = abilityAccessCtrl.createAtManager();
let bundleInfo = bundleManager.getBundleInfo('com.example.myapp', 1);
let tokenId = bundleInfo.appInfo.accessTokenId;
atManager.requestPermissionsFromUser(tokenId, ["ohos.permission.UPDATE_FONT"]);
```

**注意**: UPDATE_FONT 是系统权限，普通应用无法获取

---

### Q2: 字体安装返回 31100101 文件不存在

**问题**: 字体文件存在但返回文件不存在错误

**原因**: 路径验证失败

**解决方案**:

1. 检查路径是否为绝对路径：

```typescript
// 正确
const fontPath = '/data/storage/el2/base/files/myfont.ttf';
// 错误
const fontPath = 'myfont.ttf';
```

2. 检查文件是否存在：

```typescript
import fs from '@ohos.file.fs';

let stat = fs.statSync(fontPath);
if (stat) {
    console.log('文件存在');
}
```

3. 检查文件权限：

```bash
ls -la /data/storage/el2/base/files/myfont.ttf
```

---

### Q3: 字体安装返回 31100102 格式不支持

**问题**: 字体文件被认为是无效格式

**原因**: 文件格式不是有效的字体文件

**解决方案**:

1. 检查文件是否为有效字体文件：

```bash
file /path/to/myfont.ttf
# 应该是 TrueType font 或 OpenType font
```

2. 确认字体文件完整性：

```bash
# 检查文件大小
ls -la /path/to/myfont.ttf

# 使用字体查看工具验证
# 可尝试在 PC 上打开验证
```

---

### Q4: 字体安装返回 31100103 复制失败

**问题**: 文件复制到安装目录失败

**可能原因**:

1. 存储空间不足
2. 安装目录权限问题
3. 文件被占用

**解决方案**:

1. 检查存储空间：

```bash
df -h /data/service/el1/
```

2. 检查安装目录：

```bash
ls -la /data/service/el1/0/for-all-app/fonts/
```

3. 检查是否有其他进程占用文件

---

### Q5: 字体卸载失败

**问题**: 调用 `uninstallFont` 返回错误

**排查步骤**:

1. 确认字体名称正确：

```typescript
// 查看已安装的字体名称
// 需要通过其他方式获取已安装字体列表（当前 API 不支持）
```

2. 检查字体是否正在使用

3. 检查文件权限

---

### Q6: SA 未启动

**问题**: 调用 API 返回服务不可用

**排查步骤**:

1. 检查 SA 配置：

```bash
cat /system/etc/sa_config/66262.json
```

2. 检查 SA 库是否存在：

```bash
ls -la /system/lib/libfont_manager_server.z.so
```

3. 查看 SA 启动日志：

```bash
hilog | grep -E "FontManagerServer|SA 66262"
```

---

### Q7: DataMigration 返回 31100111

**问题**: 数据迁移返回"正在迁移中"

**原因**: 上一次迁移尚未完成

**解决方案**:

1. 等待上一次迁移完成
2. 检查迁移状态

---

## 调试技巧

### 启用详细日志

```cpp
// 在代码中添加日志输出
FONT_LOGI("InstallFont: start");
FONT_LOGD("InstallFont: fontPath = %{public}s", fontPath.c_str());
FONT_LOGE("InstallFont: error %{public}d", errCode);
```

### 使用调试器

```bash
# 使用 hdc 调试
hdc shell
# attach 到进程进行调试
```

### 跟踪系统调用

```bash
# 使用 strace 跟踪（需要 root）
strace -f -e open,read,write,close <process>
```

## 相关文档

- [N-API 参考](02_NAPI_Reference.md)
- [安全评审](06_Security_Review.md)
- [构建目标](04_Build_Targets.md)
