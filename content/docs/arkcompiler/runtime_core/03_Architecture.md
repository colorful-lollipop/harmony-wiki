# 架构说明

## 总体架构

ArkCompiler Runtime Core 采用分层架构设计，实现语言无关的运行时核心能力：

```mermaid
graph TB
    subgraph 应用层
        App[ArkTS 应用]
    end
    
    subgraph 语言运行时层
        ETS[ETS Runtime]
        Interop[JS Interop]
    end
    
    subgraph Runtime Core
        subgraph API层
            ANI[ANI API]
            NAPI[N-API]
        end
        
        subgraph 执行引擎
            Interpreter[解释器]
            JIT[JIT 编译器]
            AOT[AOT 编译器]
        end
        
        subgraph 运行时服务
            TL[类型加载器]
            TM[线程管理]
            MM[内存管理]
        end
        
        subgraph 基础能力
            File[字节码文件]
            Base[基础库]
            ISA[指令集架构]
        end
    end
    
    subgraph 平台抽象层
        OSAL[OS 抽象层]
    end
    
    subgraph 操作系统
        OS[Linux/OpenHarmony]
    end
    
    App --> ETS
    ETS --> Interop
    ETS --> ANI
    Interop --> NAPI
    ANI --> TL
    NAPI --> TL
    TL --> File
    Interpreter --> ISA
    JIT --> MM
    AOT --> File
    TM --> Base
    MM --> Base
    File --> Base
    Base --> OSAL
    OSAL --> OS
```

## 组件架构

### 1. 字节码文件子系统

```mermaid
graph LR
    subgraph 字节码文件子系统
        ABC[ABC 文件]
        File[File 类]
        CDA[ClassDataAccessor]
        MDA[MethodDataAccessor]
        Cache[Metadata Cache]
    end
    
    ABC --> File
    File --> CDA
    File --> MDA
    CDA --> Cache
    MDA --> Cache
```

**职责**:
- 加载和解析 .abc 文件
- 提供类、方法、字段的元数据访问
- 管理常量池和字符串表

**关键类**:
- `panda_file::File` - 字节码文件表示
- `panda_file::ClassDataAccessor` - 类数据访问
- `panda_file::MethodDataAccessor` - 方法数据访问

### 2. 内存管理子系统

```mermaid
graph TB
    subgraph 内存管理子系统
        subgraph 堆空间
            Young[年轻代]
            Tenured[老年代]
            Humongous[大对象区]
        end
        
        subgraph GC 框架
            G1[G1 GC]
            STW[STW GC]
            Epsilon[Epsilon GC]
        end
        
        subgraph 分配器
            TLAB[TLAB 分配器]
            Region[Region 分配器]
            FreeList[FreeList 分配器]
        end
        
        subgraph 内存池
            MMap[MMap 内存池]
            Malloc[Malloc 内存池]
        end
    end
    
    App[应用代码] --> TLAB
    TLAB --> Young
    Young --> Tenured
    Large[大对象] --> Humongous
    
    Young --> G1
    Tenured --> G1
    G1 --> MMap
    
    TLAB --> Region
    Region --> MMap
    
    MMap --> OS[操作系统]
    Malloc --> OS
```

**职责**:
- 托管内存分配与回收
- GC 策略实现
- 内存屏障管理

**关键类**:
- `HeapManager` - 堆管理器
- `GC` - GC 基类
- `G1GC` - G1 垃圾收集器
- `PoolManager` - 内存池管理器

### 3. 线程管理子系统

```mermaid
graph TB
    subgraph 线程管理子系统
        subgraph 线程类型
            MT[ManagedThread]
            CT[CompilerThread]
            GCT[GC Thread]
        end
        
        subgraph 线程管理器
            TM[ThreadManager]
            SM[StackfulCoroutineManager]
        end
        
        subgraph 同步原语
            Mutex[Mutex]
            Cond[ConditionVariable]
            RWLock[RWLock]
        end
    end
    
    App[应用线程] --> MT
    MT --> TM
    TM --> SM
    
    MT --> Mutex
    MT --> Cond
    
    Compiler[编译任务] --> CT
    GC[GC 任务] --> GCT
```

