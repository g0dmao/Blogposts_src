---
title: markdown渲染器测试
tags:
  - 博客
categories:
  - 其他
description: this is a test
abbrlink: 50034
date: 2025-01-18 19:24:13
---

@[TOC]

# Markdown 测试文本

## 标题2

这是一个段落，下面是**加粗**和*斜体*文本的示例。

### 列表

- 项目1
- 项目2
  - 子项目2.1
  - 子项目2.2
- 项目3

### 有序列表

1. 第一项

2. 第二项

3. 第三项

### 链接

[点击这里访问OpenAI](https://www.openai.com)

### 图片

![OpenAI logo](https://openai.com/favicon.ico)

### 代码块

```python
def hello_world():
    print("Hello, World!")
```

### 引用

> "我爱吃香肠"

### 表格

| 名称   | 年龄 | 职业  |
|--------|------|-------|
| Alice  | 28   | 医生  |
| Bob    | 34   | 工程师 |
| Carol  | 25   | 设计师 |

### 划线

~~我被划了~~

### 其他

<mark>划重点，标记</mark>

<details>
<summary>可收起文本</summary>
这是可收起文本. 可以包含任意元素, 比如 **粗体**, *斜体*,代码块

```python
print("Hello, World!")
```
</details> 

!!防剧透!!

- [ ] TODO List ！渲染失败

H~2~0
x^2^

脚注测试[^1]

[^1]: 这是一个脚注解释

#### hexo-admonition

!!! note Hexo-admonition 插件使用示例
    这是基于 hexo-admonition 插件渲染的一条提示信息。类型为 note，并设置了自定义标题。
    
    提示内容开头留 4 个空格，可以有多行，最后用空行结束此标记。



!!! warning ""
     This is warning without title.

!!! todo 
    This is todo.

!!! error
    This is error.


####  ~~hexo-tips~~ 

<details>

<summary>自2025.09.07起不再使用，改用 admonition。</summary>

:::warning
warning
:::
     
:::danger
danger
:::
    
:::tip
tip
:::
    
:::mention
mention
:::
   
:::recommend
recommend
:::   

:::note
note
:::    

:::info
info
:::     

:::success
success
:::

:::error
error
:::    

:::bug
bug
:::   

:::todo
todo
:::  

:::example
example 
:::   

:::quote
quote
::: 

:::link
link
:::

:::code
code
:::

:::update
update
:::

:::star
star
:::

:::time
time
:::  

</details>

# 标题大小对比
## 2级
### 3级
#### 4级
##### 5级
###### 6级