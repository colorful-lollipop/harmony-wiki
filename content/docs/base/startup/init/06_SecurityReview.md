# 安全风险评估

本文档对 init 模块进行深度安全分析，识别潜在漏洞并提供修复建议。

---

## R1: 服务路径未验证（高危）

**位置**：`services/init/lite/init_service.c:97`

**证据**：
```c
int ServiceExec(Service *service, const ServiceArgs *pathArgs)
{
    INIT_ERROR_CHECK(pathArgs != NULL && pathArgs->count > 0,
        return SERVICE_FAILURE, "Exec service failed! null ptr.");

    // 无路径验证，直接执行
    INIT_ERROR_CHECK(execv(pathArgs->argv[0], pathArgs->argv) == 0,
        service->lastErrno = INIT_EEXEC;
        return errno, "[startup_failed]failed to execv %d %d %s", isCritical, errno, service->name);
    return SERVICE_SUCCESS;
}
```

**触发路径**：
```
init.cfg "path" 字段 → ParseInitCfg() → ServiceExec() → execv()
```

**影响**：
- **可利用性**：高。攻击者可修改 init.cfg 配置任意服务路径
- **权限提升**：以 root 权限执行任意程序
- **影响范围**：整个系统被完全控制

**修复建议**：
```c
// 添加路径白名单检查
static const char *ALLOWED_SERVICE_PATHS[] = {
    "/system/bin/",
    "/vendor/bin/",
    "/product/bin/",
};

int ValidateServicePath(const char *path)
{
    for (int i = 0; i < ARRAY_LENGTH(ALLOWED_SERVICE_PATHS); i++) {
        if (strncmp(path, ALLOWED_SERVICE_PATHS[i],
                    strlen(ALLOWED_SERVICE_PATHS[i])) == 0) {
            return 0;  // 白名单内
        }
    }
    return -1;  // 不允许的路径
}
```

---

## R2: exec 命令参数无验证（高危）

**位置**：`services/init/lite/init_cmds.c:31-51`

**证据**：
```c
static void DoExec(const struct CmdArgs *ctx)
{
    if (ctx == NULL || ctx->argv[0] == NULL) {
        INIT_LOGE("DoExec: invalid arguments");
        return;
    }
    pid_t pid = fork();
    if (pid < 0) {
        INIT_LOGE("DoExec: failed to fork child process");
        return;
    }
    if (pid == 0) {
        // 直接执行用户配置的任意命令
        int ret = execve(ctx->argv[0], ctx->argv, NULL);
        if (ret == -1) {
            INIT_LOGE("DoExec: execute \"%s\" failed", ctx->argv[0]);
        }
        _exit(0x7f);
    }
}
```

**触发路径**：
```
init.cfg "cmds": ["exec /bin/sh -c malicious"] → DoJob() → DoExec()
```

**影响**：
- **可利用性**：高。init.cfg 中的 exec 命令可直接执行任意程序
- **权限提升**：以 root 权限执行任意命令
- **影响范围**：系统完全被控

**修复建议**：
1. 禁用 exec 命令
2. 或添加可执行文件白名单
3. 或限制 exec 只能执行系统二进制文件

---

## R3: Jobs 命令缺少参数长度检查（中危）

**位置**：`services/init/lite/init_jobs.c:71-80`

**证据**：
```c
static void ParseJob(const cJSON *jobItem, Job *resJob)
{
    cJSON *cmdsItem = cJSON_GetObjectItem(jobItem, CMDS_ARR_NAME_IN_JSON);
    if (!cJSON_IsArray(cmdsItem)) {
        INIT_LOGE("job %s is not an array", resJob->name);
        return;
    }
    int ret = GetCmdLinesFromJson(cmdsItem, &resJob->cmdLines);
    if (ret != 0) {
        INIT_LOGE("ParseJob, failed to get cmds for job!");
        return;
    }
}
// TODO: 未发现 MAX_ONE_ARG_LEN 的使用位置
```

**触发路径**：
```
init.cfg "cmds" 字段过长 → ParseAllJobs() → 缓冲区溢出风险
```

**影响**：
- **可利用性**：中。超长命令可能导致栈/堆溢出
- **影响范围**：服务崩溃或代码执行

**修复建议**：
```c
// 在 GetCmdLinesFromJson 中添加长度检查
#define MAX_CMD_LEN 256
#define MAX_ARGS_COUNT 20

int GetCmdLinesFromJson(cJSON *cmdsItem, CmdLines **cmdLines)
{
    int cmdNum = cJSON_GetArraySize(cmdsItem);
    if (cmdNum > MAX_ARGS_COUNT) {
        INIT_LOGE("Too many commands: %d, max: %d", cmdNum, MAX_ARGS_COUNT);
        return -1;
    }

    for (int i = 0; i < cmdNum; i++) {
        cJSON *cmdItem = cJSON_GetArrayItem(cmdsItem, i);
        const char *cmdStr = cJSON_GetStringValue(cmdItem);
        if (cmdStr == NULL) {
            continue;
        }
        if (strlen(cmdStr) > MAX_CMD_LEN) {
            INIT_LOGE("Command too long: %zu, max: %d", strlen(cmdStr), MAX_CMD_LEN);
            return -1;
        }
    }
    return 0;
}
```

