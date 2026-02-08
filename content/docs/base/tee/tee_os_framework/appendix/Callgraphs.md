# 关键调用链

## 1. TA 加载流程

```
CA (REE)
    │
    ├── SMC 调用 (LOAD_SECURE_APP)
    │   └── smc_cmd_t { cmd_id: GLOBAL_CMD_ID_LOAD_SECURE_APP }
    │
    ▼
TZDriver / ATF
    │
    ├── 设置 SMC 队列 in_bitmap
    └──触发 SMC 异常
        │
        ▼
teesmcmgr (tee_smc_thread)
    │
    ├── smc_wait_switch_req()
    └── ipc_msg_notification() → gtask
        │
        ▼
gtask (dispatch_ns_cmd)
    │
    ├── get_last_in_cmd()
    │   └── acquire_smc_buf_lock()
    │       └── copy smc_cmd_t from queue
    │
    ├── dispatch_ns_global_cmd()
    │   └── case GLOBAL_CMD_ID_LOAD_SECURE_APP:
    │       └── need_load_app()
    │           └── tee_app_load_srv()
    │               └── load_secure_file_image()
    │                   │
    │                   └── perm_srv_elf_verify()
    │                       │
    │                       └── ipc_msg_call() → permission_service
    │                           │
    │                           ▼
    │                       permission_service
    │                           │
    │                           ├── 验证签名
    │                           ├── 验证证书链
    │                           └── ipc_msg_reply()
    │
    ├── spawn_ta_task()
    │   └── sre_task_create() → tarunner
    │       │
    │       └── tarunner.main()
    │           │
    │           ├── dlopen() 加载 ELF
    │           └── call TA_CreateEntryPoint()
    │
    └── async_call_ta_entry()
        └── ipc_msg_snd() → TA Process
            │
            ▼
        TA Process
            │
            └── TA_OpenSessionEntryPoint()
```

---

## 2. CA → TA 命令调用

```
CA (REE)
    │
    ├── SMC 调用 (INVOKE_COMMAND)
    │   └── smc_cmd_t {
    │       cmd_id: GLOBAL_CMD_ID_INVOKE_COMMAND,
    │       context: session_id,
    │       operation_phys: params_paddr
    │   }
    │
    ▼
teesmcmgr → gtask
    │
    ├── get_last_in_cmd()
    └── dispatch_ns_cmd()
        │
        └── case CMD_TYPE_TA:
            └── start_ta_task()
                │
                ├── check_session_context()
                │   └── validate session_bitmap
                │
                ├── alloc_session()
                │   └── find free session_id
                │
                ├── async_call_ta_entry()
                │   └── ipc_msg_snd() → TA
                │       │
                │       └── global_to_ta_msg {
                │           cmd_id: CALL_TA_INVOKE_COMMAND,
                │           params: TEE_Param[4]
                │       }
                │
                └── ipc_msg_reply() → REE
                    │
                    └── put_last_out_cmd()
                        └── copy response to out queue

TA Process
    │
    ├── ipc_msg_rcv()
    │   └── receive global_to_ta_msg
    │
    ├── TA_InvokeCommandEntryPoint()
    │   └── switch(cmd_id)
    │       └── handle command
    │
    └── ipc_msg_snd() → gtask
        │
        └── ta_to_global_msg {
            ret: TEE_Result,
            session_context: uint64_t
        }
```

---

## 3. 安全存储操作

```
TA
    │
    ├── TEE_CreatePersistentObject()
    │   │
    │   └── IPC → FS Agent (TEE_FS_AGENT_ID)
    │       │
    │       ├── huk_srv_derive_ta_root_key()
    │       │   └── huk_service
    │       │       └── do_derive_takey()
    │       │           └── tee_crypto_derive_root_key()
    │       │               └── derive TA Root Key
    │       │
    │       ├── derive_file_key()
    │       │   └── CMAC(TA_Root_Key, salt)
    │       │
    │       ├── TEE_CreateObject()
    │       │   └── create meta_header
    │       │       └── encrypt with AES-XTS
    │       │
    │       └── return object_handle
    │
    ├── TEE_WriteObjectData()
    │   │
    │   ├── HMAC(data) → hmac_key
    │   └── encrypt(data) → AES-XTS
    │       └── write to SFS
    │
    └── TEE_ReadObjectData()
        │
        ├── read encrypted data
        ├── decrypt with AES-XTS
        ├── verify HMAC
        └── return decrypted data
```

---

## 4. TA → TA 通信

