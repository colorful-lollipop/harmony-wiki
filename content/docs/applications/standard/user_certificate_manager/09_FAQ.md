# 常见问题与定位路径

> 本文档提供用户证书管理部件的常见构建、运行、调试问题和定位路径

---

## 目的

本文档帮助开发者、测试人员和运维人员快速定位和解决常见问题。

## 适用范围

- OpenHarmony 应用开发者
- 测试人员
- 运维人员
- 技术支持人员

## 关键结论

1. **编译问题常见**: SDK 路径、签名配置、资源格式错误
2. **运行时问题常见**: 权限不足、服务未就绪、参数错误
3. **调试方法**: hilog 日志、hdc 工具
4. **定位路径**: 日志 TAG、代码位置、错误码

## 相关跳转

- [00_Overview.md](wiki/00_Overview.md) - 项目概览
- [06_GN_Targets.md](wiki/06_GN_Targets.md) - GN Targets
- [07_Build_Artifacts.md](wiki/07_Build_Artifacts.md) - 编译产物

---

## 1. 构建问题

### 1.1 编译失败：找不到 SDK

**症状**:
```
error: SDK not found at //prebuilts/ohos-sdk/linux
```

**原因**: SDK 路径配置错误

**定位路径**:
1. 检查 `BUILD.gn:27` 的 `sdk_home` 配置
2. 确认 SDK 是否存在

**解决方法**:
```bash
# 1. 检查 SDK 路径
ls -la //prebuilts/ohos-sdk/

# 2. 修改 BUILD.gn 中的 sdk_home
sdk_home = "//correct/path/to/sdk"

# 3. 重新编译
./build.sh --product-name rk3568 --ccache --build-target user_certificate_manager
```

---

### 1.2 编译失败：签名错误

**症状**:
```
error: Failed to sign HAP
```

**原因**: 签名证书不存在或过期

**定位路径**:
1. 检查 `BUILD.gn:21` 的 `certificate_profile` 配置
2. 检查 `signature/` 目录下的证书文件

**解决方法**:
```bash
# 1. 检查签名证书
ls -la signature/

# 2. 如果证书不存在，生成或获取正确证书
# 请参考 OpenHarmony 签名文档

# 3. 确认 BUILD.gn 中的证书路径正确
certificate_profile = "signature/your-cert.p7b"
```

---

### 1.3 编译失败：资源格式错误

**症状**:
```
error: Invalid JSON format in string.json
```

**原因**: 资源文件 JSON 格式错误

**定位路径**:
1. 检查 `certmanager/src/main/resources/*/element/string.json`
2. 检查 `certmanager/src/main/resources/base/profile/main_pages.json`

**解决方法**:
```bash
# 1. 验证 JSON 格式
cat certmanager/src/main/resources/base/element/string.json | jq .

# 2. 修复 JSON 错误
# 使用 JSON 编辑器修复语法错误
```

---

## 2. 运行时问题

### 2.1 应用启动失败：权限不足

**症状**:
```
Failed to start ability: permission denied
```

**原因**: 应用未获得必要权限

**定位路径**:
1. 检查 `certmanager/src/main/module.json:84-115` 的权限声明
2. 检查系统版本是否支持所需权限

**解决方法**:
```bash
# 1. 确认权限声明
# 检查 module.json 中的 requestPermissions

# 2. 升级系统版本
# 如果系统版本过低，升级到支持该权限的版本

# 3. 签名权限
# 某些权限需要系统签名
```

---

### 2.2 证书安装失败：格式错误

**症状**:
```
Error: CM_ERROR_INCORRECT_FORMAT
```

**原因**: 证书文件格式不正确

**定位路径**:
1. 检查 `CertMangerModel.ets:719-728` 的错误处理
2. 检查文件后缀是否正确

**解决方法**:
```typescript
// 1. 检查文件后缀
let suffix = getFileSuffix(filePath);
if (!['cer', 'pem', 'crt', 'der', 'p7b', 'spc'].includes(suffix)) {
  console.error('Unsupported file format');
  return;
}

// 2. 验证证书内容
// 使用证书验证工具检查文件是否为有效证书
```

---

### 2.3 证书安装失败：达到最大数量

**症状**:
```
Error: CM_ERROR_MAX_CERT_COUNT_REACHED
```

**原因**: 证书数量超过系统限制

**定位路径**:
1. 检查 `CertMangerModel.ets:722` 的错误处理
2. 检查系统证书数量限制

**解决方法**:
```bash
# 1. 删除不需要的证书
# 通过应用界面删除旧证书

# 2. 检查系统限制
# 查阅系统文档了解证书数量限制

# 3. 联系系统管理员
# 如果确实需要更多证书，请联系系统管理员调整限制
```

---

### 2.4 用户认证失败

**症状**:
```
Authentication failed
```

**原因**: 用户认证失败或取消

**定位路径**:
1. 检查 `CheckUserAuthModel.ets:100-112` 的认证结果处理
2. 检查日志中的错误信息

**解决方法**:
```typescript
// 1. 检查认证类型是否支持
if (!isAuthTypeSupported(authType)) {
  console.error('Auth type not supported');
  // 提示用户设置认证方式
}

// 2. 引导用户完成认证设置
// 提示用户在设置中完成指纹或密码设置
```

---

## 3. 调试方法

### 3.1 查看应用日志

```bash
# 1. 连接设备
hdc list targets

# 2. 实时查看日志
hdc shell hilog -x | grep CertManager

# 3. 查看所有日志
hdc shell hilog -x

# 4. 清除日志缓冲
hdc shell hilog -r
```

