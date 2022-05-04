[TOC]
## 1. DTS语法

https://elinux.org/Device_Tree_Usage

## 2. DTS的解析流程

启用OF_SELFTEST，make

```makefile
$(obj)/%.dtb: $(src)/%.dts FORCE #1.将dts文件转成.dtb二进制文件
        $(call if_changed_dep, dtc)
        
$(obj)/%.dtb.S: $(obj)/%.dtb #2. dtb转化为汇编dtb.S
        $(call cmd, dt_S_dtb)

#3. 汇编文件被编译成一个目标文件（testcases.dtb.o），并链接到内核映像中。
```

```
struct device_node {
    ...
    struct  device_node *parent;
    struct  device_node *child;
    struct  device_node *sibling;
    ...
};
```

4. 利用DT API[Devicetree_API.md](./Devicetree_API.md)来解析并获取数据

## 3. DT的主要用途

### 1. 平台识别  
在ARM上
函数|用途 |路径
--|--|--
setup_arch()|入口| arch/arm/kernel/setup.c
setup_machine_fdt()|搜索并选择最匹配的DT node| arch/arm/kernel/devtree.c
struct machine_desc中属性dt_compat|存放compatible|
DTS|设备节点compatiable| arch/arm/boot/dts/xxx.dts

### 2. 运行时配置

在早期启动期间
函数| 作用
--|--
of_scan_flat_dt() |扫描设备树并使用帮助程序设置分页 
early_init_dt_scan_chosen()| 解析所选节点，包括内核参数 
early_init_dt_scan_root() |初始化 DT 地址空间模型 early_init_dt_scan_memory() 
setup_machine_fdt() |负责在选择支持板子的正确 machine_desc 后对设备树进行早期扫描

Note： 在大多数情况下，DT将是从Firmware向Kernel传递数据的唯一方法，因此也用于传递运行时和配置数据，例如内核参数字符串和 initrd 映像的位置。
```
chosen {
        bootargs = "console=ttyS0,115200 loglevel=8"; #内核参数字符串
        initrd-start = <0xc8000000>;
        initrd-end = <0xc8200000>;
};
```
### 3. 设备入口

#### DTS中的设备
树根处的任何节点要么直接连接到处理器总线，要么是无法以任何其他方式描述的杂项系统设备。 

- 通常假设任何具有“兼容”属性的节点都代表某种设备，

- /chosen、/aliases 和 /memory 是不描述设备的信息节点（尽管可以说内存可以被视为设备）。

- /soc 节点的子节点是内存映射设备，

- /sound 节点代表的不是设备，而是其他设备如何连接在一起创建音频子系统。

- I2C 设备节点作为I2C总线节点的子节点出现。

- 同样适用于 SPI、MDIO、USB 等。

- 唯一不需要特定类型父设备的设备是 platform_devices（和 amba_devices），它们将愉快地存在于 Linux /sys/devices 的基础上树。

对于每一个设备节点
函数|功能
--|--
.init_early()| 用于需要在引导过程早期执行的任何特定于机器的设置|
.init_irq()| 用于设置中断处理|
.init_machine()|它主要负责用有关平台的数据填充 Linux 设备模型, 即通过解析 DT 并动态分配设备结构来获得设备列表|
of_platform_populate(NULL, NULL, NULL, NULL) |开始在树的根部发现设备，即寻找具有“兼容”属性的节点
platform_device|注册platform device
platform_driver|绑定platform driver

platform_device 是 Linux 用于硬件无法检测到的内存或 I/O 映射设备以及“复合”或“虚拟”设备的概念。虽然 DT 没有“平台设备”术语，但平台设备大致对应于树根处的设备节点和简单内存映射总线节点的子节点

#### Linux中的设备模型
- 每个 i2c_client 都是 i2c_master 的子设备。

- 每个 spi_device 都是 SPI 总线的子设备。

- 对于 USB、PCI、MDIO 等也是如此。

几乎所有 bus_types 都假定它的设备是总线控制器的子设备。

#### Linux中的驱动程序
- 一般行为是由父设备驱动程序在驱动程序 .probe() 时间注册子设备。
- i2c 总线设备驱动程序将为每个子节点注册一个 i2c_client，
- SPI 总线驱动程序将注册其 spi_device 子节点，
- 其他总线类型也是如此。
根据该模型，可以编写一个绑定到 SoC 节点并简单地为其每个子节点注册 platform_devices 的驱动程序。

“simple-bus”在 Devicetree 规范中被定义为一个属性，意思是一个简单的内存映射总线，

## 4. DT单元测试
### 动态解析DTS测试数据
OF Selftest 旨在**测试**提供给设备驱动程序开发人员以获取设备信息..等的接口（包括/linux/of.h）
函数|功能
--|--
selftest_data_add()| 通过以下内核符号读取链接到内核映像的扁平设备树数据|
[`of_fdt_unflatten_tree()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_fdt_unflatten_tree)|展平设备树
attach_node_and_children() |使用 of_attach_node() 将节点附加到活动树中
of_attach_node() |来连接所有其他节点。
update_node_properties() |将重复节点属性更新到活动树的节点。
selftest_data_remove| 以删除最初附加的设备节点（首先分离叶节点，然后向上移动父节点，最终删除整个树）

## 5. 提交DT binding

参考： https://www.kernel.org/doc/html/latest/devicetree/bindings/submitting-patches.html
