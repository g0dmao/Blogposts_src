---
title: Linux C标准IO操作
layout: post
tags:
  - Linux
  - IO
  - 内核
categories:
  - Linux
  - 内核编程
abbrlink: 45385
date: 2025-09-09 20:54:03
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

# 📌 C 标准库 I/O 函数（基于 `<stdio.h>`）

## 1. 打开和关闭文件

```c
#include <stdio.h>  
FILE *fopen(const char *filename, const char *mode); 
int fclose(FILE *stream);
```

- **fopen 参数 mode 常用值：**
    
    - `"r"`：只读（文件必须存在）
        
    - `"w"`：只写（不存在则创建，存在则清空）
        
    - `"a"`：追加写（不存在则创建）
        
    - `"r+"`：读写（必须存在）
        
    - `"w+"`：读写（不存在则创建，存在则清空）
        
    - `"a+"`：读写（追加模式）
        

---

## 2. 读写字符

```c
int fgetc(FILE *stream);     // 读取一个字符
int fputc(int c, FILE *stream);  // 写入一个字符
```

示例：

```c
char c = fgetc(fp);
fputc('A', fp);
```

---

## 3. 读写字符串

```c
char *fgets(char *s, int size, FILE *stream); // 读取一行（最多 size-1 个字符） 
int fputs(const char *s, FILE *stream);       // 写入字符串
```

示例：

```c
char buf[100]; fgets(buf, sizeof(buf), fp); 
fputs("Hello\n", fp);
```

---

## 4. 格式化 I/O

```c
int fprintf(FILE *stream, const char *format, ...);  // 格式化输出到文件 
int fscanf(FILE *stream, const char *format, ...);   // 格式化读取
```

示例：

```c
fprintf(fp, "name=%s age=%d\n", "Tom", 20); 
fscanf(fp, "%s %d", name, &age);
```

---

## 5. 块读写（适合二进制文件）

```c
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream); 
size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
```

示例：

```c
int data[10]; fwrite(data, sizeof(int), 10, fp); // 写10个整数 
fread(data, sizeof(int), 10, fp);  // 读10个整数
```

---

## 6. 文件定位

```c
int fseek(FILE *stream, long offset, int whence);  // 移动文件指针
long ftell(FILE *stream);                         // 获取当前位置 
void rewind(FILE *stream);                        // 回到开头`
```

`whence` 可取：

- `SEEK_SET`：文件开头
    
- `SEEK_CUR`：当前位置
    
- `SEEK_END`：文件末尾
    

---

## 7. 缓冲刷新

```c
int fflush(FILE *stream);  // 强制把缓冲区写入文件
```

常用于写日志，避免异常退出时丢数据。