### 3.2 查看 TAG

| TAG | 说明 | 文件位置 |
|-----|------|---------|
| `CertMangerModel` | 证书管理核心 | `CertMangerModel.ets:79` |
| `CheckUserAuthModel` | 用户认证 | `CheckUserAuthModel.ets:23` |
| `certManager BUNDLE:` | 包信息获取 | `BundleModel.ets:22` |
| `CertManager FA` | 文件 I/O | `FileIoModel.ets:22` |
| `PreventScreenshotsModel` | 防截屏 | `PreventScreenshotsModel.ets:22` |
| `CMFaPresenter: ` | 主页面逻辑 | `CmFaPresenter.ets:27` |
| `CertPickerUiExtAbility` | Extension Ability | `CertPickerUiExtAbility.ets:25` |

### 3.3 查看错误码

| 错误码 | 名称 | 说明 | 处理位置 |
|--------|------|------|---------|
| 0 | `CM_MODEL_ERROR_SUCCESS` | 成功 | 所有 Model |
| -1 | `CM_MODEL_ERROR_FAILED` | 失败 | 所有 Model |
| -2 | `CM_MODEL_ERROR_EXCEPTION` | 异常 | 所有 Model |
| -6 | `CM_MODEL_ERROR_INCORRECT_FORMAT` | 格式错误 | `CertMangerModel.ets:719` |
| -7 | `CM_MODEL_ERROR_MAX_QUANTITY_REACHED` | 达到最大数量 | `CertMangerModel.ets:722` |
| -8 | `CM_MODEL_ERROR_ALIAS_LENGTH_REACHED_LIMIT` | 别名过长 | `CertMangerModel.ets:724` |
| -9 | `CM_MODEL_ERROR_PASSWORD_ERR` | 密码错误 | `CertMangerModel.ets:752` |

---

## 4. 定位路径

### 4.1 证书安装失败

**定位步骤**:
1. 查看日志中的错误信息
   ```bash
   hdc shell hilog -x | grep "installUserCertificate"
   ```
2. 检查错误码 (`CertMangerModel.ets:719-728`)
3. 检查文件后缀是否正确 (`CmFaPresenter.ets:68-73`)
4. 检查证书文件是否存在且格式正确

**常见原因**:
- 文件格式不支持
- 文件损坏
- 系统证书数量达到上限

---

### 4.2 用户认证失败

**定位步骤**:
1. 查看日志中的认证信息
   ```bash
   hdc shell hilog -x | grep "userAuth"
   ```
2. 检查认证类型是否支持 (`CheckUserAuthModel.ets:32-42`)
3. 检查 Challenge 生成是否正常 (`CheckUserAuthModel.ets:78-84`)
4. 检查认证结果 (`CheckUserAuthModel.ets:100-112`)

**常见原因**:
- 用户未设置认证方式
- 认证设备不支持
- 用户取消认证
- 认证失败（密码错误、指纹不匹配）

---

### 4.3 应用崩溃

**定位步骤**:
1. 查看崩溃日志
   ```bash
   hdc shell hilog -x | grep -i "crash\|exception"
   ```
2. 检查内存泄露 (文件大小限制)
3. 检查空指针或类型错误
4. 使用 DevEco Studio 调试

**常见原因**:
- 文件过大导致 OOM
- 类型错误
- 异常未捕获

---

## 5. 性能问题

### 5.1 应用启动缓慢

**症状**: 应用启动时间超过 5 秒

**定位步骤**:
1. 检查主页面加载数量
2. 检查证书列表数量
3. 检查是否有阻塞操作

**优化建议**:
- 使用懒加载
- 分页加载证书列表
- 避免在主线程进行大量计算

---

### 5.2 内存占用过高

**症状**: 应用占用 RAM 超过 10MB

**定位步骤**:
1. 使用系统工具查看内存占用
   ```bash
   hdc shell ps -A | grep certmanager
   ```
2. 检查是否有内存泄露
3. 检查大文件加载

**优化建议**:
- 及时释放大对象
- 避免缓存大量数据
- 使用对象池

---

## 6. 兼容性问题

### 6.1 系统版本不兼容

**症状**: 应用无法在新版本系统上运行

**定位步骤**:
1. 检查 `build-profile.json5:21-22` 中的 SDK 版本
2. 检查系统版本支持的 API 级别

**解决方法**:
```json5
{
  "compileSdkVersion": 23,  // 更新为最新版本
  "compatibleSdkVersion": 23   // 保持兼容性
}
```

---

### 6.2 多语言支持问题

**症状**: 部分语言显示乱码

**定位步骤**:
1. 检查 `certmanager/src/main/resources/` 目录下的语言资源
2. 检查 `string.json` 是否包含所需语言的翻译

**解决方法**:
- 添加缺失语言的资源文件
- 补充缺失的翻译

---

## 7. 获取帮助

### 7.1 社区支持

- **OpenHarmony 官方论坛**: https://developer.harmonyos.com/cn/forum/
- **Gitee 仓库**: https://gitee.com/openharmony-sig/applications_user_certificate_manager
- **GitHub 仓库**: https://github.com/openharmony-sig/applications_user_certificate_manager

### 7.2 报告 Bug

1. 在 Gitee/GitHub 提交 Issue
2. 提供以下信息:
   - 系统版本
   - 复现步骤
   - 日志信息
   - 错误截图

---

**END OF 09_FAQ.md**
