# 应用文件服务 (AppFileService) Wiki

## 文档说明

| 项目 | 内容 |
|------|------|
| **覆盖范围** | `foundation/filemanagement/app_file_service` 完整模块 |
| **生成时间** | 2026-02-06 |
| **更新方式** | 随代码变更手动更新 |
| **适用读者** | OpenHarmony 开发者、系统架构师、安全评审人员 |

## 项目定位

应用文件服务是为 OpenHarmony 设备提供**应用间文件分享**和**数据备份恢复**能力的核心服务模块。

### 核心能力

1. **文件分享 (FileShare)**：应用间 URI 授权和持久化
2. **URI 管理 (FileURI)**：文件路径与 URI 的相互转换
3. **备份恢复 (Backup)**：完整备份、增量备份、应用数据恢复

### System Capabilities

```
SystemCapability.FileManagement.AppFileService
SystemCapability.FileManagement.StorageService.Backup
SystemCapability.FileManagement.AppFileService.FolderAuthorization
```

### 依赖子系统

- `ability_runtime`：Ability 框架和扩展能力
- `bundle_framework`：应用包管理
- `safwk/samgr`：系统能力框架
- `ipc`：进程间通信
- `storage_service`：存储管理
- `access_token`：权限管理

## 目录结构

```
app_file_service/
├── interfaces/           # 接口层
│   ├── api/             # JS N-API 内部实现
│   ├── inner_api/       # 对内内部接口
│   ├── innerkits/       # 对内 Native 接口
│   └── kits/            # 对外接口 (JS/NDK/CJ/泰河)
├── frameworks/          # 框架层
│   ├── js/              # JS 框架
│   └── native/          # Native 框架
├── services/            # 服务层
│   └── backup_sa/       # 备份恢复服务 (SA)
├── utils/               # 工具库
├── tools/               # 工具
├── BUILD.gn             # 根构建配置
└── bundle.json          # 组件配置
```

## 阅读路径建议

1. **新人入门**：`00_Overview.md` → `01_Architecture.md` → `10_NAPI_JS.md`
2. **API 使用者**：`10_NAPI_JS.md` / `11_NAPI_NDK.md` → `20_Inner_API.md`
3. **系统开发者**：`01_Architecture.md` → `02_Service_SA.md` → `03_Utils.md`
4. **构建工程师**：`04_GN_Build.md` → 产物映射表
5. **安全评审**：`05_Security_Review.md`

## 关键入口

| 模块 | 入口文件 | 说明 |
|------|----------|------|
| FileShare JS | `interfaces/kits/js/file_share/fileshare_n_exporter.cpp:226` | NAPI_MODULE 注册 |
| FileURI JS | `interfaces/kits/js/file_uri/module.cpp:49` | napi_module_register |
| Backup JS | `interfaces/kits/js/backup/module.cpp:49` | napi_module_register |
| Backup SA | `services/backup_sa/src/module_ipc/service.cpp:75` | SA 注册 |

## 文档贡献

欢迎提交 PR 完善文档：
1. 克隆仓库
2. 在 `wiki/` 目录修改或新增 `.md` 文件
3. 确保链接在 `SUMMARY.md` 中注册
4. 运行一致性检查
