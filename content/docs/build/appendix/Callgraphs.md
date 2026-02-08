# 调用链分析

本文档分析 OpenHarmony build 系统的关键调用链。

## 1. 构建入口调用链

### build.sh → hb

```
build_scripts/build.sh:176-217
    │
    ├──→ tools_checker.py (环境检查)
    │
    └──→ hb/main.py build
            │
            ├──→ Main.main()
            │       │
            │       ├──→ Main._init_build_module()
            │       │       └──→ OHOSBuildModule()
            │       │
            │       └──→ module.run()
            │               │
            │               ├──→ _prebuild_and_preload()
            │               │       ├──→ _prebuild()
            │               │       └──→ _preload() → Preloader.run()
            │               │
            │               ├──→ _load() → Loader.run()
            │               │
            │               ├──→ _gn() → Gn.run() → gn gen
            │               │
            │               ├──→ _ninja() → Ninja.run() → ninja
            │               │
            │               └──→ _post_build()
            │
            └──→ Build complete
```

## 2. GN 生成调用链

### gn gen

```
hb/services/gn.py:54-55
    │
    ├──→ Gn.run()
    │       │
    │       └──→ _execute_gn_gen_cmd()
    │               │
    │               ├──→ _convert_args() (参数转换)
    │               │
    │               └──→ subprocess.run([gn, 'gen', ...])
    │                       │
    │                       └──→ //prebuilts/build-tools/linux-x86/bin/gn
    │                               │
    │                               ├──→ 解析 .gn (root)
    │                               ├──→ 解析 BUILDCONFIG.gn
    │                               ├──→ 递归解析所有 BUILD.gn
    │                               ├──→ 展开模板 (.gni)
    │                               ├──→ 解析 declare_args()
    │                               └──→ 生成 build.ninja
    │
    └──→ build.ninja generated
```

## 3. Ninja 编译调用链

### ninja -C out

```
hb/services/ninja.py:38-39
    │
    ├──→ Ninja.run()
    │       │
    │       └──→ _execute_ninja_cmd()
    │               │
    │               ├──→ 设置环境变量
    │               │
    │               └──→ subprocess.run([ninja, '-C', out_path])
    │                       │
    │                       └──→ //prebuilts/build-tools/linux-x86/bin/ninja
    │                               │
    │                               ├──→ 解析 build.ninja
    │                               ├──→ 构建依赖图
    │                               ├──→ 计算需要构建的目标
    │                               ├──→ 并行执行构建任务
    │                               │       ├──→ 编译 (.c/.cpp/.rs → .o)
    │                               │       ├──→ 链接 (.o → .so/executable)
    │                               │       └──→ 其他操作
    │                               └──→ 生成产物
    │
    └──→ Build complete
```

## 4. 预加载调用链

### Preloader

```
hb/services/preloader.py:28-353
    │
    ├──→ OHOSPreloader.run()
    │       │
    │       ├──→ _generate_platforms_build()
    │       │       └──→ platforms.build
    │       │
    │       ├──→ _generate_features_json()
    │       │       └──→ features.json
    │       │
    │       ├──→ _generate_syscap_json()
    │       │       └──→ syscap.json
    │       │
    │       ├──→ _generate_parts_json()
    │       │       └──→ parts.json
    │       │
    │       └──→ _generate_build_config_json()
    │               └──→ build_config.json
    │
    └──→ Config files generated in out/{device}/build_configs/
```

## 5. 加载器调用链

### Loader

```
hb/services/loader.py:36-1000
    │
    ├──→ OHOSLoader.run()
    │       │
    │       ├──→ _check_args()
    │       │       ├──→ 验证产品配置
    │       │       └──→ 验证目标 CPU/OS
    │       │
    │       ├──→ _check_product_part_feature()
    │       │       └──→ 验证部件特性
    │       │
    │       ├──→ _generate_syscap_files()
    │       │       └──→ 生成系统能力文件
    │       │
    │       ├──→ _generate_subsystem_configs()
    │       │       └──→ subsystem_config.gni
    │       │
    │       └──→ _generate_target_gn()
    │               └──→ target_platform.gn
    │
    └──→ Target GN files generated
```

## 6. 模板展开调用链

