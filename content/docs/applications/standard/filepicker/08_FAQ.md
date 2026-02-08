# 常见问题

## 目的

本文档提供 FilePicker 应用在构建、运行、调试过程中的常见问题和定位路径。

## 适用范围

本文档适用于：
- 遇到构建失败的开发者
- 遇到运行时错误的使用者
- 需要调试 FilePicker 的开发者

## 关键结论

1. **纯 ArkTS 实现**：无 C/C++ 代码，所有问题可通过 TypeScript 源码定位
2. **Hvigor 构建系统**：非 GN，使用 Hvigor 工具链
3. **日志系统完善**：使用 HiLog 日志框架，可配置日志级别

## 构建问题

### Q1: 编译报错 "Cannot find module '@ohos.file.fileAccess'"

**问题描述**：编译时找不到系统模块。

**原因**：
1. SDK 版本不匹配
2. DevEco Studio 未正确导入系统模块
3. 模块路径错误

**解决方案**：
```bash
# 1. 检查 build-profile.json5 中的 SDK 版本
{
  "compileSdkVersion": 23,
  "compatibleSdkVersion": 23
}

# 2. 更新 DevEco Studio SDK
# Tools → SDK Manager → 更新到 API 23+

# 3. 清理并重新构建
hvigorw clean
hvigorw --mode module -p product=default assembleHap
```

**证据**：`build-profile.json5:7-8`

---

### Q2: 签名失败 "Invalid keystore or certificate"

**问题描述**：HAP 签名时报错证书或密钥库无效。

**原因**：
1. 签名证书路径错误
2. 密钥库密码错误
3. 签名证书过期

**解决方案**：
```bash
# 1. 检查 build-profile.json5 签名配置
{
  "material": {
    "certpath": "./signature/default_applications_filepicker_*.cer",
    "keyAlias": "debugKey",
    "keyPassword": "0000001B51325657C496FFB58D0925F364A7ABD91E8D36A914BE1F5E308C0DE2353DE4F805EF7BA9F4B545",
    "storeFile": "./signature/default_applications_filepicker_*.p12",
    "storePassword": "0000001B5727F90E843C257A05BB2F1E779C07EAFE6ACF1489795E0BC8D16B7EA407E09791AA9DA9FC714D"
  }
}

# 2. 验证签名文件存在
ls -la signature/
-rw-r--r-- 1 user  staff  0B Feb  5 23:20 default_applications_filepicker_*.cer
-rw-r--r-- 1 user  staff  3.2K Feb  5 23:20 default_applications_filepicker_*.p12

# 3. 检查密码是否正确（signature/default_applications_filepicker_6oaExOUKH552zeTOyp7pR0CmcmhFdlIgGU3DTAKskMw=.p12）
```

**证据**：`build-profile.json5:12-24`

---

### Q3: 产物 HAP 文件体积过大

**问题描述**：生成的 HAP 文件体积异常大（如 > 50MB）。

**原因**：
1. 未正确配置资源过滤
2. 包含了大文件（如高清图片）
3. 未启用资源压缩

**解决方案**：
```javascript
// hvigorfile.js 配置资源优化
module.exports = require('@ohos/hvigor-ohos-plugin').appTasks(
    {
        // 启用资源压缩
        enableCompress: true,

        // 排除不必要的资源
        excludeAssets: ['**/*.psd', '**/*.ai'],

        // 配置资源混淆
        obfuscation: {
            enable: true,
            policy: 'default'
        }
    }
)
```

---

## 运行问题

### Q4: 启动 FilePicker 时无响应

**问题描述**：通过 startAbilityForResult 拉起 FilePicker 后，界面无响应或闪退。

**定位步骤**：

```typescript
// 1. 检查 Want 参数是否正确
let want: Want = {
    bundleName: 'com.ohos.filepicker',
    abilityName: 'com.ohos.filepicker.MainAbility',
    parameters: {
        'startMode': 'choose',  // ✅ 必须小写
        'key_pick_num': 5,
        'key_select_mode': 0
    }
};
console.log('Want params:', JSON.stringify(want));

// 2. 检查调用者是否有所需权限
let bundleInfo = await bundleManager.getBundleInfoForSelf();
console.log('Requested permissions:', bundleInfo.requestPermissions);

// 3. 查看 FilePicker 日志
// hilog -T FilePicker
```

**常见原因**：
1. `startMode` 参数错误（如大写 "CHOOSE"）
2. 缺少必要的系统权限
3. FilePicker 服务未正确安装

**解决方案**：
```typescript
// 1. 确保 startMode 参数正确
parameters: {
    'startMode': 'choose'  // ✅ 小写，不是 'CHOOSE'
}

// 2. 在 module.json5 中声明所需权限
"requestPermissions": [
  {
    "name": "ohos.permission.FILE_ACCESS_MANAGER"
  }
]
```

**证据**：
- Want 参数解析：`FilePickerUtil.ets:70-107`
- 日志工具：`Logger.ts:16-48`

---

### Q5: 返回的 URI 无法访问

**问题描述**：FilePicker 返回的 URI 在调用者应用中无法打开或访问。