**职责**:
- 托管线程生命周期管理
- 线程状态转换
- 协程支持

**关键类**:
- `Thread` - 线程基类
- `ManagedThread` - 托管线程
- `ThreadManager` - 线程管理器
- `CoroutineManager` - 协程管理器

### 4. 执行引擎子系统

```mermaid
graph TB
    subgraph 执行引擎
        subgraph 执行模式
            INT[解释器]
            JIT[JIT 编译器]
            AOT[AOT 代码]
        end
        
        subgraph 编译流程
            Parser[字节码解析]
            IRBuilder[IR 构建]
            Optimizer[优化器]
            CodeGen[代码生成]
        end
        
        subgraph 运行时支持
            Entrypoints[入口点]
            Intrinsics[内建函数]
            Bridge[桥接函数]
        end
    end
    
    Code[字节码] --> Parser
    Parser --> INT
    
    Hot[热点代码] --> IRBuilder
    IRBuilder --> Optimizer
    Optimizer --> CodeGen
    CodeGen --> JIT
    
    Precompiled[预编译代码] --> AOT
    
    INT --> Entrypoints
    JIT --> Entrypoints
    AOT --> Entrypoints
    
    Entrypoints --> Intrinsics
    Entrypoints --> Bridge
```

**职责**:
- 字节码解释执行
- JIT 热点编译
- AOT 预编译执行

**关键类**:
- `Interpreter` - 解释器
- `Compiler` - 编译器
- `Graph` - IR 图
- `CodeGenerator` - 代码生成器

## 数据流

### 类加载数据流

```mermaid
sequenceDiagram
    participant App as 应用代码
    participant CL as ClassLinker
    participant File as PandaFile
    participant Cache as Class Cache
    participant Heap as 堆内存
    
    App->>CL: FindClass("MyClass")
    CL->>Cache: 查找缓存
    alt 缓存命中
        Cache-->>CL: 返回 Class*
    else 缓存未命中
        CL->>File: 读取类元数据
        File-->>CL: ClassDataAccessor
        CL->>Heap: 分配 Class 对象
        CL->>CL: 解析父类、接口
        CL->>CL: 解析方法、字段
        CL->>Cache: 加入缓存
    end
    CL-->>App: 返回 Class*
```

### 方法调用数据流

```mermaid
sequenceDiagram
    participant Caller as 调用者
    participant RT as Runtime
    participant Linker as MethodLinker
    participant Code as 代码执行
    
    Caller->>RT: 调用方法
    RT->>Linker: 解析方法
    alt 已编译
        Linker-->>RT: 返回编译后代码地址
        RT->>Code: 直接执行
    else 未编译
        Linker-->>RT: 返回字节码地址
        RT->>Code: 解释执行
        opt 热点代码
            RT->>RT: 触发 JIT 编译
        end
    end
```

### 内存分配数据流

```mermaid
sequenceDiagram
    participant App as 应用代码
    participant ObjAlloc as ObjectAllocator
    participant TLAB as TLAB
    participant Region as RegionSpace
    participant GC as GC
    
    App->>ObjAlloc: 分配对象(size)
    ObjAlloc->>TLAB: 尝试 TLAB 分配
    alt TLAB 足够
        TLAB-->>ObjAlloc: 返回地址
    else TLAB 不足
        ObjAlloc->>Region: 分配新 TLAB
        alt Region 足够
            Region-->>ObjAlloc: 返回 TLAB
        else Region 不足
            ObjAlloc->>GC: 触发 GC
            GC->>Region: 回收空间
            Region-->>ObjAlloc: 返回 TLAB
        end
    end
    ObjAlloc-->>App: 返回对象引用
```

## 线程模型

### 托管线程状态机