---

## R4: 路径遍历风险（中危）

**位置**：`services/init/lite/init_cmds.c`（mkdir/mount 命令）

**触发路径**：
```
init.cfg "cmds": ["mkdir /data/../../../etc/malicious"] → DoCmd() → mkdir()
```

**影响**：
- **可利用性**：中。攻击者可能写入恶意配置文件到受保护目录
- **影响范围**：权限绕过，文件写入越权

**修复建议**：
```c
// 添加路径规范化验证
char *NormalizePath(const char *path)
{
    char realPath[PATH_MAX];
    if (realpath(path, realPath) == NULL) {
        return NULL;  // 路径无效
    }
    // 检查是否在允许的目录内
    if (strncmp(realPath, ALLOWED_BASE_PATH, strlen(ALLOWED_BASE_PATH)) != 0) {
        return NULL;  // 路径越界
    }
    return strdup(realPath);
}
```

---

## R5: SIGCHLD 处理中的 TOCTOU 风险（中危）

**位置**：`services/init/lite/init_signal_handler.c:47-61`

**证据**：
```c
static void SigHandler(int sig)
{
    switch (sig) {
        case SIGCHLD: {
            pid_t sigPID;
            int procStat = 0;
            while (1) {
                sigPID = waitpid(-1, &procStat, WNOHANG);
                if (sigPID <= 0) {
                    break;
                }
                // TOCTOU: GetServiceByPid() 时 pid 可能已无效
                ReapService(GetServiceByPid(sigPID));
            }
            break;
        }
    }
}
```

**触发路径**：
```
子进程退出 → SIGCHLD → SigHandler() → waitpid() → GetServiceByPid()
```

**影响**：
- **可利用性**：中。竞态条件可能导致使用已释放的服务对象
- **影响范围**：服务管理混乱，可能跳过某些退出事件

**修复建议**：
```c
static void SigHandler(int sig)
{
    if (sig == SIGCHLD) {
        // 使用信号安全的方式收集状态
        siginfo_t info;
        while (waitid(P_ALL, 0, &info, WNOHANG | WEXITED) == 0) {
            if (info.si_pid == 0) {
                break;  // 没有更多子进程
            }
            // 直接使用收集到的信息
            HandleChildExit(info.si_pid, info.si_status, info.si_code);
        }
    }
}
```

---

## R6: 关键进程导致系统无限重启（中危）

**位置**：`services/init/lite/init_signal_handler.c:39-42`

**证据**：
```c
void ReapService(Service *service)
{
    if (service == NULL) {
        return;
    }
    if (service->attribute & SERVICE_ATTR_IMPORTANT) {
        // 关键进程退出 → 直接重启系统
        service->pid = -1;
        StopAllServices(0, NULL, 0, NULL);
        RebootSystem();  // 无额外验证
    }
    ServiceReap(service);
}
```

**触发路径**：
```
importance=1 服务崩溃 → SIGCHLD → ReapService() → RebootSystem()
```

**影响**：
- **可利用性**：中。攻击者使关键进程崩溃即可导致系统重启
- **影响范围**：拒绝服务
- **附加风险**：可能形成重启循环（如果关键进程本身有问题）

**修复建议**：
```c
#define MAX_RESTART_COUNT 5
#define RESTART_WINDOW_SECONDS 60

typedef struct {
    pid_t pid;
    time_t lastExitTime;
    int restartCount;
} ServiceRestartInfo;

int ShouldRebootForImportantService(Service *service)
{
    static ServiceRestartInfo info = {0};
    time_t now = time(NULL);

    if (info.pid != service->pid || info.pid == 0) {
        // 新服务，重置计数器
        info.pid = service->pid;
        info.lastExitTime = now;
        info.restartCount = 0;
        return 0;  // 不重启
    }

    // 检查是否在时间窗口内频繁重启
    if (difftime(now, info.lastExitTime) < RESTART_WINDOW_SECONDS) {
        info.restartCount++;
        if (info.restartCount >= MAX_RESTART_COUNT) {
            // 频繁重启，停止重启，进入安全模式
            INIT_LOGE("Important service restarting too frequently, entering safe mode");
            return -1;  // 停止重启，不执行 RebootSystem()
        }
    } else {
        // 重置计数器
        info.restartCount = 0;
        info.lastExitTime = now;
    }

    return 0;  // 正常重启
}
```