**定位步骤**：

```typescript
// 1. 检查返回的 URI 格式
context.startAbilityForResult(want).then((result) => {
    let uris = result.want?.parameters?.['ability.params.stream'] as string[];
    console.log('Returned URIs:', uris);  // 应为 'file://media/...' 格式

    // 2. 检查 URI 是否合法
    for (let uri of uris) {
        if (!uri.startsWith('file://')) {
            console.error('Invalid URI format:', uri);
        }
    }

    // 3. 尝试打开文件
    try {
        let file = await fs.open(uri);
        console.log('File opened:', file.name);
    } catch (error) {
        console.error('Failed to open file:', error.code, error.message);
    }
});
```

**常见原因**：
1. URI 权限未正确授予
2. URI 格式错误
3. 调用者应用没有相应的读权限

**解决方案**：
```typescript
// 1. 确认 URI 权限已授予
context.startAbilityForResult(want).then((result) => {
    if (result.resultCode === 0) {
        let uris = result.want?.parameters?.['ability.params.stream'] as string[];
        console.log('Returned URIs:', uris);

        // 等待权限生效
        setTimeout(() => {
            accessFile(uris);
        }, 100);
    }
});

// 2. 确保调用者有读权限
"requestPermissions": [
  {
    "name": "ohos.permission.READ_IMAGEVIDEO"
  }
]
```

---

### Q6: 批量授权后无法获取文件

**问题描述**：选择 ≥50 个文件后，调用者通过 udKey 无法获取文件列表。

**定位步骤**：

```typescript
// 1. 检查返回的 udKey
context.startAbilityForResult(want).then((result) => {
    let udKey = result.want?.parameters?.['ability.params.udkey'] as string;
    console.log('Returned udKey:', udKey);  // 应为非空字符串

    // 2. 通过 UDMF 查询文件
    import { unifiedDataChannel } from '@kit.ArkData';
    let data = await unifiedDataChannel.queryData(
        { intention: unifiedDataChannel.Intention.PICKER, key: udKey }
    );
    console.log('UDMF query result:', data);
});
```

**常见原因**：
1. udKey 传递或解析错误
2. UDMF 数据已过期或被清理
3. 调用者应用没有 UDMF 访问权限

**解决方案**：
```typescript
// 1. 验证 udKey 格式
if (typeof udKey !== 'string' || udKey.length === 0) {
    console.error('Invalid udKey');
    return;
}

// 2. 增加重试机制
async function getBatchFiles(udKey: string, retries = 3): Promise<string[]> {
    for (let i = 0; i < retries; i++) {
        try {
            let data = await unifiedDataChannel.queryData({
                intention: unifiedDataChannel.Intention.PICKER,
                key: udKey
            });
            return data[0].getRecords().map(r => r.getValue());
        } catch (error) {
            console.error(`Query failed (attempt ${i + 1}):`, error);
            if (i === retries - 1) {
                throw error;
            }
            await new Promise(resolve => setTimeout(resolve, 1000));  // 延迟 1 秒
        }
    }
}
```

**证据**：
- 批量授权：`FilePickerUtil.ets:123-144`
- UDMF 查询：`FilePickerUtil.ets:181-196`

---

### Q7: 文件保存时名称冲突

**问题描述**：保存文件时提示"文件名已存在"，但用户期望自动重命名。

**定位步骤**：

```typescript
// 检查 PathPicker 的重命名逻辑
// /entry/src/main/ets/pages/PathPicker.ets:186-224
async saveFiles(path: string, nameList: string[]): Promise<string[]> {
    // ...
    try {
        result = await FileUtil.createFile(fileAccessHelper, dirPath, newName);
    } catch (error) {
        if (result.err.code === ErrorCodeConst.FILE_ACCESS.FILE_NAME_EXIST) {
            // 自动重命名逻辑
            newName = FileUtil.renameFile(name, renameCount++, suffix);
        }
    }
}
```

**常见原因**：
1. 文件重命名逻辑未正确触发
2. 用户期望行为与实现不一致
3. 文件名校验导致重命名失败

**解决方案**：
```typescript
// 1. 增强重命名策略
function generateUniqueFileName(baseName: string, existingNames: string[]): string {
    let name = baseName;
    let counter = 1;
    const [base, ext] = name.split('.');

    while (existingNames.includes(name)) {
        name = `${base} (${counter})${ext ? '.' + ext : ''}`;
        counter++;
    }
    return name;
}

// 2. 在 saveFiles 中实现自动重命名
async saveFiles(path: string, nameList: string[]): Promise<string[]> {
    // 获取目录下现有文件名
    let existingFiles = await getExistingFileNames(path);

    for (let i = 0; i < nameList.length; i++) {
        let originalName = nameList[i];
        let uniqueName = generateUniqueFileName(originalName, existingFiles);
        let result = await FileUtil.createFile(fileAccessHelper, path, uniqueName);
        existingFiles.push(uniqueName);
        successArr.push(result.uri);
    }
}
```

**证据**：`PathPicker.ets:186-224`

---

## 调试技巧

### 使用 HiLog 日志系统

**配置日志级别**：

