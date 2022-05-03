
### Linux 使用DT的三个主要目的
#### 1. 平台识别  

函数|用途 |路径
--|--|--
setup_arch()|入口| arch/arm/kernel/setup.c
setup_machine_fdt()|搜索并选择最匹配的DT node| arch/arm/kernel/devtree.c
struct machine_desc属性dt_compat|存放compatible|
DTS|设备节点compatiable| arch/arm/boot/dts/xxx.dts

#### 2. 运行时配置

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
#### 3. 设备人口

在 .init_machine() 时，Tegra 板支持代码将需要查看此 DT 并决定为哪些节点创建 platform_devices。
但是，查看树，并不能立即看出每个节点代表什么样的设备，或者即使一个节点根本代表一个设备。
/chosen、/aliases 和 /memory 是不描述设备的信息节点（尽管可以说内存可以被视为设备）。
/soc 节点的子节点是内存映射设备，
codec @ 1a是 i2c 设备，sound 节点代表的不是设备，而是其他设备如何连接在一起创建音频子系统。

诀窍是内核从树的根开始并寻找具有“兼容”属性的节点。
1.通常假设任何具有“兼容”属性的节点都代表某种设备，
2.1树根处的任何节点要么直接连接到处理器总线，
2.2要么是无法以任何其他方式描述的杂项系统设备。
对于这些节点中的每一个，Linux 分配并注册一个 platform_device，而后者又可能绑定到一个 platform_driver。

对于 Linux 对设备建模的方式，几乎所有 bus_types 都假定它的设备是总线控制器的子设备。
每个 i2c_client 都是 i2c_master 的子设备。
每个 spi_device 都是 SPI 总线的子设备。
对于 USB、PCI、MDIO 等也是如此。

在 DT 中也可以找到相同的层次结构，其中 I2C 设备节点仅作为 I2C总线节点的子节点出现。同样适用于 SPI、MDIO、USB 等。唯一不需要特定类型父设备的设备是 platform_devices（和 amba_devices，但稍后会详细介绍），它们将愉快地存在于 Linux /sys/devices 的基础上树。
因此，如果 DT 节点位于树的根部，那么它确实可能最好注册为 platform_device。

Linux 板支持代码调用 of_platform_populate(NULL, NULL, NULL, NULL) 以开始在树的根部发现设备。参数都是 NULL，因为从树的根开始时，不需要提供起始节点（第一个 NULL）、父节点（最后一个 NULL），而且我们还没有使用匹配表. 对于只需要注册设备的板子，.init_machine() 可以完全为空，除了调用。struct deviceof_platform_populate()

在 Tegra 示例中，这说明了 /soc 和 /sound 节点，但是 SoC 节点的子节点呢？他们不应该也注册为平台设备吗？对于 Linux DT 支持，一般行为是由父设备驱动程序在驱动程序 .probe() 时间注册子设备。因此，i2c 总线设备驱动程序将为每个子节点注册一个 i2c_client，SPI 总线驱动程序将注册其 spi_device 子节点，其他总线类型也是如此。根据该模型，可以编写一个绑定到 SoC 节点并简单地为其每个子节点注册 platform_devices 的驱动程序。板卡支持代码将分配和注册 SoC 设备，（理论上）SoC 设备驱动程序可以绑定到 SoC 设备，并为 /soc/interrupt-controller、/soc/serial、/soc/i2s 和 /soc 注册 platform_devices /i2c 在其 . 探针（）钩子。容易，对吧？

实际上，将某些平台设备的子设备注册为更多平台设备是一种常见模式，设备树支持代码反映了这一点，并使上述示例更简单。to 的第二个参数of_platform_populate()是一个 of_device_id 表，任何与该表中的条目匹配的节点也将注册其子节点。在 Tegra 案例中，代码可能如下所示：

static void __init harmony_init_machine(void)
{
      /* ... */
      of_platform_populate(NULL, of_default_bus_match_table, NULL, NULL);
}

“simple-bus”在 Devicetree 规范中被定义为一个属性，意思是一个简单的内存映射总线，因此of_platform_populate()可以编写代码来假设简单总线兼容的节点将始终被遍历。但是，我们将它作为参数传递，以便板支持代码始终可以覆盖默认行为。
