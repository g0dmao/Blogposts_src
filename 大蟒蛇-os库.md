---
title: 大蟒蛇-os库
layout: post
tags:
  - python
  - os
categories:
  - python
abbrlink: 37113
date: 2025-09-28 15:25:58
ai: ChatGPT
ai_ver: "5"
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

# Python `os` 模块函数大全速查表

`os` 模块是 Python 标准库中用于与操作系统交互的接口，涵盖了文件/目录操作、环境变量管理、系统信息获取、进程控制等诸多功能。本文整理了常用函数及示例，便于查阅使用。

## os.getcwd()
描述：获取当前工作目录  
语法：`os.getcwd()`  
示例：
```python
import os
print(os.getcwd())
```

## os.chdir()
描述：改变当前工作目录  
语法：`os.chdir(path)`  
示例：
```python
os.chdir("/home/user")
```

## os.listdir()
描述：返回指定目录下的文件和文件夹列表  
语法：`os.listdir(path=".")`  
示例：
```python
print(os.listdir("."))
```

## os.mkdir()
描述：创建单层目录  
语法：`os.mkdir(path)`  
示例：
```python
os.mkdir("test")
```

## os.makedirs()
描述：创建多级目录  
语法：`os.makedirs(path, exist_ok=False)`  
示例：
```python
os.makedirs("a/b/c", exist_ok=True)
```

## os.rmdir()
描述：删除空目录  
语法：`os.rmdir(path)`  
示例：
```python
os.rmdir("test")
```

## os.removedirs()
描述：删除多级空目录  
语法：`os.removedirs(path)`  
示例：
```python
os.removedirs("a/b/c")
```

## os.remove()
描述：删除文件  
语法：`os.remove(path)`  
示例：
```python
os.remove("file.txt")
```

## os.rename()
描述：重命名文件或目录  
语法：`os.rename(src, dst)`  
示例：
```python
os.rename("old.txt", "new.txt")
```

## os.replace()
描述：重命名文件，如果目标已存在会被覆盖  
语法：`os.replace(src, dst)`  
示例：
```python
os.replace("a.txt", "b.txt")
```

## os.stat()
描述：获取文件状态信息  
语法：`os.stat(path)`  
示例：
```python
info = os.stat("file.txt")
print(info.st_size)
```

## os.walk()
描述：递归遍历目录树  
语法：`os.walk(top, topdown=True, onerror=None, followlinks=False)`  
示例：
```python
for root, dirs, files in os.walk("."):
    print(root, dirs, files)
```
补充：
`top`：要遍历的顶级目录的路径。
`topdown (可选)`：如果为 True（默认值），则从顶级开始向下遍历。如果为 False，则从底部的子目录开始向上遍历。
`onerror (可选)`：是一个函数，用于错误处理。如果指定，则应该是一个接受单个参数（异常实例）的函数。如果未指定或为 None，错误将被忽略。
`followlinks (可选)`：如果为 True，则会遍历符号链接指向的目录。

## os.scandir()
描述：高效迭代目录项  
语法：`os.scandir(path=".")`  
示例：
```python
def traversal_files(path):
    for item in os.scandir(path):
        if item.is_dir():
          dirs.append(item.path)

        elif item.is_file():
          files.append(item.path)

    print('dirs:')
    print('\n'.join(dirs))

    print()

    print('files:')
    print('\n'.join(files))

traversal_files(r'./test')
```

## os.path.join()
描述：拼接路径  
语法：`os.path.join(path, *paths)`  
示例：
```python
print(os.path.join("folder", "file.txt"))
```

## os.path.abspath()
描述：获取绝对路径  
语法：`os.path.abspath(path)`  
示例：
```python
print(os.path.abspath("test.txt"))
```

## os.path.basename()
描述：获取路径中的文件名  
语法：`os.path.basename(path)`  
示例：
```python
print(os.path.basename("/home/user/file.txt"))
```

## os.path.dirname()
描述：获取路径中的目录部分  
语法：`os.path.dirname(path)`  
示例：
```python
print(os.path.dirname("/home/user/file.txt"))
```

## os.path.exists()
描述：判断路径是否存在  
语法：`os.path.exists(path)`  
示例：
```python
print(os.path.exists("file.txt"))
```

