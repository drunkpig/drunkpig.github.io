## 上传自己的包到pypi

### 工程结构

```text
/project
  /module_A
    __init__.py
    xxx.py
  setup.py
  LICENSE
  README.md
  requirements.txt
  MANIFEST.in
```

- setup.py 后面详细说
- LICENSE 采用的开源协议
- README.md 在setup.py里可以读取这个文件作为在pypi上的介绍页面的内容，看了setup.py内容之后发现从任何文件读都可以，不一定是README.md。
- requirements.txt 项目的依赖库
- MANIFEST.in  打包为sdist（源码包）的时候要包含或者排除哪些文件，可以根据语法写在里面。[官方文档](<https://docs.python.org/2/distutils/sourcedist.html#the-manifest-in-template>)



#### setup.py

```python
import setuptools

with open("README.md", "r", encoding='utf-8') as fh:
    long_description = fh.read()

setuptools.setup(
    name="fear-quant-lib",
    version="0.0.1",
    author="goldencold",
    author_email="xuchaoo@gmail.com",
    description="quant fear of market",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/jscrapy/quantlib",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    install_requires=[
        'sqlalchemy',
        'python-dateutil',
        'python-levenshtein',
        'requests',
    ]
    python_requires='>=3.7',
)
```





### 打包过程

1. `python setup.py check`  检查有没有错误
2. `python setup.py sdist bdist_wheel`     sdist是源码包，bdist_wheel 是预编译好的包，两者同时存在的情况下pip优先选择wheel
3. `twine upload dist/*` 上传打好的包，会让你输入pypi的用户密码， twine如果没有，要用pip安装一下。



### 其他问题

1. 如何不上传只是把打包好的文件安装到本地库里

   > python setup.py install

2. 如何更新本地库？

   > pip install --upgrade module-name



### 参考文献

1. <http://wsfdl.com/python/2015/09/06/Python%E5%BA%94%E7%94%A8%E7%9A%84%E6%89%93%E5%8C%85%E5%92%8C%E5%8F%91%E5%B8%83%E4%B8%8A.html>
2. <http://wsfdl.com/python/2015/09/08/Python%E5%BA%94%E7%94%A8%E7%9A%84%E6%89%93%E5%8C%85%E5%92%8C%E5%8F%91%E5%B8%83%E4%B8%8B.html>