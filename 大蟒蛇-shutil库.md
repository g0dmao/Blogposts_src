---
title: 大蟒蛇-shutil库
layout: post
tags:
  - python
  - shutil
categories:
  - python
ai: ChatGPT
ai_ver: '5'
abbrlink: 22415
date: 2025-09-29 20:41:07
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

shutil库主要用于文件的高级操作，比如移动、复制、压缩和解压

## shutil.copy
描述：复制文件到指定路径。
语法：`shutil.copy(src, dst)`
示例：
```python
import shutil
shutil.copy('source.txt', 'destination.txt')
```

## shutil.copy2
描述：复制文件到指定路径，同时保留文件的元数据。
语法：`shutil.copy2(src, dst)`
示例：
```python
import shutil
shutil.copy2('source.txt', 'destination.txt')
```

## shutil.copytree
描述：递归复制整个目录树。
语法：`shutil.copytree(src, dst)`
示例：
```python
import shutil
shutil.copytree('source_dir', 'destination_dir')
```

## shutil.move
描述：移动文件或目录到指定路径。
语法：`shutil.move(src, dst)`
示例：
```python
import shutil
shutil.move('source.txt', 'destination.txt')
```

## shutil.rmtree
描述：递归删除目录树。慎用！ 
语法：`shutil.rmtree(path)`
示例：
```python
import shutil
shutil.rmtree('directory')
```

## shutil.disk_usage
描述：返回磁盘的总容量、已用空间和剩余空间。
语法：`shutil.disk_usage(path)`
示例：
```python
import shutil
total, used, free = shutil.disk_usage('/')
print(f"Total: {total}, Used: {used}, Free: {free}")
```

## shutil.chown
描述：改变文件或目录的所有者和组。
语法：`shutil.chown(path, user=None, group=None)`
示例：
```python
import shutil
shutil.chown('file.txt', user='username', group='groupname')
```

## shutil.unpack_archive
描述：解压归档文件。
语法：`shutil.unpack_archive(filename, extract_dir=None)`
示例：
```python
import shutil
shutil.unpack_archive('archive.zip', 'destination_folder')
```

## shutil.get_archive_formats
描述：获取支持的归档格式列表。
语法：`shutil.get_archive_formats()`
示例：
```python
import shutil
formats = shutil.get_archive_formats()
print(formats)
```

## shutil.make_archive
描述：创建一个归档文件。
语法：`shutil.make_archive(base_name, format, root_dir=None, base_dir=None)`
示例：
```python
import shutil
shutil.make_archive('archive_name', 'zip', 'folder_to_compress')
```

## shutil.move
描述：移动文件或目录到指定路径。
语法：`shutil.move(src, dst)`
示例：
```python
import shutil
shutil.move('source.txt', 'destination_folder')
```

