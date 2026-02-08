# 03_Build_Integration - OH 构建适配

## 概述

Mesa3D 在 OpenHarmony 中采用 **三层混合构建架构**：
1. **GN 构建系统** - OH 标准构建系统
2. **Python 脚本** - 配置生成和 Meson 调用
3. **Meson 构建系统** - Mesa3D 原生构建系统

---

## 构建架构

```
OpenHarmony 构建流程
┌─────────────────────────────────────────────────────────────────┐
│                        OH GN 构建                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ./build.sh --product-name=rk3568 --build-target=mesa3d │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                    │
│                           ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              third_party/mesa3d/BUILD.gn                │   │
│  │              ohos/BUILD.gn                              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Python 构建脚本                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    build_ohos64.py                      │   │
│  │         ├── meson_cross_process64.py                   │   │
│  │         │    └── 生成 cross_file (交叉编译配置)         │   │
│  │         │    └── 生成 pkgconfig 模板                    │   │
│  │         └── pkgconfig_template/*.pc                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Meson 构建系统                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  meson setup ${source} thirdparty/mesa3d/build-ohos    │   │
│  │      -Dplatforms=ohos                                   │   │
│  │      -Dgallium-drivers=zink                             │   │
│  │      ...                                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                    │
│                           ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  ninja -C build-ohos                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      构建产物                                    │
│  libEGL_mesa.so     (libEGL.so.1.0.0)                           │
│  libgallium-25.0.1.so                                           │
│  libGLESv2.so.2.0.0  (包含 GLESv3 别名)                         │
│  libGLESv1_CM.so.1.1.0                                          │
│  libglapi.so.0.0.0                                               │
│  libgbm.so.1.0.0                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## OH 特有构建配置

### 1. GN 构建配置

#### 根目录 BUILD.gn

```gn
# third_party/mesa3d/BUILD.gn

import("//build/ohos.gni")
import("ohos/dependency_inputs.gni")

SUBSYSTEM_NAME="thirdparty"
PART_NAME="mesa3d"

action("mesa3d_action_build") {
  script = "//third_party/mesa3d/ohos/build_ohos64.py"
  inputs = deps_inputs

  outputs = [
    "${root_build_dir}/thirdparty/mesa3d/lib/libEGL_mesa.so",
    "${root_build_dir}/thirdparty/mesa3d/lib/libgallium-25.0.1.so",
  ]

  # ASAN 选项
  if (is_asan) {
    asan_option = "hwasan"  # 或 "swasan"
  } else {
    asan_option = "noasan"
  }

  # 代码覆盖率
  if (use_clang_coverage) {
    coverage_option = "use_clang_coverage"
  }

  # Skia 版本兼容
  if (mesa3d_feature_upgrade_skia) {
    skia_version = "new_skia"  # skia m133
  } else {
    skia_version = "skia"
  }

  args = [
    ohos_root_path,
    product_name,
    mesa3d_source_path,
    asan_option,
    coverage_option,
    skia_version
  ]

  external_deps = [
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
    "graphic_surface:surface",
    "zlib:libz",
  ]

  # 根据 Skia 版本选择 expat
  if (mesa3d_feature_upgrade_skia) {
    external_deps += [ "skia:expatm133" ]
  } else {
    external_deps += [ "skia:expat" ]
  }
}

# 预编译库配置
ohos_prebuilt_shared_library("mesa3d_libEGL") {
  source = "${root_build_dir}/thirdparty/mesa3d/lib/libEGL_mesa.so"
  subsystem_name = "$SUBSYSTEM_NAME"
  part_name = "$PART_NAME"
  module_install_dir = "lib64"
  install_enable = true
  enable_strip = true
}

ohos_prebuilt_shared_library("mesa3d_libgallium") {
  source = "${root_build_dir}/thirdparty/mesa3d/lib/libgallium-25.0.1.so"
  subsystem_name = "$SUBSYSTEM_NAME"
  part_name = "$PART_NAME"
  module_install_dir = "lib64"
  install_enable = true
  enable_strip = true
}