## os.path.isfile()
描述：判断路径是否为文件  
语法：`os.path.isfile(path)`  
示例：
```python
print(os.path.isfile("file.txt"))
```

## os.path.isdir()
描述：判断路径是否为目录  
语法：`os.path.isdir(path)`  
示例：
```python
print(os.path.isdir("folder"))
```

## os.path.splitext()
描述：分离文件名与扩展名  
语法：`os.path.splitext(path)`  
示例：
```python
print(os.path.splitext("file.txt"))
```

## os.path.getsize()
描述：获取文件大小（字节）  
语法：`os.path.getsize(path)`  
示例：
```python
print(os.path.getsize("file.txt"))
```

## os.environ
描述：环境变量相关  
语法：`os.environ`  
示例：
```python
# 获取环境变量
value = os.environ.get("HOME") # 如果不存在返回 None
# 修改环境变量
os.environ["MY_VAR"] = "123"
os.environ["PATH"] = os.environ["PATH"] + ":/my/custom/path"
# 删除 MY_VAR
del os.environ["MY_VAR"]  
```

## os.system()
描述：执行系统命令  **不推荐，推荐`subprocess`**
语法：`os.system(command)`  
示例：
```python
os.system("ls")
```

## os.popen()
描述：执行命令并获取输出  
语法：`os.popen(command)`  
示例：
```python
output = os.popen("ls").read()
print(output)
```

## os.getpid()
描述：获取当前进程 ID  
语法：`os.getpid()`  
示例：
```python
print(os.getpid())
```

## os.getppid()
描述：获取父进程 ID  
语法：`os.getppid()`  
示例：
```python
print(os.getppid())
```

## os.getlogin()
描述：获取当前登录用户  
语法：`os.getlogin()`  
示例：
```python
print(os.getlogin())
```

## os.getuid() / os.getgid()
描述：获取当前进程用户/组 ID  
语法：`os.getuid()` / `os.getgid()`  
示例：
```python
print(os.getuid(), os.getgid())
```

## os.setuid() / os.setgid()
描述：设置用户/组 ID  
语法：`os.setuid(uid)` / `os.setgid(gid)`  
示例：
```python
os.setuid(1000)
```

## os.umask()
描述：设置文件创建掩码  
语法：`os.umask(mask)`  
示例：
```python
os.umask(0o022)
```

## os.chmod()
描述：修改文件权限  
语法：`os.chmod(path, mode)`  
示例：
```python
os.chmod("file.txt", 0o644)
```

## os.link()
描述：创建硬链接  
语法：`os.link(src, dst)`  
示例：
```python
os.link("a.txt", "b.txt")
```

## os.symlink()
描述：创建符号链接  
语法：`os.symlink(src, dst)`  
示例：
```python
os.symlink("a.txt", "link.txt")
```

## os.readlink()
描述：获取符号链接指向的路径  
语法：`os.readlink(path)`  
示例：
```python
print(os.readlink("link.txt"))
```

## os.utime()
描述：修改文件的访问和修改时间  
语法：`os.utime(path, times=None)`  
示例：
```python
os.utime("file.txt", (1620000000, 1620000000))
```

## os.truncate()
描述：截断文件到指定大小  
语法：`os.truncate(path, length)`  
示例：
```python
os.truncate("file.txt", 100)
```

## os.urandom()
描述：生成随机字节数据  
语法：`os.urandom(n)`  
示例：
```python
print(os.urandom(16))
```

## os.cpu_count()
描述：返回 CPU 核心数  
语法：`os.cpu_count()`  
示例：
```python
print(os.cpu_count())
```

## os.uname()
描述：返回操作系统信息  
语法：`os.uname()`  
示例：
```python
print(os.uname())
```

## os.name
描述：获取操作系统类型标识  
语法：`os.name`  
示例：
```python
print(os.name)
```

## os.sep
描述：路径分隔符  
语法：`os.sep`  
示例：
```python
print(os.sep)
```

## os.linesep
描述：行分隔符  
语法：`os.linesep`  
示例：
```python
print(repr(os.linesep))
```

## os.pathsep
描述：系统路径分隔符  
语法：`os.pathsep`  
示例：
```python
print(os.pathsep)
```

## os.devnull
描述：空设备路径，用于丢弃输出  
语法：`os.devnull`  
示例：
```python
with open(os.devnull, "w") as f:
    f.write("This goes nowhere")
```
