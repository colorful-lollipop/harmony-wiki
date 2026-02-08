# 配置开关参考

> 本文档描述 update_app 模块的所有配置开关、宏定义和 feature flags。

## 1 编译时配置

### 1.1 GN 配置变量

#### 基础配置

| 变量名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `update_app_module_name` | string | "update_app" | 模块名称 |
| `update_app_version` | string | "1.0.0" | 模块版本 |
| `build_type` | string | "release" | 构建类型 |
| `cpp_std` | string | "c++17" | C++ 标准 |
| `warning_level` | string | "all" | 警告级别 |

#### 优化配置

| 变量名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `optimize` | string | "speed" | 优化级别 |
| `enable_neon` | bool | true | 启用 NEON 优化 |
| `enable_multithread` | bool | true | 启用多线程 |
| `enable_lto` | bool | false | 启用链接时优化 |

#### 功能配置

| 变量名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `enable_backup` | bool | true | 启用备份功能 |
| `enable_rollback` | bool | true | 启用回滚功能 |
| `enable_debug_log` | bool | false | 启用调试日志 |
| `enable_trace` | bool | false | 启用跟踪 |
| `use_brotli` | bool | false | 使用 Brotli 压缩 |

### 1.2 GN 配置示例

```bash
# 调试构建
gn gen out/debug --args='
build_type = "debug"
enable_debug_log = true
enable_trace = true
optimize = "none"
'

# 性能构建
gn gen out/release --args='
build_type = "release"
enable_neon = true
enable_multithread = true
enable_lto = true
'

# 功能完整构建
gn gen out/full --args='
build_type = "release"
enable_backup = true
enable_rollback = true
use_brotli = true
enable_lto = true
'
```

### 1.3 配置模板

```gn
# config_extension.gni

# 功能开关
declare_args() {
  enable_feature_a = true
  enable_feature_b = false
  enable_feature_c = true
}

# 阈值配置
declare_args() {
  max_patch_size = 209715200  # 200MB
  max_retry_count = 3
  download_timeout = 300      # 5分钟
  verify_timeout = 60         # 1分钟
}

# 配置验证
template("validated_config") {
  config(target_name) {
    asserts = [
      # 验证阈值范围
      "${invoker.max_patch_size}" <= 500 * 1024 * 1024,
      "${invoker.max_retry_count}" <= 10,
      "${invoker.download_timeout}" <= 600,
    ]
  }
}
```

## 2 编译宏定义

### 2.1 功能宏

| 宏名 | 定义位置 | 说明 |
|------|----------|------|
| `UPDATE_VERSION` | BUILD.gn | 模块版本 |
| `UPDATE_BUILD_TYPE` | BUILD.gn | 构建类型 |
| `ENABLE_BACKUP` | BUILD.gn | 启用备份 |
| `ENABLE_ROLLBACK` | BUILD.gn | 启用回滚 |
| `ENABLE_DEBUG_LOG` | BUILD.gn | 启用调试日志 |
| `ENABLE_TRACE` | BUILD.gn | 启用跟踪 |
| `USE_NEON` | BUILD.gn | 使用 NEON |
| `USE_MULTITHREAD` | BUILD.gn | 使用多线程 |
| `USE_BROTLI` | BUILD.gn | 使用 Brotli |

### 2.2 安全宏

| 宏名 | 定义位置 | 说明 |
|------|----------|------|
| `VERIFY_REQUIRE_SIGNATURE` | BUILD.gn | 必需签名验证 |
| `VERIFY_REQUIRE_TIMESTAMP` | BUILD.gn | 必需时间戳验证 |
| `PATH_VALIDATION_STRICT` | BUILD.gn | 严格路径验证 |
| `ENABLE_SECURITY_LOG` | BUILD.gn | 启用安全日志 |

### 2.3 调试宏

| 宏名 | 定义位置 | 说明 |
|------|----------|------|
| `DEBUG_LOG` | BUILD.gn | 调试日志开关 |
| `TRACE_FUNCTION` | BUILD.gn | 函数跟踪 |
| `TRACE_CALLCHAIN` | BUILD.gn | 调用链跟踪 |
| `DUMP_PACKET` | BUILD.gn | 数据包转储 |
| `DUMP_PATCH` | BUILD.gn | 补丁转储 |

### 2.4 宏使用示例

