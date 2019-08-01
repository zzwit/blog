
[TOC]

# 实现 打包Python项目 和 上传

> Python Package，在实际项目中应用广泛，包含Public Package和Private Package。它有助于我们对代码进行功能封装，使得软件更加结构化、有序化、层次化。
>
> 建议：创建虚拟环境

### 1 如何打包Python Package

####  1.1   目录 和 文件结构

要在本地创建此项目，请创建以下文件结构：  

```python
/ceshiwenjian
  /example_pkg
    __init__.py
LICENCE
README.md
setup.py
创建此结构后，cd  ceshiwenjian 编辑 /example_pkg/__init__. py文件
name = "example_pkg"
这个名字要注意，为了避免的和其他的包名有冲突，要建立自己独特名字
```



#### 1.2 vim setup.py

`setup.py`是[setuptools](https://packaging.python.org/key_projects/#setuptools)的构建脚本。它告诉setuptools你的包（例如名称和版本）以及要包含的代码文件。

打开`setup.py`并输入以下内容。更新软件包名称以包含您的用户名（例如，`example-pkg-theacodes`），这可确保您拥有唯一的软件包名称，并且您的软件包与本教程后其他人上传的软件包不会发生冲突。

```Python
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="example-pkg-your-username", # 建立和/example_pkg/__init__. py文件 name 保持一致
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)
```



`setup()`需要几个论点。此示例包使用相对最小的集：

- `name`是包的*分发名称*。只要包含字母，数字`_`和，就可以是任何名称`-`。它也不能在pypi.org上使用。**请务必使用您的用户名更新此内容，**因为这样可确保您不会尝试上传与上传程序包时已存在的程序包相同的程序包。
- `version` 是包版本看 [**PEP 440**](https://www.python.org/dev/peps/pep-0440)有关版本的更多详细信息。
- `author`并`author_email`用于识别包的作者。
- `description` 是一个简短的，一句话的包的总结。
- `long_description`是包的详细说明。这显示在Python Package Index的包详细信息包中。在这种情况下，加载长描述`README.md`是一种常见模式。
- `long_description_content_type`告诉索引什么类型的标记用于长描述。在这种情况下，它是Markdown。
- `url`是项目主页的URL。对于许多项目，这只是一个指向GitHub，GitLab，Bitbucket或类似代码托管服务的链接。
- `packages`是应包含在[分发包](https://packaging.python.org/glossary/#term-distribution-package)中的所有Python [导入包](https://packaging.python.org/glossary/#term-import-package)的列表。我们可以使用 自动发现所有包和子包，而不是手动列出每个包。在这种情况下，包列表将是example_pkg，因为它是唯一存在的包。`find_packages()`
- `classifiers`给出了指数和[点子](https://packaging.python.org/key_projects/#pip)你的包一些额外的元数据。在这种情况下，该软件包仅与Python 3兼容，根据MIT许可证进行许可，并且与操作系统无关。您应始终至少包含您的软件包所使用的Python版本，软件包可用的许可证以及您的软件包将使用的操作系统。有关分类器的完整列表，请参阅https://pypi.org/classifiers/。

除了这里提到的还有很多。有关详细信息，请参阅 [打包和分发项目](https://packaging.python.org/guides/distributing-packages-using-setuptools/)。

#### 1.3 编辑 README.md

这里的主要的是你的项目简介什么，注意书写格式

####  1.4 编辑 LICENCE 设置验证许可

上传到Python Package Index的每个包都包含许可证，这一点很重要。这告诉用户安装您的软件包可以使用您的软件包的条款。有关选择许可证的帮助，请访问 https://choosealicense.com/。选择许可证后，打开 LICENSE并输入许可证文本。例如，如果您选择了MIT许可证：

```Python
MIT License

Copyright (c) [year] [fullname]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```



#### 1.5 生成分发档案

确保您拥有`setuptools`并`wheel` 安装了最新版本：

```lixun
python3 -m pip install --user --upgrade setuptools wheel
```

如果报这个错误：

```liunx
ERROR: Can not perform a '--user' install. User site-packages are not visible in this virtualenv.
```

请执行:

```liunx
ython -m ensurepip --default-pip
```

如果还解决不了问题 请查看[安装包教程](https://packaging.python.org/tutorials/installing-packages/)

在`setup.py`位于的同一目录运行此命令：

```liunx
python3 setup.py sdist bdist_wheel
```

此命令应输出大量文本，一旦完成，应在`dist`目录中生成两个文件：

```liunx
dist/
  example_pkg_your_username-0.0.1-py3-none-any.whl
  example_pkg_your_username-0.0.1.tar.gz
```

该`tar.gz`文件是[源存档，](https://packaging.python.org/glossary/#term-source-archive)而该`.whl`文件是 [构建的分发](https://packaging.python.org/glossary/#term-built-distribution)。较新的[pip](https://packaging.python.org/key_projects/#pip)版本优先安装构建的发行版，但如果需要，将回退到源代码存档。您应该始终上传源存档并为项目兼容的平台提供构建的存档。在这种情况下，我们的示例包在任何平台上都与Python兼容，因此只需要一个构建的发行版。

#### 2 上传分发档案

最后，是时候将您的包上传到Python Package Index了！

您需要做的第一件事是在Test PyPI上注册一个帐户。Test PyPI是用于测试和实验的包索引的单独实例，要注册帐户，请访问https://test.pypi.org/account/register/并完成该页面上的步骤。在您上传项目之前，您还需要验证您的电子邮件地址。有关Test PyPI的更多详细信息，请参阅 [使用TestPyPI](https://packaging.python.org/guides/using-testpypi/)。

#### 2.1 现在您已注册，您可以使用[twine](https://packaging.python.org/key_projects/#twine)上传分发包。你需要安装Twine：

```liunx
python3 -m pip install --user --upgrade twine
```

安装完成后，运行Twine以上传所有存档`dist`：

```liunx
python3 -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```

系统将提示您输入使用Test PyPI注册的用户名和密码。命令完成后，您应该看到与此类似的输出：

```liunx
Enter your username: xiaowen
Enter your password:
Uploading distributions to https://test.pypi.org/legacy/
Uploading wchi_pkg-0.0.1-py3-none-any.whl
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 5.49k/5.49k [00:00<00:00, 22.5kB/s]
Uploading wence-0.0.1-py3-none-any.whl
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 5.45k/5.45k [00:01<00:00, 3.68kB/s]
Uploading wchi_pkg-0.0.1.tar.gz
```

上传后，您的软件包应该可以在TestPyPI上查看，例如，[https：](https://test.pypi.org/project/example-pkg-your-username) //test.pypi.org/project/example-pkg-your-username

#### 2.3 完结

更多请参考[官网](https://packaging.python.org/tutorials/packaging-projects/)






