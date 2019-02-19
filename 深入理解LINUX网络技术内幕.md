# 深入理解LINUX网络技术内幕

## 关键数据结构

### net_device

存储特定网络设备的所有信息，每一个设备都有一个这样的结构

https://www.linuxjournal.com/article/7268 介绍VLAN

网络设备分为Ethernet卡和Token Ring卡。

字段可分为**配置 统计数据 设备状态 列表管理 流量管理 功能专用 通用 函数指针**

#### 标识符

ifindex  独一无二的ID，以dev_new_index注册时分配给每个设备

iflink (虚拟)隧道设备使用，标识抵达隧道另一端的真是设备

dev_id 

#### 配置字段

有些字段根据网络设备的种类给定默认值，其他字段驱动程序填写。

p54

#### 接口类型和端口

## 用户空间与内核的接口

### 概论

内核通过不同的接口把内部信息输出到用户空间

1. 系统调用sys call

2. procfs(/proc文件系统) 虚拟文件系统，挂在(mount)/proc，允许内核一文件的形式向用户空间输出内部信息。这些文件没有实际存在于磁盘中，但可使用cat more >shell 重定向字符写入磁盘，也可以指定这些文件.用户不能把文件添加到这个目录下面，也不能删除

   主要输出只读数据

   /proc中的目录使用`proc_mkdir`创建

   /proc/net使用`proc_net_fops_create`和`proc_net_remove`注册和删除，都是包裹函数

3. sysctl(/proc/sys) 允许用户读取修改内核变量,内核应明确说明哪些变量接口可见。用户空间两种方式访问sysctl: sysctl 系统调用，procfs.

   信息可写入，只有superuser可写

   这个目录下的文件和目录是以`ctl_table`结构定义的，使用`register_sysctl_table`和`unregister_sysctl_table`函数完成

   ```c
   struct ctl_table
   {
           int ctl_name;                   /* Binary ID */
           const char *procname;           /* Text ID for /proc/sys, or zero */ 
           void *data;
           int maxlen;
           mode_t mode;
           ctl_table *child;
           proc_handler *proc_handler;     /* Callback for text formatting */
           ctl_handler *strategy;          /* Callback function for all r/w */
           struct proc_dir_entry *de;      /* /proc control block */
           void *extra1;
           void *extra2;
   };
   proc_handle strategy 根据文件相关联的变量种类而定，可以被定义的值在 kernel/sysctl.c中P73
   ```

   ```c
   ctl_table初始化
   {
                           .ctl_name       = NET_IPV4_CONF_FORWARDING,
                           .procname       = "forwarding",文件名
                           .data           = &ipv4_devconf.forwarding,输出的内核变量
                           .maxlen         = sizeof(int),参数声明为整数
                           .mode           = 0644,访问权限
                           .proc_handler   = &devinet_sysctl_forward,handler函数
                   }
   ```

   注册文件

   两个参数，指向ctl_table实体的指针, 标识指出新元素放在ctl_table列表的何处1 head 0 tail

   

4. sysfs(sys文件系统)

   

5. ioctl系统调用, 对象是一个文件

   

6. netlind套接字(socket)

   