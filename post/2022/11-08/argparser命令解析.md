tags: python
    argparse



[TOC]



## argparse 命令行解析



### 例1



```python
import argparse

parser = argparse.ArgumentParser(
                    prog = 'ProgramName',
                    description = 'What the program does',
                    epilog = 'Text at the bottom of help')

parser.add_argument('filename')           # positional argument
parser.add_argument('-c', '--count'， metavar='COUNT')      # option that takes a value
parser.add_argument('-v', '--verbose',
                    action='store_true')  # on/off flag
args = parser.parse_args()
print(args.filename, args.count, args.verbose)
```

输出

```shell
(dsdl-sdk) PS D:\workspace\s3utils> python main.py -h
usage: ProgramName [-h] [-c COUNT] [-v] filename
What the program does
positional arguments:
  filename
options:
  -h, --help            show this help message and exit
  -c COUNT, --count COUNT
  -v, --verbose
Text at the bottom of help

```

可见使用argparse一共3个步骤

1. 创建ArgumentParser对象

2. 利用add_argument()方法增加命令选项， 对选项做一些有必要的互斥、默认值等设置

3. 调用parse_args()解析参数，得到一个Namescape对象

   

### ArgumentParser对象的创建

- [prog](https://docs.python.org/zh-cn/3/library/argparse.html#prog) - 程序的名称 (默认值: `os.path.basename(sys.argv[0])`)
- [usage](https://docs.python.org/zh-cn/3/library/argparse.html#usage) - 描述程序用途的字符串（默认值：从添加到解析器的参数生成）
- [description](https://docs.python.org/zh-cn/3/library/argparse.html#description) - Text to display before the argument help (by default, no text)
- [epilog](https://docs.python.org/zh-cn/3/library/argparse.html#epilog) - Text to display after the argument help (by default, no text)
- [parents](https://docs.python.org/zh-cn/3/library/argparse.html#parents) - 一个 [`ArgumentParser`](https://docs.python.org/zh-cn/3/library/argparse.html#argparse.ArgumentParser) 对象的列表，它们的参数也应包含在内
- [formatter_class](https://docs.python.org/zh-cn/3/library/argparse.html#formatter-class) - 用于自定义帮助文档输出格式的类
- [prefix_chars](https://docs.python.org/zh-cn/3/library/argparse.html#prefix-chars) - 可选参数的前缀字符集合（默认值： '-'）
- [fromfile_prefix_chars](https://docs.python.org/zh-cn/3/library/argparse.html#fromfile-prefix-chars) - 当需要从文件中读取其他参数时，用于标识文件名的前缀字符集合（默认值： `None`）
- [argument_default](https://docs.python.org/zh-cn/3/library/argparse.html#argument-default) - 参数的全局默认值（默认值： `None`）
- [conflict_handler](https://docs.python.org/zh-cn/3/library/argparse.html#conflict-handler) - 解决冲突选项的策略（通常是不必要的）
- [add_help](https://docs.python.org/zh-cn/3/library/argparse.html#add-help) - 为解析器添加一个 `-h/--help` 选项（默认值： `True`）
- [allow_abbrev](https://docs.python.org/zh-cn/3/library/argparse.html#allow-abbrev) - 如果缩写是无歧义的，则允许缩写长选项 （默认值：`True`）
- [exit_on_error](https://docs.python.org/zh-cn/3/library/argparse.html#exit-on-error) - 决定当错误发生时是否让 ArgumentParser 附带错误信息退出。 (默认值: `True`)



### add_argument()方法参数

| Name                                                         | Description                                          | Values                                                       |
| :----------------------------------------------------------- | :--------------------------------------------------- | :----------------------------------------------------------- |
| [action](https://docs.python.org/zh-cn/3/library/argparse.html#action) | Specify how an argument should be handled            | `'store'`, `'store_const'`, `'store_true'`, `'append'`, `'append_const'`, `'count'`, `'help'`, `'version'` |
| [choices](https://docs.python.org/zh-cn/3/library/argparse.html#choices) | 限定参数值取值范围                                   | `['foo', 'bar']`, `range(1, 10)`, or [`Container`](https://docs.python.org/zh-cn/3/library/collections.abc.html#collections.abc.Container) instance |
| [const](https://docs.python.org/zh-cn/3/library/argparse.html#const) | Store a constant value                               |                                                              |
| [default](https://docs.python.org/zh-cn/3/library/argparse.html#default) | 默认值                                               | Defaults to `None`                                           |
| [dest](https://docs.python.org/zh-cn/3/library/argparse.html#dest) | 例如add_argument里用的 --name, 想在程序里用 myname   |                                                              |
| [help](https://docs.python.org/zh-cn/3/library/argparse.html#help) | Help message for an argument                         |                                                              |
| [metavar](https://docs.python.org/zh-cn/3/library/argparse.html#metavar) | 打印usage的时候参数的默认值                          | 仅在显示层面替换，dest在code的变量层面替换                   |
| [nargs](https://docs.python.org/zh-cn/3/library/argparse.html#nargs) | 参数可取的个数                                       | [`int`](https://docs.python.org/zh-cn/3/library/functions.html#int), `'?'`, `'*'`, `'+'`, or `argparse.REMAINDER` |
| [required](https://docs.python.org/zh-cn/3/library/argparse.html#required) | Indicate whether an argument is required or optional | `True` or `False`                                            |
| [type](https://docs.python.org/zh-cn/3/library/argparse.html#type) | Automatically convert an argument to the given type  | [`int`](https://docs.python.org/zh-cn/3/library/functions.html#int), [`float`](https://docs.python.org/zh-cn/3/library/functions.html#float), `argparse.FileType('w')`, or callable function |



### 互斥





### 分组