### ohos_shared_library

```
templates/cxx/cxx.gni:574
    │
    ├──→ ohos_shared_library("my_lib")
    │       │
    │       ├──→ check_target (依赖检查)
    │       │
    │       ├──→ collect_module_target (模块收集)
    │       │
    │       ├──→ generate_module_info (模块信息生成)
    │       │
    │       ├──→ 处理 innerapi_tags
    │       │       ├──→ ndk
    │       │       ├──→ platformsdk
    │       │       └──→ chipsetsdk
    │       │
    │       ├──→ 处理 shlib_type
    │       │       ├──→ sa
    │       │       ├──→ hdi
    │       │       └──→ napi
    │       │
    │       ├──→ 设置编译标志
    │       │       ├──→ cflags
    │       │       ├──→ ldflags
    │       │       └──→ configs
    │       │
    │       └──→ shared_library (GN 内置)
    │               │
    │               └──→ 生成 .so 构建规则
    │
    └──→ Target defined
```

## 7. 镜像制作调用链

### system.img

```
ohos/images/BUILD.gn
    │
    ├──→ system_image target
    │       │
    │       ├──→ 收集 system 目录内容
    │       │       ├──→ bin/
    │       │       ├───→ lib64/
    │       │       └──→ ...
    │       │
    │       ├──→ mkextimage.py
    │       │       │
    │       │       ├──→ 读取 system_image_conf.txt
    │       │       │       ├──→ size = 1610612224
    │       │       │       └──→ fs_type = ext4
    │       │       │
    │       │       ├──→ 创建空镜像文件
    │       │       ├──→ 格式化为 ext4
    │       │       ├──→ 挂载镜像
    │       │       ├──→ 复制文件
    │       │       └──→ 卸载镜像
    │       │
    │       └──→ system.img
    │
    └──→ Image created
```

## 8. SDK 打包调用链

### ohos-sdk

```
ohos/sdk/BUILD.gn:315
    │
    ├──→ make_sdk_modules target
    │       │
    │       ├──→ 解析 ohos_sdk_description_std.json
    │       │       ├──→ toolchains
    │       │       ├──→ js
    │       │       ├──→ ets (static/dynamic)
    │       │       └──→ previewer
    │       │
    │       ├──→ copy_sdk_modules.py
    │       │       └──→ 复制 SDK 组件
    │       │
    │       ├──→ 收集声明文件
    │       │       └──→ notice/merge_notice_files.py
    │       │
    │       ├──→ 打包归档
    │       │       └──→ ohos-sdk-linux.tar.gz
    │       │
    │       └──→ check_sdk_completeness.py
    │               └──→ 验证 SDK 完整性
    │
    └──→ SDK package created
```

## 9. HAP 构建调用链

### ohos_hap

```
ohos/app/app.gni
    │
    ├──→ ohos_hap("my_app")
    │       │
    │       ├──→ 处理 hap_profile (module.json)
    │       │
    │       ├──→ 编译 JS/ETS 资源
    │       │       ├──→ build_js_assets.py
    │       │       └──→ generate_js_bytecode.py
    │       │
    │       ├──→ 处理 resources
    │       │       └──→ compile_resources.py
    │       │
    │       ├──→ 打包 HAP
    │       │       └──→ hapbuilder.py
    │       │
    │       ├──→ 签名
    │       │       └──→ app_sign.py
    │       │
    │       └──→ my_app.hap
    │
    └──→ HAP created
```

## 10. Rust 构建调用链

### ohos_rust_executable

```
templates/rust/rust_template.gni:263
    │
    ├──→ ohos_rust_executable("my_rust_app")
    │       │
    │       ├──→ rust_target (基础模板)
    │       │       │
    │       │       ├──→ 设置 crate_type = "bin"
    │       │       ├──→ 设置 rustc_lints
    │       │       └──→ 调用 rustc
    │       │
    │       ├──→ ohos_executable (C++ 模板)
    │       │       │
    │       │       ├──→ check_target
    │       │       ├──→ collect_module_target
    │       │       └──→ generate_module_info
    │       │
    │       └──→ 生成可执行文件
    │
    └──→ Rust executable built
```

---

*文档生成时间: 2025-02-06*
