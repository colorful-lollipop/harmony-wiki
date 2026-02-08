# XDevice 构建文档

## 1. GN 构建配置

### 1.1 根构建入口

**文件**: `BUILD.gn`

```python
import("//test/xts/tools/lite/build/suite_lite.gni")

deploy_suite("xdevice") {
  suite_name = "acts,hits,ssts"
}
```

**证据来源**：`BUILD.gn`

### 1.2 bundle.json 配置

**文件**: `bundle.json`

```json
{
    "name": "@openharmony/xdevice",
    "version": "2.30.0",
    "description": "xdevice",
    "component": {
        "name": "xdevice",
        "subsystem": "testfwk",
        "adapted_system_type": ["mini", "small", "standard"],
        "build": {
            "sub_component": [],
            "inner_kits": [],
            "test": []
        }
    }
}
```

**证据来源**：`bundle.json`

---

## 2. Python 包构建

### 2.1 主包配置

**文件**: `setup.py`

```python
install_requires = [
    "requests",
    "urllib3<2.1;python_version<'3.8'",
]

setup(
    name='xdevice',
    package_dir={'': 'src'},
    packages=[
        'xdevice',
        'xdevice._core',
        'xdevice._core.command',
        'xdevice._core.config',
        'xdevice._core.driver',
        'xdevice._core.environment',
        'xdevice._core.executor',
        'xdevice._core.report',
        'xdevice._core.testkit',
        'xdevice._core.context',
        'xdevice._core.cluster',
        'xdevice._core.cluster.controller',
        'xdevice._core.cluster.worker'
    ],
    package_data={
        'xdevice._core': [
            'resource/*.txt',
            'resource/config/*.xml',
            'resource/template/*',
            'resource/template/static/*',
            'resource/template/static/components/*',
            'resource/template/static/css/*',
            'resource/tools/*'
        ]
    },
    entry_points={
        'console_scripts': [
            'xdevice=xdevice.__main__:main_process',
            'xdevice_report=xdevice._core.report.__main__:main_report'
        ]
    },
    extras_require={
        "full": [
            "cryptography",
            "psutil",
            "fastapi;python_version>='3.10'",
            "filelock;python_version>='3.10'",
            "python-multipart;python_version>='3.10'",
            "sqlmodel;python_version>='3.10'",
            "uvicorn;python_version>='3.10'"
        ]
    }
)
```

**证据来源**：`setup.py`

### 2.2 ohos 插件配置

**文件**: `plugins/ohos/setup.py`

```python
entry_points={
    'device': [
        'device=ohos.environment.device',
        'device_lite=ohos.environment.device_lite'
    ],
    'manager': [
        'manager=ohos.managers.manager_device',
        'manager_lite=ohos.managers.manager_lite'
    ],
    'driver': [
        'cpp_driver=ohos.drivers.cpp_driver',
        'cpp_driver_lite=ohos.drivers.cpp_driver_lite',
        'jsunit_driver=ohos.drivers.jsunit_driver',
        'ltp_posix_driver=ohos.drivers.ltp_posix_driver',
        'oh_jsunit_driver=ohos.drivers.oh_jsunit_driver',
        'oh_kernel_driver=ohos.drivers.oh_kernel_driver',
        'oh_yara_driver=ohos.drivers.oh_yara_driver',
        'c_driver_lite=ohos.drivers.c_driver_lite',
        'opensource_driver_lite=ohos.drivers.opensource_driver_lite',
        'build_only_driver_lite=ohos.drivers.build_only_driver_lite',
        'vulkan_driver=ohos.drivers.vulkan_driver'
    ],
    'listener': [
        'listener=ohos.executor.listener',
    ],
    'testkit': [
        'kit=ohos.testkit.kit',
        'kit_lite=ohos.testkit.kit_lite'
    ],
    'parser': [
        'build_only_parser_lite=ohos.parser.build_only_parser_lite',
        'c_parser_lite=ohos.parser.c_parser_lite',
        'cpp_parser_lite=ohos.parser.cpp_parser_lite',
        'jsunit_parser_lite=ohos.parser.jsunit_parser_lite',
        'opensource_parser_lite=ohos.parser.opensource_parser_lite',
        'cpp_parser=ohos.parser.cpp_parser',
        'jsunit_parser=ohos.parser.jsunit_parser',
        'junit_parser=ohos.parser.junit_parser',
        'oh_jsunit_parser=ohos.parser.oh_jsunit_parser',
        'oh_kernel_parser=ohos.parser.oh_kernel_parser',
        'oh_rust_parser=ohos.parser.oh_rust_parser',
        'oh_yara_parser=ohos.parser.oh_yara_parser',
        'vulkan_parser=ohos.parser.vulkan_parser'
    ]
}
```

### 2.3 devicetest 插件配置

**文件**: `plugins/devicetest/setup.py`

```python
install_requires = [
    "jinja2",
    "xdevice"
]

extras_require={
    "full": [
        "numpy",
        "pillow",
        "opencv-python"
    ]
}

entry_points={
    'driver': [
        'device_test=devicetest.driver.device_test',
        'windows=devicetest.driver.windows'
    ]
}
```

---

## 3. 编译产物

### 3.1 Python 包产物

| 产物 | 类型 | 说明 |
|------|------|------|
| `dist/xdevice-*.tar.gz` | Python 包 | 主框架安装包 |
| `dist/xdevice-ohos-*.tar.gz` | Python 包 | OHOS 插件安装包 |
| `dist/xdevice-devicetest-*.tar.gz` | Python 包 | DeviceTest 插件安装包 |

### 3.2 安装命令

```bash
# 安装 XDevice 主包
cd /path/to/xdevice
python setup.py sdist
pip install dist/xdevice-*.tar.gz

# 安装 ohos 插件
cd plugins/ohos
python setup.py sdist
pip install dist/xdevice-ohos-*.tar.gz

# 安装 devicetest 插件
cd plugins/devicetest
python setup.py sdist
pip install dist/xdevice-devicetest-*.tar.gz
```

### 3.3 命令行工具

| 命令 | 入口 | 说明 |
|------|------|------|
| `xdevice` | `xdevice.__main__:main_process` | 主测试框架 CLI |
| `xdevice_report` | `xdevice._core.report.__main__:main_report` | 报告工具 |

---

## 4. 依赖清单

### 4.1 核心依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| Python | >= 3.7.5 | 运行环境 |
| requests | any | HTTP 请求 |
| urllib3 | < 2.1 (Python < 3.8) | URL 处理 |
| pyserial | >= 3.3 | 串口通信 |
| paramiko | >= 2.7.1 | SSH/SFTP |
| rsa | >= 4.0 | 加密 |

### 4.2 可选依赖

| 依赖 | Python 版本 | 用途 |
|------|------------|------|
| cryptography | 3.10+ | 加密增强 |
| psutil | 3.10+ | 系统监控 |
| fastapi | 3.10+ | Web 服务 |
| uvicorn | 3.10+ | ASGI 服务器 |
| filelock | 3.10+ | 文件锁 |
| python-multipart | 3.10+ | 表单解析 |
| sqlmodel | 3.10+ | ORM |
| jinja2 | - | 模板渲染 |
| numpy | - | 数值计算 |
| pillow | - | 图像处理 |
| opencv-python | - | 图像识别 |

---

## 5. 构建检查清单

- [ ] Python 版本 >= 3.7.5
- [ ] 安装依赖：`pip install -e .`
- [ ] 安装插件：`pip install -e plugins/ohos/`
- [ ] 验证安装：`xdevice --help`

---

## 相关文档

- [配置说明](05_Configuration.md)
- [使用指南](07_Usage.md)
