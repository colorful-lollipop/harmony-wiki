# zlib Wiki

## OpenHarmony zlib 适配文档

本 Wiki 详细记录 zlib 库在 OpenHarmony (OH) 中的集成与适配情况，重点关注 OH 特定的修改、Patch 分析和依赖关系。

---

## 文档导航

| 文档 | 内容 |
|-----|-----|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 中的定位 |
| [02_Patches.md](./02_Patches.md) | **Patch 详细分析**（核心文档）|
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | OH 依赖关系和使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

---

## 快速概览

### 库基本信息

```yaml
名称: zlib
版本: 1.3.1
许可证: zlib/libpng License
上游地址: https://github.com/madler/zlib

OH 组件名: @ohos/zlib
OH 版本: 3.1
所属子系统: thirdparty
```

### OH 适配概况

| 适配项 | 状态 | 说明 |
|-------|-----|-----|
| Patch 数量 | 1 个 | huawei_zlib_CMakeList.patch |
| Patch 类型 | 构建适配 | 禁用 MINGW 代码和测试二进制 |
| OH 源码修改 | 无 | 无 `#ifdef OHOS` 宏 |
| BUILD.gn | 完整 | 静态库 + 共享库 + CRC 优化库 |
| NDK 导出 | 94 个 API | 完整 zlib + MiniZip |

### 核心修改点

1. **构建系统适配**: 提供 BUILD.gn 替代 CMake，禁用测试代码
2. **ARM64 优化**: 启用 ARM CRC32 硬件加速 (`-march=armv8-a+crc`)
3. **MiniZip 集成**: 包含 contrib/minizip 提供 ZIP 文件处理能力
4. **多目标输出**: libz (静态)、shared_libz (动态)、libz_crc (CRC 专用)

---

## OH 中的典型使用场景

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                          │
├─────────────────────────────────────────────────────────────┤
│  ArkUI  ◄──►  HAP 包处理  ◄──►  图像处理  ◄──►  HTTP 通信    │
│    │            │              │              │            │
│    ▼            ▼              ▼              ▼            │
│  libz.so    minizip        libpng          curl          │
│    │            │              │              │            │
│    └────────────┴──────────────┴──────────────┘            │
│                              │                              │
│                         third_party/zlib                   │
└─────────────────────────────────────────────────────────────┘
```

1. **HAP 包解压**: Bundle Manager 使用 zlib 解压应用安装包
2. **图像解码**: Image Framework 通过 libpng 间接依赖 zlib
3. **网络传输**: curl 使用 zlib 支持 HTTP gzip 压缩
4. **数据压缩**: KV Store、Resource Manager 使用 zlib 压缩数据
5. **字节码归档**: ArkCompiler 使用 zlib 压缩字节码文件

---

## 维护与升级

### 当前状态
- ✅ 使用最新稳定版 1.3.1 (2024-01 发布)
- ✅ 无已知高危 CVE
- ✅ Patch 维护成本低

### 升级注意事项
1. Patch 文件需要重新应用 (prepare.sh)
2. API 向后兼容，无需修改依赖模块
3. 建议关注上游安全公告

---

## 更多信息

- 工作目录: [./_work/](./_work/)
- 评估报告: [./_work/ASSESSMENT.md](./_work/ASSESSMENT.md)
- 分析笔记: [./_work/NOTES.md](./_work/NOTES.md)
