# 使用指南

## 快速开始

### 环境要求

| 依赖 | 版本要求 | 说明 |
|------|----------|------|
| Python | 3.x | 运行 download_iana.py |
| Bash | 4.0+ | 运行 compile.sh |
| Make | 3.0+ | 编译 zic 工具 |
| GCC/Clang | 支持 C 编译 | 编译 zic 源码 |

### 前置条件

1. 确保工作目录在仓库根目录
2. 确保 `data/prebuild/posix/` 目录存在
3. 确保 `data/iana/` 目录可写

## 时区数据更新流程

### 步骤 1: 切换到更新工具目录

```bash
cd tool/update_tool
```

**证据**: `tool/update_tool/download_iana.py:92-93`

```python
file_path = os.path.abspath(__file__)
file_dir = os.path.dirname(file_path)
```

### 步骤 2: 执行下载脚本

```bash
python3 download_iana.py
```

**或**

```bash
chmod 755 download_iana.py
./download_iana.py
```

### 步骤 3: 验证下载结果

```bash
ls -la ../../data/iana/
```

成功输出示例:
```
tzdata2023c.tar.gz
tzcode2023c.tar.gz
version.txt
africa/          europe/
asia/            ...
```

**证据**: `tool/update_tool/download_iana.py:104-111`

```python
file_name = download(file_type, download_path, version)
if file_name != '':
    decompress(file_name, download_path)
    file_type = "tzcode"
    new_version = file_name[6:11]
    try_download(file_type, new_version, download_path, version)
    decompress(file_type + new_version + '.tar.gz', download_path)
```

## 时区数据编译流程

### 步骤 1: 切换到编译工具目录

```bash
cd tool/compile_tool
```

**证据**: `tool/compile_tool/compile.sh:15-18`

```bash
script_path=$(cd $(dirname $0);pwd)
iana_path="${script_path}/../../data/iana"
posix_path="${iana_path}/../prebuild/posix"
zic_path="${iana_path}/../prebuild/tool/linux"
```

### 步骤 2: 执行编译脚本

```bash
chmod 755 compile.sh
./compile.sh
```

### 步骤 3: 验证编译结果

```bash
ls -la ../../data/prebuild/posix/
```

成功输出示例:
```
version.txt
Africa/      Europe/      Australia/
America/     Pacific/      Etc/
Arctic/      Atlantic/     Indian/
Antarctica/  SystemV/      leapseconds
iso3166.tab  zone.tab     zone1970.tab
```

**证据**: `tool/compile_tool/compile.sh:29-31`

```bash
rm -rf ${posix_path}/*
mv ${iana_path}/zoneinfo/* ${posix_path}
mv ${iana_path}/version.txt ${posix_path}
```

## 端到端操作示例

### 完整更新流程

```bash
# 1. 下载最新时区数据
cd tool/update_tool
python3 download_iana.py

# 2. 编译时区数据
cd ../compile_tool
chmod 755 compile.sh
./compile.sh

# 3. 验证输出
ls -la ../../data/prebuild/posix/
```

**预期输出**:
```
start to find the lastest version of tzdata and tzcode.
start to download tzdata2023c.tar.gz
download finished!
decompress finished!
start to download tzcode2023c.tar.gz
download finished!
decompress finished!
done
```

### 版本检查

```bash
cat data/prebuild/posix/version.txt
```

输出示例:
```
2023c
```

## 常见问题与解决方案

### 问题 1: 下载失败 - 网络超时

**错误信息**:
```
URLError: <urlopen error timed out>
```

**解决方案**:
1. 检查网络连接
2. 配置代理 (如需要):
   ```bash
   export http_proxy=http://proxy:port
   export https_proxy=http://proxy:port
   ```
3. 手动下载:
   - 访问 https://data.iana.org/time-zones/releases/
   - 下载 `tzdataYYYYx.tar.gz` 和 `tzcodeYYYYx.tar.gz`
   - 解压到 `data/iana/` 目录

---

### 问题 2: 编译失败 - zic 工具缺失