```mermaid
stateDiagram-v2
    [*] --> CREATED: 创建线程
    CREATED --> RUNNING: 开始执行
    
    RUNNING --> NATIVE: 调用 Native 代码
    NATIVE --> RUNNING: 返回托管代码
    
    RUNNING --> BLOCKED: 等待锁/IO
    BLOCKED --> RUNNING: 获取锁/IO 完成
    
    RUNNING --> WAITING: 调用 wait()
    WAITING --> RUNNING: 被 notify()
    
    RUNNING --> SUSPENDED: GC 暂停
    SUSPENDED --> RUNNING: GC 完成
    
    RUNNING --> TERMINATED: 执行完成
    TERMINATED --> [*]
```

**状态说明**:
- **RUNNING**: 执行托管代码
- **NATIVE**: 执行原生代码（不受 GC 暂停影响）
- **BLOCKED**: 阻塞等待
- **WAITING**: 等待通知
- **SUSPENDED**: 被 GC 暂停

### GC 线程交互

```mermaid
sequenceDiagram
    participant MT as 托管线程
    participant GC as GC 线程
    participant Worker as GC Worker
    
    MT->>MT: 分配对象失败
    MT->>GC: 请求 GC
    
    GC->>GC: 进入 GC 暂停
    GC->>MT: 暂停所有托管线程
    MT->>MT: 保存上下文
    MT->>GC: 确认暂停
    
    GC->>Worker: 启动标记任务
    Worker->>Worker: 并发标记
    Worker->>GC: 标记完成
    
    GC->>GC: 重新标记
    GC->>GC: 清理/压缩
    
    GC->>MT: 恢复执行
    MT->>MT: 恢复上下文
```

## 关键时序

### 运行时启动时序

```mermaid
sequenceDiagram
    participant Main as main()
    participant RT as Runtime
    participant MM as MemoryManager
    participant CL as ClassLinker
    participant Plugin as LanguagePlugin
    
    Main->>RT: Runtime::Create(options)
    RT->>RT: 初始化配置
    RT->>MM: 创建堆管理器
    MM->>MM: 初始化 GC
    RT->>CL: 创建类链接器
    RT->>Plugin: 加载语言插件
    Plugin->>Plugin: 初始化语言上下文
    RT->>RT: 加载启动类
    RT-->>Main: Runtime 实例
```

### 应用启动时序

```mermaid
sequenceDiagram
    participant User as 用户
    participant Ark as ark 命令
    participant RT as Runtime
    participant File as ABCFile
    participant Main as main()
    
    User->>Ark: ark app.abc EntryPoint
    Ark->>RT: 初始化运行时
    RT->>File: 加载 app.abc
    File->>File: 解析文件头
    File->>File: 验证校验和
    RT->>RT: 查找 EntryPoint
    RT->>Main: 调用 main()
    Main->>Main: 执行业务逻辑
    Main-->>RT: 返回
    RT->>RT: 清理资源
    RT-->>Ark: 退出
```

## 模块依赖关系

```mermaid
graph TB
    subgraph 上层依赖
        App[应用代码]
        Lang[语言插件]
    end
    
    subgraph RuntimeCore
        subgraph 接口层
            API[ANI/N-API]
        end
        
        subgraph 核心服务
            Exec[执行引擎]
            Type[类型系统]
            Thread[线程管理]
            Mem[内存管理]
        end
        
        subgraph 基础层
            File[字节码文件]
            Base[基础库]
        end
    end
    
    subgraph 系统层
        OS[操作系统]
    end
    
    App --> API
    Lang --> API
    
    API --> Type
    API --> Exec
    
    Exec --> Mem
    Exec --> Thread
    Type --> File
    Thread --> Base
    Mem --> Base
    File --> Base
    
    Base --> OS
```

## 下一步

- 了解 [对外 API](04_Public_API.md) 进行 Native 开发
- 深入 [内部 API](05_Inner_API.md) 理解模块接口
- 查看 [GN 构建目标](06_GN_Targets.md) 了解构建系统