```typescript
// /entry/src/main/ets/base/log/Logger.ts
const enum LogVersion {
    Debug = 1,
    Info = 2,
    Warn = 3,
    Error = 4,
    Fatal = 5,
}

const LOG_LEVEL = LogVersion.Debug  // ✅ 开发环境设置为 Debug
const APP_TAG = 'FilePicker'
const LOG_DOMAIN = 0

// 日志输出示例
Logger.i(TAG, 'This is an info message');
Logger.e(TAG, 'This is an error message');
```

**查看日志**：

```bash
# 查看所有日志
hilog -T FilePicker

# 按级别过滤
hilog -T FilePicker -l I  # 仅 Info 级别
hilog -T FilePicker -l E  # 仅 Error 级别

# 实时监控
hilog -T FilePicker -v  # 实时输出
```

**证据**：`Logger.ts:27-48`

---

### 使用 DevEco Studio 调试器

**设置断点**：

```typescript
// 1. 在源码中设置断点
export async function terminateFilePicker(result: string[] = [],
    resultCode: number = ResultCodePicker.SUCCESS, startModeOptions: StartModeOptions): Promise<void> {
    Logger.i(TAG, 'enter terminateFilePicker');  // ⬅️ 设置断点
    const NORMAL_PICKER_SELECT_NUM: number = 50;
    if (result.length > NORMAL_PICKER_SELECT_NUM) {  // ⬅️ 设置断点
        // ...
    }
}
```

**调试 Want 参数**：

```typescript
// 在 FilePickerUtil.getStartModeOptions 中设置断点
export function getStartModeOptions(want: Want): StartModeOptions {  // ⬅️ 设置断点
    let options = new StartModeOptions();
    if (!want) {
        Logger.e(TAG, 'getDocumentSelectOptions want is undefined')
        return options;
    }
    options.action = want.action as string || '';  // ⬅️ 查看实际传入的值
    options.callerAbilityName = want.parameters?.['ohos.aafwk.param.callerAbilityName'] as string || '';
    // ...
    Logger.i(TAG, 'getDocumentOptions : ' + JSON.stringify(options));  // ⬅️ 查看解析后的值
    return options;
}
```

---

### 查看 UI 状态

```typescript
// 在 Page 组件中使用 @Watch 和 console.log
@State checkedNum: number = 0;
@State fileListSource: FileDataSource = new FileDataSource();

checkedNumChange(): void {  // ⬅️ 设置断点
    this.selectAll = this.checkedNum === this.fileListSource.totalCount();
    console.log('checkedNum changed:', this.checkedNum);  // 查看状态变化
    this.checkedList = this.fileListSource.getSelectedFileList();
}
```

**证据**：`MyPhone.ets:62-69`

---

### 常见错误码速查

| 错误码 | 含义 | 排查步骤 |
|--------|------|----------|
| 1000 | 正常 | ✅ 无需处理 |
| 1001 | 文件名已存在 | 检查文件是否已创建 |
| 1002 | 文件名非法 | 检查文件名格式 |
| 13900015 | 文件名已存在（系统层） | 检查 FileAccess API 返回 |
| 14000001 | 文件名非法（系统层） | 检查文件名正则 |
| -102825984 | IPC 异常 | 检查跨进程通信状态 |
| '3' | 媒体库查询为空 | 检查媒体库权限 |

**证据**：`ErrorCodeConst.ts:19-73`

---

## 问题定位路径

### 1. 确认问题现象

```bash
# 记录问题发生的详细步骤
1. 操作步骤：...
2. 期望结果：...
3. 实际结果：...
4. 错误信息：...
```

### 2. 查看日志

```bash
# 提取关键日志
hilog -T FilePicker -l E > error.log
hilog -T FilePicker -l W > warn.log

# 搜索关键词
grep "grantUriPermission" error.log
grep "terminateSelfWithResult" error.log
```

### 3. 复现问题

```typescript
// 编写最小复现用例
async function reproduceIssue() {
    let want = {
        bundleName: 'com.ohos.filepicker',
        abilityName: 'com.ohos.filepicker.MainAbility',
        parameters: {
            'startMode': 'choose',
            'key_pick_num': 1
        }
    };

    let result = await context.startAbilityForResult(want);
    console.log('Result:', JSON.stringify(result));
}
```

### 4. 定位代码位置

```bash
# 使用 grep 搜索关键代码
grep -rn "grantUriPermission" entry/src/main/ets/
grep -rn "FILE_NAME_EXIST" entry/src/main/ets/

# 使用代码导航工具（DevEco Studio）
# Ctrl + Click 方法名 → 跳转到定义
```

### 5. 修复并验证

```bash
# 1. 修改代码
# 2. 重新编译
hvigorw --mode module -p product=default assembleHap -m entry

# 3. 安装测试
hdc install entry-default-signed.hap

# 4. 验证修复
# 测试复现用例，确认问题已解决
```

## 相关跳转

- [架构设计](02_Architecture.md) - 理解数据流
- [权限机制](06_Permissions.md) - 了解授权流程
- [安全评审](07_Security_Review.md) - 查看已知风险
