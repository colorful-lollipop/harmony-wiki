# OH 构建适配

## BUILD.gn 文件结构

OpenHarmony 使用 GN (Generate Ninja) 构建系统替代了 iptables 原生的 autotools。整个库包含 4 个 BUILD.gn 文件：

```
third_party/iptables/
├── BUILD.gn              # 主构建文件 - 可执行文件
├── extensions/
│   └── BUILD.gn          # 扩展模块构建
├── libiptc/
│   └── BUILD.gn          # iptables 控制库构建
└── libxtables/
    └── BUILD.gn          # xtables 共享库构建
```

## 构建目标详解

### 1. 可执行文件 (BUILD.gn)

定义了 3 个主要的 iptables 可执行文件：

#### iptables
```gn
ohos_executable("iptables") {
  sources = [
    "//third_party/iptables/iptables/ip6tables-standalone.c",
    "//third_party/iptables/iptables/ip6tables.c",
    "//third_party/iptables/iptables/iptables-restore.c",
    "//third_party/iptables/iptables/iptables-save.c",
    "//third_party/iptables/iptables/iptables-standalone.c",
    "//third_party/iptables/iptables/iptables-xml.c",
    "//third_party/iptables/iptables/iptables.c",
    "//third_party/iptables/iptables/xshared.c",
    "//third_party/iptables/iptables/xtables-legacy-multi.c",
  ]
  # ...
  symlink_target_name = [ "ip6tables" ]
}
```

- **功能**: IPv4/IPv6 防火墙规则配置工具
- **源文件**: 9 个 C 源文件
- **Symlink**: 创建 `ip6tables` 软链接指向同一二进制文件

#### iptables-save
```gn
ohos_executable("iptables-save") {
  # 与 iptables 相同的 sources
  symlink_target_name = [ "ip6tables-save" ]
}
```

- **功能**: 保存当前 iptables 规则到文件
- **Symlink**: 创建 `ip6tables-save` 软链接

#### iptables-restore
```gn
ohos_executable("iptables-restore") {
  # 与 iptables 相同的 sources
  symlink_target_name = [ "ip6tables-restore" ]
}
```

- **功能**: 从文件恢复 iptables 规则
- **Symlink**: 创建 `ip6tables-restore` 软链接

**设计说明**: 三个可执行文件使用相同的源文件集合，通过不同的入口点（通过 argv[0] 判断）实现不同功能。这是 iptables 上游的标准做法（xtables-multi）。

### 2. 静态库构建

#### libip4tc / libip6tc (libiptc/BUILD.gn)

```gn
ohos_static_library("libip4tc") {
  sources = [ "//third_party/iptables/libiptc/libip4tc.c" ]
  # ...
  part_name = "netmanager_base"
  subsystem_name = "communication"
}

ohos_static_library("libip6tc") {
  sources = [ "//third_party/iptables/libiptc/libip6tc.c" ]
  # ...
  part_name = "netmanager_base"
  subsystem_name = "communication"
}
```

- **功能**: 提供与内核 netfilter 通信的底层接口
- **归属**: netmanager_base part，communication 子系统

#### libxtables (libxtables/BUILD.gn)

```gn
ohos_static_library("libxtables") {
  sources = [
    "//third_party/iptables/libxtables/xtables.c",
    "//third_party/iptables/libxtables/xtoptions.c",
  ]
  # ...
  part_name = "netmanager_base"
  subsystem_name = "communication"
}
```

- **功能**: xtables 共享库，提供扩展模块支持
- **归属**: netmanager_base part，communication 子系统

#### libext / libext4 / libext6 (extensions/BUILD.gn)

三个静态库分别对应不同类型的扩展模块：

| 库名 | 源文件前缀 | 说明 |
|-----|-----------|------|
| libext | libxt_*.c | 通用扩展（IPv4/IPv6 共用） |
| libext4 | libipt_*.c | IPv4 特有扩展 |
| libext6 | libip6t_*.c | IPv6 特有扩展 |

**扩展模块数量**:
- libext: 66 个通用扩展
- libext4: 9 个 IPv4 扩展
- libext6: 13 个 IPv6 扩展

## 关键编译选项

### 通用 cflags (所有目标)

