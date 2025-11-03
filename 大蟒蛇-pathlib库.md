---
title: 大蟒蛇-pathlib库
layout: post
tags:
  - python
  - pathlib
categories:
  - python
ai: ChatGPT
ai_ver: '5'
abbrlink: 63515
date: 2025-09-28 19:21:17
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

## Path()
描述：创建路径对象  
语法：`Path(path_str)`  
示例：
```python
from pathlib import Path
p = Path('/home/user/docs')

```

## cwd()
描述：获取当前工作目录
语法：`Path.cwd()`
示例：
```python
Path.cwd()

```

## home()
描述：获取用户主目录路径
语法：`Path.home()`
示例：
```python
Path.home()
```

## exists()
描述：判断路径是否存在
语法：`path.exists()`
示例：
```python
Path('test.txt').exists()

```

## is_file()
描述：判断是否为文件
语法：`path.is_file()`
示例：
```python
Path('test.txt').is_file()

```

## is_dir()
描述：判断是否为目录
语法：`path.is_dir()`
示例：
```python
Path('/etc').is_dir()

```

## is_symlink()
描述：判断是否为符号链接
语法：`path.is_symlink()`
示例：
```python
Path('link').is_symlink()

```

## iterdir()
描述：列出目录内容（生成器）
语法：`path.iterdir()`
示例：
```python
for item in Path('.').iterdir():
    print(item)
```

## glob()
描述：使用通配符匹配文件
语法：`path.glob(pattern)`
示例：
```python
list(Path('.').glob('*.py'))
```

## rglob()
描述：递归匹配文件
语法：`path.rglob(pattern)`
示例：
```python
list(Path('.').rglob('*.py'))
```

## mkdir()
描述：创建目录
语法：`path.mkdir(parents=False, exist_ok=False)`
示例：
```python
Path('new_dir').mkdir(exist_ok=True)

```

## rmdir()
描述：删除空目录
语法：`path.rmdir()`
示例：
```python
Path('empty_dir').rmdir()

```

## unlink()
描述：删除文件或符号链接
语法：`path.unlink(missing_ok=False)`
示例：
```python
Path('file.txt').unlink()

```

## rename()
描述：重命名文件或目录
语法：`path.rename(target)`
示例：
```python
Path('old.txt').rename('new.txt')

```

## replace()
描述：重命名（如目标存在则覆盖）
语法：`path.replace(target)`
示例：
```python
Path('a.txt').replace('b.txt')

```

## resolve()
描述：返回绝对路径（解析符号链接）
语法：`path.resolve()`
示例：
```python
Path('..').resolve()

```

## absolute()
描述：返回绝对路径（不解析符号链接）
语法：`path.absolute()`
示例：
```python
Path('file.txt').absolute()

```

## stat()
描述：获取文件状态信息
语法：`path.stat()`
示例：
```python
info = Path('file.txt').stat()
print(info.st_size)

```

## read_text()
描述：读取文本文件内容
语法：`path.read_text(encoding=None)`
示例：
```python
Path('a.txt').read_text()

```

## write_text()
描述：写入文本内容
语法：`path.write_text(data, encoding=None)`
示例：
```python
Path('a.txt').write_text('Hello World')

```

## read_bytes()
描述：读取二进制文件内容
语法：`path.read_bytes()`
示例：
```python
data = Path('a.bin').read_bytes()

```

## write_bytes()
描述：写入二进制数据
语法：`path.write_bytes(data)`
示例：
```python
Path('a.bin').write_bytes(b'\x00\x01')

```

## joinpath()
描述：拼接路径
语法：`path.joinpath(*other)`
示例：
```python
Path('/home').joinpath('user', 'docs')

```

## with_name()
描述：返回更改文件名的新路径
语法：`path.with_name(name)`
示例：
```python
Path('a.txt').with_name('b.txt')

```

## with_suffix()
描述：修改文件扩展名
语法：`path.with_suffix(suffix)`
示例：
```python
Path('a.txt').with_suffix('.md')

```

## suffix
描述：返回路径后缀
语法：`path.suffix`
示例：
```python
Path('a.txt').suffix

```

## suffixes
描述：返回所有后缀（列表）
语法：`path.suffixes`
示例：
```python
Path('archive.tar.gz').suffixes

```

## stem
描述：返回不带后缀的文件名
语法：`path.stem`
示例：
```python
Path('a.txt').stem

```

## name
描述：返回文件名
语法：`path.name`
示例：
```python
Path('/home/user/a.txt').name

```

## parent
描述：返回父路径
语法：`path.parent`
示例：
```python
Path('/home/user/a.txt').parent

```

## parents
描述：返回所有上级目录
语法：`path.parents`
示例：
```python
for p in Path('/a/b/c').parents:
    print(p)

```

## as_posix()
描述：以 POSIX 格式返回路径字符串
语法：`path.as_posix()`
示例：
```python
Path('C:\\a\\b').as_posix()

```

## as_uri()
描述：返回文件URI
语法：`path.as_uri()`
示例：
```python
Path('/home/user/a.txt').as_uri()

```

## relative_to()
描述：返回相对路径
语法：`path.relative_to(base)`
示例：
```python
Path('/home/user/docs/a.txt').relative_to('/home/user')

```

## match()
描述：判断路径是否匹配模式
语法：`path.match(pattern)`
示例：
```python
Path('a.txt').match('*.txt')

```

## touch()
描述：创建空文件或更新修改时间
语法：`path.touch(exist_ok=True)`
示例：
```python
Path('a.txt').touch()

```

## open()
描述：打开文件（与内置 open 类似）
语法：`path.open(mode='r', encoding=None)`
示例：
```python
with Path('a.txt').open('w') as f:
    f.write('data')

```
