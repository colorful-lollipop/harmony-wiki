# 安全评审

## 目的

本文档对 FilePicker 应用进行安全风险评审，识别攻击面、信任边界和可被利用点。

## 适用范围

本文档适用于：
- 安全审计人员
- 需要了解安全风险的开发者

## 关键结论

1. **特权权限使用**：FilePicker 拥有多项特权权限（FILE_ACCESS_MANAGER、PROXY_AUTHORIZATION_URI 等）
2. **URI 权限授权机制**：存在过度授权风险（未过滤敏感路径）
3. **输入校验不充分**：文件名校验存在绕过可能
4. **路径遍历防护不足**：部分文件操作未充分检查路径边界
5. **任务池并发风险**：TaskManager 最大并发 5 个任务，可能导致资源耗尽

## 攻击面清单

### 1. 文件选择器攻击面

| 攻击面 | 描述 | 影响组件 | 证据 |
|---------|------|----------|-------|
| Want 参数注入 | 恶意应用构造特殊 Want 参数导致异常 | MainAbility | `MainAbility.ets:35` |
| 文件名注入 | 恶意文件名包含特殊字符或过长 | PathPicker.ets:95 | `PathPicker.ets:116` |
| URI 路径遍历 | 构造恶意 URI 访问敏感目录 | FileAccessExec.ets | `FileAccessExec.ets:38-73` |
| 权限提升 | 利用授权机制获取未授权的文件访问 | FilePickerUtil | `FilePickerUtil.ets:123` |
| 拒绝服务 | 大量并发请求导致服务不可用 | TaskManager | `TaskManager.ts:36` |

### 2. 下载授权攻击面

| 攻击面 | 描述 | 影响组件 | 证据 |
|---------|------|----------|-------|
| Bundle Name 欺骗 | 假冒合法应用名称获取下载授权 | DownloadAuth | `DownloadAuth.ets:74` |
| URI 越权 | 通过合法 URI 获取敏感数据 | DownloadAuth | `DownloadAuth.ets:82` |

### 3. 系统权限滥用

| 权限 | 潜在滥用风险 | 证据 |
|------|----------|-------|
| FILE_ACCESS_MANAGER | 访问任意文件系统 | `module.json5:72-73` |
| PROXY_AUTHORIZATION_URI | 代理授权任意 URI | `module.json5:79-81` |
| GET_BUNDLE_INFO_PRIVILEGED | 获取任意应用信息 | `module.json5:75-77` |

## 信任边界

### 边界 1：调用者应用 ↔ FilePicker

```
信任边界：不可信
数据流向：Want 参数 → FilePicker → URI 权限
威胁：调用者可能传递恶意参数
保护机制：
- ✅ 文件名校验（正则）
- ✅ 文件长度限制（225 字符）
- ✅ 文件数量限制（默认 50）
- ❌ URI 路径遍历检查不足
```

**证据**：
- Want 解析：`FilePickerUtil.ets:70-107`
- 文件名校验：`Constant.ts:32-34`
- 数量限制：`FilePickerUtil.ets:95`

### 边界 2：FilePicker ↔ 系统文件系统

```
信任边界：部分信任
数据流向：FileAccess API → 文件系统
威胁：FilePicker 可能被滥用访问敏感文件
保护机制：
- ✅ 依赖系统权限（MEDIA_LOCATION 等）
- ❌ 未进行路径白名单检查
- ❌ 未限制访问范围（如仅 Download 目录）
```

**证据**：
- FileAccess 初始化：`AbilityCommonUtil.ts:createFileAccessHelper`
- 文件访问：`FileAccessExec.ets:38-187`

### 边界 3：FilePicker ↔ URI 权限管理器

```
信任边界：信任
数据流向：URI 列表 → 权限管理器 → 调用者
威胁：过度授权可能泄露敏感数据
保护机制：
- ✅ 使用官方 API（FileShare.grantUriPermission）
- ✅ 支持批量授权优化
- ❌ 未验证 URI 敏感性
```

**证据**：
- 单个授权：`AbilityCommonUtil.ts:grantUriPermission`
- 批量授权：`FilePickerUtil.ets:123-144`

## 可被利用点

### 1. 文件名校验绕过

