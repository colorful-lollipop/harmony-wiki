# 调用链附录

## 目的

本文档记录 Updater 子系统的关键调用链，便于理解代码执行流程。

## 适用范围

- 子系统开发者
- 调试工程师

## OTA 升级调用链

### 1. 触发升级流程

```
应用调用
  ↓
RebootAndInstallUpgradePackage() [interfaces/kits/updaterkits/updaterkits.cpp]
  ↓
WriteUpdaterMiscMsg() [interfaces/kits/misc_info/misc_info.cpp]
  ↓
reboot("updater") [系统调用]
```

### 2. Updater 启动流程

```
Bootloader 加载 Kernel + Updater.img
  ↓
Kernel 启动 init
  ↓
init 读取 init.cfg
  ↓
init 启动 updater 进程 [services/main.cpp:main()]
  ↓
ReadUpdaterMiscMsg() [interfaces/kits/misc_info/misc_info.cpp]
  ↓
IsUpdater(msg) / IsFlashd(msg)
  ↓
UpdaterMain() [services/updater_main.cpp]
  ↓
  ├─ InitUI() [services/ui/]
  ├─ LoadPackage() [services/package/]
  └─ StartUpdaterProc() [services/updater.cpp]
```

### 3. 升级包加载流程

```
StartUpdaterProc()
  ↓
PkgManager::LoadPackage() [services/package/pkg_manager/pkg_manager_impl.cpp]
  ↓
ZipPkgFile::LoadPackage() [services/package/pkg_package/pkg_zipfile.cpp]
  ↓
  ├─ 解析 ZIP 结构
  ├─ HashDataVerifier::Verify() [services/package/pkg_verify/hash_data_verifier.cpp]
  │   ↓
  │   PkgVerifyUtil::GetSignature() [services/package/pkg_verify/pkg_verify_util.cpp]
  │   ↓
  │   Pkcs7SignedData::ParsePkcs7Data() [services/package/pkg_verify/pkcs7_signed_data.cpp]
  │   ↓
  │   CertVerify::Verify() [services/package/pkg_verify/cert_verify.cpp]
  │   ↓
  │   OpensslUtil::VerifyDigestByPubKey() [services/package/pkg_verify/openssl_util.cpp]
  │
  └─ 返回文件列表
```

### 4. 脚本执行流程

```
StartUpdaterProc()
  ↓
ExtractUpdaterBinary() [services/updater.cpp]
  ↓
fork() + execl() 启动 updater_binary
  ↓
updater_binary 主进程 [services/updater_binary/main.cpp]
  ↓
UpdateProcessor::Process() [services/updater_binary/update_processor.cpp]
  ↓
ScriptManager::ExecuteScript() [services/script/script_manager/script_manager_impl.cpp]
  ↓
ScriptInterpreter::Parse() [services/script/script_interpreter/script_interpreter.cpp]
  ↓
  ├─ mount 指令 → MountInstruction::Execute() → mount() 系统调用
  ├─ format 指令 → FormatInstruction::Execute() → FormatPartition()
  ├─ write_raw_image 指令 → WriteImageInstruction::Execute() → 写入分区
  └─ apply_patch 指令 → ApplyPatchInstruction::Execute() → ApplyPatch()
```

### 5. 差分补丁应用流程

```
apply_patch 指令
  ↓
ApplyPatch() [services/applypatch/apply_patch.cpp]
  ↓
TransferManager::ParseTransfers() [services/applypatch/transfer_manager.cpp]
  ↓
  ├─ 读取 transfer.list
  └─ 解析指令序列
  ↓
TransferManager::ExecuteTransfers()
  ↓
  ├─ new 指令 → 直接写入新数据
  ├─ erase 指令 → 擦除块
  └─ diff 指令 → 应用差分
  ↓
BlockWriter::Write() [services/applypatch/block_writer.cpp]
  ↓
写入目标分区
```

## AB 分区切换调用链

```
升级完成
  ↓
PostUpdater(clearMisc = true)
  ↓
SetActiveSlot() [interfaces/kits/slot_info/slot_info.cpp]
  ↓
  ├─ 调用 HAL 接口 (若 AB 支持)
  └─ 切换活跃槽位
  ↓
ClearMisc() [services/updater.cpp]
  ↓
WriteUpdaterMiscMsg() (清空命令)
  ↓
reboot() [系统调用]
```

## Flashd 调用链

```
HDC 客户端
  ↓
HDC 服务端
  ↓
FlashdDaemon [services/flashd/daemon/daemon_updater.cpp]
  ↓
CommanderFactory::CreateCommander()
  ↓
  ├─ format → FormatCommander::Execute() → FormatPartition()
  ├─ erase → EraseCommander::Execute() → WipePartition()
  ├─ flash → FlashCommander::Execute() → WriteImage()
  └─ update → UpdateCommander::Execute() → 触发升级流程
```

## 日志系统调用链

```
业务代码调用 LOGI/LOGE
  ↓
LogMessage() [services/log/log.cpp]
  ↓
  ├─ 写入本地文件
  └─ HiLog::Info()/HiLog::Error() [services/log/updater_hilog.cpp]
      ↓
      Hilog 系统服务

崩溃时:
UPDATER_LAST_WORD()
  ↓
UpdaterLastWord() [utils/utils.cpp]
  ↓
写入 Misc 分区 faultinfo 字段
```

## UI 更新调用链

```
脚本执行进度更新
  ↓
ScriptManager::SetProgress()
  ↓
UI::UpdateProgress() [services/ui/]
  ↓
ProgressStrategy::Update() [services/ui/strategy/progress_strategy.cpp]
  ↓
View::SetProgress() [services/ui/view/]
  ↓
GraphicDrv::Flush() [services/ui/driver/graphic_drv.cpp]
  ↓
DRM/Framebuffer 刷新
```

## 流式更新调用链 (AB Streaming)

```
下载服务
  ↓
BinChunkUpdate::WriteDataChunk() [services/stream_update/bin_chunk_update.cpp]
  ↓
  ├─ 缓冲数据块
  └─ 哈希验证
  ↓
验证通过 → 直接写入目标分区
  ↓
写入完成 → 记录进度到 Misc 分区 (支持断电续传)
```

## 相关跳转

- [内部接口文档](./04_Inner_API.md)
- [架构说明](./01_Architecture.md)
- [目录结构](./02_Directory_Structure.md)
