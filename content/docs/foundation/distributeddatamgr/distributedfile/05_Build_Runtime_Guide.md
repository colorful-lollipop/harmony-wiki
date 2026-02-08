# 构建与运行时指南

> ⚠️ 本文档待代码仓库完整后补充

## 构建环境要求

### 系统要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux (Ubuntu 20.04+) |
| 内存 | 16GB+ |
| 磁盘 | 100GB+ |
| Python | 3.8+ |
| Git | 2.28+ |

### 依赖工具

```bash
# Ubuntu/Debian
sudo apt-get install build-essential python3 git-lfs

# 安装 hb (HarmonyOS Build)
pip3 install ohos-build
```

## 获取源码

```bash
# 克隆代码仓库
git clone https://gitee.com/openharmony/distributeddatamgr_file.git

# 初始化子模块（如果有）
cd distributeddatamgr_file
git submodule update --init --recursive
```

## 构建命令

### 完整构建

```bash
# 设置构建环境
source build.sh

# 执行构建
./build.sh --product-name <product_name> --target-cpu arm64
```

### 模块构建

```bash
# 仅构建 distributedfile 子系统
hb build -p distributeddatamgr

# 仅构建特定 target
hb build -p distributeddatamgr -T //foundation/distributeddatamgr/distributedfile:fileio_napi
```

### GN 直接构建

```bash
# 生成构建配置
gn gen out/default --args="target_os=\"ohos\" target_cpu=\"arm64\""

# 构建
ninja -C out/default //foundation/distributeddatamgr/distributedfile:fileio_napi
```

## 编译产物

| 产物 | 路径 | 说明 |
|------|------|------|
| `libfileio_napi.z.so` | `out/.../system/lib/` | N-API 动态库 |
| `libn.z.a` | `out/.../system/lib/` | 静态库 |
| `libhilog.z.so` | `out/.../system/lib/` | 日志库 |

## 安装路径

| 路径 | 说明 |
|------|------|
| `/system/lib/libfileio_napi.z.so` | 系统库 |
| `/system/lib64/libn.z.a` | 静态库（64位） |
| `/data/app/<bundle_name>/` | 应用沙箱目录 |

## 运行时加载

### 应用沙箱

```
/data/app/{bundle_name}/
├── cache/              # internal://cache/
├── files/              # internal://app/
│   ├── share/          # internal://share/ (可被其他应用访问)
│   └── ...
└── preferences/
```

### URI 映射

| URI 前缀 | 映射路径 |
|----------|----------|
| `internal://cache/` | `/data/app/{bundle}/cache/` |
| `internal://app/` | `/data/app/{bundle}/files/` |
| `internal://share/` | `/data/app/{bundle}/files/share/` |

## 调试方法

### 日志查看

```bash
# 查看 fileio 相关日志
hilog | grep -i "fileio\|distributedfile"

# 实时日志
hilog -w &
```

### 调试技巧

#### 1. 断点调试

```bash
# 使用 hdc 调试
hdc shell
gdbserver64 :5000 --attach <pid>
```

#### 2. 跟踪系统调用

```bash
# 跟踪文件操作
strace -f -e open,close,read,write -p <pid>
```

#### 3. 内存检查

```bash
# 使用 valgrind 检测内存问题
valgrind --leak-check=full ./test_fileio
```

### 常见问题（待补充）

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| TODO | 待代码补充 | 待代码补充 |

## 测试

### 运行单元测试

```bash
# 进入测试目录
cd test

# 运行测试
./run_tests.sh

# 或使用 pytest
python3 -m pytest test/
```

### API 测试

```javascript
// test/fileio.test.js
import fileio from '@OHOS.distributedfile.fileio';

describe('fileio', () => {
    it('should create file', () => {
        let fd = fileio.openSync('/data/test.txt', 'w');
        expect(fd).not.toBeNull();
        fileio.closeSync(fd);
    });
});
```

## 参考

- [项目概览](00_Overview.md)
- [N-API 文档](01_APIs.md)
- [构建系统](03_Build_System.md)
- [OpenHarmony 构建指南](https://docs.openharmony.cn)
- [hb 工具文档](https://gitee.com/openharmony/build/blob/master/docs/quick_start/Readme-CN.md)
