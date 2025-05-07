# Python笔记(Python高级技巧)

**初学者请先完成菜鸟教程中Python3基础教程的学习**

**[菜鸟教程网址](https://www.runoob.com/python3/python3-tutorial.html)**

## 使用虚拟环境

>  Linux下

```shell
python3 -m venv virEnv
# 进入虚拟环境
source virEnv/bin/activate
# 退出虚拟环境
deactivate
```

>  Windows下

```powershell
pip install virtualenv
virtualenv virEnv
./virEnv/Scripts/activate.bat
# 退出虚拟环境
deactivate
```

## 使用conda环境

官方安装链接：https://www.anaconda.com/docs/getting-started/miniconda/install

> 创建conda文件夹

```bash
mkdir miniconda3
```

> 根据官方步骤下载并运行脚本安装

```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
# rm ~/miniconda3/miniconda.sh
```

> 调整其中的源配置

```bash
cp ~/miniconda3/.condarc ~/miniconda3/.condarc.bak
vim ~/miniconda3/.condarc
# 替换为以下内容
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch
  - defaults
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
show_channel_urls: True
```

> 激活conda

```bash
source ~/miniconda3/bin/activate
```

> 初始化conda

```bash
conda init --all
```

> 退出conda

```bash
deactivate
```

---

**以下操作均是在上述环境的前提下进行**

> 查看当前conda配置

```bash
conda config --show
```

这会列出当前 Conda 的所有配置，包括配置的源。

> 创建虚拟环境

```bash
conda create -n virEnv python=3.x
```

**注：这里所指定的Python环境需要本地环境中被安装才能成功！！！若本地环境中不存在对应版本的Python环境需现安装。**

> 激活和切换环境

```bash
conda activate virEnv
```

> 移除环境

```bash
conda env remove -n virEnv
```

> 查看已安装环境

```bash
conda env list
```

这条命令会列出所有 Conda 环境，以及当前激活的环境。

> 查询库

```bash
conda search numpy
```

> 安装库

```bash
conda install numpy
```

> 更新库

```bash
conda update numpy
```

> 卸载库

```bash
conda remove numpy
```

> 导出环境

```bash
conda list --explicit > environment.txt
# 或者使用yml文件作为环境配置
conda env export > environment.yml
```

> 导入环境

```bash
conda env create -f environment.yml
```

---

应确保conda环境中所使用的pip与虚拟环境对应，如下所示：

```bash
# .bashrc文件中的配置应保证优先使用conda虚拟环境中的pip
export PATH="$CONDA_PREFIX/bin:$PATH"
source ~/.bashrc
# 查看当前所使用的pip所属
which pip
# 若使用的是conda虚拟环境中的pip则应会有以下类似的输出
/home/<username>/miniconda3/envs/virEnv/bin/pip
# 可以通过以下命令查看当前的所有pip
which -a pip
# 若显示的路径并不是虚拟环境相关的路径，则证明当前使用的是全局的pip
# 调整方法
# 先通过激活当前虚拟环境，在当前虚拟环境中安装pip
conda activate virEnv
conda install pip
# 通过which命令查看pip安装情况
which pip
# 此时应该会出现虚拟环境中的pip
# 然后通过调整.bashrc文件中的配置并重载该配置实现pip的切换
```



## 从源码安装Python（Linux环境）

**在Ubuntu 18.04-24.04通过测试**

>  **依赖安装**

```shell
sudo apt install vim git make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-dev
```

---

**实际安装时将下面所写的python3.x替换为实际的具体版本即可**

---

> **从官方下载源码压缩包**

下载地址（可能会失效）：https://www.python.org/downloads/source/

> **解压后并进入文件夹**

```bash
tar -xvzf Python3.x.tar.gz
```

>  **创建文件夹**

```shell
sudo mkdir /opt/python3.x
```

>  **配置makefile**

```shell
./configure --with-ssl --prefix=/opt/python3.x
```

>  **编译**

```shell
make
```

>  **安装**

```shell
sudo make install
```

>  **删除原有软链接**

```shell
ls -l /usr/bin/python
sudo rm /usr/bin/python
# rm ./.local/bin/pip
```

>  **建立新的软链接**

```shell
ln -s /opt/python3.x/bin/python3 /usr/bin/python
ln -s /opt/python3.x/bin/pip3 /usr/bin/pip
```

> **pip源**

临时使用：

```txt
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple <包名>
```

设为默认（pip版本大于等于10.0.0）：

```shell
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

配置多个源：

```shell
pip config set global.extra-index-url "<url1> <url2>..."
```

可用的 `pypi` 源列表（校园网联合镜像站）：https://mirrors.cernet.edu.cn/list/pypi

## collection容器输出

一行输出字典，输出格式：一行一个键值

```python
m = {"a": 1, "b": 2, "c": 3}
print(*[f"{k}: {v}" for k, v in m.items()], sep="\n")
```

这里，我们使用了 `print` 函数的 `*` 操作符来解包一个列表，每个元素是由列表推导生成的，它包含格式化的字符串，表示键和值。`sep='\n'` 指定了每个元素应当如何分隔，这里使用换行符，这样每对键值都会打印在单独的行上。

上述方式可以推广至所有类型的容器，只需要修改该输出中内部list中关于该容器的遍历方式即可实现，比如对于list类型的容器则可以将该list直接替换为目标list。

## 字典

- **普通字典**:

  - 在尝试读取不存在的键时会抛出 `KeyError` 异常。
  - 不会自动为不存在的键创建默认值。

- **defaultdict**:

  - 在尝试读取不存在的键时，会自动为该键创建默认值。
  - 需要在初始化时提供一个默认工厂函数，用于生成默认值。

  该字典的使用：

  ```python
  ddict = defaultdict(SomeClass)
  print(ddict['key'].attribute)  # 输出: Default Value
  print(ddict_partial['key'].attribute)  # 输出: Default Value
  ```

  在你尝试访问一个尚未存在于字典中的键时，它会使用一个默认工厂函数来创建该键的值，然后将这个键-值对添加到字典中。

  如果你将`defaultdict`的默认工厂设置为某个类的构造器，并且这个类的构造器只接受一个参数（除了`self`），那么在尝试使用`defaultdict`自动创建一个此类实例时不会引发错误。这是因为`defaultdict`在创建新值时调用默认工厂函数是不传递任何参数（也就是说，它只调用了构造器而没有提供额外的参数），这对于不需要参数或只有默认参数的情况是可行的。

  而，若是`MyClass`的构造方法需要多个参数，这将会引发类型错误（`TypeError`），因为`MyClass`的构造方法在被`defaultdict`调用时需要多个参数，但是`defaultdict`不会传递任何参数给它。这种尝试创建`MyClass`实例的操作会失败。

  为了解决这个问题，你可以提供一个无参数的包装器函数或使用`functools.partial`来预填充所需的参数，如下：

  ```python
  from collections import defaultdict
  from functools import partial
  
  class MyClass:
      def __init__(self, param):
          self.attribute = param
  
  # 定义一个包装器函数，预设构造方法的参数
  def myclass_factory():
      return MyClass(param="Default Value")
  
  # 或者使用 functools.partial 来达到同样效果
  myclass_factory_partial = partial(MyClass, "Default Value")
  
  # 使用包装器函数作为默认工厂
  ddict = defaultdict(myclass_factory)
  # 或者使用 partial
  ddict_partial = defaultdict(myclass_factory_partial)
  
  print(ddict['key'].attribute)  # 输出: Default Value
  print(ddict_partial['key'].attribute)  # 输出: Default Value
  ```

  

## 模块

获取当前目录路径：

```python
current_module_dir = os.path.dirname(os.path.abspath(__file__))
```

## 生成器函数

生成器函数与一般的函数不同，最大的特点是调用之后返回一个迭代器。每次迭代时，都会从函数体中的`yield`语句处获取一个值，然后暂停函数的执行，等待下一次迭代。在迭代过程中，函数会保持其当前的执行环境（包含所有的局部变量），这使得它可以在任何的`yield`语句之间保持状态。这个特性赋予了Python在处理大型数据集，或者需要动态生成结果集，或者在需要控制复杂的控制流程的情况下的强大能力。

```python
def search_completer(self, document: Document):
	word = document.get_word_before_cursor()

	# Business logic that gives a list of possible words
	possible_words = self.console.business_interface(word)

	for word in possible_words:
		yield Completion(word, start_position=-len(word))
```

上述的例子中，``` search_completer ```是一个补全器，当tab键按下时将会触发它。该补全器函数会在yield处返回，看上去是返回一个Completion实例，实际上是返回一个迭代器，即当tab按下时都会返回一个不同的Completion实例，当``` word ```迭代完成之后，再次按下tab键将再次重新调用该函数，重新计算``` possible_words```列表，并再次返回迭代器。

## 装饰器

装饰器的原理：

当带有装饰器的对象被使用时，会优先触发装饰器中定义的代码，再根据其中的代码决定是否触发带有装饰器的对象的代码。

位于函数上的装饰器：

```python
from functools import wraps

def register_command(func_name: str, func_description: str, func_alias: str = ""):
    def inner_wrapper(func):
        func.func_name = func_name
        func.alias = func_alias
        func.func_description = func_description
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            # 这里调用原始函数，并得到返回结果
            result = func(*args, **kwargs)
            # 在这里可以对返回结果进行处理
            print(f"Executing '{func.func_name}' with result: {result}")
            return result
        return wrapper
    return inner_wrapper
```

上述代码中，定义了一个名为``` register_command ```的装饰器，其内部定义了一个拦截函数``` inner_wrapper ```，该拦截函数是该装饰器的核心代码，当带有该装饰器的对象的代码被调用时，会先触发该拦截函数。

在拦截函数中能通过装饰器传入的``` func ```参数变量取到原函数（带有该装饰器的函数）的函数（类型）变量，从而实现为该函数对象添加一些需要的成员，这些成员的值来自于装饰器上的传入参数。

通过定义在拦截函数中的内部函数``` wrapper ```实现对原函数的调用。注意，定义这个内部函数的本质就是为了使原函数被调用，若不定义该内部函数，则原函数无法被调用。

上述代码中所给出的装饰器``` register_command ```本质上完成两件事情：在装饰器的外层（`inner_wrapper`）为`func`添加属性，在装饰器的内层（`wrapper`）增强或修改`func`的执行逻辑（比如记录日志、修改返回值等）。

还可以为装饰器传入函数类型的变量，例子如下：

```python
def decorator_factory(preprocessor, *preprocessor_args, **preprocessor_kwargs):
    """
    一个装饰器工厂，接收一个预处理函数及其参数作为输入。
    """
    def decorator(func):
        def wrapper(*args, **kwargs):
            # 调用预处理函数，并将额外的参数传入
            preprocessed_args = preprocessor(*args, *preprocessor_args, **preprocessor_kwargs)
            return func(*preprocessed_args, **kwargs)
        return wrapper
    return decorator

# 预处理函数的一个示例，现在它接收额外的参数
def preprocessor(*args, factor=1):
    print("预处理函数被调用")
    # 在这里可以根据factor对args进行某种形式的预处理
    preprocessed_args = [arg * factor for arg in args if isinstance(arg, (int, float))]
    return preprocessed_args

# 应用装饰器时传递额外参数给preprocessor函数
@decorator_factory(preprocessor, factor=10)
def my_func(x):
    print(f"函数被调用，参数值为: {x}")

# 此时，my_func被装饰了，且preprocessor将用factor=10来预处理入参
my_func(5)  # 期望输出为函数被调用，参数值为: 50，因为预处理函数将其乘以了10
```

一个更复杂的例子：

```python
def multi_preprocessor_factory(*preprocessor_with_args):
    """
    装饰器工厂，接受多个预处理函数及其参数。
    *preprocessor_with_args: 元组（preprocessor_function, args, kwargs）
    """
    def decorator(func):
        def wrapper(*args, **kwargs):
            # 遍历所有预处理函数及其参数
            current_args = args
            for preprocessor, preprocessor_args, preprocessor_kwargs in preprocessor_with_args:
                # 更新当前参数为预处理函数调用的结果
                current_args = preprocessor(*current_args, *preprocessor_args, **preprocessor_kwargs)
            # 调用原函数
            return func(*current_args, **kwargs)
        return wrapper
    return decorator

# 示例预处理函数
def preprocessor1(*args, factor=1):
    return [a * factor for a in args]

def preprocessor2(*args, increment=1):
    return [a + increment for a in args]

# 使用装饰器，并且传入多个预处理函数以及它们的参数
@multi_preprocessor_factory(
    (preprocessor1, (10,), {}),  # preprocessor1 with factor=10
    (preprocessor2, (), {"increment": 5})  # preprocessor2 with increment=5
)
def my_func(x):
    print(f"函数被调用，最终参数值为: {x}")

my_func(2)  # 预期先乘以10得到20，然后加上5得到25
```



## 包结构以及Python项目管理

一个标准的Python项目结构：

```
my_project/
│
├── src/
│   ├── __init__.py
│   ├── module1.py
│   └── module2.py
│
├── tests/
│   ├── __init__.py
│   ├── test_module1.py
│   └── test_module2.py
│
├── docs/
│   └── ...
│
├── README.md
├── LICENSE
└── setup.py
```

- `src/`：这是项目的主包目录，包含项目的所有主要代码。每个包或子包目录中应该有一个`__init__.py`文件（可以为空），标记该目录可被Python识别为包。
- `tests/`：这个目录包含所有的单元测试代码，帮助你验证包内各模块的行为。
- `docs/`：项目的文档，比如API说明、使用指南等。
- `README.md`、`LICENSE`和`setup.py`等：这些文件用于项目描述、版权信息和项目依赖设置等。

为了方便后续开发，应在项目的根目录中编写``` setup.py ```文件或``` pyproject.toml ```文件

两种文件的简单格式如下：

> **setup.py**

```python
from setuptools import setup, find_packages

setup(
    name='example_project',
    version='0.1',
    packages=find_packages(),
)
```

稍微复杂、详细一点的写法：

```python
from setuptools import setup, find_packages

setup(
    name='your_package_name',  # 包名
    version='0.1',  # 版本号
    packages=find_packages(),  # 包含所有 src 中的包
    description='A short description of your project',  # 简短描述
    long_description=open('README.md').read(),  # 长描述，通常是你的 README
    long_description_content_type='text/markdown',  # 说明长描述是 markdown 格式
    author='Your Name',  # 作者名
    author_email='your_email@example.com',  # 作者邮箱
    url='https://github.com/yourname/your_project',  # 项目首页/代码仓库URL
    install_requires=[
        # 这里面填写项目依赖的其他包
        'requests>=2.20.0',  # 一个示例依赖
    ],
    classifiers=[
        # 分类器：https://pypi.org/classifiers/
        'Programming Language :: Python :: 3',
        'License :: OSI Approved :: MIT License',
        'Operating System :: OS Independent',
    ],
)
```

**关键参数说明：**

- **name**: 包名称，这是项目发布和安装时使用的名字。
- **version**: 项目版本号。
- **packages**: 说明哪些包含在你的项目里。`find_packages()` 自动找到这些包。
- **description**: 项目的简短描述。
- **long_description**: 项目的长描述，通常是README文件的内容。
- **author** 和 **author_email**: 项目作者的姓名和邮箱。
- **url**: 项目的官方网站地址，通常是代码仓库的URL。
- **install_requires**: 列出项目运行时依赖的包。这些包将被自动安装。
- **classifiers**: 项目的分类信息，帮助用户找到项目并了解项目可以在哪些环境下运行。

> **pyproject.toml**

```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "example_project"
version = "0.1"
authors = [{ name = "Your Name", email = "your.email@example.com" }]
```

**创建好以上两种文件中的一种后，使用pip命令进行项目初始化**

```shell
pip install -e .
```

出现错误后使用的一些命令：

```shell
# 清理项目构建文件
python setup.py clean
# 输出构建时的详细日志
pip install -e . -v
# 通过find命令查找目录
find /path/to/your/project -type d -name "*.egg-info"
```



## 面向对象

### 元类

**元类的构造方法在带有这个元类的类被构造之前被调用。**简单来说，元类的主要作用是控制类的创建过程，元类允许我们拦截类的创建，修改类的定义，或者返回修改后的类。

在Python中，元类的使用可以通过定义一个继承自 `type` 的新类并覆写其 `__new__` 或 `__init__` 方法来实现。当创建一个类时，Python会首先查找它的元类（通过 `metaclass` 属性指定），然后使用这个元类来创建这个类。

这里说明一下 `__new__` 和 `__init__` 在元类中的作用：

- `__new__` 方法在创建类的实例（这里的实例实际上是一个类，因为我们在讨论元类）之前被调用，负责返回一个新的实例。对于元类，`__new__` 方法通常用于在类被正式创建之前拦截和修改类的定义。
- `__init__` 方法在类被创建并由 `__new__` 返回之后被调用，用于初始化这个新创建的类。此时，类已经被创建出来了，`__init__` 方法可以对这个类做一些额外的设置或初始化工作。

举个例子说明元类的工作流程：

```python
class Meta(type):
    def __new__(cls, name, bases, dct):
        # 在类的构造之前被调用
        # 这里可以修改类的定义
        print("Creating class", name)
        return super().__new__(cls, name, bases, dct)

    def __init__(cls, name, bases, dct):
        # 在类的构造之后被调用
        # 这里可以初始化类
        print("Initialized class", name)
        super().__init__(name, bases, dct)

class MyClass(metaclass=Meta):
    def method(self):
        pass
```

输出如下：

```
Creating class MyClass
Initialized class MyClass
```

这表明元类的 `__new__` 方法首先被调用以创建 `MyClass` 类，然后一旦类被创建，`__init__` 方法随后被调用来初始化这个新创建的类。

### 数据类

```python 
class DataKlass:
    def __init__(self, name, value, data):
        self.name = name
        self.value = value
        self.data = data
```

从Python3.7开始，为了简化上述对象类的开发，官方提供一个装饰器，用以方便地定义这种在面向对象中常用的数据类，具体使用如下：

```python
@dataclasses.dataclass
class DataKlass:
    name: str
    value: str
    data: []
```

使用时也很方便，只需要在构造时对应填入对应的值即可完成构造：

```python
def function():
    obj = DataKlass(name="abc", value="123", data=[1, 2, 3])
```

### 反射

Python 的内置函数如 `getattr`、`setattr`、`hasattr` 和 `delattr` 等提供了反射能力

> 将实例方法绑定实例

```python
def build_smart_completer(command_infos: {}):
    completer_nest = {}
    for command_name, command_info in command_infos.items():
        # 使用方法的的__get__魔术方法
        completer_nest[command_name] = command_info.completer.__get__(command_info.command_obj)
    return completer_nest
```

> 反射调用方法

```python
class MyClass:
    def my_method(self, x, y):
        return x + y

# 创建MyClass的一个实例
instance = MyClass()

# 获取my_method方法的引用，注意我们这里并没有调用它，所以没有加()
method_variable = instance.my_method

# 现在，你可以像调用任何其他函数一样调用method_variable
# 因为method_variable是从实例上获取的，它自动绑定到那个实例上
# 所以你不需要显式传递self参数，就可以像下面这样调用
result = method_variable(3, 4)

print(result)  # 输出7，因为3 + 4 = 7
```





### 系统开发中插件功能开发(动态模块导入、元类的使用)

在很多场合我们需要实现已有系统的插件/模块功能，拥有这种能力意味着需要先在系统中定义好一个基类，然后在系统源码外的任何地方继承该基类并拓展该基类的功能以完成插件或者模块的功能，但是由于Python解释器工作时只会扫描已导入的模块，对于未导入的模块Python解释器无法对其进行加载，这也就导致了就算在插件类中导入了系统中的基类，Python解释器也无法加载该插件。

上述问题中存在两个难点：

1. 子类位于其他目录中
2. 子类未被系统显示导入

对于上述问题的解决思路应该是先通过*importlib.util*库中的函数动态导入这些自定义的模块，然后由于这些模块继承的基类定义了元类，再通过元类中的代码将这些子类加入到系统的管理中。

#### 使用*importlib.util*实现动态导入模块

```python
def import_dir(modules_dir):
    if os.path.isdir(modules_dir):
        for file in os.listdir(modules_dir):
            if file.endswith('.py') and not file.startswith('_'):
                module_name = str(file)
                module_path = os.path.join(modules_dir, module_name)

                spec = importlib.util.spec_from_file_location(module_name, module_path)
                print(spec)
                module = importlib.util.module_from_spec(spec)
                spec.loader.exec_module(module)

                return module
```

#### 定义元类实现子类的注册

```python
class Meta(type):
    _registry = []

    def __init__(cls, name, bases, nmspc):
        super(Meta, cls).__init__(name, bases, nmspc)
        Meta._registry.append(cls)

class Base(metaclass=Meta):
    @classmethod
    def get_registered_subclasses(cls):
        return Meta._registry
```



## 编码检测

### chardet库

`chardet`是Python中的一个第三方库，用于编码检测。它可以自动检测一个给定字节字符串的编码，是处理未知字符编码数据时非常有用的工具。在进行文本数据处理时，尤其是处理来源不明确或者支持多种编码的数据时，`chardet`可以帮助你确定正确的字符编码，以便正确地解码字节序列到字符串。

#### 主要用途

- **自动字符编码检测**：当你不确定一个文本文件或字节流的编码方式时，你可以使用`chardet`来检测其编码。这在处理用户上传的文件或爬取的网页内容时尤其有用。
- **数据清洗和预处理**：在机器学习或数据分析项目中，数据来自多个源，编码可能不统一。使用`chardet`进行编码检测和统一，是数据清洗步骤中的一个重要环节。

#### 使用示例

首先，你需要安装`chardet`。如果尚未安装，可以通过pip安装：

```sh
pip install chardet
```

下面是一个简单的例子，展示如何使用`chardet`来检测字符编码：

```python
import chardet

# 假设我们有一些不确定编码的字节数据
data = b'\xcf\x84o\xcf\x81\xce\xbdo\xcf\x82'

# 使用chardet来检测数据的编码
detected_encoding = chardet.detect(data)

print(detected_encoding)
# 输出示例：{'encoding': 'Windows-1253', 'confidence': 0.73, 'language': 'Greek'}

# 现在我们知道了编码，可以正确地解码这些字节
text = data.decode(detected_encoding['encoding'])
print(text)
# 输出应该是原始文本内容，根据实际编码而定
```

`chardet.detect`函数返回的是一个字典，包含了编码(`encoding`)、检测的置信度(`confidence`)和语言(`language`)。置信度是一个0到1之间的浮点数，表示检测结果的可靠程度。

#### 注意事项

尽管`chardet`能够非常有用，但它并不是万能的。它的检测结果有时可能不是100%准确，特别是对于较短的文本或者混合了多种编码的文本。因此，在使用`chardet`检测结果时，评估置信度，并在可能的情况下进行验证是一个好的实践。

此外，需要注意的是，`chardet`的检测过程可能会相对耗时，尤其是在处理大量或大尺寸的数据时。在性能敏感的应用中，合理使用`chardet`是很重要的。

除了`chardet`，确实还有其他一些库可以用于检测字符编码。根据应用场景和性能要求的不同，你可以选择下面列出的一些流行的库：

1. **cChardet**

   `cChardet`是`chardet`的一个C语言实现版本，它和`chardet`有相似的API，但是速度更快。这是因为它是使用C语言编写的，而不是Python，所以在处理大量数据时表现更佳。如果你的应用需要频繁地检测字符编码，且对性能有较高的要求，`cChardet`可能是一个更好的选择。

   安装方式（通过pip）：

   ```sh
   pip install cchardet
   ```

2. **python-magic**

   `python-magic`是一个库，可以用来识别文件的类型。这包括基于文件的内容来判断其编码类型。`python-magic`是基于Unix文件系统中的`libmagic`库的，所以它不仅能识别编码，还能识别大量不同的文件类型。这对于需要处理多种不同文件和数据类型的应用来说非常有用。

   安装方式（通过pip）：

   ```sh
   pip install python-magic
   ```

3. **UnicodeDammit**

   `UnicodeDammit`是`Beautiful Soup`库的一部分，虽然它是作为HTML/XML解析器库的一部分被引入的，但`UnicodeDammit`也可以单独用作编码检测工具。它尝试检测一个给定文本块的编码，并将其转换为UTF-8编码。如果你已经在使用`Beautiful Soup`进行网页抓取或解析工作，那么使用`UnicodeDammit`来处理编码问题会非常方便。

   安装`Beautiful Soup`（`UnicodeDammit`包含其中）：

   ```sh
   pip install beautifulsoup4
   ```

这些库与`chardet`一起，为处理字符编码检测提供了多种选择。选择哪一个主要取决于你的具体需求，比如处理速度、依赖关系以及你的项目是否已经使用了其中的某个库。

### 编码检测算法

编码检测库通过分析给定文本的字节模式和频率等特征来推断文本的编码。不是所有库都会显式给出检测的置信度。置信度是一个实用的指标，表示库对其检测结果准确性的自信程度。即便没有提供置信度，检测结果也不一定绝对准确，因为自动编码检测固有的是一个基于统计和概率的过程。以下是几种主流的编码推断算法及其工作原理的概述：

### 1. 频率分析

- **原理**：不同语言和编码体系中，特定字节或字节序列出现的频率有明显差异。例如，某些字节在UTF-8编码的中文文本中出现的频率会与在ISO-8859-1编码的英文文本中的频率大不相同。通过分析这些频率模式，可以推断出文本的可能编码。
- **局限性**：对于较短的文本或内容不足以显现出典型频率分布的文本，准确率会下降。

### 2. 字节序标记（BOM）检测

- **原理**：某些Unicode编码（如UTF-8, UTF-16, UTF-32）可以包含一个字节序标记（BOM），这是一系列特定的字节，位于文本的开始位置，用来标识编码和字节序。检测BOM是判断文本编码的直接和准确方法。
- **局限性**：并非所有文本都包含BOM。此外，依赖BOM只适用于支持BOM的Unicode编码。

### 3. 特定编码模式匹配

- **原理**：某些编码方式有特定的字节模式或结构特征。例如，UTF-8编码规定了多字节字符的特定字节序列模式。通过检测这些模式，可以推断出文本的编码。
- **局限性**：需要详细了解各种编码规范，并且对于包含少量多字节字符的文本，检测可能不够准确。

### 4. 语言提示

- **原理**：如果已知或可以推断文本的语言，这可以进一步帮助确定编码，因为不同语言可能更倾向于使用特定的编码（例如，日文可能使用Shift_JIS或EUC-JP）。
- **局限性**：需要额外的语言检测步骤，且语言与编码间的对应关系并不总是一对一的。

### 示例和实践

- `chardet`和`cChardet`主要采用基于频率分析的算法，并给出置信度。这使得它们在处理各种文本时相对通用，但也意呴着结果并非总是100%准确。
- `python-magic`通过利用文件内容的特定模式以及可能的BOM来判断文件类型和编码，其更倾向于系统级的文件类型检测，而不是纯粹的文本编码检测。
- `UnicodeDammit`（从`Beautiful Soup`库）尝试使用多种方法（包括BOM检测和内置编码推断逻辑）来猜测文档编码，尽管它特别为HTML或XML文档设计，但它的方法可以泛用于其他文本类型。

总的来说，自动编码检测库提供了有用的工具来猜测未知编码的文本，但由于依赖于概率和统计，任何自动检测方法都不能保证100%的准确率。在可能的情况下，结合文档的上下文信息和编码检测的置信度，对结果进行人工检查或确认，可以提高处理文本编码的准确性和鲁棒性。

## 并发（Concurrency）

并发是指系统处理多个任务的能力，这不一定意味着同时发生。在单核CPU系统中，通过任务之间快速切换给用户一种同时执行的错觉。并发主要用于IO密集型任务，如网络操作、磁盘操作等，这些任务的特点是CPU计算量不大，但等待时间长。

#### 并发的适用场景：

- **网络请求处理**：服务器同时处理多个客户端请求。
- **用户界面应用程序**：在等待用户输入的同时，还需要响应其他事件，如动画显示或网络数据接收。

### 线程（Threading）

Python的`threading`模块允许并发执行。在Python中，由于全局解释器锁（GIL）的存在，同一时刻只允许一个线程执行Python字节码。因此，线程在Python中主要适用于IO密集型任务。



### 协程

Python的协程调度由异步事件循环负责完成，`asyncio`库提供了默认的事件循环实现。协程的调度依据主要包括：

- **事件循环**：事件循环负责管理和调度执行所有注册的协程。当一个协程挂起等待某个操作完成时（例如，通过`await`），事件循环会检查是否有其他的协程已经准备好继续执行，从而实现协程之间的切换。
- **挂起点**：协程通过`await`表达式指定了挂起点，在这些点上协程可以暂停执行，使得事件循环可以安排其他任务的执行。事件循环使用这些挂起点来决定何时切换执行其他协程。
- **Future对象和Task对象**：在底层，`asyncio`用`Future`对象表示一个异步操作的最终结果，而`Task`对象则是`Future`的子类，代表事件循环中的一个可调度的协程。当一个协程等待`Future`对象时，它实际上被挂起，直到该对象的状态变为完成（resolved）。事件循环会监控这些`Future`和`Task`对象的状态，一旦某个对象标记为完成，它就安排相应的协程恢复执行。

一个简单的示例：

```python
import asyncio

async def task1():
    print('Task 1 开始执行')
    await asyncio.sleep(2)  # 模拟耗时2秒的I/O操作
    print('Task 1 完成')

async def task2():
    print('Task 2 开始执行')
    await asyncio.sleep(1)  # 模拟耗时1秒的I/O操作
    print('Task 2 完成')

async def main():
    # 创建task1和task2的任务, 并启动它们. 注意这并不会等待这两个任务完成
    task1_coro = task1()
    task2_coro = task2()

    # 等待两个任务都完成
    await asyncio.gather(task1_coro, task2_coro)

# 运行事件循环
asyncio.run(main())
```

流程是这样的：

1. **main** 函数开始执行。
2. **main** 函数中，**task1** 函数被调用，当遇到`await asyncio.sleep(2)`时，**task1** 挂起，控制权返回给事件循环。
3. 控制权回到**main** 函数后，随即开始执行**task2**。**task2** 同样在遇到`await asyncio.sleep(1)`时挂起，控制权再次返回给事件循环。
4. 此时，两个协程**task1** 和 **task2** 都处于挂起状态，事件循环将等待它们所等待的操作（在此示例中是`asyncio.sleep()`中模拟的时间）完成。
5. 当**task2** 的`await asyncio.sleep(1)`等待完成后（因为它只等待1秒），**task2** 被唤醒并继续执行，打印`"Task 2 完成"`。之后**task2** 运行结束，控制权返回给事件循环。
6. 紧接着，**task1** 的`await asyncio.sleep(2)`等待也完成了（因为它总共等待了2秒），**task1** 被唤醒并继续执行，打印`"Task 1 完成"`。之后**task1** 运行结束，控制权返回给事件循环。
7. 最后，当**task1** 和 **task2** 都完成后，**main** 函数中的`await asyncio.gather(task1_coro, task2_coro)`完成等待，**main** 函数继续执行直至最终完成。

一些常用的`asyncio`协程同步工具：

#### 1. Lock（锁）

- `asyncio.Lock`提供了一个基本的协程锁，用于保护共享资源，防止同时由多个协程访问，从而避免竞态条件。它的使用方法类似于线程锁，但是是异步的，使用`await`来获取和释放。

#### 2. Event（事件）

- `asyncio.Event`允许协程等待某些事件的发生。一旦事件被设置，所有等待这个事件的协程将被唤醒。事件对象主要用于协程之间的信号传递。

以下是如何使用`asyncio.Event`的代码示例：

```python
import asyncio

# 事件初始化
event = asyncio.Event()

async def loader():
    print("开始加载资源...")
    # 模拟异步的资源加载过程
    await asyncio.sleep(2)
    print("资源加载完成！")
    # 设置事件，通知所有等待的协程资源已经加载完成
    event.set()

async def worker(name):
    print(f"{name} 等待资源加载...")
    # 等待事件被设置
    await event.wait()
    print(f"{name} 开始执行任务...")

async def main():
    # 创建一个任务，用于加载资源
    load_task = asyncio.create_task(loader())
    
    # 创建多个等待资源加载完成的工作任务
    workers = [asyncio.create_task(worker(f"工作协程 {i}")) for i in range(3)]
    
    # 等待所有任务完成
    await load_task
    await asyncio.gather(*workers)

asyncio.run(main())
```

在这个示例中：

- 我们定义了一个`loader`协程来模拟资源的加载过程，它在加载过程结束后调用`event.set()`来设置事件。设置事件表示通知所有等待此事件的协程可以继续执行。
- `worker`是工人协程，它在开始执行任务之前通过调用`await event.wait()`等待事件被设置。`event.wait()`在事件被设置之前会挂起当前协程的执行，释放控制权给事件循环。
- `main`协程负责启动加载任务以及多个工人协程，并且等待它们全部完成。

#### 3. Condition（条件变量）

- `asyncio.Condition`用于复杂的协程同步场景，它可以使一个协程等待某个条件，而另一个协程可以在该条件发生改变时通知它。这对于满足特定状态之前需要等待的情况特别有用。

#### 4. Semaphore（信号量）

- `asyncio.Semaphore`是一个计数器，用于控制同时访问共享资源或执行某项操作的协程数量。当计数器值大于零时，`await semaphore.acquire()`会减少计数器值并立即执行；当计数器值为零时，协程等待直到它再次变为正值。

#### 5. BoundedSemaphore（有界信号量）

- `asyncio.BoundedSemaphore`与`Semaphore`类似，区别在于它不允许计数器的值超过它的初始值。这可以用来防止某些程序错误，例如将信号量释放得比获取的次数更多。

#### 6. Queue（队列）

- `asyncio.Queue`提供了一个异步先进先出（FIFO）队列，适用于协程之间的消息传递。它支持等待元素被添加到队列中（`await queue.put(item)`）和等待从队列中移除元素（`await queue.get()`），非常适于生产者-消费者模型。

## 并行（Parallelism）

并行是指多个处理器或多核处理器上的多个任务同时执行。并行是真正的同时执行，旨在通过分配独立的计算资源（如CPU核心）来提高计算速度。

#### 并行的适用场景：

- **数据分析与科学计算**：在多核处理器上同时对大量数据进行分析和计算。
- **图像处理**：图像渲染或编辑过程中，对图像的不同部分同时进行处理。

### 进程

Python的`multiprocessing`模块可以充分利用多核CPU，实现真正的并行计算。每个进程都有自己独立的GIL,内存空间，因此可以同时运行多个Python解释器。



## 官方标准库

### 文件和目录访问

#### pathlib



#### os.path



#### tempfile



#### glob



#### shutil



### 数据压缩

#### zlib



#### gzip



#### zipfile



#### tarfile



### 文件格式

#### csv



#### configparser

> 读取配置文件

```python
parser = configparser.ConfigParser()
read_ok = parser.read(config_file_abs_path)
	if not read_ok:
        print("Read Config file failure")
```

读取配置文件时可以通过其返回值判断是否读取成功，其中不会抛出异常，原因是read方法支持传入一个文件列表，read在处理时会跳过其中处理失败的文件。

> 取得其中的键值

```python
value = parser["title"][key]
```



### 加密

#### hashlib



#### hmac



#### secrets



### 通用操作系统操作

#### os



#### io



#### logging



#### time



### 并发

#### threading



#### multiprocessing

用于在运行时创建新进程，并在其之上运行函数或方法

#### subprocess

用于在运行时运行外部程序

#### concurrent



#### queue



### 文本处理

#### re



#### readline



#### 互联网数据处理

#### json



#### base64



### 数据类型

#### copy



#### enum



### 数字/数学

#### math



#### random



### 函数式编程

#### functools



### tty

#### shlex



### 网络

#### socket



#### socketserver

该标准库简化了Python下的TCP、UDP服务器的开发工作，通过继承其基类可以实现对Unix下的TCP、UDP服务器的快速开发



#### asyncio



#### select



#### selectors

```python
import selectors
import socket

# 创建默认的选择器
sel = selectors.DefaultSelector()

# 接受连接
def accept(sock, mask):
    conn, addr = sock.accept()  # Should be ready
    print('accepted', conn, 'from', addr)
    conn.setblocking(False)
    sel.register(conn, selectors.EVENT_READ, read)

# 读取客户端发送的数据并回复
def read(conn, mask):
    data = conn.recv(1000)  # 每次读取1000字节
    if data:
        print('echoing', repr(data), 'to', conn)
        conn.send(data)  # 回复客户端
    else:
        print('closing', conn)
        sel.unregister(conn)
        conn.close()

# 创建套接字
sock = socket.socket()
sock.bind(('localhost', 1234))
sock.listen(100)
sock.setblocking(False)

# 注册套接字为可读，以接受连接
sel.register(sock, selectors.EVENT_READ, accept)

# 事件循环
while True:
    events = sel.select()
    for key, mask in events:
        callback = key.data
        callback(key.fileobj, mask)
```

代码解释：

1. **创建选择器**：首先，创建一个 `selectors.DefaultSelector()` 实例，它用于监控文件对象（在这个例子中是套接字）上的 I/O 事件。
2. **接受连接**：在 `accept` 函数中，服务器接受一个新的连接并将其注册到选择器中，监听读取事件。`conn.setblocking(False)` 确保了套接字的非阻塞模式。
3. **读取数据并回复**：`read` 函数负责从连接读取数据，并将接收到的数据回发给客户端。如果收到的数据为空，表示客户端关闭了连接，服务器也应关闭对应的套接字并从选择器中取消注册。
4. **设置服务器套接字**：服务器套接字绑定到`localhost`的`1234`端口并开始监听。将套接字设置为非阻塞模式，以免在等待连接时阻塞整个程序。
5. **事件循环**：无限循环中调用`sel.select()`等待注册的事件。每当事件发生（连接准备接受或连接准备读取），事件循环就会根据注册的回调函数处理这些事件。

这个示例展示了如何使用`selectors`模块处理多个连接，它通过在单个线程中轮询各个套接字的状态来避免使用多线程或多进程。这种模型能够高效处理大量的网络连接，并广泛应用于网络服务器和客户端的开发中。

#### signal



### 运行时服务

#### abc



#### sys



#### gc



#### inspect



### 导入模块

#### importlib



## 第三方库

### prompt_toolkit

#### Style

样式字符串支持以下一些选项：

- `fg` ：设置前景色，即文本颜色。
- `bg` ：设置背景色。
- `bold` ：设置加粗文本。
- `italic` ：设置斜体文本。
- `underline` ：设置下划线文本。
- `blink` ：设置闪烁文本，但注意该效果在许多现代终端中可能不支持或被禁用。

在文本样式中，可以通过设置 `fg`（前景色，即文字颜色）来改变文本的颜色。以下是一些基本颜色的 `fg` 设置值，包括红橙黄绿青蓝紫以及黑白这些纯颜色。在 `prompt_toolkit` 或其他支持 ANSI 颜色代码的文本处理库中，您可以使用这些颜色值来为文本着色。

- **红色**: `'fg:red'` 或 `'fg:#ff0000'`
- **橙色**: `'fg:ansiyellow'` (橙色通常会被编码为黄色的暗色版本) 或 `'fg:#ffa500'`
- **黄色**: `'fg:yellow'` 或 `'fg:#ffff00'`
- **绿色**: `'fg:green'` 或 `'fg:#008000'`
- **青色** (Cyan): `'fg:cyan'` 或 `'fg:#00ffff'`
- **蓝色**: `'fg:blue'` 或 `'fg:#0000ff'`
- **紫色**: `'fg:magenta'` 或 `'fg:#ff00ff'`
- **黑色**: `'fg:black'` 或 `'fg:#000000'`
- **白色**: `'fg:white'` 或 `'fg:#ffffff'`

请注意，ANSI 颜色标准中的 "yellow" 在可视化上更接近一般所指的 "橙色"，尤其是在暗色系主题中。同时，不同的终端和显示环境可能对颜色的呈现有所区别，因此建议实际测试以确认颜色的显示效果。

ANSI颜色代码通常在终端中有良好的兼容性，而十六进制颜色代码 (`#rrggbb`) 提供了更广泛的颜色选择，但可能不会在所有终端中均被支持。

#### PromptSession

```python
import sys
import inspect

from prompt_toolkit import PromptSession
from prompt_toolkit.completion import WordCompleter, NestedCompleter, PathCompleter

# module_name = 'ui.commands.general' module_obj = sys.modules[module_name] classes_in_module = [member for member,
# member_type in inspect.getmembers(module_obj, inspect.isclass) if member_type.__module__ == module_name]


completer = WordCompleter([
    'apple', 'banana', 'grape', 'orange', 'mango',
    'apricot', 'blueberry', 'cherry'
], ignore_case=False)


smart_completer = NestedCompleter.from_nested_dict({
    "show": NestedCompleter.from_nested_dict({
        "auxiliary": completer,
        "exploit": completer,
        "post": completer
    }),
    "path": PathCompleter()
})


if __name__ == '__main__':
    # for klass in classes_in_module:
    #     print(klass)
    sessionX = PromptSession("MSF X > ", completer=smart_completer)
    sessionY = PromptSession("MSF Y > ", completer=completer)
    while True:
        try:
            msgX = sessionX.prompt()
            if msgX == "jump":
                while True:
                    try:
                        msgY = sessionY.prompt()
                        print("Y msg[" + msgY + "]")
                        print("X msg[" + msgX + "]")
                        if msgY == "exit":
                            break
                    except KeyboardInterrupt:
                        break
            elif msgX == "exit":
                break
        except KeyboardInterrupt:
            break


```

通过``` PromptSession ```创建一个可交互的命令行会话，通过该命令行会话的``` prompt ```方法取得来自命令行的输出。

``` PromptSession ```创建时可以为其指定该命令行所使用的补全器，校验器。



#### Completer

下面分别介绍几种补全器的适用场景与代码示例。

##### 1. WordCompleter

`WordCompleter` 适用于从预定义的单词列表中进行补全。这在诸如编程语言关键词、城市名或常用短语补全时非常有用。

```python
from prompt_toolkit.completion import WordCompleter
from prompt_toolkit.shortcuts import prompt

completer = WordCompleter(['select', 'from', 'where', 'insert', 'update', 'delete'], ignore_case=True)

user_input = prompt('SQL> ', completer=completer)
print(user_input)
```

##### 2. FuzzyCompleter

`FuzzyCompleter` 适用于需要模糊匹配补全候选项的场景，例如搜索提示功能，用户输入的部分字符可以以非连续方式匹配候选项。

```python
from prompt_toolkit.completion import WordCompleter, FuzzyCompleter
from prompt_toolkit.shortcuts import prompt

completer = FuzzyCompleter(WordCompleter(['dog', 'cat', 'camel', 'elephant']))

user_input = prompt('Enter animal: ', completer=completer)
print(user_input)
```

##### 3. NestedCompleter

`NestedCompleter` 适用于有层级结构的补全场景，比如配置项的编辑或命令行工具的子命令补全。

```python
from prompt_toolkit.completion import NestedCompleter
from prompt_toolkit.shortcuts import prompt

completer = NestedCompleter.from_nested_dict({
    'show': {'version': None, 'interfaces': None},
    'exit': None
})

user_input = prompt('$ ', completer=completer)
print(user_input)
```

##### 4. PathCompleter

`PathCompleter` 适用于需要补全文件或目录路径的场景，如文件管理程序或需要输入路径的命令行工具。

```python
from prompt_toolkit.completion import PathCompleter
from prompt_toolkit.shortcuts import prompt

completer = PathCompleter()

user_input = prompt('File: ', completer=completer)
print(user_input)
```

##### 5. DynamicCompleter

`DynamicCompleter` 适用于补全逻辑需要根据用户输入动态变化的场景。比如基于用户之前的命令历史或其他上下文信息来生成补全建议。

```python
from prompt_toolkit.completion import DynamicCompleter, WordCompleter
from prompt_toolkit.shortcuts import prompt


def get_completer():
    # 这里可以根据实际情况动态选取或构建补全器
    words = ['dynamic', 'completion', 'example']
    return WordCompleter(words)

dynamic_completer = DynamicCompleter(get_completer)

user_input = prompt('Type something: ', completer=dynamic_completer)
print(user_input)
```

以上示例覆盖了广泛的使用场景，为集成强大的补全功能提供了良好的起点。

除了之前介绍的几种补全器，`prompt_toolkit` 还提供了其他一些补全器，如 `ThreadedCompleter`、`ConditionalCompleter` 和 `DeduplicateCompleter`。下面是它们的使用示例：

##### 6. ThreadedCompleter

`ThreadedCompleter` 用于在后台线程中执行补全逻辑，这对于可能需要较长时间来生成补全建议的情况特别有用，从而不会阻塞用户界面。

```python
from prompt_toolkit.completion import ThreadedCompleter, WordCompleter
from prompt_toolkit.shortcuts import prompt

# 假设这是一个比较耗时的补全操作
def get_slow_completions():
    return WordCompleter(['slow', 'completion', 'example'])

completer = ThreadedCompleter(get_slow_completions())

user_input = prompt('Type something: ', completer=completer)
print(user_input)
```

##### 7. ConditionalCompleter

`ConditionalCompleter` 根据所提供的条件动态启用或禁用补全器。这在只在特定条件下启用某些补全逻辑时非常有用。

```python
from prompt_toolkit.completion import ConditionalCompleter, WordCompleter
from prompt_toolkit.shortcuts import prompt

def is_evening():
    # 假设这是判断当前是否是晚上的函数
    return True  # 这里简化为总是返回 True

completer = ConditionalCompleter(
    WordCompleter(['good', 'evening']),
    filter_condition=is_evening
)

user_input = prompt('Say something: ', completer=completer)
print(user_input)
```

##### 8. DeduplicateCompleter

`DeduplicateCompleter` 用于去除从多个补全器中可能得到的重复补全建议。

```python
from prompt_toolkit.completion import DeduplicateCompleter, WordCompleter, merge_completers
from prompt_toolkit.shortcuts import prompt

completer = DeduplicateCompleter(
    merge_completers([
        WordCompleter(['duplicate', 'example']),
        WordCompleter(['example', 'unique'])
    ])
)

user_input = prompt('Type something: ', completer=completer)
print(user_input)
```

注意，`merge_completers` 函数用于合并多个补全器，而 `DeduplicateCompleter` 确保结果中不会有重复项。

通过上述示例，可以看出 `prompt_toolkit` 提供了丰富的补全功能，可以灵活地根据不同的需求选择或组合使用。

```python
from prompt_toolkit.completion import FuzzyCompleter, WordCompleter, PathCompleter, NestedCompleter
from prompt_toolkit.completion import CompleteEvent
from prompt_toolkit.document import Document
from prompt_toolkit import PromptSession

def build_smart_completer(commands: list[Command]):
    nested = {}
    for command in commands:
        nested.setdefault(command.name, command.completer())
    return NestedCompleter.from_nested_dict(nested)


class BaseCompleter(WordCompleter):
    def __init__(self, words: list[str]):
        super().__init__(words)


class SimpleCompleter(FuzzyCompleter):
    def get_completions(self, document: Document, complete_event: CompleteEvent):
        pass


# 创建一个包含几个候选词的 WordCompleter 实例
word_completer = WordCompleter([
    'apple', 'banana', 'grape', 'orange', 'mango',
    'apricot', 'blueberry', 'cherry'
], ignore_case=False)

# 使用 FuzzyCompleter 对 WordCompleter 进行封装，以启用模糊匹配
fuzzy_completer = FuzzyCompleter(word_completer)

path_completer = PathCompleter()

nested_completer = NestedCompleter.from_nested_dict({
    'Path': path_completer,
    'Search': fuzzy_completer,
})

smart_completer = nested_completer
```

#### Validation

下面是针对 `prompt_toolkit.validation` 模块中各种验证器的具体实现和适用场景的代码示例。

##### 1. ConditionalValidator

**适用场景**：用户输入需要在特定条件下进行验证。比如，只有当用户标记了输入为重要时才需要进行格式校验。

```python
from prompt_toolkit.validation import Validator, ValidationError, ConditionalValidator
from prompt_toolkit import prompt

class EmailValidator(Validator):
    def validate(self, document):
        text = document.text
        if "@" not in text or "." not in text:
            raise ValidationError(message="请输入有效的电子邮件地址", cursor_position=len(text))

# 假设 is_important 为 True 时需要对输入进行验证
is_important = True 
validator = ConditionalValidator(EmailValidator(), filter=is_important)

user_input = prompt("请输入您的邮箱：", validator=validator)
```

##### 2. DummyValidator

**适用场景**：开发阶段，或在用户输入不需要进行严格校验的情况下，需要快速通过验证。

```python
from prompt_toolkit.validation import DummyValidator
from prompt_toolkit import prompt

validator = DummyValidator()

user_input = prompt("请输入任意内容：", validator=validator)
```

##### 3. DynamicValidator

**适用场景**：需要根据运行时的某些条件动态决定使用哪个验证器。

```python
from prompt_toolkit.validation import DynamicValidator, Validator, ValidationError
from prompt_toolkit import prompt

def get_validator():
    # 基于某些条件动态返回不同的验证器
    if condition_to_use_email_validator:
        return EmailValidator()
    else:
        return DummyValidator()

validator = DynamicValidator(get_validator)

user_input = prompt("请输入内容：", validator=validator)
```

##### 4. ThreadedValidator

**适用场景**：当验证逻辑可能较为耗时，比如需要网络请求校验输入时，使用 ThreadedValidator 避免界面冻结。

```python
from prompt_toolkit.validation import ThreadedValidator, Validator, ValidationError
from prompt_toolkit import prompt

class SlowValidator(Validator):
    def validate(self, document):
        # 模拟耗时的验证过程
        time.sleep(2)
        if not is_valid_input(document.text):
            raise ValidationError(message="输入不有效", cursor_position=len(document.text))

validator = ThreadedValidator(SlowValidator())

user_input = prompt("请输入内容（可能需要耗时校验）：", validator=validator)
```

##### 5. Validator 从可调用对象创建

**适用场景**：对于简单的验证需求，可以直接从一个验证函数创建 `Validator` 实例，避免定义一个新的 `Validator` 子类。

```python
from prompt_toolkit.validation import Validator
from prompt_toolkit import prompt

def is_valid(text):
    return text.isdigit()

validator = Validator.from_callable(is_valid, error_message="请输入数字", move_cursor_to_end=True)

user_input = prompt("请输入数字：", validator=validator)
```

每个示例对应一种验证器的应用场景和基本用法，以及如何通过 `prompt_toolkit` 的 `prompt` 函数将验证器集成到用户输入过程中。这些示例为如何在实际程序中应用不同的验证器提供了参考。

##### 6. 动态校验器

要为不同的命令适配不同的验证器，并且在一个控制台会话中整合它们，可以类似于使用 `NestedCompleter` 的方式去动态地根据当前的输入（即命令）选择相应的验证器。这需要创建一个能够动态选择验证器的机制，依赖于用户输入的上下文。

一种实现这一思路的方式是创建一个自定义的验证器，它能根据用户已经输入的命令动态地选择并委托给相应的验证器。这种自定义验证器可以称为 `DynamicCommandValidator`，它内部维护一个命令到验证器的映射，并实现相应的逻辑来选择和使用正确的验证器。

下面是这种思路的一个简单实现示例：

```python
from prompt_toolkit.validation import Validator, ValidationError, Document
from prompt_toolkit import PromptSession
from collections import defaultdict

# 定义一个简单的验证器作为示例
class EmailValidator(Validator):
    def validate(self, document: Document) -> None:
        text = document.text
        if "@" not in text or "." not in text:
            raise ValidationError(message="请输入有效的电子邮件地址", cursor_position=len(text))

class NumberValidator(Validator):
    def validate(self, document: Document) -> None:
        text = document.text
        if not text.isdigit():
            raise ValidationError(message="请输入一个数字", cursor_position=len(text))

# 动态命令验证器
class DynamicCommandValidator(Validator):
    def __init__(self):
        # 映射命令到它们的验证器
        self.validators = defaultdict(Validator)
        self.validators["email"] = EmailValidator()
        self.validators["number"] = NumberValidator()

    def validate(self, document: Document) -> None:
        # 假设命令和参数由空格分隔
        parts = document.text.split()
        if not parts:
            # 没有输入命令
            raise ValidationError(message="请输入命令", cursor_position=0)
        cmd = parts[0]

        # 根据命令选择验证器
        validator = self.validators.get(cmd)
        if not validator:
            # 未知命令
            raise ValidationError(message=f"未知命令: {cmd}", cursor_position=0)
        
        # 让选定的验证器来验证输入
        validator.validate(Document(text=' '.join(parts[1:])))

# 创建一个PromptSession使用自定义的动态验证器
session = PromptSession(validator=DynamicCommandValidator())

while True:
    try:
        text = session.prompt('> ')
        print('Accepted:', text)
    except ValidationError as e:
        print(e)
```

在这个示例中，我们定义了两个简单的验证器 `EmailValidator` 和 `NumberValidator`，然后创建了一个 `DynamicCommandValidator` 去动态选择和使用这些验证器基于用户输入的命令。这里的 `DynamicCommandValidator` 假设命令和参数是由空格分隔的，并且当遇到未知命令时，会抛出一个验证错误。

这种方法允许你在一个控制台会话中为不同命令应用不同的验证逻辑，实现了类似于 `NestedCompleter` 的功能但用于验证场景。
