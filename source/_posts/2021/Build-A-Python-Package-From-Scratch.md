---
title: 一个Python包是如何产生的？
top: false
cover: false
toc: true
mathjax: true
tags:
  - Python
categories:
date: 2021-07-14 16:11:05
summary:
---

# 一个Python包是如何产生的？
> 我们都用过`pip install`，但是你尝试过自己打包一个Python包并提供给他人下载使用吗？

本文提供了一个使用setuptools+click实现的CLI示例，以帮助读者了解Python包的生成过程。

代码已发布在GitHub上：https://github.com/doutv/Python-Package-Demo
## 目录架构及文件内容
这是一个根据一定的语法规则翻译字符串的CLI，项目并未展示全部细节，只展示了与本文有关的部分代码。

项目目录架构是这样的：
```
.
│  setup.py
│
└─myproject
    │  api.py
    │  README.md
    │  __init__.py
    │
    ├─data
    │      data.yaml
    │      __init__.py
    │
    └─utils
            common.py
            __init__.py
```

### `setup.py`
setuptools使用的配置，setup的参数很多，以下是部分参数的解释:
- `packages` 项目中要安装的包的名字
- `install_requires` 项目的依赖包
  - 可指定版本，使用`==`,`<=`,`!=`等等
- `entry_points` CLI命令与函数的对应关系
  - `api=myproject.api:cli`
  - 当我在命令行输入`api`时，就会调用`myproject.api`中的`cli`函数
- `package_data` 要包含的静态文件


```python
# setup.py
from setuptools import setup, find_packages

setup(
    name='myproject',
    version='0.1',
    author="",
    author_email="",
    packages=find_packages(),
    install_requires=[
        'Click==8.0.1',
        'ruamel.yaml==0.17.10'
    ],
    entry_points='''
        [console_scripts]
        api=myproject.api:cli
    ''',
    package_data={
        "": ["*.yaml"]
    }
)
```

### `api.py`
暴露出来的CLI接口，click库的具体用法请参考官方文档

比较简单的理解：
- `click.group()` 可以把多个函数绑定到一个函数上面，这样就可以实现多个command
- `click.argument()` 是必填参数 
- `click.option()` 是选填参数

```python
# api.py
from myproject.utils.common import prepare_init_data, get_rule, parse_data_by_rule
import click


@click.group()
def cli():
    prepare_init_data()


@cli.command()
@click.argument('data')
@click.option('--schema', default="无")
def parse(data, schema):
    print(f"parsing {data} in {schema}")
    print(f"result is {parse_data_by_rule(data)}")


@cli.command()
def rule():
    print(get_rule())
```

### `common.py`

`api.py`中需要用到的一些辅助函数

- 读取Python包内自带的数据
  - 使用`importlib.resources`中的方法
  - 第一个参数是数据所在的包名
  - 第二个参数是文件名
  - 这种方法的优点：https://stackoverflow.com/questions/6028000/how-to-read-a-static-file-from-inside-a-python-package

```python
# common.py
RULE = {}


def prepare_init_data():
    from ruamel.yaml import YAML
    try:
        import importlib.resources as pkg_resources
    except ImportError:
        # Try backported to PY<37 `importlib_resources`.
        import importlib_resources as pkg_resources
    from myproject import data
    yaml = YAML()
    filename = "data.yaml"
    in_data = pkg_resources.read_text(data, filename)
    global RULE
    RULE = yaml.load(in_data)


def parse_data_by_rule(data):
    global RULE
    return RULE.get(data)


def get_rule():
    global RULE
    return RULE
```

## 调试及安装
- 在Debug Mode下调试: `pip install -e .`
- 在本地安装: `python setup.py install`

调用方法：
1. 在任意路径的命令行输入`api --help`
   1. `api parse --help`
   2. `api rule --help`
2. 安装成功后，可以在别的Python包中`import myproject`使用


## 发布到PyPi或Conda源上
请参考网络上的其他文章

# References
- https://stackoverflow.com/questions/6323860/sibling-package-imports
- https://stackoverflow.com/questions/6028000/how-to-read-a-static-file-from-inside-a-python-package
- https://setuptools.readthedocs.io/en/latest/userguide/quickstart.html
- https://click-docs-zh-cn.readthedocs.io/zh/latest/why.html