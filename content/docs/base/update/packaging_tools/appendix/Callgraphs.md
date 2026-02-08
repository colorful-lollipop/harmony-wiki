# 关键调用链（Callgraphs）

> 本文档记录 OpenHarmony 升级包制作工具的关键调用链，从入口到核心逻辑的完整调用路径，帮助调试人员和开发人员理解代码执行流程。

## 1 入口调用链

### 1.1 build_update.py 入口调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    build_update.py 入口调用链                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  python build_update.py                                              │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def main():                                                   │   │
│  │  1. 解析命令行参数                                             │   │
│  │  2. 加载配置                                                   │   │
│  │  3. 验证参数                                                   │   │
│  │  4. 调用处理流程                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  main()                                                             │
│  ├── argparse.parse_args()  ← 参数解析                              │
│  ├── Options() ← 配置管理                                          │
│  ├── validate_args() ← 参数验证                                     │
│  └── create_update_package() ← 包创建（条件分支）                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 2 镜像处理调用链

### 2.1 镜像解析调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    镜像解析调用链                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  image_class.py                                                     │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def parse_image(file_path):                                  │   │
│  │  1. 检测镜像格式（raw/sparse）                                 │   │
│  │  2. 选择对应的解析器                                            │   │
│  │  3. 执行解析                                                   │   │
│  │  4. 返回镜像数据块                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def parse_sparse_image(file_path):                            │   │
│  │  1. 读取 sparse header                                        │   │
│  │  2. 解析 chunk 数据                                            │   │
│  │  3. 展开 sparse 数据                                           │   │
│  │  4. 验证完整性                                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def parse_raw_image(file_path):                               │   │
│  │  1. 直接读取文件内容                                           │   │
│  │  2. 验证文件大小                                               │   │
│  │  3. 计算校验和                                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  parse_image(file_path)                                             │
│  ├── detect_format(file_path)                                       │
│  │   ├── is_sparse_image(file_path)                              │
│  │   └── is_raw_image(file_path)                                 │
│  ├── parse_sparse_image(file_path) ← 条件调用                      │
│  └── parse_raw_image(file_path) ← 条件调用                         │
│                                                                      │
│  依赖模块：                                                          │
│  - update_package.py ← 镜像数据传递                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 镜像格式检测调用链

```
detect_format(file_path)
     │
     ├── is_sparse_image(file_path)
     │    ├── open(file_path, 'rb')
     │    ├── read_header(16 bytes)
     │    ├── validate_magic_number()
     │    └── return True/False
     │
     └── is_raw_image(file_path)
          ├── open(file_path, 'rb')
          ├── read_first_bytes()
          └── check_not_sparse_magic()
```

## 3 升级包生成调用链

### 3.1 全量升级包创建调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    全量升级包创建调用链                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  create_update_package.py                                           │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ class CreateUpdatePackage:                                    │   │
│  │  def run():                                                   │   │
│  │    1. 加载目标镜像                                             │   │
│  │    2. 生成升级脚本                                            │   │
│  │    3. 计算签名                                                │   │
│  │    4. 组装升级包                                              │   │
│  │    5. 写入输出                                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ├─────────────────────────────────────────────────────────┐   │
│       │                                                         │   │
│       ▼                                                         ▼   │
│  ┌───────────────────────┐    ┌───────────────────────────────┐   │
│  │ image_class.py         │    │ script_generator.py           │   │
│  │ parse_target_images() │    │ generate_full_script()        │   │
│  │                       │    │                               │   │
│  │ 返回：                │    │ 返回：                         │   │
│  │ - system.img          │    │ - Action List                 │   │
│  │ - vendor.img          │    │ - Install commands            │   │
│  │ - product.img         │    │ - Verify commands             │   │
│  └───────────────────────┘    └───────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  CreateUpdatePackage.run()                                          │
│  ├── image_class.parse_target_images(target_dir)                    │
│  ├── script_generator.generate_full_script(image_data)              │
│  ├── build_pkcs7.sign(data) ← 签名处理                            │
│  └── update_package.create_package(data, script, signature)        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 差分升级包创建调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    差分升级包创建调用链                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  create_update_package.py (差分模式)                                 │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def run_diff_mode(source_path, target_dir):                  │   │
│  │  1. 解析源镜像                                               │   │
│  │  2. 解析目标镜像                                             │   │
│  │  3. 计算差分数据                                             │   │
│  │  4. 生成差分脚本                                             │   │
│  │  5. 签名和打包                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ├─────────────────────────────────────────────────────────┐   │
│       │                                                         │   │
│       ▼                                                         ▼   │
│  ┌───────────────────────┐    ┌───────────────────────────────┐   │
│  │ image_class.py         │    │ patch_package_process.py     │   │
│  │ parse_source_image()   │    │ calculate_diff()            │   │
│  │                       │    │                               │   │
│  │ 返回：                │    │ 调用：                         │   │
│  │ - source_image_data    │    │ - blocks_manager.alloc()     │   │
│  │                       │    │ - bsdiff.calculate()          │   │
│  │                       │    │ - imgdiff.calculate()         │   │
│  └───────────────────────┘    └───────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  run_diff_mode(source, target)                                      │
│  ├── image_class.parse_source_image(source)                        │
│  ├── image_class.parse_target_image(target)                        │
│  ├── blocks_manager.prepare_blocks(source_data, target_data)      │
│  ├── patch_package_process.calculate_diff(source, target)          │
│  │   ├── blocks_manager.get_diff_blocks()                        │
│  │   └── subprocess.call(['bsdiff', ...])                        │
│  ├── gigraph_process.optimize_diff(diff_data)                     │
│  ├── transfers_manager.create_transfer_info(diff_data)            │
│  ├── script_generator.generate_diff_script(transfer_info)          │
│  ├── build_pkcs7.sign(diff_data)                                   │
│  └── update_package.create_package(diff_data, script, signature)  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 4 签名处理调用链

