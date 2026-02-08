# 关键调用链

## 1. 策略加载调用链

### 1.1 LoadPolicy 入口

```
init_lite (system startup)
    │
    └── LoadPolicy() [policycoreutils.h:25]
            │
            ├── load_policy.cpp:LoadPolicyImpl()
            │        │
            │        ├── open("/etc/selinux/targeted/policy/policy.31", O_RDONLY)
            │        │         │
            │        │         └── 读取策略文件
            │        │
            │        └── Selinux_load_policy() [libselinux]
            │                  │
            │                  ├── open("/sys/fs/selinux/policy", O_WRONLY)
            │                  └── write(fd, policy_data, policy_size)
            │
            └── return 0 / error_code
```

**证据来源**:
- `interfaces/policycoreutils/include/policycoreutils.h:25` - 函数声明
- `framework/policycoreutils/src/load_policy.cpp` - 实现

### 1.2 进程标签设置

```
init_lite
    │
    └── setcon(service_context)
            │
            └── selinux_setexeccon() [libselinux]
                      │
                      └── Linux kernel SELinux
                                │
                                └── 设置进程的 scontext
```

## 2. 文件恢复调用链

### 2.1 Restorecon 通用接口

```
调用者
    │
    └── RestoreconCommon(path, flag, nthreads) [policycoreutils.h:31]
            │
            ├── Selinux_restorecon_init()
            │        │
            │        ├── fts_open() [FreeBSD]
            │        │         │
            │        │         └── 初始化文件遍历
            │        │
            │        └── restorecon_flags()
            │                  └── 解析 flag
            │
            ├── fts_read() [FreeBSD]
            │        │
            │        └── 遍历目录条目
            │
            └── Selinux_lsetfilecon()
                       │
                       └── libselinux:lsetfilecon()
                                 │
                                 └── kernel: setxattr(SELINUX_LABEL)
```

**证据来源**:
- `framework/policycoreutils/src/selinux_restorecon.c` - 实现
- `policycoreutils.h:31` - 接口定义

### 2.2 并行恢复

```
RestoreconRecurseParallel(path, nthreads)
    │
    ├── 创建线程池 (nthreads)
    │
    ├── 分割目录为多个任务
    │
    ├── 线程执行:
    │    │
    │    ├── fts_read()  // 每个线程独立遍历
    │    │
    │    └── lsetfilecon() // 设置标签
    │
    └── 等待所有线程完成
```

## 3. 参数检查调用链

### 3.1 SetParamCheck

```
参数服务 (收到 SET 请求)
    │
    └── SetParamCheck(paraName, destContext, &info) [param_checker.h:43]
            │
            ├── GetParamLabel(paraName) [selinux_parameter.h:74]
            │        │
            │        ├── contexts_trie.c:TrieFind()
            │        │         │
            │        │         └── 查找参数上下文
            │        │
            │        └── return context / NULL
            │
            ├── GetParamLabelIndex(paraName)
            │        │
            │        └── return index
            │
            └── selinux_check_access()
                       │
                       ├── 获取调用者 scontext
                       ├── 获取目标 tcontext
                       ├── 检查权限
                       └── return 0 (允许) / -1 (拒绝)
```

**证据来源**:
- `interfaces/policycoreutils/include/param_checker.h:43`
- `framework/policycoreutils/src/param_checker.c`
- `interfaces/policycoreutils/include/selinux_parameter.h:74`

## 4. 服务检查调用链

### 4.1 ServiceChecker

```
SAMgr (收到 GetService 请求)
    │
    └── ServiceChecker::GetServiceCheck(callingSid, serviceName) [service_checker.h:36]
            │
            ├── GetServiceContext(serviceName)
            │        │
            │        ├── 读取 service_contexts
            │        │
            │        └── 查找服务上下文
            │
            └── CheckPerm(callingSid, serviceName, "get")
                       │
                       └── selinux_check_access()
                                  │
                                  ├── 读取 callingSid 的域
                                  ├── 读取 serviceName 的类型
                                  └── 检查 { get_service } 权限
```

### 4.2 HDF ServiceChecker

```
HDF Framework (收到 GetService 请求)
    │
    └── HdfGetServiceCheck(callingSid, serviceName) [hdf_service_checker.h:26]
            │
            ├── 读取 hdf_service_contexts
            │
            └── selinux_check_access()
```

**证据来源**:
- `interfaces/policycoreutils/include/service_checker.h`
- `framework/policycoreutils/src/service_checker.cpp`

## 5. HAP 上下文调用链

### 5.1 HAP 文件标签恢复

```
BmsBundleMgr (应用安装)
    │
    └── HapFileRestoreContext::GetInstance()
            │
            └── SetFileConForce(hapFileInfo, remainingNum, resultInfo)
                     │
                     ├── InitRestoreTask(task, hapFileInfo)
                     │        │
                     │        └── 创建 RestoreTask
                     │
                     ├── ProcessRestorePath(task, path, hapFileInfo)
                     │        │
                     │        ├── fts_open() [FreeBSD]
                     │        │
                     │        ├── fts_read()
                     │        │
                     │        └── GetSecontext(...)
                     │                  │
                     │                  ├── HapLabelLookup()
                     │                  │        │
                     │                  │        └── 查表返回 secontext
                     │                  │
                     │                  └── lsetfilecon()
                     │
                     └── FinishRestoreTask(...)
```

