---
title: 大蟒蛇-json库
layout: post
tags:
  - json
  - python
categories:
  - python
ai: ChatGPT
ai_ver: '5'
abbrlink: 50277
date: 2025-10-23 11:25:00
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

## json.load()
**描述：** 从文件对象读取 JSON 数据并反序列化为 Python 对象  
**语法：**
```python
json.load(fp, *, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)
```
**示例：**
```python
import json
with open('data.json', 'r') as f:
    data = json.load(f)
```

## json.loads()
**描述：** 从字符串读取 JSON 数据并反序列化为 Python 对象  
**语法：**
```python
json.loads(s, *, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)
```
**示例：**
```python
data = json.loads('{"name": "Alice", "age": 25}')
```

## json.dump()
**描述：** 将 Python 对象序列化为 JSON 并写入文件  
**语法：**
```python
json.dump(obj, fp, *, skipkeys=False, ensure_ascii=True, check_circular=True,
          allow_nan=True, cls=None, indent=None, separators=None,
          default=None, sort_keys=False, **kw)
```
**示例：**
```python
data = {"name": "Bob", "age": 30}
with open('data.json', 'w') as f:
    json.dump(data, f, indent=4)
```

## json.dumps()
**描述：** 将 Python 对象序列化为 JSON 字符串  
**语法：**
```python
json.dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True,
           allow_nan=True, cls=None, indent=None, separators=None,
           default=None, sort_keys=False, **kw)
```
**示例：**
```python
s = json.dumps({"x": 1, "y": 2}, indent=2)
```

## json.JSONDecoder
**描述：** JSON 反序列化类，可自定义解析逻辑  
**语法：**
```python
class json.JSONDecoder(object_hook=None, object_pairs_hook=None, parse_float=None,
                       parse_int=None, parse_constant=None, strict=True, **kw)
```
**示例：**
```python
decoder = json.JSONDecoder()
obj = decoder.decode('{"a":1,"b":2}')
```

## json.JSONEncoder
**描述：** JSON 序列化类，可自定义编码行为  
**语法：**
```python
class json.JSONEncoder(skipkeys=False, ensure_ascii=True, check_circular=True,
                       allow_nan=True, sort_keys=False, indent=None, separators=None, default=None)
```
**示例：**
```python
encoder = json.JSONEncoder(indent=2)
s = encoder.encode({"name": "test"})
```

## json.JSONDecodeError
**描述：** 解码 JSON 时的异常类型（继承自 `ValueError`）  
**语法：**
```python
try:
    json.loads('invalid json')
except json.JSONDecodeError as e:
    print(e)
```

## json.tool（命令行模块）
**描述：** 命令行工具，用于格式化 JSON 输出  
**语法：**
```bash
python -m json.tool input.json
```
**示例：**
```bash
echo '{"a":1,"b":2}' | python -m json.tool
```

## 参数说明（适用于 dump/dumps/load/loads）

### indent
控制缩进格式（整数或字符串）  
```python
json.dumps(data, indent=4)
```

### separators
自定义分隔符（默认 `(', ', ': ')`）  
```python
json.dumps(data, separators=(',', ':'))
```

### sort_keys
按键名排序输出  
```python
json.dumps(data, sort_keys=True)
```

### ensure_ascii
是否转义非 ASCII 字符  
```python
json.dumps({"中文": "测试"}, ensure_ascii=False)
```

### default
定义无法序列化对象的处理函数  
```python
def encode_complex(obj):
    if isinstance(obj, complex):
        return {"real": obj.real, "imag": obj.imag}
    raise TypeError()

json.dumps(1+2j, default=encode_complex)
```

### object_hook
自定义反序列化处理函数  
```python
def as_person(d):
    return Person(d['name']) if 'name' in d else d

json.loads('{"name":"Alice"}', object_hook=as_person)
```

### parse_float, parse_int, parse_constant
用于自定义解析数字/常量的函数  
```python
json.loads('{"num": 1.23}', parse_float=lambda x: round(float(x), 1))
```

## 常见用法对比

| 操作   | 从字符串         | 从文件         | 输出为字符串       | 输出为文件       |
| ---- | ------------ | ----------- | ------------ | ----------- |
| 反序列化 | json.loads() | json.load() | —            | —           |
| 序列化  | —            | —           | json.dumps() | json.dump() |