### 4.1 PKCS7 签名调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PKCS7 签名调用链                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  build_pkcs7.py                                                     │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def sign_data(data, private_key_path, algorithm):             │   │
│  │  1. 加载私钥                                                  │   │
│  │  2. 计算数据哈希                                              │   │
│  │  3. 生成签名                                                  │   │
│  │  4. 打包 PKCS7                                                │   │
│  │  5. 返回签名块                                                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 详细步骤：                                                    │   │
│  │  1. load_private_key(private_key_path)                      │   │
│  │     ├── open(private_key_path, 'rb')                        │   │
│  │     ├── deserialize_pem_key()                                │   │
│  │     └── return PrivateKey object                             │   │
│  │                                                              │   │
│  │  2. hash_data(data, algorithm)                                │   │
│  │     ├── select_hash_algorithm(algorithm)                     │   │
│  │     ├── hash_func.update(data)                                │   │
│  │     └── hash_func.finalize()                                  │   │
│  │                                                              │   │
│  │  3. generate_signature(private_key, hash)                    │   │
│  │     ├── padding.OAEP.new()                                    │   │
│  │     ├── private_key.sign()                                   │   │
│  │     └── return signature bytes                                │   │
│  │                                                              │   │
│  │  4. pack_pkcs7(signature, algorithm, certificates)          │   │
│  │     ├── PKCS7.new()                                           │   │
│  │     ├── PKCS7.add_signer()                                   │   │
│  │     └── PKCS7.serialize()                                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  sign_data(data, key_path, algo)                                   │
│  ├── load_private_key(key_path)                                    │
│  ├── compute_hash(data, algo)                                       │
│  ├── private_key.sign(hash)                                         │
│  └── wrap_pkcs7(signature, algo)                                   │
│                                                                      │
│  依赖模块：                                                          │
│  - cryptography ← 加密库依赖                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 5 脚本生成调用链

### 5.1 升级脚本生成调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    升级脚本生成调用链                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  script_generator.py                                                │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ class ScriptGenerator:                                      │   │
│  │  def generate_script(image_data, script_type):               │   │
│  │    1. 创建 Action 列表                                        │   │
│  │    2. 添加分区操作                                            │   │
│  │    3. 添加校验操作                                            │   │
│  │    4. 生成脚本格式                                            │   │
│  │    5. 返回脚本内容                                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def generate_full_script(image_data):                       │   │
│  │  返回全量升级脚本                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def generate_diff_script(diff_data):                         │   │
│  │  返回差分升级脚本                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def generate_actions(image_data):                             │   │
│  │  1. for each partition:                                      │   │
│  │     - create_write_action() ← 写入操作                       │   │
│  │     - create_verify_action() ← 校验操作                      │   │
│  │  2. return [Action1, Action2, ...]                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def format_script(action_list):                              │   │
│  │  1. serialize_actions(action_list)                           │   │
│  │  2. add_header()                                            │   │
│  │  3. add_footer()                                            │   │
│  │  4. return script_content                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  generate_script(image_data, type)                                  │
│  ├── generate_actions(image_data)                                   │
│  │   ├── for partition in image_data.partitions:               │   │
│  │   │   ├── create_write_action(partition)                    │   │
│  │   │   └── create_verify_action(partition)                   │   │
│  │   └── return action_list                                     │   │
│  └── format_script(action_list)                                    │
│      ├── serialize_actions(action_list)                           │
│      ├── add_metadata()                                            │
│      └── return formatted_script                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 6 Block 管理调用链