### 5.2 HAP 进程域设置

```
AppSpawn (应用启动)
    │
    └── HapContext::HapDomainSetcontext(hapDomainInfo) [hap_restorecon.h:89]
            │
            ├── 解析 hapDomainInfo
            │        │
            │        ├── apl (应用权限级别)
            │        ├── packageName
            │        ├── hapFlags
            │        └── uid
            │
            ├── GetDomainContext(...)
            │        │
            │        └── 根据 apl 查找 domain
            │
            └── selinux_setexeccon()
                       │
                       └── 设置应用进程域
```

**证据来源**:
- `interfaces/policycoreutils/include/hap_restorecon.h:89`
- `framework/policycoreutils/src/hap_restorecon.cpp`

## 6. 策略编译调用链

### 6.1 build_policy.py

```
GN build (build_policy action)
    │
    └── build_policy.py
            │
            ├── 收集策略文件 (*.te)
            │        │
            │        └── sepolicy/base/, sepolicy/ohos_policy/, etc.
            │
            ├── 检查策略语法
            │        │
            │        └── checkpolicy -c 31 -o policy.mod policy/*.te
            │
            ├── 链接策略模块
            │        │
            │        └── checkmodule -M -m -o policy.mod policy/*.te
            │
            └── 编译为二进制策略
                       │
                       └── secilc -c 31 -o policy.31 policy.mod
```

### 6.2 build_contexts.py

```
GN build (build_contexts action)
    │
    ├── 依赖: build_policy (确保 policy.31 存在)
    │
    └── build_contexts.py
            │
            ├── 编译 file_contexts
            │        │
            │        └── sefcontext_compile -o file_contexts.bin file_contexts
            │
            ├── 编译 parameter_contexts
            │        │
            │        └── sefcontext_compile -o parameter_contexts.bin ...
            │
            ├── 编译 service_contexts
            │
            ├── 编译 hdf_service_contexts
            │
            └── 编译 sehap_contexts
```

## 7. 共享内存调用链

### 7.1 参数上下文共享内存

```
InitParamSelinux(isInit)
    │
    ├── SelinuxInitParameter()
    │        │
    │        ├── GetParameterFileSize()
    │        │        │
    │        │        └── 读取 parameter_contexts 大小
    │        │
    │        ├── SelinuxInitShareMem()
    │        │        │
    │        │        ├── shm_open()
    │        │        │
    │        │        ├── ftruncate()
    │        │        │
    │        │        └── mmap()
    │        │
    │        └── LoadParameterContexts()
    │                  │
    │                  └── 读取 parameter_contexts 到共享内存
    │
    └── return 0 / error
```

**证据来源**:
- `framework/policycoreutils/src/selinux_parameter.c`
- `framework/policycoreutils/src/selinux_share_mem.c`

## 8. 错误处理调用链

### 8.1 SELinux 错误处理

```
selinux_check_access() 返回拒绝
    │
    ├── 生成 AVC 拒绝日志
    │        │
    │        └── audit: type=1400 ... avc: denied ...
    │
    ├── hilog 输出错误
    │        │
    │        └── SELINUX_LOG(ERROR, "AVC denied ...")
    │
    └── 返回错误码 (-1)
```

**证据来源**:
- `framework/policycoreutils/src/selinux_error.cpp`
- `framework/policycoreutils/src/selinux_log.c`

## 9. 调用关系图

### 9.1 模块间调用

```
┌──────────────────────────────────────────────────────────────────────┐
│                           调用者                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────────┐    │
│  │ init_lite│ │ BmsBundle│ │ SAMgr    │ │ 参数服务              │    │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └──────────────────────┘    │
├───────┼────────────┼────────────┼─────────────────────────────────────┤
│       │            │            │                                     │
│       ▼            ▼            ▼                                     │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                   libload_policy.so                           │    │
│  │                    LoadPolicy()                               │    │
│  └──────────────────────────────────────────────────────────────┘    │
│       │                                                                │
│       ▼                                                                │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                   librestorecon.so                            │    │
│  │         Restorecon / RestoreconRecurse / RestoreconCommon     │    │
│  └──────────────────────────────────────────────────────────────┘    │
│       │                                                                │
│       ▼                                                                │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                   libhap_restorecon.so                       │    │
│  │            SetFileConForce / HapDomainSetcontext             │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                   libparaperm_checker.so                     │    │
│  │                     SetParamCheck()                          │    │
│  └──────────────────────────────────────────────────────────────┘    │
│       │                                                                │
│       ▼                                                                │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                   libselinux_parameter_static                 │    │
│  │            GetParamLabel / InitParamSelinux                  │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
├──────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                   libservice_checker.so                       │    │
│  │            GetServiceCheck / HdfGetServiceCheck               │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │    libselinux (third_party)   │
              │  - selinux_check_access()     │
              │  - selinux_setexeccon()       │
              │  - lsetfilecon()             │
              └───────────────────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │     Linux Kernel SELinux       │
              │   - 访问控制决策               │
              │   - AVC 缓存                   │
              └───────────────────────────────┘
```

## 10. 相关文档

| 文档 | 链接 |
|------|------|
| 架构设计 | [02_Architecture.md](../02_Architecture.md) |
| API 接口 | [03_API.md](../03_API.md) |
| 构建系统 | [04_Build.md](../04_Build.md) |
| 故障排查 | [06_Troubleshooting.md](../06_Troubleshooting.md) |
