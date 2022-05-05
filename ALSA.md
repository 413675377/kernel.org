## File Tree Structure
```

```

## PCI 驱动程序的基本流程

1. 定义 PCI ID 表

2. 创建probe回调。

3. 创建remove回调。

4. 创建一个包含上述三个指针的结构。struct pci_driver

5. 创建一个init函数，只调用pci_register_driver()来注册上面定义的 pci_driver 表。

6. 创建一个exit函数来调用该pci_unregister_driver()函数。


## Write an ALSA driver

```c
//1. 检查索引
//2. 创建声卡
//3. 创建组件
//4. 设置driver ID，name
//5. 创建mixer
//6. 注册声卡(析构函数)
//7. 设置Data，返回
//8. 析构函数
//TODO example?
```

## 应用driver
### 单一源文件时（simon.c）
```
# 1. makefile
snd-simon-objs := simon.o
obj-$(CONFIG_SND_SIMON) += snd-simon.o

# 2. Kconfig #TODO 参考Kbuild文档
```
### 多个源文件时（目录）
```
# 1. add below in parent makefile
obj-$(CONFIG_SND) += sound/pci/simon/

# 2. makefile
snd-simon-objs := simon.o abc.o def.o
obj-$(CONFIG_SND_simon) += snd-simon.o

# 3. Kconfig #TODO 参考Kbuild文档
```

## Debug
```
snd_printk()
snd_printdd()仅在CONFIG_SND_DEBUG_VERBOSE设置时编译
snd_printd()  CONFIG_SND_DEBUG
dev_err()
pr_err()
snd_BUG_ON()与WARN_ON()宏类似
```