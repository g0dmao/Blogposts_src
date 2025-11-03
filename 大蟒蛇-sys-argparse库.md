---
title: 大蟒蛇-sys/argparse库
layout: post
tags:
  - python
  - sys
  - argparse
categories:
  - python
ai: ChatGPT
ai_ver: '5'
abbrlink: 22025
date: 2025-09-29 20:53:30
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

# sys库

`sys` 是 Python 标准库中的一个模块，提供了与 Python 解释器及其环境交互的功能。
通过 `sys` 库，你可以访问与 Python 解释器相关的变量和函数，例如命令行参数、标准输入输出、程序退出等。命令行参数部分可用`argparse`库替代。

## sys.argv
描述：命令行参数的列表。`sys.argv[0]` 是脚本的名称，后续元素是传递给脚本的参数。
语法：`sys.argv[]`
示例：
```python
import sys  
  
print("脚本名称:", sys.argv[0])  
print("参数列表:", sys.argv[1:])
```

## sys.exit
描述：用于退出程序。你可以传递一个整数作为退出状态码，通常 `0` 表示成功，非零值表示错误。
语法：`sys.exit(num）`
示例：
```python
import sys

print("程序开始")
sys.exit(0)
print("这行代码不会执行")
```

## sys.stdin/stdout/stderr
描述：`sys.stdin`、`sys.stdout` 和 `sys.stderr` 分别代表标准输入、标准输出和标准错误流。你可以重定向这些流以实现自定义的输入输出行为。
语法：`sys.stderr.write("err")`
示例：
```python
import sys

# 重定向标准输出到文件
with open('output.txt', 'w') as f:
    sys.stdout = f
    print("这行内容将写入 output.txt")

# 恢复标准输出
sys.stdout = sys.__stdout__
print("这行内容将显示在控制台")
```

## sys.version/version_info
描述：提供了当前 Python 解释器的版本信息。
语法：`sys.version`
示例：
```python
import sys

print("Python 版本:", sys.version)
print("版本信息:", sys.version_info)
```

## sys.path
描述：列表，包含了 Python 解释器在导入模块时搜索的路径。可以修改这个列表来添加自定义的模块搜索路径。
示例：
```python
import sys

print("模块搜索路径:", sys.path)
sys.path.append('/custom/path')
print("更新后的模块搜索路径:", sys.path)
```

## 其他

| 属性               | 说明                                           |
| ---------------- | -------------------------------------------- |
| `sys.modules`    | 已加载模块的字典                                     |
| `sys.platform`   | 操作系统平台标识（如 `'win32'`, `'linux'`, `'darwin'`） |
| `sys.executable` | Python 解释器的绝对路径                              |
| `sys.byteorder`  | 字节序（`'little'` 或 `'big'`）                    |
| `sys.maxsize`    | 最大整数值（`2**31-1` 或 `2**63-1`）                 |

# argparser库

## ArgumentParser
描述：用于创建解析命令行参数的对象。
语法：
```python
argparse.ArgumentParser(prog=None, usage=None, description=None, epilog=None, parents=[], formatter_class=<class 'argparse.HelpFormatter'>, prefix_chars='-', fromfile_prefix_chars=None, argument_default=None, conflict_handler='error', add_help=True)
```
示例：
```python
import argparse
parser = argparse.ArgumentParser(description="Example parser")
```

## add_argument——重点
描述：向ArgumentParser对象添加命令行参数。
语法：
```python
ArgumentParser.add_argument(name or flags..., action='store', nargs=None, const=None, default=None, type=None, choices=None, required=False, help=None, metavar=None)
```
示例：
```python
parser.add_argument('--verbose', action='store_true', help='Increase output verbosity')
```

补充：

| 键        | 接受的值                         | 作用                          | 举例                  |
| -------- | ---------------------------- | --------------------------- | ------------------- |
| name     | 字符串                          | 变量的名字                       | ‘radius’            |
| nargs    | 数字或’?‘或’‘或’+’                | 用来说明传入的参数个数（符号意义和正则表达式里的一致) | nargs=’?’ nargs=2   |
| type     | list, str, tuple, set, dict等 | 设置读取参数的类型                   | type=int            |
| default  | 类型跟type统一                    | 设置默认值                       | default=1           |
| choices  | 装选项的list                     | 参数值只能从几个选项里面选择              | choices=[1,2,3,4]   |
| required | True或False                   | 这个可选参数是否必须有（只能用于可选参数！否则报错）  | required=True       |
| help     | 字符串                          | 说明一下这个参数是干嘛的                | help=“I don’t know” |
| action   | 六种内置动作                       | 一旦这个有参数，就会触发相应的动作           | action=‘store_true’ |

## parse_args
描述：解析命令行参数。
语法：
```python
ArgumentParser.parse_args(args=None, namespace=None)
```
示例：
```python
args = parser.parse_args()
print(args.[arg name])
```

## set_defaults
描述：为指定的参数设置默认值。
语法：
```python
ArgumentParser.set_defaults(**kwargs)
```
示例：
```python
parser.set_defaults(name="John Doe", age=30)  
```

## print_help
描述：打印帮助信息。
语法：
```python
ArgumentParser.print_help()
```
示例：
```python
parser.print_help()
```

## add_subparsers
描述：为ArgumentParser对象添加子命令解析器。
语法：
```python
ArgumentParser.add_subparsers(dest=None, parser_class=<class 'argparse.ArgumentParser'>, required=False, help=None)
```
示例：
```python
subparsers = parser.add_subparsers(dest='command')
```


> 这上面几个是最常用的。

---

## add_mutually_exclusive_group
描述：创建一个互斥的参数组，同一时间只能使用组内一个参数。
语法：
```python
ArgumentParser.add_mutually_exclusive_group(required=False)
```
示例：
```python
group = parser.add_mutually_exclusive_group()
group.add_argument('--foo', action='store_true')
group.add_argument('--bar', action='store_true')
```

## parse_known_args
描述：解析命令行参数，但返回一个包含已知参数和未知参数的元组。
语法：
```python
ArgumentParser.parse_known_args(args=None, namespace=None)
```
示例：
```python
args, unknown = parser.parse_known_args()
```

## convert_arg_line_to_args
描述：将从文件读取的一行转换为参数列表。
语法：
```python
ArgumentParser.convert_arg_line_to_args(arg_line)
```
示例：
```python
args = parser.convert_arg_line_to_args("foo --bar=3")
```

## error
描述：在命令行参数解析过程中发生错误时，抛出异常并输出错误信息。
语法：
```python
ArgumentParser.error(message)
```
示例：
```python
parser.error("Invalid argument")
```

## exit
描述：在解析命令行参数时遇到错误时，退出程序。
语法：
```python
ArgumentParser.exit(status=0, message=None)
```
示例：
```python
parser.exit(message="Exiting due to error")
```

## parse_args_from_file
描述：从文件中读取参数并解析。
语法：
```python
ArgumentParser.parse_args_from_file(filename)
```
示例：
```python
parser.parse_args_from_file('args.txt')
```

## add_argument_group
描述：创建一个参数组，用于组织和描述相关参数。
语法：
```python
ArgumentParser.add_argument_group(title=None, description=None)
```
示例：
```python
group = parser.add_argument_group('Optional arguments')
group.add_argument('--output', type=str, help='Output file')
```

## format_help
描述：返回当前帮助信息的格式化字符串。
语法：
```python
ArgumentParser.format_help()
```
示例：
```python
help_text = parser.format_help()
```
