---
title: 大蟒蛇-rich库
layout: post
tags:
  - rich
  - python
categories:
  - python
ai: ChatGPT
ai_ver: '5'
abbrlink: 30447
date: 2025-10-28 21:10:46
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

## 🪶 Rich 库新手使用指南


`Rich` 是一个超强的 Python 库，用来让终端输出变得**漂亮、有色彩、有格式**。
它可以显示**彩色文字、表格、进度条、Markdown、高亮代码、日志信息**等。

安装非常简单：

```bash
pip install rich
```

导入方式：

```python
from rich import print
from rich.console import Console
```


### 一、彩色输出基础

Rich 自带的 `print` 可以直接使用类似 BBCode 的标签上色。

```python
from rich import print

print("[bold red]错误：[/bold red] 文件未找到！")
print("[green]任务完成！[/green]")
print("[yellow on black]警告：内存不足[/yellow on black]")
```

`[color]文字[/color]`
支持样式如：`bold`, `italic`, `underline`, `red`, `green`, `blue`, `on color`, `blink` 等。

### 二、使用 Console 对象（更灵活）

```python
from rich.console import Console

console = Console()
console.print("普通输出")
console.print("[bold magenta]高亮输出[/bold magenta]")
console.log("这是带时间戳的日志输出")
```

### 三、打印表格

```python
from rich.console import Console
from rich.table import Table

table = Table(title="学生成绩")

table.add_column("姓名", style="cyan", no_wrap=True)
table.add_column("科目", style="magenta")
table.add_column("分数", justify="right", style="green")

table.add_row("张三", "数学", "88")
table.add_row("李四", "英语", "93")
table.add_row("王五", "物理", "75")

console = Console()
console.print(table)
```

输出：彩色、有边框的漂亮表格。

### 四、进度条

```python
import time
from rich.progress import track

for i in track(range(10), description="[cyan]正在处理..."):
    time.sleep(0.3)
```

它会在终端显示一个动态进度条，非常直观。

### 五、打印 Markdown 文本

```python
from rich.console import Console
from rich.markdown import Markdown

md = Markdown("""
# 标题
- 支持列表
- **加粗**
- *斜体*
""")

console = Console()
console.print(md)
```


### 六、代码高亮

```python
from rich.console import Console
from rich.syntax import Syntax

code = '''
def hello():
    print("Hello, world!")
'''

syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console = Console()
console.print(syntax)
```

### 七、日志与追踪

```python
from rich.console import Console
from rich.traceback import install

install()  # 让报错显示彩色堆栈
console = Console()

def divide(a, b):
    return a / b

divide(5, 0)  # 会输出带颜色的错误信息

```

### 八、组合示例：多种元素一起展示

```python
from rich.console import Console
from rich.table import Table
from rich.progress import track
from time import sleep

console = Console()

console.rule("[bold green]Rich 示例")

table = Table(title="任务状态")
table.add_column("任务名", style="cyan")
table.add_column("状态", style="magenta")

for i in track(range(5), description="[yellow]执行任务中..."):
    sleep(0.2)
    table.add_row(f"任务{i+1}", "完成")

console.print(table)
console.rule("[bold blue]执行结束")

```

### 九、常见功能速览

| 功能 | 模块 | 示例 |
| ---- | ---- | ---- |
| 彩色文本 | rich.print | [red]Error[/red] |
| 表格 | rich.table.Table | 美观的表格输出 |
| 进度条 | rich.progress | track() |
| Markdown | rich.markdown.Markdown | 渲染 Markdown |
| 代码高亮 | rich.syntax.Syntax | 高亮 Python 等代码 |
| 日志输出 | console.log() | 彩色时间戳日志 |
| 彩色错误追踪 | rich.traceback.install() | 让异常更清晰 |

## 🌈 Rich 进阶组件使用指南

这些功能让你不仅能“美化输出”，还能在终端中**构建交互式信息界面**。
主要包括：`Panel`、`Layout`、`Tree`、`Live`、`Columns` 等模块。

### 一、Panel —— 给内容加上“信息卡片边框”

`Panel` 用来让内容看起来像一块醒目的提示板。

```python
from rich.console import Console
from rich.panel import Panel

console = Console()

panel = Panel(
    "[bold yellow]操作成功！[/bold yellow]\n数据已保存到数据库。",
    title="通知",
    subtitle="rich.panel 示例",
    border_style="green"
)

console.print(panel)
```

可以设置：

