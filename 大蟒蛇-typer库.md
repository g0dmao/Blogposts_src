---
title: 大蟒蛇-typer库
layout: post
abbrlink: 40650
date: 2025-10-28 21:11:11
tags:
categories:
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

## 一、安装

```bash
pip install typer[all]
```

`[all]` 会额外安装颜色和补全功能（推荐）。

## 二、最简单的例子

```python
import typer

def main(name: str):
    typer.echo(f"Hello {name}")

if __name__ == "__main__":
    typer.run(main)
```

运行：
```bash
python3 app.py gdm
```

输出：
```
Hello gdm
```

Typer 会自动解析命令行参数并生成帮助：

```bash
python3 app.py --help
```

```
Usage: app.py [OPTIONS] NAME

Arguments:
  NAME  [required]

Options:
  --help  Show this message and exit.
```


## 三、带类型提示的参数解析

Typer 会根据**Python 类型注解**自动转换输入：

```python
def main(
name: str, 
age: int, 
active: bool = True
):
    typer.echo(f"Name: {name}, Age: {age}, Active: {active}")

if __name__ == "__main__":
    typer.run(main)

```

执行：
```bash
python3 app.py gdm 25 --active
```

或关闭 active：
```bash
python3 app.py gdm 25 --no-active
```


## 四、使用选项（Options）

如果你希望某个参数是可选的、带前缀的选项，可以用 `typer.Option`。

```python
def main(
name: str = typer.Option("World", help="名字"), 
repeat: int = typer.Option(1, help="重复次数")
):

    for _ in range(repeat):
        typer.echo(f"Hello {name}")

if __name__ == "__main__":
    typer.run(main)

```

运行：


```bash
python3 app.py --name gdm --repeat 3
```


## 五、使用参数（Arguments）

`typer.Argument()` 用于定义**位置参数**，即必需参数。

```python
def main(filename: str = typer.Argument(..., help="文件路径")):
    typer.echo(f"Processing file: {filename}")

if __name__ == "__main__":
    typer.run(main)
```

`...` 表示必须提供。

示例：
```bash
python3 app.py data.txt
```


## 六、命令分组（多个子命令）

你可以像写 Git 一样定义多命令结构。

```python
import typer

app = typer.Typer()

@app.command()
def add(a: int, b: int):
    typer.echo(a + b)

@app.command()
def sub(a: int, b: int):
    typer.echo(a - b)

if __name__ == "__main__":
    app()
```

运行：
```bash
python3 app.py add 3 5
python3 app.py sub 10 2
```

帮助信息：
```bash
python3 app.py --help
```

```
Commands:
  add
  sub
```


## 七、默认命令（入口函数）

如果只定义一个命令，也可以这样写：

```python
import typer

app = typer.Typer()

@app.command()
def greet(name: str):
    typer.echo(f"Hi {name}")

if __name__ == "__main__":
    app()

```


## 八、提示输入（Prompt）

Typer 支持交互式输入：

```python
def main(password: str = typer.Option(..., prompt=True, hide_input=True)):
    typer.echo("Password received!")

if __name__ == "__main__":
    typer.run(main)
```

运行后会提示输入密码而不回显。

## 九、颜色输出

```python
typer.secho("成功!", fg=typer.colors.GREEN, bold=True)
typer.secho("警告!", fg=typer.colors.YELLOW)
typer.secho("错误!", fg=typer.colors.RED)
```

## 十、退出与错误

```python
if error:
    typer.echo("Something went wrong.")
    raise typer.Exit(code=1)
```

或：

```python
typer.Abort()  # 相当于 Ctrl+C
```


## 十一、自动补全

为终端安装 Typer 补全脚本：

```bash
app --install-completion

```

然后重启终端。

## 十二、复杂结构（嵌套命令）

可以创建多个 `Typer` 实例，并注册子命令模块：

```python
import typer

app = typer.Typer()
user_app = typer.Typer()
app.add_typer(user_app, name="user")

@user_app.command("create")
def create_user(name: str):
    typer.echo(f"Created user: {name}")

if __name__ == "__main__":
    app()
```

运行：
```bash
python app.py user create gdm
```


## 十三、与Click兼容

Typer 基于 Click，可以混用 Click 的函数：

```python
import typer
import click

@app.command()
@click.option("--debug/--no-debug", default=False)
def run(debug):
    typer.echo(f"Debug = {debug}")

```

## 十四、完整示例

一个带颜色、帮助信息、命令组、选项的综合例子：

```python
import typer

app = typer.Typer(help="一个简单的文件处理工具")

@app.command(help="读取文件内容")
def read(file: str):
    with open(file, "r", encoding="utf-8") as f:
        typer.echo(f.read())

@app.command(help="统计文件行数")
def count(file: str):
    lines = sum(1 for _ in open(file, "r", encoding="utf-8"))
    typer.secho(f"{file}: {lines} lines", fg=typer.colors.GREEN)

if __name__ == "__main__":
    app()

```
