# AI Engine 配置开关

## 编译时配置

### GN 构建宏

| 宏 | 位置 | 默认值 | 说明 |
|---|------|--------|------|
| `USE_NNIE` | `services/server/plugin/*/BUILD.gn` | 未定义 | 启用海思 NNIE 推理引擎 |
| `USE_HIAI` | - | 未定义 | 预留：启用 HIAI 推理引擎 |
| `USE_CPU` | - | 未定义 | 预留：启用 CPU 推理 |

#### USE_NNIE 配置

**启用场景**：在 hispark_taurus 开发板上使用 CV/ASR 插件

```gn
# services/server/plugin/asr/keyword_spotting/BUILD.gn
lite_library("asr_keyword_spotting") {
  sources = [ "kws_plugin.cpp" ]
  defines = [ "USE_NNIE" ]  # 启用 NNIE
  deps = [
    "//vendor/hispark_taurus/hardware/nnie_adapter:nnie_adapter",
    "//vendor/hispark_taurus/hardware/nnie:nnie",
  ]
}
```

**禁用场景**：在没有 NNIE 硬件的开发板上运行

```gn
# 注释掉 defines
# defines = [ "USE_NNIE" ]
```

---

### 编译器标志

| 标志 | 作用域 | 用途 |
|------|--------|------|
| `-fPIC` | 22+ BUILD.gn | 位置无关代码（共享库必需） |
| `-fexceptions` | test BUILD.gn | 启用 C++ 异常支持 |
| `-fno-omit-frame-pointer` | debug builds | 保留帧指针（调试用） |
| `-O2` | release builds | 优化级别 |
| `-g` | debug builds | 调试符号 |

**证据**：`services/server/BUILD.gn:22` cflags 定义

### 链接器标志

| 标志 | 位置 | 用途 |
|------|------|------|
| `-lstdc++` | services/server/BUILD.gn | C++ 标准库 |
| `-lpthread` | services/server/BUILD.gn | POSIX 线程 |
| `-ldl` | services/server/BUILD.gn | 动态加载（dlopen/dlsym） |
| `-lnnie` | 插件 BUILD.gn | NNIE 推理库 |
| `-lnnie_adapter` | 插件 BUILD.gn | NNIE 适配层 |
| `-lmpi` | ASR 插件 BUILD.gn | 媒体处理接口 |
| `-lhdi_media` | audio_loader BUILD.gn | 媒体硬件驱动接口 |

---

## GNI 配置文件

### ai_plugin_config.gni

**路径**：`services/ai_plugin_config.gni`

**内容**：
```gni
# 插件激活列表
# 通过产品配置文件 config.json 覆盖
activate_plugin_list = []
```

**使用方式**：
```json
// 产品 config.json
{
  "ai_plugin_config": {
    "activate_plugin_list": [
      "asr_keyword_spotting",
      "cv_image_classification"
    ]
  }
}
```

**证据**：`services/ai_plugin_config.gni` 配置定义

---

## 运行时配置

### 插件配置文件

**路径**：`/system/etc/ai_engine_plugin.ini`

**格式**：
```ini
[plugin]
# 插件名称
name = keyword_spotting

# 插件路径
path = /system/lib/libasr_keyword_spotting.so

# 算法类型
algo_type = 20001002

# 版本
version = 20001001

# 推理模式 (sync/async)
infer_mode = sync
```

**证据**：`services/server/plugin_manager/BUILD.gn:24-31` gen_etc_ini action

### 环境变量

| 变量 | 值 | 说明 |
|------|------|------|
| `AIE_LOG_LEVEL` | 0-5 | 日志级别（0=DEBUG, 5=ERROR） |
| `AIE_PLUGIN_PATH` | 路径 | 插件搜索路径 |
| `AIE_MODEL_PATH` | 路径 | 模型文件搜索路径 |

**设置方式**：
```bash
# 设置日志级别
export AIE_LOG_LEVEL=1

# 运行
./ai_server
```

---

## 产品配置

### config.json 配置项

**路径**：产品级配置文件

**示例**：
```json
{
  "ai_engine": {
    "enable": true,
    "plugins": [
      {
        "name": "keyword_spotting",
        "path": "/system/lib/libasr_keyword_spotting.so"
      },
      {
        "name": "image_classification",
        "path": "/system/lib/libcv_image_classification.so"
      }
    ],
    "max_clients": 16,
    "max_sessions_per_client": 8,
    "shared_memory_size": 10485760
  }
}
```

---

## 功能开关

### 编译时特性

| 特性 | 宏 | 默认 | 说明 |
|------|-----|------|------|
| 同步推理 | - | 启用 | 支持 SyncProcess |
| 异步推理 | - | 启用 | 支持 AsyncProcess |
| 插件热插拔 | - | 启用 | 支持动态加载/卸载 |
| 共享内存优化 | - | 启用 | 大数据使用共享内存 |
| 崩溃恢复 | - | 禁用 | 服务端死亡自动重启 |

### 运行时特性

| 特性 | 配置项 | 默认 | 说明 |
|------|--------|------|------|
| 调试模式 | AIE_DEBUG=1 | 关闭 | 启用额外检查和日志 |
| 性能分析 | AIE_PROFILE=1 | 关闭 | 输出性能数据 |
| 内存追踪 | AIE_MEMTRACE=1 | 关闭 | 输出内存使用 |

---

## 端口化配置

### 适配新开发板

```gn
# 新开发板 BUILD.gn 片段
if (board_name == "new_board") {
  defines += [ "USE_NNIE" ]
  deps += [
    "//vendor/new_board/hardware/nnie_adapter:nnie_adapter",
    "//vendor/new_board/hardware/nnie:nnie",
  ]
} else {
  # 无 NNIE 硬件的降级配置
  defines -= [ "USE_NNIE" ]
}
```

### 禁用 NNIE 的配置

```gn
# 无 NNIE 开发板 BUILD.gn
lite_component("ai") {
  features = [
    "client:client",
    "server:server",
    # 禁用插件
    # "server/plugin:plugin",
  ]
}
```

---

## 配置优先级

| 优先级 | 配置来源 | 说明 |
|--------|----------|------|
| 1 | 命令行参数 | 最高优先级 |
| 2 | 环境变量 | 可覆盖运行时配置 |
| 3 | 产品 config.json | 产品级配置 |
| 4 | ai_plugin_config.gni | 组件级默认配置 |
| 5 | 代码默认值 | 最低优先级 |

---

## 配置验证

### 验证脚本

```bash
# 检查配置有效性
python3 validate_config.py --config /system/etc/ai_engine_plugin.ini

# 检查插件签名
python3 verify_plugin.py --plugin /system/lib/libasr_keyword_spotting.so

# 检查依赖库
ldd /system/lib/libasr_keyword_spotting.so
```

### 常见配置错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| 插件加载失败 | 路径错误 | 检查 ini 文件路径 |
| dlopen 失败 | 权限不足 | chmod +x .so |
| NNIE 初始化失败 | 硬件不支持 | 注释 USE_NNIE |
| 共享内存创建失败 | 权限不足 | 检查 /dev/shm 权限 |