- `title` / `subtitle`：上、下标题
- `border_style`：边框颜色
- `expand=True`：让面板占满终端宽度

### 二、Layout —— 构建终端“界面布局”

`Layout` 能让终端像网页一样分区。适合日志面板、状态栏等。

```python
from rich.console import Console
from rich.layout import Layout
from rich.panel import Panel

console = Console()
layout = Layout()

# 整体分为上下两部分
layout.split(
    Layout(name="header", size=3),
    Layout(name="body", ratio=1)
)

# 再把 body 分为左右
layout["body"].split_row(
    Layout(name="left"),
    Layout(name="right")
)

layout["header"].update(Panel("[bold magenta]Rich Layout 示例"))
layout["left"].update(Panel("左侧面板内容"))
layout["right"].update(Panel("右侧面板内容"))

console.print(layout)
```

可以使用 `split()`（上下）或 `split_row()`（左右）来划分区域。
每个区域都能用 `.update()` 放入 `Panel`、`Table`、`Markdown` 等内容。

### 三、Tree —— 生成漂亮的目录结构

`Tree` 是一个树形结构输出工具，很适合打印文件层级或配置结构。

```python
from rich.console import Console
from rich.tree import Tree

console = Console()

tree = Tree("📂 项目结构")
src = tree.add("src")
src.add("main.py")
src.add("utils.py")

docs = tree.add("docs")
docs.add("readme.md")

tree.add("requirements.txt")

console.print(tree)
```

可搭配 emoji 和颜色使用，让终端目录更直观。

### 四、Live —— 实时更新终端内容（动态界面）

`Live` 能让你持续刷新输出，而不是刷屏。

```python
import time
from rich.live import Live
from rich.table import Table

table = Table()
table.add_column("任务")
table.add_column("进度")

with Live(table, refresh_per_second=4):
    for i in range(5):
        table.add_row(f"任务 {i+1}", f"{(i+1)*20}%")
        time.sleep(1)
```

`Live` 会在同一块区域动态更新内容，非常适合实时监控或进度显示。



### 五、Columns —— 多列并排显示内容

当你想让一堆内容整齐地排在一行（比如日志或状态栏）时，`Columns` 是利器。

```python
from rich.console import Console
from rich.columns import Columns
from rich.panel import Panel

console = Console()

panels = [
    Panel("CPU: 24%", title="系统状态", border_style="cyan"),
    Panel("内存: 1.2 GB", title="内存", border_style="green"),
    Panel("网络: 正常", title="网络", border_style="magenta"),
]

console.print(Columns(panels))
```

会自动根据终端宽度排版，多列自适应。

### 六、Progress（进阶版）—— 多任务进度条

之前的 `track()` 是简化用法。
进阶用法可以同时显示多个任务。

```python
import time
from rich.progress import Progress

with Progress() as progress:
    task1 = progress.add_task("[cyan]下载文件A...", total=100)
    task2 = progress.add_task("[magenta]下载文件B...", total=100)

    while not progress.finished:
        progress.update(task1, advance=2)
        progress.update(task2, advance=1)
        time.sleep(0.05)
```

你可以随时 `update(task_id, advance=x)` 来推进进度，非常适合多线程或循环任务。

### 七、组合示例：一个简易“系统监控界面”

```python
import time
from rich.console import Console
from rich.layout import Layout
from rich.panel import Panel
from rich.live import Live

console = Console()

layout = Layout()
layout.split_column(
    Layout(name="header", size=3),
    Layout(name="main", ratio=1),
    Layout(name="footer", size=3)
)

layout["header"].update(Panel("[bold green]🖥 系统监控"))
layout["footer"].update(Panel("按 Ctrl+C 退出", border_style="red"))

def make_main_panel(cpu, mem):
    return Panel(f"CPU 使用率: {cpu}%\n内存占用: {mem} MB", border_style="cyan")

with Live(layout, refresh_per_second=2):
    for i in range(100):
        cpu = (i * 3) % 100
        mem = 500 + (i * 2)
        layout["main"].update(make_main_panel(cpu, mem))
        time.sleep(0.1)
```

运行后，你会看到一个实时更新的动态“界面”，像个迷你监控台。

## 高级

Rich 的开发者还写了一个基于它的**终端应用框架**：[**Textual**](https://github.com/Textualize/textual)  
它能让你用 Rich 的基础语法写出带窗口、按钮、滚动区的完整 TUI（终端UI）程序。

有余力可以学习一下。
