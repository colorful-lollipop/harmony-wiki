# 命令参考 (Command Reference)

> bundle_tool 所有命令的详细用法、参数说明与示例

## 命令概览

bundle_tool 提供以下命令：

| 命令 | 功能 | 优先级 |
|------|------|--------|
| [help](#帮助命令-help) | 显示帮助信息 | 高 |
| [install](#安装命令-install) | 安装 HAP/HSP | **极高** |
| [uninstall](#卸载命令-uninstall) | 卸载应用 | 高 |
| [dump](#查询命令-dump) | 查询应用信息 | 高 |
| [clean](#清理命令-clean) | 清理缓存/数据 | 中 |
| [enable](#使能命令-enable) | 使能应用 | 中 |
| [disable](#禁用命令-disable) | 禁用应用 | 中 |
| [get](#获取udid命令-get) | 获取设备 UDID | 低 |
| [quickfix](#快速修复命令-quickfix) | 快速修复 | 中 |
| [compile](#编译命令-compile) | AOT 编译 | 低 |
| [copy-ap](#拷贝ap命令-copy-ap) | 拷贝 AP 文件 | 低 |
| [dump-overlay](#查询overlay命令-dump-overlay) | 查询 Overlay | 低 |
| [dump-target-overlay](#查询目标overlay命令-dump-target-overlay) | 查询目标 Overlay | 低 |
| [dump-shared](#查询hsp命令-dump-shared) | 查询 HSP | 低 |
| [dump-dependencies](#查询依赖命令-dump-dependencies) | 查询依赖 | 低 |
| [install-plugin](#安装插件命令-install-plugin) | 安装插件 | 低 |
| [uninstall-plugin](#卸载插件命令-uninstall-plugin) | 卸载插件 | 低 |

---

## 使用说明

### 调用方式

```bash
# 方式 1: 通过 hdc shell
hdc shell
bm <command> <options>

# 方式 2: 直接执行
hdc shell bm <command> <options>
```

### 通用选项

| 选项 | 说明 |
|------|------|
| `-h, --help` | 显示帮助信息 |
| `-u, --user-id <user-id>` | 指定用户 ID |

**说明**: 用户 ID 仅支持当前活跃用户或 0

---

## 帮助命令 (help)

### 用法

```bash
bm help
```

### 说明

显示 bm 工具支持的命令列表。

### 示例

```bash
$ bm help
usage: bm <command> <options>
These are common bm commands list:
  help         list available commands
  install      install a bundle with options
  uninstall    uninstall a bundle with options
  install-plugin install a plugin with options
  uninstall-plugin  uninstall a plugin with option
  dump         dump the bundle info
  get          obtain device udid
  quickfix     quick fix, including query and install
  compile      Compile the software package
  copy-ap      Copy software ap file to /data/local/pgo
  dump-overlay dump overlay info of the specific overlay bundle
  dump-target-overlay dump overlay info of the specific target bundle
  dump-dependencies dump dependencies by given bundle name and module name
  dump-shared dump inter-application shared library information by bundle name
```

**证据来源**: `bundle_command.h:28-43`

---

## 安装命令 (install)

### 用法

```bash
bm install [-h] [-p filePath] [-r] [-w waitingTime] [-s hspDirPath] [-u userId] [-d]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-p, --bundle-path <file-path>` | 路径 | **是** | HAP/HSP/APP 路径 |
| `-r, --replace` | - | 否 | 覆盖安装（默认） |
| `-w, --waitting-time <time>` | 数值 | 否 | 等待时间 (180-600s，默认 180s) |
| `-s, --shared-bundle-dir-path <path>` | 路径 | 条件 | HSP 路径 |
| `-u, --user-id <user-id>` | 数值 | 否 | 用户 ID |
| `-d, --downgrade` | - | 否 | 允许降级安装 |

### 使用限制

1. `-p` 每次只能安装一个 APP
2. `-s` 每个目录只能存在一个同包名的 HSP
3. `-u` 仅支持当前活跃用户或 0
4. `-d` 仅支持三方应用降级

### 示例

```bash
# 安装单个 HAP
bm install -p /data/local/tmp/ohos.app.hap

# 指定用户安装
bm install -p /data/local/tmp/ohos.app.hap -u 100

# 覆盖安装
bm install -p /data/local/tmp/ohos.app.hap -r

# 安装 HSP
bm install -s xxx.hsp

# 同时安装 HAP 和 HSP
bm install -p aaa.hap -s xxx.hsp yyy.hsp

# 指定等待时间
bm install -p /data/local/tmp/ohos.app.hap -w 300

# 降级安装
bm install -p /data/local/tmp/ohos.app.hap -d
```

### 调用链

```
bm install
  └── RunAsInstallCommand()
        ├── GetBundlePath() - 解析路径
        └── InstallOperation()
              └── IBundleInstaller::Install()
                    └── IPC → BundleManagerService
```

**证据来源**: `bundle_command.h:65-82`, `bundle_command.cpp`

---

## 卸载命令 (uninstall)

### 用法

```bash
bm uninstall [-h] [-n bundleName] [-m moduleName] [-k] [-s] [-v versionCode] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-n, --bundle-name <name>` | 字符串 | **是** | 包名 |
| `-m, --module-name <name>` | 字符串 | 否 | 模块名 |
| `-k, --keep-data` | - | 否 | 保留用户数据 |
| `-s, --shared` | - | 条件 | 卸载 HSP |
| `-v, --version <code>` | 数值 | 否 | HSP 版本号 |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 示例

```bash
# 卸载应用
bm uninstall -n com.ohos.app

# 指定用户卸载
bm uninstall -n com.ohos.app -u 100

# 卸载单个模块
bm uninstall -n com.ohos.app -m entry

# 卸载 HSP
bm uninstall -n com.ohos.example -s

# 卸载指定版本 HSP
bm uninstall -n com.ohos.example -s -v 100001

# 卸载并保留数据
bm uninstall -n com.ohos.app -k
```

### 调用链

```
bm uninstall
  └── RunAsUninstallCommand()
        └── UninstallOperation()
              └── IBundleInstaller::Uninstall()
                    └── IPC → BundleManagerService
```

**证据来源**: `bundle_command.h:84-93`

---

## 查询命令 (dump)

### 用法

```bash
bm dump [-h] [-a] [-g] [-n bundleName] [-s shortcutInfo] [-d deviceId] [-l label] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-a, --all` | - | 否 | 查询所有已安装应用 |
| `-g, --debug-bundle` | - | 否 | 查询调试签名应用 |
| `-n, --bundle-name <name>` | 字符串 | 否 | 指定包名 |
| `-s, --shortcut-info` | - | 否 | 查询快捷方式 |
| `-d, --device-id <id>` | 字符串 | 否 | 指定设备 ID |
| `-l, --label` | - | 否 | 查询 label 值 |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 示例

```bash
# 显示所有已安装应用
bm dump -a

# 查询调试签名应用
bm dump -g

# 查询应用详细信息
bm dump -n com.ohos.app

# 指定用户查询
bm dump -n com.ohos.app -u 100

# 查询快捷方式
bm dump -s -n com.ohos.app

# 跨设备查询
bm dump -n com.ohos.app -d xxxxx

# 查询应用名称
bm dump -n com.ohos.app -l

# 显示所有应用名称
bm dump -a -l
```

### 调用链

```
bm dump
  └── RunAsDumpCommand()
        ├── DumpBundleList() - 全部列表
        ├── DumpBundleInfo() - 详细信息
        ├── DumpShortcutInfos() - 快捷方式
        └── DumpDistributedBundleInfo() - 跨设备
              └── IBundleMgr::GetBundleInfo()
                    └── IPC → BundleManagerService
```

**证据来源**: `bundle_command.h:102-112`

---

## 清理命令 (clean)

### 用法

```bash
bm clean [-h] [-c] [-n bundleName] [-d] [-i appIndex] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-c, --cache` | - | 条件 | 清理缓存（需配合 -n） |
| `-d, --data` | - | 条件 | 清理数据（需配合 -n） |
| `-n, --bundle-name <name>` | 字符串 | **是** | 包名 |
| `-i, --app-index <index>` | 数值 | 否 | 分身应用索引（默认 0） |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 权限说明

此命令在 **root 版本**下可用，在 **user 版本**下需打开开发者模式。

### 示例

```bash
# 清理缓存
bm clean -c -n com.ohos.app

# 指定用户清理缓存
bm clean -c -n com.ohos.app -u 100

# 清理数据
bm clean -d -n com.ohos.app

# 清理分身应用数据
bm clean -d -n com.ohos.app -i 1
```

### 调用链

```
bm clean
  ├── CleanBundleCacheFilesOperation()
  │      └── IBundleMgr::CleanBundleCacheFiles()
  │            └── IPC → BundleManagerService
  └── CleanBundleDataFilesOperation()
         └── IBundleMgr::CleanBundleDataFiles()
               └── IPC → BundleManagerService
```

**证据来源**: `bundle_command.h:114-122`

---

## 使能命令 (enable)

### 用法

```bash
bm enable [-h] [-n bundleName] [-a abilityName] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-n, --bundle-name <name>` | 字符串 | **是** | 包名 |
| `-a, --ability-name <name>` | 字符串 | 否 | 能力名 |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 权限说明

此命令在 **root 版本**下可用，在 **user 版本**下不可用。

### 示例

```bash
# 使能应用
bm enable -n com.ohos.app

# 指定用户使能
bm enable -n com.ohos.app -u 100

# 使能指定能力
bm enable -n com.ohos.app -a com.ohos.app.EntryAbility
```

### 输出

```
enable bundle successfully.
```

**证据来源**: `bundle_command.h:124-130`

---

## 禁用命令 (disable)

### 用法

```bash
bm disable [-h] [-n bundleName] [-a abilityName] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-n, --bundle-name <name>` | 字符串 | **是** | 包名 |
| `-a, --ability-name <name>` | 字符串 | 否 | 能力名 |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 权限说明

此命令在 **root 版本**下可用。

### 示例

```bash
# 禁用应用
bm disable -n com.ohos.app

# 指定用户禁用
bm disable -n com.ohos.app -u 100

# 禁用指定能力
bm disable -n com.ohos.app -a com.ohos.app.EntryAbility
```

### 输出

```
disable bundle successfully.
```

**证据来源**: `bundle_command.h:132-138`

---

## 获取 UDID 命令 (get)

### 用法

```bash
bm get [-h] [-u]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-u, --udid` | - | **是** | 获取 UDID |

### 示例

```bash
# 获取设备 UDID
bm get -u

# 输出
udid of current device is:
23CADE0C
```

### 调用链

```
bm get -u
  └── RunAsGetCommand()
        └── GetUdid()
              └── IPC → 系统服务
```

**证据来源**: `bundle_command.h:140-144`

---

## 快速修复命令 (quickfix)

### 用法

```bash
bm quickfix [-h] [-a -f filePath [-t targetPath] [-d] [-o]] [-q -b bundleName] [-r -b bundleName]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-a, --apply` | - | 条件 | 应用快速修复 |
| `-f, --file-path <path>` | 路径 | 条件 | HQF 文件路径 |
| `-t, --target <path>` | 路径 | 否 | 目标路径 |
| `-d, --debug` | - | 否 | 调试模式 |
| `-o, --overwrite` | - | 否 | 覆盖模式 |
| `-q, --query` | - | 条件 | 查询快速修复 |
| `-r, --remove` | - | 条件 | 卸载快速修复 |
| `-b, --bundle-name <name>` | 字符串 | 条件 | 包名 |

### 示例

```bash
# 查询补丁信息
bm quickfix -q -b com.ohos.app

# 应用快速修复
bm quickfix -a -f /data/app/

# 卸载快速修复
bm quickfix -r -b com.ohos.app

# 调试模式应用
bm quickfix -a -f /data/app/ -d
```

### 输出

```
Information as follows:
ApplicationQuickFixInfo:
  bundle name: com.ohos.app
  bundle version code: xxx
  bundle version name: xxx
  patch version code: x
  ...
```

**证据来源**: `bundle_command.h:146-159`

---

## 编译命令 (compile)

### 用法

```bash
bm compile [-h] [-m mode] [-r bundleName] [-a]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-m, --mode <mode>` | 字符串 | 否 | 编译模式 (partial/full) |
| `-r, --reset <name>` | 字符串 | 否 | 重置编译结果 |
| `-a, --all` | - | 否 | 编译/重置所有 |

### 示例

```bash
# 按包名编译
bm compile -m partial com.example.myapplication

# 编译所有
bm compile -m full -a
```

**证据来源**: `bundle_command.h:50-56`

---

## 拷贝 AP 命令 (copy-ap)

### 用法

```bash
bm copy-ap [-h] [-a] [-n bundleName]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-a, --all` | - | 否 | 拷贝所有 |
| `-n, --bundle-name <name>` | 字符串 | 否 | 指定包名 |

### 功能

拷贝 AP 文件到 `/data/local/pgo` 目录。

### 示例

```bash
# 拷贝指定包
bm copy-ap -n com.example.myapplication
```

**证据来源**: `bundle_command.h:58-63`

---

## 查询 Overlay 命令 (dump-overlay)

### 用法

```bash
bm dump-overlay [-h] [-b bundleName] [-m moduleName] [-t targetModuleName] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-b, --bundle-name <name>` | 字符串 | **是** | Overlay 包名 |
| `-m, --module-name <name>` | 字符串 | 否 | Overlay 模块名 |
| `-t, --target-module-name <name>` | 字符串 | 否 | 目标模块名 |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 示例

```bash
# 查询 Overlay 信息
bm dump-overlay -b com.ohos.app

# 指定模块查询
bm dump-overlay -b com.ohos.app -m libraryModuleName

# 指定用户查询
bm dump-overlay -b com.ohos.app -u 100
```

**证据来源**: `bundle_command.h:161-169`

---

## 查询目标 Overlay 命令 (dump-target-overlay)

### 用法

```bash
bm dump-target-overlay [-h] [-b bundleName] [-m moduleName] [-u userId]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-b, --bundle-name <name>` | 字符串 | **是** | 目标包名 |
| `-m, --module-name <name>` | 字符串 | 否 | 目标模块名 |
| `-u, --user-id <id>` | 数值 | 否 | 用户 ID |

### 示例

```bash
# 查询目标应用的所有关联 Overlay
bm dump-target-overlay -b com.ohos.app

# 指定用户查询
bm dump-target-overlay -b com.ohos.app -u 100

# 指定模块查询
bm dump-target-overlay -b com.ohos.app -m entry
```

**证据来源**: `bundle_command.h:171-178`

---

## 查询 HSP 命令 (dump-shared)

### 用法

```bash
bm dump-shared [-h] [-a] [-n bundleName]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-a, --all` | - | 否 | 查询所有 HSP |
| `-n, --bundle-name <name>` | 字符串 | 否 | 指定包名 |

### 示例

```bash
# 显示所有 HSP
bm dump-shared -a

# 查询指定 HSP
bm dump-shared -n com.ohos.lib
```

**证据来源**: `bundle_command.h:180-186`

---

## 查询依赖命令 (dump-dependencies)

### 用法

```bash
bm dump-dependencies [-h] [-n bundleName] [-m moduleName]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-n, --bundle-name <name>` | 字符串 | **是** | 包名 |
| `-m, --module-name <name>` | 字符串 | 否 | 模块名 |

### 示例

```bash
# 查询依赖
bm dump-dependencies -n com.ohos.app -m entry
```

**证据来源**: `bundle_command.h:188-194`

---

## 安装插件命令 (install-plugin)

### 用法

```bash
bm install-plugin [-h] [-n hostBundleName] [-p filePath]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-n, --host-bundle-name <name>` | 字符串 | **是** | 宿主应用包名 |
| `-p, --plugin-path <path>` | 路径 | **是** | 插件 HSP 路径 |

### 限制

1. 不支持降级安装
2. 不推荐安装与宿主同名的插件

### 示例

```bash
# 安装插件
bm install-plugin -n com.ohos.app -p /data/plugin.hsp
```

**证据来源**: `bundle_command.h:195-199`

---

## 卸载插件命令 (uninstall-plugin)

### 用法

```bash
bm uninstall-plugin [-h] [-n hostBundleName] [-p pluginBundleName]
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `-h, --help` | - | 否 | 显示帮助 |
| `-n, --host-bundle-name <name>` | 字符串 | **是** | 宿主应用包名 |
| `-p, --plugin-bundle-name <name>` | 字符串 | **是** | 插件包名 |

### 示例

```bash
# 卸载插件
bm uninstall-plugin -n com.ohos.app -p com.ohos.plugin
```

**证据来源**: `bundle_command.h:201-205`

---

## 错误码说明

### 通用错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败（见具体消息） |

### 常见错误消息

| 消息 | 说明 |
|------|------|
| `install bundle successfully.` | 安装成功 |
| `error: failed to install bundle.` | 安装失败 |
| `uninstall bundle successfully.` | 卸载成功 |
| `error: failed to uninstall bundle.` | 卸载失败 |
| `enable bundle successfully.` | 使能成功 |
| `disable bundle successfully.` | 禁用成功 |
| `clean bundle cache files successfully.` | 清理缓存成功 |
| `clean bundle data files successfully.` | 清理数据成功 |

**证据来源**: `bundle_command.h:222-243`

---

## 相关文档

- [01_Architecture.md](./01_Architecture.md) - 架构设计
- [06_Troubleshooting.md](./06_Troubleshooting.md) - 问题定位