```
TA_A
    │
    ├── TEE_OpenTASession(TA_B_UUID)
    │   │
    │   ├── IPC → gtask
    │   │   └── handle_ta2ta_cmd()
    │   │       └── start_ta_task() → TA_B
    │   │
    │   └── return session_handle
    │
    └── TEE_InvokeTACommand()
        │
        ├── IPC → gtask
        │   └── ipc_msg_snd() → TA_B
        │
        └── wait for response

gtask
    │
    ├── receive TA_A's request
    ├── start_ta_task(TA_B)
    │   └── ipc_msg_snd() → TA_B
    │
    └── queue TA_A's context

TA_B
    │
    ├── ipc_msg_rcv()
    │   └── receive TA2TA request
    │
    ├── TA_InvokeCommandEntryPoint()
    │   └── handle command
    │
    └── ipc_msg_snd() → gtask
        │
        └── response with session_context
            │
            ▼
        gtask
            │
            ├── find TA_A's session
            └── forward response to TA_A
```

---

## 5. TA → Driver 通信

```
TA
    │
    ├── TEE_DrvOpen()
    │   │
    │   └── IPC → drvmgr
    │       │
    │       ├── caller_open_auth_check()
    │       │   └── check TA permissions
    │       │
    │       ├── get_valid_drv_node()
    │       │   └── spawn_driver_handle() if needed
    │       │       └── posix_spawn_ex() → tarunner
    │       │           └── load driver ELF
    │       │
    │       └── drv_open_handle()
    │           └── IPC → Driver Process
    │
    └── TEE_DrvIoctl()
        │
        └── IPC → drvmgr → Driver
            │
            ├── ioctl request
            ├── driver processes
            └── return result
```

---

## 6. 密钥派生调用链

```
TA
    │
    ├── huk_srv_derive_ta_root_key()
    │   └── IPC → huk_service
    │       │
    │       ├── huk_task_takey_param_check()
    │       │   └── validate salt and key buffers
    │       │
    │       ├── huk_srv_map_from_task()
    │       │   └── map sender's shared memory
    │       │
    │       ├── huk_derive_takey()
    │       │   └── combine salt + UUID
    │       │
    │       └── do_derive_takey()
    │           └── tee_crypto_derive_root_key()
    │               └── CMAC(HUK, salt + UUID)
    │                   │
    │                   └── secure cleanup
    │                       └── memset_s(key_buffer)
    │
    └── derive_file_key()
        │
        ├── CMAC(TA_Root_Key, FILEKEY_SALT)
        ├── CMAC(TA_Root_Key, ENCRYPTION1_SALT)
        └── CMAC(TA_Root_Key, ENCRYPTION2_SALT)
```

---

## 7. ELF 验证调用链

```
gtask
    │
    └── load_secure_file_image()
        │
        ├── perm_srv_elf_verify()
        │   └── IPC → permission_service
        │       │
        │       └── perm_thread_handle_async_file_msg()
        │           │
        │           └── perm_srv_elf_verify()
        │               │
        │               ├── secure_elf_verify()
        │               │   ├── unpack_copy_check_params()
        │               │   │   └── validate buffer
        │               │   │
        │               │   ├── tee_secure_img_unpack_v3()
        │               │   │   ├── decrypt cipher layer
        │               │   │   ├── verify signature
        │               │   │   └── parse manifest
        │               │   │
        │               │   └── tee_secure_img_unpack_v2()
        │               │       └── legacy format handling
        │               │
        │               └── perm_srv_ta_run_authorization_check()
        │                   ├── check_ta_not_deactivated()
        │                   ├── verify_manifest_properties()
        │                   ├── anti_version_rollback()
        │                   └── check_heap_stack_limits()
        │
        └── return verified image info
```

---

## 8. IPC 消息流

```
┌─────────────────────────────────────────────────────────────────┐
│                    IPC 消息类型                                  │
├───────────────────┬─────────────────────────────────────────────┤
│ MSG_TYPE_NOTIF    │ 异步通知，无回复                             │
│ MSG_TYPE_CALL     │ 同步调用，需要回复                           │
└───────────────────┴─────────────────────────────────────────────┘

发送消息:
    ipc_msg_snd(dst, msg, len, hdl, flags)
        │
        ├── validate dst (channel exists)
        ├── copy msg to buffer
        └── notify dst task

接收消息:
    ipc_msg_rcv(channel, buffer, size, hdl, info, timeout)
        │
        ├── wait for message
        ├── copy from buffer
        └── return message

同步调用:
    ipc_msg_call(dst, msg, rsp, timeout)
        │
        ├── ipc_msg_snd() → MSG_TYPE_CALL
        ├── wait for response
        └── return result
```

---

## 相关文档

- 架构设计 → `01_Architecture.md`
- 模块详情 → `02_Module_Detail.md`
- Native API → `03_Native_API.md`
- 安全评审 → `05_Security_Review.md`