group("zink_opengl") {
  deps = [
    ":mesa3d_libEGL",
    ":mesa3d_libgallium",
  ]
}
```

#### ohos/BUILD.gn

```gn
# ohos/BUILD.gn (传统配置，32位)

mesa3d_libs_dir = "$root_build_dir/packages/phone/mesa3d"

mesa3d_all_lib_items = [
  [ "libEGL.so.1.0.0", [ "libEGL.so", "libEGL.so.1", "libEGL_impl.so" ] ],
  [ "libgbm.so.1.0.0", [ "libgbm.so", "libgbm.so.1" ] ],
  [ "libglapi.so.0.0.0", [ "libglapi.so", "libglapi.so.0" ] ],
  [ "libGLESv1_CM.so.1.1.0", [ "libGLESv1_CM.so", "libGLESv1_CM.so.1", "libGLESv1_impl.so" ] ],
  [ "libGLESv2.so.2.0.0", [ "libGLESv2.so", "libGLESv2.so.2", "libGLESv3.so", "libGLESv2_impl.so", "libGLESv3_impl.so" ] ],
  [ "libgallium_dri.so", [ "panfrost_dri.so" ] ],
]
```

---

### 2. Python 构建脚本

#### build_ohos64.py

```python
#!/usr/bin/env python3

import sys
import os

def main():
    if len(sys.argv) < 7:
        print("需要参数: OH 源码路径, 产品名, mesa 源码路径, ASAN 选项, 覆盖率选项, Skia 版本")
        exit(-1)

    script_dir = os.path.dirname(os.path.abspath(__file__))

    # 1. 生成交叉编译配置
    run_cross = f'python3 {script_dir}/meson_cross_process64.py {sys.argv[1]} {sys.argv[2]} {sys.argv[4]} {sys.argv[5]} {sys.argv[6]}'
    os.system(run_cross)

    # 2. Meson 配置
    run_build_cmd = (
        'PKG_CONFIG_PATH=./thirdparty/mesa3d/pkgconfig '
        f'meson setup {sys.argv[3]} thirdparty/mesa3d/build-ohos '
        '-Dplatforms=ohos '
        '-Degl-native-platform=ohos '
        '-Dgallium-drivers=zink '
        '-Dbuildtype=release '
        '-Degl-lib-suffix=_mesa '
        '-Dvulkan-drivers= '
        '-Degl=enabled '
        '-Dgles1=enabled '
        '-Dgles2=enabled '
        '-Dopengl=true '
        '-Dcpp_rtti=false '
        '-Dglx=disabled '
        '-Dtools= '
        '-Dglvnd=disabled '
        '-Dshared-glapi=enabled '
        '-Dshader-cache=enabled '
        f'--cross-file={script_dir}/cross_file '
        f'--prefix={os.getcwd()}/thirdparty/mesa3d'
    )

    os.system(run_build_cmd)

    # 3. 编译和安装
    os.system('ninja -C thirdparty/mesa3d/build-ohos -j126')
    os.system('ninja -C thirdparty/mesa3d/build-ohos install')

if __name__ == '__main__':
    main()
```

#### meson_cross_process64.py

```python
# 生成 Meson 交叉编译配置文件