**严重程度**：中

**描述**：文件名校验正则可能被绕过，导致创建非法文件。

**证据**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/base/constants/Constant.ts
export const FILENAME_REGEXP = /^[^\\/:*?<>\"|]+$/  // 第 32 行
export const FILENAME_MAX_LENGTH = 225  // 第 34 行
```

**问题**：
1. 正则 `/^[^\\/:*?<>\"|]+$/` 可被以下方式绕过：
   - Unicode 控制字符（如 `\u0000`）
   - 字符编码绕过（URL 编码）
2. 未检查保留文件名（如 `CON`, `PRN`, `AUX`）
3. 未检查路径分隔符（如 `..` 在某些场景下）

**触发路径**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/pages/PathPicker.ets
async saveFiles(path: string, nameList: string[]): Promise<string[]> {  // 第 95 行
    // ...
    let result: ErrUri = await FileUtil.createFile(fileAccessHelper, dirPath, currName);  // 第 116 行
    // ...
}
```

**影响**：
- 恶意应用可创建非法文件名，可能导致：
  - 系统文件操作异常
  - 文件系统损坏
  - 安全机制绕过

**修复建议**：
```typescript
// 1. 增强正则表达式
export const FILENAME_REGEXP = /^[^\\/:*?<>\"|\x00-\x1F\x7F]+$/;

// 2. 检查保留文件名
const RESERVED_NAMES = ['CON', 'PRN', 'AUX', 'NUL', 'COM1-9', 'LPT1-9'];
function isReservedName(name: string): boolean {
    return RESERVED_NAMES.includes(name.toUpperCase().split('.')[0]);
}

// 3. 检查路径遍历字符
const PATH_TRAVERSAL_CHARS = ['..', '~'];
function containsPathTraversal(path: string): boolean {
    return PATH_TRAVERSAL_CHARS.some(char => path.includes(char));
}

// 4. 在 saveFiles 中增加检查
async saveFiles(path: string, nameList: string[]): Promise<string[]> {
    for (let i = 0; i < len; i++) {
        const currName = nameList[i];
        if (isReservedName(currName) || containsPathTraversal(currName)) {
            throw new Error('Invalid file name');
        }
        // ...
    }
}
```

### 2. URI 路径遍历

**严重程度**：高

**描述**：FileAccessExec 获取文件列表时未充分验证 URI 路径边界，可能导致路径遍历攻击。

**证据**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/base/utils/FileAccessExec.ets
export function getFileByCurIterator(fileInfo: fileAccess.FileInfo): FilesData[] {  // 第 90 行
    // ...
    let result = fileIterator.next();  // 第 98 行
    let isDone = result.done;
    while (!isDone) {
        const data = result.value;
        tempFile.fileName = data.fileName;  // 第 106 行
        tempFile.uri = data.uri;  // 第 108 行
        // ... 未验证 uri 合法性
        result = fileIterator.next();
        isDone = result.done;
    }
    return fileList;
}
```

**问题**：
1. `data.uri` 和 `data.fileName` 直接使用，未验证是否包含路径遍历字符
2. 未检查 URI 是否指向预期目录范围（如 `/data/storage` 以外）
3. `getRootFolder()` 未限制根目录范围

**证据**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/base/utils/FileAccessExec.ets
export function getRootFolder(): FilesData[] {  // 第 166 行
    const rootFolder: fileAccess.RootInfo = globalThis.rootInfoArr.find(  // 第 173 行
        (item: fileAccess.RootInfo) => item.deviceType === fileExtensionInfo.DeviceType.DEVICE_LOCAL_DISK);
    if (rootFolder) {
        fileList = getFileByCurIterator(rootFolder);  // 直接使用，未验证
    }
    return fileList;
}
```

**影响**：
- 恶意应用通过构造特殊 URI 可能：
  - 访问系统敏感目录（如 `/data`, `/system`）
  - 绕过权限检查访问其他应用数据
  - 泄露系统配置文件

**修复建议**：
```typescript
// 1. 定义允许的根目录白名单
const ALLOWED_ROOT_TYPES = [
    fileExtensionInfo.DeviceType.DEVICE_LOCAL_DISK
];

// 2. URI 合法性验证
function isValidUri(uri: string): boolean {
    // 检查路径遍历
    if (uri.includes('..') || uri.includes('~')) {
        return false;
    }
    // 检查 URI 格式
    if (!uri.startsWith('file://media')) {
        return false;
    }
    return true;
}

// 3. 在 getFileByCurIterator 中增加验证
export function getFileByCurIterator(fileInfo: fileAccess.FileInfo): FilesData[] {
    let fileList: FilesData[] = [];
    let result = fileIterator.next();
    let isDone = result.done;
    while (!isDone) {
        const data = result.value;
        if (!isValidUri(data.uri)) {
            Logger.e(TAG, 'Invalid URI detected: ' + data.uri);
            continue;  // 跳过非法 URI
        }
        // ...
    }
    return fileList;
}
```

### 3. URI 过度授权风险

**严重程度**：中

**描述**：FilePicker 对 URI 授权未进行充分检查，可能授权敏感文件或过度授权。

**证据**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/base/utils/AbilityCommonUtil.ts
export function grantUriPermission(uriList: Array<string>, bundleName: string): Promise<boolean> {  // 第 1 行
    return new Promise(async (resolve, reject) => {
        for (let uri of uriList) {  // 第 4 行
            try {
                await FileShare.grantUriPermission(  // 第 7 行
                    uri,
                    bundleName,
                    wantConstant.Flags.FLAG_AUTH_READ_URI_PERMISSION |
                    wantConstant.Flags.FLAG_AUTH_WRITE_URI_PERMISSION
                );
            } catch (error) {
                resolve(false);
                return;
            }
        }
        resolve(true);
    });
}
```

**问题**：
1. 未验证 URI 的敏感性（如是否指向 `/data/app`）
2. 未验证 bundleName 的合法性（是否为空或非法）
3. 授权给整个 URI 列表，未区分文件类型
4. 批量授权模式（≥50 个文件）未进行单独验证

**影响**：
- 恶意应用可能：
  - 获取对敏感文件的访问权限
  - 过度授权导致权限泄露
  - 利用批量授权绕过单文件检查

**修复建议**：
```typescript
// 1. URI 敏感性检查
function isSensitiveUri(uri: string): boolean {
    const SENSITIVE_PATHS = [
        '/data/app',
        '/data/system',
        '/data/user'
    ];
    return SENSITIVE_PATHS.some(path => uri.includes(path));
}

// 2. 增强授权检查
export function grantUriPermission(uriList: Array<string>, bundleName: string): Promise<boolean> {
    return new Promise(async (resolve, reject) => {
        // 验证 bundleName
        if (!bundleName || bundleName.trim().length === 0) {
            resolve(false);
            return;
        }

        let authorizedCount = 0;
        for (let uri of uriList) {
            // 验证 URI
            if (!isValidUri(uri) || isSensitiveUri(uri)) {
                Logger.w(TAG, 'Skipped sensitive URI: ' + uri);
                continue;
            }
            try {
                await FileShare.grantUriPermission(uri, bundleName, flags);
                authorizedCount++;
            } catch (error) {
                resolve(false);
                return;
            }
        }
        // 检查是否全部授权成功
        resolve(authorizedCount === uriList.length);
    });
}
```

### 4. Bundle Name 欺骗风险

**严重程度**：低

**描述**：DownloadAuth 未充分验证调用者 bundleName，可能被欺骗。

**证据**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/pages/DownloadAuth.ets
async downloadDialogConfirm(): Promise<void> {  // 第 73 行
    const bundleName: string = this.startModeOptions.callerBundleName;  // 第 74 行
    // 直接使用，未验证合法性
    this.downloadNewUri = VirtualUri.DOWNLOAD + '/' + bundleName;  // 第 76 行
    AbilityCommonUtil.grantUriPermission([this.downloadNewUri], bundleName);  // 第 82 行
    // ...
}
```

**问题**：
1. 未验证 bundleName 是否为空或非法
2. 未检查 bundleName 是否包含特殊字符
3. 未验证 bundleName 是否为系统应用

**影响**：
- 恶意应用可能：
  - 欺骗合法应用名称
  - 获取其他应用的下载授权
  - 创建恶意目录名称

**修复建议**：
```typescript
// 1. Bundle 名称合法性验证
function isValidBundleName(bundleName: string): boolean {
    if (!bundleName || bundleName.trim().length === 0) {
        return false;
    }
    const BUNDLE_NAME_REGEXP = /^[a-zA-Z0-9_.]+$/;
    return BUNDLE_NAME_REGEXP.test(bundleName);
}

// 2. 在 downloadDialogConfirm 中增加验证
async downloadDialogConfirm(): Promise<void> {
    const bundleName: string = this.startModeOptions.callerBundleName;

    if (!isValidBundleName(bundleName)) {
        Logger.e(TAG, 'Invalid bundle name: ' + bundleName);
        this.session.terminateSelfWithResult({
            resultCode: -1,
            want: {}
        });
        return;
    }

    // 后续授权流程...
}
```

### 5. 任务池资源耗尽风险

**严重程度**：低

**描述**：TaskManager 固定最大并发任务数为 5，可能导致资源耗尽或任务排队延迟。

**证据**：
```typescript
// /Volumes/lexar/code/d/work/oh/applications/standard/filepicker/entry/src/main/ets/taskpool/manager/TaskManager.ts
export class TaskManager {
    static readonly maxRunningTask: number = 5;  // 第 36 行
    static executorMap: HashMap<string, TaskExecutor> = new HashMap();  // 第 40 行

    execute<T>(task: BaseTask, maxRunningTask: number = TaskManager.maxRunningTask, ...): void {  // 第 75 行
        let executor = TaskManager.executorMap.get(taskName);
        if (!executor) {
            executor = new TaskExecutor(taskName, maxRunningTask, isOrder, maxWaitLimit);  // 第 91 行
            TaskManager.executorMap.set(taskName, executor);
        }
        executor.execute<T>(task);
    }
}
```

**问题**：
1. 固定最大并发数（5），无法根据系统资源动态调整
2. 未检查当前系统负载（CPU、内存）
3. 大量批量授权任务可能导致任务堆积

**影响**：
- 高并发场景下：
  - 任务执行延迟
  - 内存占用增加
  - 用户体验下降

**修复建议**：
```typescript
// 1. 动态调整并发数
import { systemCapability } from '@ohos.systemParameter';

export class TaskManager {
    static async getMaxRunningTask(): Promise<number> {
        // 获取系统内存信息
        const memInfo = await systemCapability.getMemoryInfo();
        const availableMemory = memInfo.availMem;

        // 根据可用内存动态调整
        if (availableMemory < 1024 * 1024 * 100) {  // < 100MB
            return 3;
        } else if (availableMemory < 1024 * 1024 * 300) {  // < 300MB
            return 5;
        } else {
            return 8;
        }
    }

    static async execute<T>(task: BaseTask, maxRunningTask?: number, ...): Promise<void> {
        const actualMax = maxRunningTask ?? await TaskManager.getMaxRunningTask();
        // 使用动态并发数...
    }
}
```

## 防护措施总结

| 风险点 | 当前防护 | 修复优先级 |
|---------|----------|------------|
| 文件名校验绕过 | ✅ 正则 + 长度限制<br/>❌ 无 Unicode/保留名检查 | 高 |
| URI 路径遍历 | ✅ URI 格式验证<br/>❌ 无路径白名单检查 | 高 |
| URI 过度授权 | ✅ 使用官方 API<br/>❌ 无 URI 敏感性检查 | 中 |
| Bundle Name 欺骗 | ❌ 无验证 | 中 |
| 任务池资源耗尽 | ❌ 固定并发数 | 低 |

## 安全检查清单

- [ ] 文件名校验增强（Unicode、保留名检查）
- [ ] URI 路径遍历防护（白名单、路径规范化）
- [ ] URI 敏感性检查（过滤敏感路径）
- [ ] Bundle Name 合法性验证
- [ ] 任务池并发数动态调整
- [ ] 错误信息脱敏（避免泄露路径信息）
- [ ] 日志安全（不记录敏感 URI 和路径）

## 相关跳转

- [架构设计](02_Architecture.md) - 理解数据流
- [权限机制](06_Permissions.md) - 了解授权流程
