
## Linux 使用DT的三个主要目的
### 1. 平台识别  

函数|用途 |路径
--|--|--
setup_arch()|入口| arch/arm/kernel/setup.c
setup_machine_fdt()|搜索并选择最匹配的DT node| arch/arm/kernel/devtree.c
struct machine_desc属性dt_compat|存放compatible|
DTS|设备节点compatiable| arch/arm/boot/dts/xxx.dts

### 2. 运行时配置

在早期启动期间
函数| 作用
--|--
of_scan_flat_dt() |代码扫描设备树并使用帮助程序设置分页 
early_init_dt_scan_chosen()| 用于解析所选节点，包括内核参数
early_init_dt_scan_root() |用于初始化 DT 地址空间模型 early_init_dt_scan_memory() |用于确定可用 RAM 的大小和位置。
setup_machine_fdt() |负责在选择支持板子的正确 machine_desc 后对设备树进行早期扫描

Note： 在大多数情况下，DT将是从Firmware向Kernel传递数据的唯一方法，因此也用于传递运行时和配置数据，例如内核参数字符串和 initrd 映像的位置。
```
chosen {
        bootargs = "console=ttyS0,115200 loglevel=8"; #内核参数字符串
        initrd-start = <0xc8000000>;
        initrd-end = <0xc8200000>;
};
```
### 3. 设备人口

#### DTS中的设备
1. 通常假设任何具有“兼容”属性的节点都代表某种设备，
2.1 树根处的任何节点要么直接连接到处理器总线，
2.2 要么是无法以任何其他方式描述的杂项系统设备。
3. /chosen、/aliases 和 /memory 是不描述设备的信息节点（尽管可以说内存可以被视为设备）。
4. /soc 节点的子节点是内存映射设备，
5. sound 节点代表的不是设备，而是其他设备如何连接在一起创建音频子系统。
对于每一个节点，板支持代码调用 of_platform_populate(NULL, NULL, NULL, NULL) 以开始在树的根部发现设备，即寻找具有“兼容”属性的节点，然后分配并注册一个platform_device，而后又可能绑定到一个platform_driver。

#### DTS中关系
- I2C 设备节点作为I2C总线节点的子节点出现。
- 同样适用于 SPI、MDIO、USB 等。
- 唯一不需要特定类型父设备的设备是 platform_devices（和 amba_devices，但稍后会详细介绍），它们将愉快地存在于 Linux /sys/devices 的基础上树。

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