**错误信息**:
```
make: *** No targets specified and no makefile found
```

**解决方案**:
1. 确保已执行下载脚本
2. 检查 zic 源码是否存在:
   ```bash
   ls -la data/iana/
   ```
3. 手动编译 zic:
   ```bash
   cd data/iana
   make
   mv zic ../prebuild/tool/linux/
   ```

---

### 问题 3: 版本号不一致

**错误信息**:
```
current version is 2023a
trying version 2023b... success
but version.txt still shows old version
```

**解决方案**:
1. 检查版本文件权限
2. 手动更新版本号:
   ```bash
   echo "2023b" > data/prebuild/posix/version.txt
   ```

---

### 问题 4: 权限错误

**错误信息**:
```
Permission denied: 'data/iana/version.txt'
```

**解决方案**:
1. 确保目录有写权限
   ```bash
   chmod -R u+w data/
   ```
2. 使用 sudo (谨慎):
   ```bash
   sudo ./compile.sh
   ```

---

### 问题 5: 磁盘空间不足

**错误信息**:
```
No space left on device
```

**解决方案**:
1. 清理临时文件
2. 检查磁盘空间:
   ```bash
   df -h
   ```
3. 清理 IANA 下载目录:
   ```bash
   rm -rf data/iana/tzdata*.tar.gz
   rm -rf data/iana/tzcode*.tar.gz
   ```

## 手动操作指南

### 手动下载 IANA 数据

1. 访问 IANA 官网: https://data.iana.org/time-zones/releases/
2. 下载最新的 `tzdataYYYYx.tar.gz` 和 `tzcodeYYYYx.tar.gz`
3. 解压到 `data/iana/` 目录:
   ```bash
   cd data/iana
   tar -xzf tzdata2023c.tar.gz
   tar -xzf tzcode2023c.tar.gz
   ```
4. 创建版本文件:
   ```bash
   echo "2023c" > version.txt
   ```

### 手动编译 zic

1. 进入 IANA 源码目录:
   ```bash
   cd data/iana
   ```
2. 编译 zic:
   ```bash
   make
   ```
3. 移动到工具目录:
   ```bash
   mv zic ../prebuild/tool/linux/
   ```

### 手动生成时区数据

```bash
cd data/iana
./zic -d zoneinfo africa antarctica asia australasia europe etcetera \
    northamerica southamerica backward
```

## 自动化脚本示例

### 定时更新脚本

```bash
#!/bin/bash
# update_timezone.sh - 自动更新时区数据

set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$SCRIPT_DIR"

echo "[$(date)] Starting timezone update..."

# 下载
cd tool/update_tool
python3 download_iana.py

# 编译
cd ../compile_tool
./compile.sh

echo "[$(date)] Timezone update completed!"
```

### CI/CD 集成

```yaml
# .github/workflows/update-timezone.yml
name: Update Timezone

on:
  schedule:
    - cron: '0 0 1 * *'  # 每月1日执行
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      - name: Download IANA data
        run: python3 tool/update_tool/download_iana.py
      - name: Compile timezone data
        run: chmod 755 tool/compile_tool/compile.sh && ./compile.sh
      - name: Create Pull Request
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git checkout -b update/timezone-$(date +%Y%m%d)
          git add data/
          git commit -m "Update timezone data $(date +%Y-%m-%d)"
          git push -u origin update/timezone-$(date +%Y%m%d)
```

## 调试技巧

### 启用详细日志

```bash
# 在 Python 脚本中添加调试输出
python3 -u download_iana.py  # -u 禁用输出缓冲
```

### 调试网络请求

```python
# 在 download_iana.py 中添加
import http.client
http.client.HTTPConnection.debuglevel = 1
```

### 验证时区数据

```bash
# 检查时区文件格式
zdump -v America/New_York

# 列出所有可用时区
zic -l
```

## 相关文档

- [架构说明](./01_Architecture.md)
- [构建指南](./02_Build.md)
- [安全评审](./03_Security.md)