corss_file_content = '''
[properties]
needs_exe_wrapper = true

c_args = [
    '--target=aarch64-linux-ohosmusl',
    '--sysroot=sysroot_stub',
    '-fno-emulated-tls',
    '-fPIC',
    '-g', '-O2',
    '-D_FORTIFY_SOURCE=2',
    '-fstack-protector-all',
]

cpp_args = [
    '--target=aarch64-linux-ohosmusl',
    '--sysroot=sysroot_stub',
    '-fno-emulated-tls',
    '-fPIC',
    '-g', '-O2',
    '-D_FORTIFY_SOURCE=2',
    '-fstack-protector-all',
]

c_link_args = [
    '--target=aarch64-linux-ohosmusl',
    '-fPIC',
    '--sysroot=sysroot_stub',
    '-Lsysroot_stub/usr/lib/aarch64-linux-ohos',
    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/current/lib/aarch64-linux-ohos',
    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/aarch64-linux-ohos/c++',
    '--rtlib=compiler-rt',
    '-Wl,--exclude-libs=libunwind_llvm.a',
]

cpp_link_args = [
    '--target=aarch64-linux-ohosmusl',
    '--sysroot=sysroot_stub',
    '-Lsysroot_stub/usr/lib/aarch64-linux-ohos',
    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/current/lib/aarch64-linux-ohos',
    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/aarch64-linux-ohos/c++',
    '-fPIC',
    '--rtlib=compiler-rt',
]

[binaries]
ar = 'project_stub/prebuilts/clang/ohos/linux-x86_64/llvm/bin/llvm-ar'
c = ['ccache', 'project_stub/prebuilts/clang/ohos/linux-x86_64/llvm/bin/clang']
cpp = ['ccache', 'project_stub/prebuilts/clang/ohos/linux-x86_64/llvm/bin/clang++'
c_ld = 'lld'
cpp_ld = 'lld'
strip = 'project_stub/prebuilts/clang/ohos/linux-x86_64/llvm/bin/llvm-strip'

[host_machine]
system = 'linux'
cpu_family = 'aarch64'
cpu = 'armv8'
endian = 'little'
'''
```

---

### 3. Meson 构建选项

#### 关键编译参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `-Dplatforms` | `ohos` | 启用 OHOS 平台支持 |
| `-Degl-native-platform` | `ohos` | 设置 EGL 原生平台 |
| `-Dgallium-drivers` | `zink` | 使用 Zink 驱动 |
| `-Degl-lib-suffix` | `_mesa` | EGL 库后缀 |
| `-Dvulkan-drivers` | 空 | **禁用**原生 Vulkan 驱动 |
| `-Degl` | `enabled` | 启用 EGL |
| `-Dgles1` | `enabled` | 启用 GLES 1.x |
| `-Dgles2` | `enabled` | 启用 GLES 2.x |
| `-Dopengl` | `true` | 启用 OpenGL |
| `-Dglx` | `disabled` | **禁用** GLX |
| `-Dtools` | 空 | **禁用**工具程序 |
| `-Dglvnd` | `disabled` | **禁用** GLVND |
| `-Dshared-glapi` | `enabled` | 启用共享 GL API |
| `-Dshader-cache` | `enabled` | 启用着色器缓存 |
| `-Dcpp_rtti` | `false` | 禁用 C++ RTTI |

#### ASAN/调试选项

```python
# HWASAN (硬件地址消毒)
compile_args = ['-shared-libasan', '-fsanitize=hwaddress']
link_args = ['-shared-libasan', '-fsanitize=hwaddress']

# SWASAN (软件地址消毒)
compile_args = ['-fsanitize=address', '-fno-inline-functions']
link_args = ['-lclang_rt.asan']

# 代码覆盖率
compile_args = ['--coverage']
link_args = ['--coverage']
```

---

### 4. pkgconfig 模板系统

#### 模板目录结构

```
ohos/pkgconfig_template/
├── expat.pc.template
├── libbegetutil.pc.template
├── libdrm.pc.template
├── libhilog.pc.template
├── libhitrace.pc.template
├── libjpeg.pc.template
├── libpng.pc.template
├── libsurface.pc.template
├── libudev.pc.template
├── libxml2.pc.template
├── libz.pc.template
├── wayland-client.pc.template
├── wayland-cursor.pc.template
├── wayland-egl-backend.pc.template
├── wayland-egl.pc.template
├── wayland-protocols.pc.template
├── wayland-scanner.pc.template
├── wayland-server.pc.template
└── zlib.pc.template
```

#### libsurface.pc.template 示例

```pc
prefix=ohos_project_directory_stub/out/ohos-arm-release/obj/third_party
exec_prefix=${prefix}
libdir=${exec_prefix}/third_party/surface/lib
includedir=${prefix}/third_party/surface/interfaces/native