```gn
cflags = [
  # 大文件支持
  "-D_LARGEFILE_SOURCE=1",
  "-D_LARGE_FILES",
  "-D_FILE_OFFSET_BITS=64",
  
  # 线程安全
  "-D_REENTRANT",
  
  # 协议支持
  "-DENABLE_IPV4",
  "-DENABLE_IPV6",
  
  # 静态链接配置
  "-DNO_SHARED_LIBS=1",        # 禁用共享库
  "-DXTABLES_INTERNAL",         # 内部 xtables API
  
  # xtables 库目录（禁用动态加载）
  "-DXTABLES_LIBDIR=\"xtables_libdir_not_used\"",
  
  # 警告控制
  "-Wall",
  "-Wno-error",                 # 不将警告视为错误
  "-Wno-pointer-arith",         # 允许指针算术
  "-Wno-sign-compare",          # 允许有符号/无符号比较
  "-Wno-unused-parameter",      # 允许未使用参数
  "-Wno-missing-field-initializers",
  "-Wno-parentheses-equality",
]
```

### OH 特有配置详解

#### 1. 静态链接模式 (`-DNO_SHARED_LIBS=1`)

**上游默认**: 支持共享库 (.so)，扩展模块动态加载

**OH 配置**: 仅静态链接

**影响**:
- 所有扩展模块编译进主程序
- 运行时无需加载 .so 文件
- 减少运行时依赖
- 增加二进制文件大小

**优势**:
- 简化部署（无额外库文件）
- 提高启动速度（无动态加载开销）
- 减少攻击面（无 dlopen）

#### 2. 内部 xtables API (`-DXTABLES_INTERNAL`)

**作用**: 暴露内部 API 供扩展模块使用

**相关配置**: `-DXTABLES_LIBDIR="xtables_libdir_not_used"`

**说明**: 由于使用静态链接，动态库目录设置为无用值，避免运行时尝试加载外部扩展。

#### 3. 警告控制

OH 配置中使用了大量 `-Wno-*` 选项：

| 选项 | 原因 |
|-----|------|
| `-Wno-error` | 编译器升级时避免构建失败 |
| `-Wno-pointer-arith` | iptables 大量使用 void* 运算 |
| `-Wno-sign-compare` | 有符号/无符号比较警告 |
| `-Wno-unused-parameter` | 回调函数中常用 |
| `-Wno-format` | 格式化字符串警告 |

**注意**: 这些选项在上游代码中常见，是为了兼容不同编译器版本。

### 依赖关系

```
iptables (可执行文件)
    │
    ├── deps: libext (通用扩展)
    ├── deps: libext4 (IPv4 扩展)
    ├── deps: libext6 (IPv6 扩展)
    ├── deps: libip4tc (IPv4 控制库)
    ├── deps: libip6tc (IPv6 控制库)
    └── deps: libxtables (xtables 库)

libext / libext4 / libext6 (扩展库)
    │
    └── (无额外 deps，自包含)

libip4tc / libip6tc / libxtables (基础库)
    │
    └── (无额外 deps，直接调用内核接口)
```

## 特殊构建处理

### 1. 扩展模块代码生成 (genInit.py)

#### 位置
`extensions/genInit.py`

#### 功能
自动生成扩展模块的初始化代码文件。

#### 工作原理

1. **扫描源文件**
   ```python
   for filter_file in os.listdir(ori_path):
       if (prefix in filter_file) and (extension == ".c"):
           # 处理文件
   ```

2. **提取初始化函数**
   ```python
   if "_init(void)" in content:
       # 重命名函数以避免命名冲突
       replace_init_text = filter_file.rstrip(".c") + "_init(void)"
       new_content = content.replace("_init(void)", replace_init_text)
   ```

3. **生成初始化文件**
   - 生成 `initext.c` / `initext4.c` / `initext6.c`
   - 包含所有扩展的初始化函数声明
   - 包含统一的初始化函数调用所有扩展

#### 调用参数

```gn
# libext (通用扩展)
args_libext = [
  "libxt_",  # 文件前缀
  "[libxt_cgroup.c,libxt_ipvs.c,...]",  # 排除列表
  "initext.c",  # 输出文件名
  "extensions",  # 内部方法名
]

# libext4 (IPv4 扩展)
args_libext4 = ["libipt", "[]", "initext4.c", "extensions4"]

# libext6 (IPv6 扩展)
args_libext6 = ["libip6t_", "[]", "initext6.c", "extensions6"]
```

#### 生成的代码示例

```c
// initext.c (片段)
void init_extensions(void);
void init_extensions(void)
{
    libxt_addrtype_init();
    libxt_AUDIT_init();
    libxt_bpf_init();
    // ... 所有通用扩展
}
```

#### MD5 缓存机制

```python
def _need_rebuild(src_file, dest_file, src_md5_file):
    if os.path.exists(src_file) and os.path.exists(dest_file):
        this_md5 = md5sum(src_file)
        last_md5 = read_file(src_md5_file)
        if this_md5 == last_md5:
            return 0  # 无需重建
    return 1  # 需要重建
```

