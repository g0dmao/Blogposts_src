---
title: Linux文件操作调用链探究
layout: post
tags:
  - Linux
  - 调用链
categories:
  - Linux
  - 内核编程
ai: ChatGPT
ai_ver: "5"
abbrlink: 31088
date: 2025-10-09 20:57:35
password: "123123"
---
<details>

<summary>版权信息</summary>

!!! warning

    本文章为博主原创文章。遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans) 版权协议，转载请附上原文出处链接和本声明。


</details>

---

理解完整调用链相当于打通 Linux 内核的“主经脉”，之后再看字符设备、块设备、文件系统、网络设备——都会变得清晰许多。

> 文中提到的方法表是我自己用的叫法，更通用的好像叫文件操作函数集/驱动函数集。

## 创建设备文件：以字符设备为例

### 注册设备号

驱动中首先要注册设备号（主设备号 + 次设备号）：
```c
#include <linux/fs.h>
dev_t dev_num; // 设备号
// 动态分配设备号
alloc_chrdev_region(&dev_num, 0, 1, "mychardev");
// 或者静态分配
// register_chrdev_region(MKDEV(240, 0), 1, "mychardev");
```

相当于你去医院看病，先挂号。

### 初始化并注册cdev结构

你申请了一个字符设备描述对象，该结构体的原型为：
```c
struct cdev {
    struct kobject kobj;               // 供 sysfs 使用
    struct module *owner;              // 模块所属
    const struct file_operations *ops; // 你的驱动函数集合
    dev_t dev;                         // 设备号 (主+次)
    unsigned int count;                // 管理的次设备数量
};
```

记录了：
- 我是谁（dev_t 设备号）
- 我要调用哪组函数（file_operations）    
- 我属于哪个模块（owner）

这些都是需要填充的（用后面的两个函数）。**注意你只需要申请即可。**

然后你将自己的自定义驱动函数注册到了file_operation结构体，这个你要自己填充：
```c
static const struct file_operations my_fops = {
    .owner = THIS_MODULE,
    .open  = my_open,
    .read  = my_read,
    .write = my_write,
    .release = my_release,
};
```

并将这个方法表注册到了全局表：
```c
cdev_init(&my_cdev, &my_fops); //初始化字符设备描述对象，绑定方法表
//将该对象注册进表，告诉内核：“从设备号 dev 开始的 count 个次设备，由这个 cdev 驱动负责。”
cdev_add(&my_cdev, dev_num, 【次设备号数量】);
```

这两个函数会将你的cdev结构体填好，看函数的定义就明白了（简化版）：
```c
void cdev_init(struct cdev *cdev, const struct file_operations *fops)
{
    memset(cdev, 0, sizeof(*cdev));
    INIT_LIST_HEAD(&cdev->list);
    kobject_init(&cdev->kobj, &ktype_cdev_dynamic);
    cdev->ops = fops;
}

int cdev_add(struct cdev *p, dev_t dev, unsigned count)
{
    p->dev = dev;
    p->count = count;
    kobject_add(&p->kobj, NULL, "%s", "char_dev");

    // 把设备登记到全局字符设备表
    return cdev_map_add(p);
}
```

这一步就是你被叫到号，去描述你的病情，填充你的病历。

## 文件操作调用链
### 用户空间层

当我们使用open函数时，

```c
int fd = open("/dev/led", O_RDWR);
```

- 实际上这不是直接进入内核的函数，而是 glibc 对系统调用的封装。
- 它最终会执行一条汇编指令（x86 下是 `syscall`，ARM 下是 `svc #0`）  
    触发**软中断**，陷入内核态。
此时 CPU 从用户态切换到内核态，进入系统调用表。

### 内核系统调用

Linux 内核在 `fs/open.c` 中定义：

```c
SYSCALL_DEFINE3(open, const char __user *filename, int flags, umode_t mode)
{
    return do_sys_open(AT_FDCWD, filename, flags, mode);
}
```

- `SYSCALL_DEFINE3` 是一个宏，定义了系统调用的入口。
- 参数从用户空间复制进内核（使用 `copy_from_user()`）。

> 所以内核做的事就是呼叫Virtual File System层，让VFS层这个专家来处理文件。

### VFS层

> VFS（Virtual File System）是 Linux 文件系统抽象层，
   让所有“文件”看起来一致，不管是 ext4、procfs 还是字符设备。

关键任务：
1. **解析路径 `/dev/led`**：通过 `namei()` 查找 inode。
2. **创建 file 结构体**：`struct file` 表示一个打开的文件描述。
3. **找到 inode 对应的 file_operations**：首先VFS会判断文件类型（inode->i_mode），怎么找，不同文件系统、设备类型找的方法不同，感觉每个类型设备都可以单开一篇介绍了....因为本文是了解大体框架所以这里先略过了...找到的方法表地址这里我们用`f_op`表示。 
4. **调用对应的 open 回调**：`f_op->open(inode, file)`。这里就是调用驱动函数了，你自己开发的驱动函数，好耶！

后续补充：
> 1. inode是索引节点，相当于文件的身份证，记录了文件的元数据，比如文件类型、权限、所有者等....
> 2. 创建file结构体就相当于在进程中实例化了一个文件对象，就是你打开的文件对象。需要知道的是这个结构体里面包含了这个文件对象的属性，包括该对象的**方法表**的地址f_op。
> 3. 怎么找示例：例如字符设备，VFS通过inode查询该设备为字符设备，VFS通过inode提取设备号，以设备号为索引，调用函数查找哈希表`chrdevs[]`——这相当于是设备<->f_ops映射表，找到其中对应的cdev结构体（字符设备对象），然后设置`file->f_op = cdev->ops;` 

同时我们不要混淆inode和file结构体：

|对象|代表什么|生命周期|共享性|
|---|---|---|---|
|**inode**|文件自身（存放在磁盘上）|文件存在期间|所有打开者共享同一个 inode|
|**file**|一次打开操作的实例|从 open 到 close|每个打开者独立一份|

### 设备驱动层

然后你的屎山代码就要开始发力了！

```c
if (file->f_op && file->f_op->open)
    file->f_op->open(inode, file);
```

此时就进入了驱动中你自己写的

```c
static int led_open(struct inode *inode, struct file *file)
{
    // 初始化硬件、设置状态等
    return 0;
}
```

之后，用户执行：

```c
write(fd, buf, len);
read(fd, buf, len);
```

也都会走：
```c
sys_write() → vfs_write() → file->f_op->write()
sys_read()  → vfs_read()  → file->f_op->read()
```