```cpp
// 版本信息
#ifdef UPDATE_VERSION
#define VERSION_STRING UPDATE_VERSION
#else
#define VERSION_STRING "unknown"
#endif

// 功能开关
#ifdef ENABLE_BACKUP
void Backup(const std::string& path) {
    // 备份实现
}
#else
void Backup(const std::string& path) {
    // 空实现或返回错误
}
#endif

// 调试日志
#ifdef DEBUG_LOG
#define LOG_DEBUG(fmt, ...) \
    printf("[DEBUG] " fmt "\n", ##__VA_ARGS__)
#else
#define LOG_DEBUG(fmt, ...) ((void)0)
#endif
```

## 3 运行时配置

### 3.1 update.cfg 配置

#### 基础配置

```json
{
  "version": "1.0.0",
  
  "update": {
    "max_patch_size": 209715200,
    "max_retry_count": 3,
    "retry_interval": 5,
    "backup_enabled": true,
    "rollback_enabled": true,
    "auto_update": false,
    "require_signature": true
  },
  
  "download": {
    "max_concurrent": 3,
    "timeout": 300,
    "buffer_size": 8192,
    "use_proxy": false,
    "verify_ssl": true
  },
  
  "verify": {
    "require_signature": true,
    "require_timestamp": true,
    "hash_algorithm": "sha256",
    "signature_algorithm": "sha256WithRSA"
  },
  
  "paths": {
    "temp_dir": "/data/update/temp",
    "backup_dir": "/data/update/backup",
    "patch_dir": "/data/update/patch",
    "log_dir": "/data/update/log"
  },
  
  "cache": {
    "enabled": true,
    "max_size": 104857600,
    "ttl": 86400
  }
}
```

### 3.2 环境变量

| 变量名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `UPDATE_DEBUG` | bool | false | 启用调试模式 |
| `UPDATE_LOG_LEVEL` | string | "INFO" | 日志级别 |
| `UPDATE_TRACE` | bool | false | 启用跟踪 |
| `UPDATE_SKIP_VERIFY` | bool | false | 跳过验证 |
| `UPDATE_MOCK_SERVER` | string | "" | Mock 服务器 URL |

```bash
# 设置环境变量
export UPDATE_DEBUG=1
export UPDATE_LOG_LEVEL=DEBUG
export UPDATE_TRACE=1

# 运行
./update_cli check
```

### 3.3 命令行参数

| 参数 | 说明 |
|------|------|
| `--help` | 显示帮助 |
| `--version` | 显示版本 |
| `--config <path>` | 指定配置文件 |
| `--verbose` | 详细输出 |
| `--debug` | 调试模式 |
| `--trace` | 跟踪模式 |
| `--timeout <sec>` | 超时时间 |
| `--skip-verify` | 跳过验证 |

```bash
# 使用示例
./update_cli check --verbose
./update_cli download --config /path/to/config.json
./update_cli apply --skip-verify
./update_cli --version
```

## 4 Feature Flags

### 4.1 功能开关

| Flag | 默认值 | 说明 |
|------|--------|------|
| `FEATURE_BACKUP` | enabled | 备份功能 |
| `FEATURE_ROLLBACK` | enabled | 回滚功能 |
| `FEATURE_INCREMENTAL` | enabled | 增量更新 |
| `FEATURE_FULLUPDATE` | enabled | 全量更新 |
| `FEATURE_BACKGROUND` | disabled | 后台更新 |
| `FEATURE_SCHEDULED` | disabled | 定时更新 |
| `FEATURE_WIFI_ONLY` | disabled | 仅 Wi-Fi 下载 |
| `FEATURE_AUTO_INSTALL` | disabled | 自动安装 |

### 4.2 实验性功能

| Flag | 默认值 | 说明 | 风险 |
|------|--------|------|------|
| `FEATURE_DELTA_V2` | disabled | Delta V2 算法 | 低 |
| `FEATURE_COMPRESS_1` | disabled | LZ4 压缩 | 低 |
| `FEATURE_SIGN_V4` | disabled | V4 签名 | 中 |
| `FEATURE_VERIFY_PARALLEL` | disabled | 并行校验 | 中 |

### 4.3 开关配置

```json
{
  "features": {
    "backup": {
      "enabled": true,
      "keep_count": 3,
      "compression": "zstd"
    },
    
    "rollback": {
      "enabled": true,
      "timeout": 60
    },
    
    "incremental": {
      "enabled": true,
      "min_saving": 0.2,
      "algorithm": "bsdiff"
    },
    
    "experimental": {
      "delta_v2": false,
      "compress_lz4": false,
      "sign_v4": false,
      "verify_parallel": false
    }
  }
}
```

