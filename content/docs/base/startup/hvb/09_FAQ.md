# 常见问题

## 目的

本文档收集 HVB 组件的常见问题、定位方法和解决方案，帮助开发者快速排查和解决问题。

---

## 适用范围

- 目标读者：Bootloader 开发者、init 开发者、构建工程师
- 知识储备：C 语言、GN 构建、Bootloader 工作流程

---

## 核心结论

1. **集成问题**：主要集中在 `hvb_ops` 实现和平台适配
2. **构建问题**：GN 配置错误、拼写错误、依赖缺失
3. **运行时问题**：校验失败、启动参数格式错误
4. **调试方法**：通过 `hvb_print()` 输出调试信息

---

## 集成问题

### Q1: 链接错误 - undefined reference to `hvb_chain_verify`

**现象**：
```
error: undefined reference to 'hvb_chain_verify'
```

**原因**：
- 未正确链接 `libhvb_static.a`
- 链接顺序错误（依赖库放后面）

**解决方案**：
```makefile
# 错误：库放前面
gcc bootloader.o -lhvb_static -o bootloader

# 正确：库放后面
gcc bootloader.o -o bootloader -lhvb_static

# 或使用 -L 指定路径
gcc bootloader.o -L $(OUT_DIR)/system/lib -lhvb_static -o bootloader
```

---

### Q2: hvb_ops 实现后编译错误 - incompatible pointer types

**现象**：
```
error: passing 'enum hvb_io_errno (*)(...)' to parameter of incompatible pointer type
```

**原因**：
- 函数签名不匹配
- 参数类型错误

**解决方案**：
```c
// 参考准确签名
enum hvb_io_errno my_read_partition(
    struct hvb_ops *ops,
    const char *ptn,
    int64_t offset,
    uint64_t num_bytes,
    void *buffer,
    uint64_t *out_num_read
) {
    // 实现...
}
```

**证据**：`libhvb/include/hvb_ops.h:40-41`

---

### Q3: 校验总是返回 HVB_ERROR_IO

**现象**：
```c
enum hvb_errno ret = hvb_chain_verify(&ops, "rvt", ptn_list, &vd);
printf("ret = %d\n", ret);  // 总是 3 (HVB_ERROR_IO)
```

**原因**：
1. `ops->read_partition()` 实现错误
2. 分区名错误
3. 存储设备访问失败

**排查步骤**：
1. 检查分区名是否正确（`rvt`、`boot` 等）
2. 检查分区是否存在
3. 添加调试输出：
```c
enum hvb_io_errno my_read_partition(...) {
    printf("Reading partition: %s, offset=%lld, size=%llu\n",
           ptn, offset, num_bytes);
    // 实现...
}
```

---

### Q4: 校验返回 HVB_ERROR_PUBLIC_KEY_REJECTED

**现象**：
```c
ret = hvb_chain_verify(...);
// ret = 7 (HVB_ERROR_PUBLIC_KEY_REJECTED)
```

**原因**：
- `ops->valid_rvt_key()` 返回 `is_trusted = false`
- 公钥不在可信根白名单中

**排查步骤**：
1. 检查 `ops->valid_rvt_key()` 实现
2. 验证可信根公钥是否正确
3. 检查公钥格式（PEM/DER）
4. 调试输出：
```c
enum hvb_io_errno my_valid_rvt_key(..., bool *out_is_trusted) {
    printf("Validating pubkey, len=%llu\n", pubkey_length);
    // 实现...
    printf("is_trusted = %d\n", *out_is_trusted);
}
```

---

### Q5: 设备解锁后仍校验失败

**现象**：
- 设备已解锁
- 但校验仍然返回错误

**原因**：
- Bootloader 未正确实现 `ops->read_lock_state()`
- 锁状态判断逻辑错误

**解决方案**：
```c
// 检查锁状态，解锁时跳过校验
bool is_locked;
ops->read_lock_state(ops, &is_locked);
if (!is_locked) {
    printf("Device unlocked, skip verification\n");
    return HVB_OK;  // 或直接返回，不校验
}
```

---

## 构建问题

### Q6: GN 构建错误 - include_dirs not found

**现象**：
```
ERROR:include_dirs value "incldue" is not in the default list
```

**原因**：
- `libhvb/BUILD.gn:51,67` 中存在拼写错误
- `incldue` 应为 `include`

**解决方案**：
```gn
# 错误
include_dirs = [ "incldue" ]

# 正确
include_dirs = [ "include" ]
```

**证据**：`libhvb/BUILD.gn:51,67`

---

### Q7: 找不到 securec 头文件

**现象**：
```
fatal error: securec.h: No such file or directory
```

**原因**：
- 未正确配置 `bounds_checking_function` 依赖
- SDK 路径配置错误

**解决方案**：
```gn
# 确保依赖正确
external_deps = [ "bounds_checking_function:libsec_shared" ]

# 检查 SDK 路径
ohos_sysroot = /path/to/sdk
```

**证据**：`libhvb/BUILD.gn:50,66`

---

### Q8: 编译警告 - unused variable

**现象**：
```
warning: unused variable 'ret'
```

**原因**：
- 变量定义后未使用
- 条件编译导致部分代码未执行

**影响**：无功能性影响，但可能掩盖真实问题

**解决方案**：
```c
// 添加 (void) 或删除未使用变量
(void)ret;  // 明确标记未使用
```

---

## hvbtool 问题

### Q9: hvbtool.py 报错 - Python version too old

**现象**：
```
Python 3.0 or higher required
```