### 6.1 差分 Block 分配调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Block 管理调用链                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  blocks_manager.py                                                  │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ class BlocksManager:                                          │   │
│  │  def allocate_blocks(source_data, target_data):               │   │
│  │    1. 划分源数据 Block                                        │   │
│  │    2. 划分目标数据 Block                                      │   │
│  │    3. 标记相同 Block                                          │   │
│  │    4. 标记差异 Block                                          │   │
│  │    5. 返回 Block 映射                                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def split_into_blocks(data, block_size):                     │   │
│  │  1. 计算 Block 数量                                           │   │
│  │  2. 读取每个 Block                                            │   │
│  │  3. 计算 Block 哈希                                           │   │
│  │  4. 返回 Block 列表                                           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def compare_blocks(source_blocks, target_blocks):           │   │
│  │  1. for each target_block:                                   │   │
│  │     - find matching_source_block(target_block)               │   │
│  │     - if found: mark as same                                 │   │
│  │     - else: mark as different                                │   │
│  │  2. return comparison_result                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  调用关系：                                                          │
│  allocate_blocks(source, target)                                   │
│  ├── split_into_blocks(source, BLOCK_SIZE)                         │
│  ├── split_into_blocks(target, BLOCK_SIZE)                         │
│  └── compare_blocks(source_blocks, target_blocks)                 │
│      ├── for target_block in target_blocks:                        │
│      │   ├── hash_block(target_block)                            │
│      │   └── find_in_source_blocks(hash)                         │
│      └── return mapping                                           │
│                                                                      │
│  依赖模块：                                                          │
│  - patch_package_process.py ← 差分计算使用                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 7 错误处理调用链

```
┌─────────────────────────────────────────────────────────────────────┐
│                    错误处理调用链                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  log_exception.py                                                   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ class CustomException(Exception):                           │   │
│  │  def __init__(self, message, error_code):                   │   │
│  │    - 记录错误信息                                            │   │
│  │    - 记录错误码                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ def handle_exception(exception):                            │   │
│  │  1. 记录错误日志                                             │   │
│  │  2. 格式化错误信息                                           │   │
│  │  3. 决定是否继续执行                                         │   │
│  │  4. 抛出或返回                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  调用链示例：                                                        │
│  any_function()                                                     │
│  ├── try:                                                          │
│  │   └── risky_operation()                                      │
│  │       ├── FileNotFoundError → caught                        │
│  │       ├── ValueError → caught                               │
│  │       └── CustomException → caught                          │
│  └── except Exception as e:                                      │
│      └── log_exception.handle_exception(e)                        │
│          ├── log_error(e)                                        │
│          └── re-raise or return error code                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## 8 完整调用关系索引

### 8.1 模块间调用矩阵

| 调用模块 | 被调用模块 | 调用函数 | 调用类型 |
|---------|-----------|---------|---------|
| build_update.py | utils.py | Options() | 直接调用 |
| build_update.py | image_class.py | parse_image() | 直接调用 |
| build_update.py | create_update_package.py | CreateUpdatePackage() | 直接调用 |
| create_update_package.py | image_class.py | parse_*_image() | 直接调用 |
| create_update_package.py | script_generator.py | generate_*_script() | 直接调用 |
| create_update_package.py | build_pkcs7.py | sign_data() | 直接调用 |
| create_update_package.py | update_package.py | create_package() | 直接调用 |
| patch_package_process.py | blocks_manager.py | allocate_blocks() | 直接调用 |
| patch_package_process.py | subprocess | call() | 外部调用 |
| script_generator.py | transfers_manager.py | create_transfer_info() | 间接调用 |

### 8.2 入口到终点的调用链

| 场景 | 入口 | 终点 | 关键中间节点 |
|-----|------|------|-------------|
| 全量升级 | build_update.main() | update_package.create_package() | image_class → script_generator → build_pkcs7 |
| 差分升级 | build_update.main() | update_package.create_package() | image_class → blocks_manager → patch_package_process → gigraph_process |
| 镜像解析 | image_class.parse_image() | 镜像数据对象 | detect_format → parse_sparse/raw |
| 签名生成 | build_pkcs7.sign_data() | PKCS7 签名块 | load_private_key → compute_hash → private_key.sign |

## 9 相关文档

| 文档 | 说明 |
|-----|-----|
| 02_Architecture.md | 系统架构 |
| 03_Usage.md | 使用说明 |
| appendix/Config_Flags.md | 配置参数 |
| 01_Directory_Structure.md | 模块职责 |
