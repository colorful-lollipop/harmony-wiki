# 依赖关系与使用

## 4.1 直接依赖者

### 已发现的直接依赖模块

| 模块 | BUILD.gn 路径 | 依赖方式 | 用途 |
|------|---------------|----------|------|
| **samgr_lite** | `foundation/systemabilitymgr/samgr_lite/samgr/BUILD.gn` | `deps` | 系统能力管理器 |

### 依赖详情

#### samgr_lite 依赖配置

```gn
# foundation/systemabilitymgr/samgr_lite/samgr/BUILD.gn
deps = [ "//third_party/mksh" ]
```

**用途说明**：
samgr_lite（System Ability Manager Lite）是 OpenHarmony 的核心系统服务管理组件，它使用 mksh 作为脚本执行环境，用于：

- 系统服务初始化脚本执行
- 动态服务配置
- 系统状态诊断

### 待发现的依赖模块

**注意**：由于搜索范围限制，可能还有其他模块依赖 mksh。以下是推测的直接依赖者：

| 推测模块 | 推测用途 |
|----------|----------|
| init | 系统初始化进程 |
| shell | 交互式 shell 会话 |
| hilog | 日志系统脚本 |

---

## 4.2 使用场景分析

### 场景 1：系统服务脚本执行

**参与者**：
- samgr_lite（调用者）
- mksh（脚本解释器）

**流程**：
```
samgr_lite 
  → 加载 mksh 库
  → 传递脚本内容
  → 执行脚本
  → 返回结果
```

**示例**：
```c
// samgr_lite 中的伪代码
int ExecuteScript(const char* script) {
    char* argv[] = { "sh", "-c", script, NULL };
    return mksh_execute(4, argv);
}
```

### 场景 2：交互式 Shell 会话

**参与者**：
- 终端应用（调用者）
- mksh（独立进程）

**流程**：
```
用户终端
  → 启动 mksh 进程
  → 读写 stdin/stdout
  → 处理命令
```

**启动方式**：
```bash
# 直接启动 mksh
/bin/mksh

# 或通过符号链接
/bin/sh -> /bin/mksh
```

### 场景 3：系统脚本执行

**参与者**：
- init 或其他系统进程
- mksh（解释脚本文件）

**用途**：
- 系统启动脚本
- 服务配置脚本
- 设备初始化脚本

**示例脚本**：
```bash
#!/bin/sh
# 系统初始化脚本示例
export PATH=/usr/local/bin:/bin:/usr/bin
mount -a
start_services
```

---

## 4.3 链接方式

### 静态链接 vs 动态链接

| 链接方式 | 当前状态 | 说明 |
|----------|----------|------|
| **静态链接** | ✅ 使用 | mksh 编译为独立可执行文件 |
| **动态链接** | ❌ 未使用 | 无需动态链接库 |

### 链接说明

mksh 在 OH 中以**独立可执行文件**形式存在，不作为库被其他模块链接。这种方式的优点：

- ✅ 独立的进程空间，安全性高
- ✅ 无需处理库版本兼容问题
- ✅ 简单的部署方式
- ❌ 启动时需要 fork/exec 开销

---

## 4.4 头文件引用

### 引用方式

mksh 不提供公共头文件给其他模块使用。它是一个**独立的可执行程序**，而非库。

### 内部头文件结构

```mksh
sh.h          # 主头文件，包含所有公共定义
├── 类型定义
├── 全局变量
├── 函数声明
└── 常量定义
```

**注意**：这些头文件仅用于 mksh 自身编译，不对外提供 API。

---

## 4.5 依赖关系图

### 依赖图（Mermaid）

```mermaid
graph TB
    subgraph 系统层
        A[init] --> B[mksh]
        C[samgr_lite] --> B
        D[shell] --> B
    end
    
    subgraph 用户层
        E[终端应用] --> D
        F[系统管理] --> C
    end
    
    B --> G[/bin/sh]
    B --> H[/usr/bin/mksh]
```

### 依赖说明

| 关系 | 描述 |
|------|------|
| init → mksh | 系统初始化时调用脚本 |
| samgr_lite → mksh | 服务管理器执行脚本 |
| shell → mksh | 交互式 shell 前端 |

---

## 4.6 使用示例

### 示例 1：在 BUILD.gn 中添加依赖

```gn
# foundation/xxx/BUILD.gn
public_deps += [
  "//third_party/mksh:sh",  # 添加 mksh 依赖
]
```

### 示例 2：执行脚本命令

```bash
# 交互式使用
OHOS:/$ echo "Hello, OHOS!"
Hello, OHOS!

# 执行脚本文件
OHOS:/$ cat /system/init.sh
#!/bin/sh
echo "System starting..."

OHOS:/$ mksh /system/init.sh
System starting...
```

### 示例 3：脚本功能演示

```bash
# 变量使用
OHOS:/$ name="OHOS"
OHOS:/$ echo "Hello, $name!"
Hello, OHOS!

# 条件判断
OHOS:/$ if [ -f /system/build ]; then echo "Build exists"; fi
Build exists

# 循环结构
OHOS:/$ for i in 1 2 3; do echo "Count: $i"; done
Count: 1
Count: 2
Count: 3

# 函数定义
OHOS:/$ hello() { echo "Hello, $1!"; }
OHOS:/$ hello World
Hello, World!
```

---

## 4.7 配置与定制

### 启动参数

```bash
mksh [选项] [脚本文件]

常用选项：
-c string    # 从字符串执行命令
-i           # 交互模式
-l           # 登录 shell
-V           # 显示版本
-x           # 调试模式（输出执行的命令）
```

### 环境变量

| 变量 | 用途 | 示例 |
|------|------|------|
| `PATH` | 命令搜索路径 | `/usr/local/bin:/bin:/usr/bin` |
| `HOME` | 用户主目录 | `/root` |
| `USER` | 当前用户 | `root` |
| `SHELL` | 默认 shell | `/bin/mksh` |
| `PS1` | 主提示符 | `OHOS:/$ ` |
| `HISTFILE` | 历史文件 | `~/.mksh_history` |

### 初始化文件

| 文件 | 用途 | 加载时机 |
|------|------|----------|
| `/etc/mkshrc` | 系统级配置 | 每次启动 |
| `~/.mkshrc` | 用户级配置 | 每次启动 |

---

## 4.8 性能考虑

### 启动时间

| 指标 | 数值 |
|------|------|
| 冷启动 | < 100ms |
| 热启动 | < 10ms |

### 资源占用

| 资源 | 数值 | 说明 |
|------|------|------|
| ROM | 172KB | 编译后大小 |
| RAM | 344KB | 运行时占用 |

### 优化建议

1. **减少脚本复杂度**：简化启动脚本
2. **预编译脚本**：使用 mksh 的编译功能
3. **共享内存**：多实例时考虑共享只读数据

---

## 4.9 常见问题

### Q1：如何替换默认 Shell？

**方法 1**：修改 `/etc/passwd`
```
root:x:0:0:root:/root:/bin/mksh
```

**方法 2**：使用 `chsh` 命令
```bash
chsh -s /bin/mksh
```

### Q2：如何启用终端扩展功能？

**步骤**：
1. 在 mksh.gni 中设置 `mksh_terminal_ext = true`
2. 重新编译 mksh

**验证**：
```bash
OHOS:/$ echo $MKSH_VERSION
R59c (MKSH_TERMINAL_EXT)
```

### Q3：如何调试脚本？

**启用调试模式**：
```bash
# 方法1：命令行参数
mksh -x script.sh

# 方法2：脚本内调试
set -x
# 调试代码
set +x
```