- 避免不必要的文件拷贝和重新编译
- 增量构建优化

### 2. Patch 自动应用 (install.sh)

#### 位置
根目录 `install.sh`

#### 触发方式
在 BUILD.gn 中通过 `exec_script` 调用：

```gn
iptables_path = rebase_path("//third_party/iptables")
exec_script("install.sh", [ "$iptables_path" ])
```

#### 工作流程

```bash
_all_patches=(
  "0001-musl-build-fix.patch"
)

for filename in "${_all_patches[@]}"; do
  # 1. 验证 Patch
  if patch --dry-run -p1 < ${filename} > /dev/null 2>&1; then
    echo "Verify patch ${filename} ok. start apply."
    # 2. 应用 Patch
    patch -p1 < ${filename} --fuzz=0 --no-backup-if-mismatch
  else
    echo "Verify patch ${filename} not ok. patch already apply? skip apply."
  fi
done
```

#### 特点

1. **幂等性**: 已应用的 Patch 不会重复应用
2. **验证优先**: 先验证再应用，避免部分应用失败
3. **无备份**: `--no-backup-if-mismatch` 不生成备份文件
4. **精确匹配**: `--fuzz=0` 不允许模糊匹配

### 3. 源文件拷贝机制

`genInit.py` 会将扩展源文件从 `extensions/` 拷贝到 `out/gen/`：

```
extensions/libxt_*.c  →  out/gen/libxt_*.c  →  编译为 libext
```

**原因**:
- 需要修改源文件中的 `_init(void)` 函数名
- 不能直接修改原始源文件（需要保持干净）
- 在输出目录进行修改造构建时副作用

## 与上游构建系统对比

| 特性 | 上游 (autotools) | OH (GN) |
|-----|-----------------|---------|
| **构建配置** | configure.ac + Makefile.am | BUILD.gn (4 个) |
| **配置生成** | ./configure | 预生成 config.h |
| **构建工具** | make | ninja (通过 GN 生成) |
| **库类型** | 共享库 (.so) + 静态库 (.a) | 仅静态库 |
| **扩展加载** | 动态 (dlopen) | 静态链接 |
| **安装路径** | 可配置 (--prefix) | 固定到 OH 系统目录 |
| **代码生成** | Makefile 规则 | genInit.py 脚本 |
| **Patch 应用** | 手动或 distro 打包 | install.sh 自动 |

### 关键差异说明

#### 配置系统

**上游**:
```bash
./configure --prefix=/usr --enable-static --disable-shared
make
make install
```

**OH**:
- `config.h` 预生成并硬编码在仓库中
- 无需 configure 步骤
- 所有选项在 BUILD.gn 中静态定义

#### 扩展模块处理

**上游**:
```
/usr/lib/xtables/libxt_*.so  (动态加载)
```

**OH**:
```
所有扩展静态链接到 iptables 可执行文件
```

#### 依赖声明

**上游**: 通过 pkg-config 和 configure 检查
**OH**: 在 BUILD.gn 中显式声明 deps

## 构建命令示例

### OH 构建

```bash
# 在 OpenHarmony 源码根目录
./build.sh --product {product_name} --build-target //third_party/iptables:iptables

# 构建所有 iptables 目标
./build.sh --product {product_name} --build-target //third_party/iptables:all
```

### 本地测试构建

```bash
# 进入 iptables 目录
cd third_party/iptables

# 应用 Patch
./install.sh .

# 手动编译测试 (使用 OH 工具链)
# 注意：需要完整的 OH 构建环境
```

## 常见问题

### Q: 为什么使用静态链接？
**A**: 
1. 简化系统镜像（无额外 .so 文件）
2. 提高启动性能
3. 减少运行时依赖和攻击面
4. 符合 OH 嵌入式设备的设计原则

### Q: 如何添加新的扩展模块？
**A**:
1. 将新的 `.c` 文件放入 `extensions/` 目录
2. 在对应 BUILD.gn 的 sources 列表中添加文件路径
3. 重新构建即可

### Q: 构建时 Patch 应用失败怎么办？
**A**:
1. 检查 Patch 是否已手动应用过
2. 检查源文件是否与 Patch 期望的版本一致
3. 查看具体错误信息：`patch -p1 < 0001-musl-build-fix.patch`
4. 必要时重新生成 Patch

### Q: 如何更新到新的上游版本？
**A**:
1. 更新源码到新版本
2. 尝试应用现有 Patch：`patch --dry-run -p1 < 0001-musl-build-fix.patch`
3. 如失败，分析差异并重新生成 Patch
4. 检查 BUILD.gn 是否需要更新（新扩展、新源文件等）
5. 完整构建测试