**原因**：
- Python 版本低于 3.0

**解决方案**：
```bash
# 检查 Python 版本
python3 --version

# 升级 Python
sudo apt install python3.9

# 或使用虚拟环境
python3.9 -m venv venv
source venv/bin/activate
python hvbtool.py ...
```

**证据**：`tools/readme.md:103`

---

### Q10: hvbtool.py 报错 - OpenSSL command not found

**现象**：
```
FileNotFoundError: [Errno 2] No such file or directory: 'openssl'
```

**原因**：
- OpenSSL 未安装或不在 PATH 中

**解决方案**：
```bash
# 安装 OpenSSL
sudo apt install openssl  # Ubuntu/Debian
sudo yum install openssl  # CentOS/RHEL

# 或指定完整路径
export PATH="/usr/local/bin:$PATH"
```

**证据**：`tools/hvbtool.py:169-176`（`subprocess.Popen(['openssl', ...])`）

---

### Q11: 签名后镜像大小超出分区

**现象**：
```
Error: Signed image size > partition_size
```

**原因**：
- 镜像 + verity footer 大小超过分区大小
- `--partition_size` 参数设置过小

**解决方案**：
```bash
# 计算正确大小
image_size=$(stat -f%s boot.img)
padding=4096  # Footer 大小
footer_size=$(python3 -c "import struct; print(struct.calcsize('8s4Q64s'))")
total_size=$((image_size + padding + footer_size))

# 使用正确大小
./hvbtool.py make_hash_footer \
  --image boot.img \
  --partition boot \
  --partition_size $total_size \
  ...
```

---

## 运行时问题

### Q12: dm-verity 未使能

**现象**：
- init 调用 `hvb_chain_verify()` 成功
- 但 dm-verity 未使能，文件系统可读写

**原因**：
- 启动参数未正确传递给内核
- 内核未使能 dm-verity
- fstab 未配置 hvb 标志

**排查步骤**：
1. 检查启动参数是否生成：
```c
printf("cmdline: %s\n", vd->cmdline.buf);
```

2. 检查内核参数：
```bash
cat /proc/cmdline | grep hvb
```

3. 检查 fstab 配置：
```
# /system/etc/fstab
# 确保有 hvb 标志
/dev/block/... /usr ext4 ro,barrier=1 wait,required,hvb
```

**证据**：`README_zh.md:147-152`

---

### Q13: 校验通过但镜像损坏

**现象**：
- `hvb_chain_verify()` 返回 `HVB_OK`
- 但实际镜像数据损坏

**原因**：
- 仅验证签名和哈希，不验证内容完整性
- 坏块未被检测（除非 FEC 启用）

**解决方案**：
- 实现 FEC 纠错码（当前未完整实现）
- 或使用 full-disk 校验（hash 模式）

**证据**：`libhvb/include/hvb_cert.h:144-151`（FEC 字段定义）

---

## 调试技巧

### 1. 启用调试输出

```c
// 在代码中添加调试宏
#define DEBUG_HVB 1

#ifdef DEBUG_HVB
#define hvb_print(msg) printf(msg)
#else
#define hvb_print(msg) ((void)0)
#endif
```

### 2. 打印关键数据

```c
// 打印证书信息
printf("Cert version: %d.%d\n", cert->version_major, cert->version_minor);
printf("Partition: %s\n", cert->image_name);
printf("Algorithm: %d\n", cert->hash_algo);

// 打印校验结果
printf("Verify result: %d\n", ret);
```

### 3. 使用 GDB 调试

```bash
# 编译调试版本
./build.sh --product-name <product> --ccache --gn-args=is_debug=true

# 用 GDB 调试
gdb ./bootloader
(gdb) break hvb_chain_verify
(gdb) run
(gdb) print vd
```

---

## 平台适配问题

### Q14: 如何适配 ARM64 平台？

**解决方案**：

```c
// 检查指针大小
#if UINTPTR_MAX == 0xffffffffffffffffULL
#define IS_64BIT 1
#else
#define IS_64BIT 0
#endif

// 根据平台调整
if (IS_64BIT) {
    // ARM64 特定逻辑
}
```

### Q15: 如何实现 ops->valid_rvt_key()？

**示例实现**：

```c
enum hvb_io_errno my_valid_rvt_key(
    struct hvb_ops *ops,
    const uint8_t *pubkey,
    uint64_t pubkey_length,
    const uint8_t *pubkey_metadata,
    uint64_t pubkey_metadata_length,
    bool *out_is_trusted
) {
    // 1. 读取可信根公钥（从 TPM/EFUSE）
    uint8_t trusted_key[256];
    size_t trusted_len = read_trusted_key(trusted_key);

    // 2. 对比公钥
    *out_is_trusted = (pubkey_length == trusted_len &&
                     memcmp(pubkey, trusted_key, trusted_len) == 0);

    return *out_is_trusted ? HVB_IO_OK : HVB_IO_ERROR_NO_SUCH_VALUE;
}
```

---

## 相关跳转

- [项目概览](00_Overview.md) - HVR 组件介绍
- [对外 API](04_Public_API.md) - 接口使用方法
- [安全分析](08_Security_Analysis.md) - 安全最佳实践

---

## 仍需帮助？

1. **查看代码注释**：头文件中包含详细说明
2. **阅读 README**：`README_zh.md` 有中文使用说明
3. **参考测试用例**：`test/` 目录（不在本文档范围内）
4. **咨询社区**：[OpenHarmony 社区](https://gitee.com/openharmony)

---

*最后更新: 2026-02-06*
