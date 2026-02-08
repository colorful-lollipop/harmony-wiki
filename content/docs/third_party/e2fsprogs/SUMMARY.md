# 阅读路线建议 (SUMMARY.md)

## 推荐阅读路径

### 路径一：快速了解 (15 分钟)
适合：项目经理、产品经理、初步了解

1. **[README.md](./README.md)** - 获取整体概况
2. **[01_Overview.md](./01_Overview.md)** - 了解 OH 中的定位
3. **[02_Patches.md#Patch 清单表](./02_Patches.md)** - 浏览 Patch 列表

### 路径二：开发者指南 (45 分钟)
适合：开发工程师、需要集成或修改代码

1. **[01_Overview.md](./01_Overview.md)** - 了解基本概念
2. **[02_Patches.md](./02_Patches.md)** - **仔细阅读 Patch 分析**
   - 特别关注 1003 (DAC 配置) 和 1001 (镜像制作)
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - 理解构建系统
   - BUILD.gn 结构
   - 编译选项
   - 依赖关系
4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 了解使用场景

### 路径三：深度分析 (90 分钟)
适合：架构师、维护工程师、需要全面掌握

1. **所有文档按顺序阅读**
2. **[02_Patches.md](./02_Patches.md)** - 每个 Patch 都要理解
3. **[06_Security.md](./06_Security.md)** - 安全风险评估
4. **阅读原始 Patch 文件** - 在 `../` 目录下

### 路径四：特定任务

#### 任务 A：升级上游版本
1. [02_Patches.md#升级建议](./02_Patches.md#升级建议)
2. [03_Build_Integration.md#与上游差异](./03_Build_Integration.md#与上游差异)
3. 检查每个 Patch 是否需要重新适配

#### 任务 B：添加新文件系统支持
1. [02_Patches.md#Patch-1006-add-hmfs-for-blkid](./02_Patches.md#Patch-1006-add-hmfs-for-blkid)
2. [03_Build_Integration.md#libext2_blkid](./03_Build_Integration.md#libext2_blkid)
3. 参考 HMFS 的实现方式

#### 任务 C：修改镜像制作流程
1. [02_Patches.md#Patch-1001-image-make](./02_Patches.md#Patch-1001-image-make)
2. [02_Patches.md#Patch-1003-add-dac-config](./02_Patches.md#Patch-1003-add-dac-config)
3. [04_Usage_in_OH.md#镜像制作流程](./04_Usage_in_OH.md#镜像制作流程)

#### 任务 D：排查 blkid 问题
1. [02_Patches.md#Patch-1005-read-vfat-chinese-label](./02_Patches.md#Patch-1005-read-vfat-chinese-label)
2. [02_Patches.md#Patch-1007-blkid-support-skip-specified-filesystem](./02_Patches.md#Patch-1007-blkid-support-skip-specified-filesystem)
3. [02_Patches.md#Patch-1008-blkid-enlarge-cluster-for-ntfs](./02_Patches.md#Patch-1008-blkid-enlarge-cluster-for-ntfs)

## 文档依赖关系

```
README.md
    ├── 01_Overview.md ──────┐
    │                        │
    ├── 02_Patches.md ───────┼── 04_Usage_in_OH.md
    │                        │
    ├── 03_Build_Integration ┘
    │
    ├── 05_API_Differences.md (可选)
    │
    └── 06_Security.md (可选)
```

## 重点章节速查

| 主题 | 相关文档 | 章节 |
|------|---------|------|
| DAC 配置机制 | 02_Patches.md | Patch 1003 |
| 镜像制作流程 | 02_Patches.md | Patch 1001 |
| 中文卷标支持 | 02_Patches.md | Patch 1005 |
| BUILD.gn 结构 | 03_Build_Integration.md | 完整文档 |
| 依赖关系 | 04_Usage_in_OH.md | 依赖关系与使用 |
| 安全 CVE | 06_Security.md | 安全风险分析 |

## 附录

### 术语表

| 术语 | 说明 |
|------|------|
| DAC | Discretionary Access Control，自主访问控制 |
| HMFS | Huawei Mobile File System，华为移动文件系统 |
| OTA | Over-The-Air，空中升级 |
| SELinux | Security-Enhanced Linux，安全增强型 Linux |
| ext4 | Fourth Extended Filesystem，第四代扩展文件系统 |

### 相关路径

| 路径 | 说明 |
|------|------|
| `//third_party/e2fsprogs` | 源码根目录 |
| `//third_party/e2fsprogs/BUILD.gn` | 构建配置 |
| `//third_party/e2fsprogs/*.patch` | Patch 文件 |
| `//third_party/e2fsprogs/install.sh` | 安装脚本 |