---

## R7: 配置文件路径验证不严格（低危）

**位置**：`services/init/lite/init_cmds.c:53-71`

**证据**：
```c
static bool CheckValidCfg(const char *path)
{
    static const char *supportCfg[] = {
        "/etc/patch.cfg",
        "/patch/fstab.cfg",
    };
    struct stat fileStat = { 0 };
    if (stat(path, &fileStat) != 0 || fileStat.st_size <= 0 ||
        fileStat.st_size > LOADCFG_MAX_FILE_LEN) {
        return false;
    }
    size_t cfgCnt = ARRAY_LENGTH(supportCfg);
    for (size_t i = 0; i < cfgCnt; ++i) {
        if (strcmp(path, supportCfg[i]) == 0) {
            return true;
        }
    }
    return false;
}
```

**问题**：
1. 仅硬编码两个路径，灵活性差
2. 未检查路径是否为符号链接
3. 未验证文件所有者

**修复建议**：
```c
static bool CheckValidCfg(const char *path)
{
    // 检查路径是否为绝对路径
    if (path[0] != '/') {
        return false;
    }

    // 解析真实路径
    char realPath[PATH_MAX];
    if (realpath(path, realPath) == NULL) {
        return false;
    }

    // 检查是否在允许的目录内
    const char *allowedDirs[] = {"/etc/", "/patch/"};
    for (int i = 0; i < ARRAY_LENGTH(allowedDirs); i++) {
        if (strncmp(realPath, allowedDirs[i], strlen(allowedDirs[i])) == 0) {
            // 文件存在性检查
            struct stat fileStat;
            if (stat(realPath, &fileStat) == 0 &&
                S_ISREG(fileStat.st_mode) &&
                fileStat.st_size > 0 &&
                fileStat.st_size <= LOADCFG_MAX_FILE_LEN) {
                return true;
            }
        }
    }
    return false;
}
```

---

## R8: 缺少配置完整性验证（中危）

**问题描述**：
init.cfg 文件缺少完整性校验机制。

**触发路径**：
```
恶意 init.cfg → ParseInitCfg() → 使用恶意配置
```

**影响**：
- **可利用性**：中。攻击者可修改 init.cfg 植入恶意配置
- **缓解因素**：vendor 分区通常只读，且可能有 dm-verity 保护

**修复建议**：
```c
// 在 ParseInitCfg 开头添加完整性验证
int ParseInitCfg(const char *configFile, void *context)
{
#ifdef ENABLE_CFG_INTEGRITY_CHECK
    // 验证配置文件完整性
    int fd = open(configFile, O_RDONLY);
    if (fd < 0) {
        INIT_LOGE("Cannot open config file: %s", configFile);
        return -1;
    }

    // 读取并验证签名
    char sig[256];
    size_t sigLen = sizeof(sig);
    if (ReadFileSignature(fd, sig, &sigLen) < 0) {
        INIT_LOGE("Invalid config signature: %s", configFile);
        close(fd);
        return -1;
    }

    // 验证签名
    if (VerifySignature(configFile, sig) != 0) {
        INIT_LOGE("Config signature verification failed: %s", configFile);
        close(fd);
        return -1;
    }

    close(fd);
#endif

    // 继续解析配置...
}
```

---

## 风险汇总表

| ID | 风险名称 | 等级 | 可利用性 | 影响 |
|----|---------|------|----------|------|
| R1 | 服务路径未验证 | **高危** | 高 | 系统完全控制 |
| R2 | exec 命令无验证 | **高危** | 高 | 任意代码执行 |
| R3 | 命令长度无检查 | **中危** | 中 | 溢出风险 |
| R4 | 路径遍历 | **中危** | 中 | 文件写入越权 |
| R5 | SIGCHLD TOCTOU | **中危** | 中 | 竞态条件 |
| R6 | 关键进程重启循环 | **中危** | 中 | 拒绝服务 |
| R7 | 配置路径验证不严 | **低危** | 低 | 路径注入 |
| R8 | 配置完整性缺失 | **中危** | 中 | 配置篡改 |

---

## 安全加固优先级

| 优先级 | 风险 | 加固措施 |
|--------|------|----------|
| P0 | R1, R2 | 添加路径白名单和命令验证 |
| P1 | R8 | 添加配置签名验证 |
| P2 | R4, R7 | 路径规范化 + 链接检查 |
| P3 | R6 | 添加重启频率限制 |
| P4 | R3, R5 | 添加长度检查和竞态防护 |

---

## 相关文档

- [攻击面分析](05_AttackSurface.md)
- [项目概览](01_Overview.md)
- [架构说明](02_Architecture.md)
- [代码地图](03_CodeMap.md)
- [构建配置](05_Build.md)