Name: libsurface
Description: OpenHarmony Native Window library
Version: 1.0
Libs: -L${libdir} -lsurface
Cflags: -I${includedir}/surface
```

---

## 构建产物说明

### 32位 vs 64位 差异

| 特性 | 32位 (build_ohos.py) | 64位 (build_ohos64.py) |
|------|---------------------|------------------------|
| **目标架构** | armv7-a | aarch64 |
| **GPU 驱动** | panfrost | zink |
| **GBM 库** | libgbm.so | **不构建** |
| **E_GL 后缀** | 无 | `_mesa` |

### 库清单

```
# 64位产物
libEGL_mesa.so         (libEGL.so.1.0.0 的符号链接)
libEGL.so.1.0.0        (EGL 实现)
libgallium-25.0.1.so   (Gallium 框架)
libGLESv2.so.2.0.0     (包含 GLESv3)
libGLESv1_CM.so.1.1.0  (GLES 1.1)
libglapi.so.0.0.0      (共享 GL API)

# 32位额外产物
libgbm.so.1.0.0        (GBM 缓冲管理)
panfrost_dri.so        (Panfrost DRI 驱动)
```

---

## 与上游构建差异

| 方面 | 上游 Mesa | OH 适配版 |
|------|-----------|-----------|
| **构建系统** | 纯 Meson | GN + Python + Meson |
| **平台选项** | 自动检测 Linux | 显式指定 `platforms=ohos` |
| **窗口系统** | X11/Wayland/DRM | OH NativeWindow |
| **GPU 驱动** | 多驱动可选 | zink (64位) / panfrost (32位) |
| **GLX** | 可选启用 | 强制禁用 |
| **GBM** | 可选启用 | 32位启用，64位禁用 |
| **工具程序** | 完整工具链 | 禁用 (`-Dtools=`) |
| **交叉编译** | 手动 cross-file | 自动生成 cross_file |
| **依赖查找** | 系统 pkg-config | 模板生成 .pc |

---

## 编译示例

### 完整编译流程

```bash
# 1. 编译整个系统
./build.sh --product-name=rk3568

# 2. 单独编译 mesa3d (64位)
./build.sh --product-name=rk3568 --build-target=mesa3d

# 3. 查看构建日志
cat out/rk3568/logs/mesa3d_build.log
```

### 手动编译 (开发调试)

```bash
cd third_party/mesa3d

# 1. 准备交叉编译环境
python3 ohos/build_ohos64.py \
    /path/to/openharmony \
    rk3568 \
    /path/to/openharmony/third_party/mesa3d \
    noasan \
    no_coverage \
    skia

# 2. 重新配置 (修改 meson 选项后)
rm -rf build-ohos
meson setup build-ohos . \
    -Dplatforms=ohos \
    -Dgallium-drivers=zink \
    ...

# 3. 重新编译
ninja -C build-ohos -j126
ninja -C build-ohos install
```

---

## 常见问题

### Q1: 编译失败 "cannot find -lxxx"

**解决**: 确保依赖库已编译
```bash
./build.sh --product-name=rk3568 --build-target=libsurface
./build.sh --product-name=rk3568 --build-target=expat
```

### Q2: 如何启用调试符号

**解决**: 修改 meson 选项
```bash
meson setup build-ohos . -Dbuildtype=debug
```

### Q3: 如何切换 Skia 版本

**解决**: 修改 `mesa3d_feature_upgrade_skia` gn 变量

---

## 关键文件清单

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | OH 构建入口 |
| `ohos/BUILD.gn` | 传统 32位配置 |
| `ohos/build_ohos64.py` | 64位构建脚本 |
| `ohos/build_ohos.py` | 32位构建脚本 |
| `ohos/meson_cross_process64.py` | 交叉编译配置生成 |
| `ohos/pkgconfig_template/*.pc` | pkgconfig 模板 |
| `src/util/detect_os.h` | OHOS 检测 |
| `src/egl/drivers/dri2/platform_ohos.c` | EGL 平台适配 |
| `include/vulkan/vulkan_ohos.h` | Vulkan 扩展 |