## 5 安全配置

### 5.1 安全开关

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `require_signature` | bool | true | 必需签名 |
| `require_timestamp` | bool | true | 必需时间戳 |
| `min_signature_version` | int | 2 | 最小签名版本 |
| `allow_self_signed` | bool | false | 允许自签名 |
| `check_revocation` | bool | true | 检查证书吊销 |

### 5.2 证书配置

```json
{
  "security": {
    "signature": {
      "required": true,
      "min_version": 2,
      "algorithms": ["sha256WithRSA", "sha256WithECDSA"]
    },
    
    "certificate": {
      "required": true,
      "check_expiry": true,
      "check_revocation": true,
      "trusted_roots": [
        "/etc/ssl/certs/root_ca.pem"
      ]
    },
    
    "timestamp": {
      "required": true,
      "max_age": 2592000,
      "providers": ["http://timestamp.digicert.com"]
    },
    
    "integrity": {
      "algorithm": "sha256",
      "block_size": 4096
    }
  }
}
```

## 6 日志配置

### 6.1 日志级别

| 级别 | 值 | 说明 |
|------|------|------|
| `ERROR` | 1 | 错误 |
| `WARN` | 2 | 警告 |
| `INFO` | 3 | 信息 |
| `DEBUG` | 4 | 调试 |
| `VERBOSE` | 5 | 详细 |

### 6.2 日志配置

```json
{
  "logging": {
    "level": "INFO",
    "console": {
      "enabled": true,
      "level": "INFO"
    },
    "file": {
      "enabled": true,
      "path": "/data/update/log/update.log",
      "level": "DEBUG",
      "max_size": 10485760,
      "max_files": 5
    },
    "remote": {
      "enabled": false,
      "url": "http://log-server/upload",
      "batch_size": 100
    }
  }
}
```

## 7 性能配置

### 7.1 性能参数

| 参数 | 类型 | 默认值 | 范围 |
|------|------|--------|------|
| `max_workers` | int | 4 | 1-16 |
| `io_pool_size` | int | 2 | 1-8 |
| `memory_limit` | int | 104857600 | 1MB-1GB |
| `cache_limit` | int | 52428800 | 1MB-500MB |
| `connection_timeout` | int | 30 | 5-300 |

### 7.2 性能配置

```json
{
  "performance": {
    "threads": {
      "worker_count": 4,
      "io_count": 2,
      "max_tasks": 100
    },
    
    "memory": {
      "limit": 104857600,
      "pool_size": 8388608,
      "max_allocation": 33554432
    },
    
    "cache": {
      "enabled": true,
      "memory_size": 52428800,
      "disk_size": 104857600,
      "ttl": 3600
    },
    
    "network": {
      "connections": 3,
      "timeout": 30,
      "buffer_size": 65536,
      "keep_alive": true
    }
  }
}
```

## 8 配置验证

### 8.1 配置检查

```cpp
// 配置验证器
class ConfigValidator {
public:
    static ValidationResult Validate(const Config& config) {
        // 检查阈值
        if (config.update.max_patch_size > MAX_PATCH_SIZE) {
            return ValidationError("max_patch_size too large");
        }
        
        // 检查超时
        if (config.download.timeout > MAX_TIMEOUT) {
            return ValidationError("timeout too large");
        }
        
        // 检查线程数
        if (config.performance.threads.worker_count > MAX_THREADS) {
            return ValidationError("too many worker threads");
        }
        
        return ValidationOK();
    }
};
```

### 8.2 配置示例

```bash
# 最小配置
{
  "version": "1.0.0",
  "update": {
    "max_patch_size": 209715200
  }
}

# 生产配置
{
  "version": "1.0.0",
  "update": {
    "max_patch_size": 209715200,
    "max_retry_count": 3,
    "backup_enabled": true,
    "rollback_enabled": true,
    "require_signature": true
  },
  "download": {
    "max_concurrent": 3,
    "timeout": 300,
    "verify_ssl": true
  },
  "verify": {
    "hash_algorithm": "sha256",
    "signature_algorithms": ["sha256WithRSA"]
  }
}
```

## 9 相关文档

| 文档 | 描述 |
|------|------|
| [05_Build_System.md](./05_Build_System.md) | GN 构建配置 |
| [06_Build_Artifacts.md](./06_Build_Artifacts.md) | 编译产物 |
| [07_Security_Review.md](./07_Security_Review.md) | 安全配置 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 故障排查 |
