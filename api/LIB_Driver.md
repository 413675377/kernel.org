# 设备驱动程序基础设施

## 基本设备驱动程序模型结构

- 结构 `subsys_interface`

  设备功能接口

**定义**

```
struct subsys_interface {
  const char *name;
  struct bus_type *subsys;
  struct list_head node;
  int (*add_dev)(struct device *dev, struct subsys_interface *sif);
  void (*remove_dev)(struct device *dev, struct subsys_interface *sif);
};
```

**成员**

- `name`

  设备功能名称

- `subsys`

  要附加到的设备的子系统

- `node`

  在子系统中注册的函数列表

- `add_dev`

  设备连接到设备功能处理程序

- `remove_dev`

  设备连接到设备功能处理程序

**描述**

附加到子系统的简单接口。多个接口可以连接到子系统及其设备。与司机不同的是，他们并不独占或控制设备。接口通常代表子系统/设备类的特定功能。

- `devm_alloc_percpu`（*开发*， *类型*）

  资源管理的 alloc_percpu

**参数**

- `dev`

  为每个 CPU 分配内存的设备

- `type`

  为每个 CPU 分配内存的类型

**描述**

托管 alloc_percpu。使用此函数分配的每 CPU 内存在驱动程序分离时自动释放。

**返回**

成功时指向分配内存的指针，失败时为 NULL。

- 枚举 `dl_dev_state`

  设备驱动程序存在跟踪信息。

**常数**

- `DL_DEV_NO_DRIVER`

  设备没有附加驱动程序。

- `DL_DEV_PROBING`

  一名司机正在试探。

- `DL_DEV_DRIVER_BOUND`

  驱动程序已绑定到设备。

- `DL_DEV_UNBINDING`

  驱动程序与设备解除绑定。

- 枚举 `device_removable`

  设备是否可移动。将设备归类为可移动设备的标准由其子系统或总线确定。

**常数**

- `DEVICE_REMOVABLE_NOT_SUPPORTED`

  此设备不支持此属性（默认）。

- `DEVICE_REMOVABLE_UNKNOWN`

  设备位置未知。

- `DEVICE_FIXED`

  设备不能由用户移动。

- `DEVICE_REMOVABLE`

  设备可由用户移除。

- 结构 `dev_links_info`

  与设备链接相关的设备数据。

**定义**

```
struct dev_links_info {
  struct list_head suppliers;
  struct list_head consumers;
  struct list_head defer_sync;
  enum dl_dev_state status;
};
```

**成员**

- `suppliers`

  供应商设备的链接列表。

- `consumers`

  消费者设备的链接列表。

- `defer_sync`

  挂钩到具有延迟 sync_state 的设备的全局列表。

- `status`

  驱动程序状态信息。

- 结构 `dev_msi_info`

  与 MSI 相关的设备数据

**定义**

```
struct dev_msi_info {
#ifdef CONFIG_GENERIC_MSI_IRQ_DOMAIN;
  struct irq_domain       *domain;
#endif;
#ifdef CONFIG_GENERIC_MSI_IRQ;
  struct msi_device_data  *data;
#endif;
};
```

**成员**

- `domain`

  与设备关联的 MSI 中断域

- `data`

  指向 MSI 设备数据的指针

- 结构 `device`

  基本设备结构

**定义**

```
struct device {
  struct kobject kobj;
  struct device           *parent;
  struct device_private   *p;
  const char              *init_name;
  const struct device_type *type;
  struct bus_type *bus;
  struct device_driver *driver;
  void *platform_data;
  void *driver_data;
#ifdef CONFIG_PROVE_LOCKING;
  struct mutex            lockdep_mutex;
#endif;
  struct mutex            mutex;
  struct dev_links_info   links;
  struct dev_pm_info      power;
  struct dev_pm_domain    *pm_domain;
#ifdef CONFIG_ENERGY_MODEL;
  struct em_perf_domain   *em_pd;
#endif;
#ifdef CONFIG_PINCTRL;
  struct dev_pin_info     *pins;
#endif;
  struct dev_msi_info     msi;
#ifdef CONFIG_DMA_OPS;
  const struct dma_map_ops *dma_ops;
#endif;
  u64 *dma_mask;
  u64 coherent_dma_mask;
  u64 bus_dma_limit;
  const struct bus_dma_region *dma_range_map;
  struct device_dma_parameters *dma_parms;
  struct list_head        dma_pools;
#ifdef CONFIG_DMA_DECLARE_COHERENT;
  struct dma_coherent_mem *dma_mem;
#endif;
#ifdef CONFIG_DMA_CMA;
  struct cma *cma_area;
#endif;
#ifdef CONFIG_SWIOTLB;
  struct io_tlb_mem *dma_io_tlb_mem;
#endif;
  struct dev_archdata     archdata;
  struct device_node      *of_node;
  struct fwnode_handle    *fwnode;
#ifdef CONFIG_NUMA;
  int numa_node;
#endif;
  dev_t devt;
  u32 id;
  spinlock_t devres_lock;
  struct list_head        devres_head;
  struct class            *class;
  const struct attribute_group **groups;
  void (*release)(struct device *dev);
  struct iommu_group      *iommu_group;
  struct dev_iommu        *iommu;
  enum device_removable   removable;
  bool offline_disabled:1;
  bool offline:1;
  bool of_node_reused:1;
  bool state_synced:1;
  bool can_match:1;
#if defined(CONFIG_ARCH_HAS_SYNC_DMA_FOR_DEVICE) ||     defined(CONFIG_ARCH_HAS_SYNC_DMA_FOR_CPU) ||     defined(CONFIG_ARCH_HAS_SYNC_DMA_FOR_CPU_ALL);
  bool dma_coherent:1;
#endif;
#ifdef CONFIG_DMA_OPS_BYPASS;
  bool dma_ops_bypass : 1;
#endif;
};
```

**成员**

- `kobj`

  一个顶级的抽象类，其他类派生自该类。

- `parent`

  设备的“父”设备，即它所连接的设备。在大多数情况下，父设备是某种总线或主机控制器。如果 parent 为 NULL，则设备是顶级设备，这通常不是您想要的。

- `p`

  保存设备驱动核心部分的私有数据。有关详细信息，请参阅 struct device_private 的注释。

- `init_name`

  设备的初始名称。

- `type`

  设备的类型。这标识了设备类型并携带特定类型的信息。

- `bus`

  总线设备类型已打开。

- `driver`

  哪个司机分配了这个

- `platform_data`

  特定于设备的平台数据。

- `driver_data`

  驱动程序特定信息的私有指针。

- `lockdep_mutex`

  一个可选的调试锁，子系统可以使用它作为对等锁来获得 device_lock 的本地化 lockdep 覆盖。

- `mutex`

  互斥锁以同步对其驱动程序的调用。

- `links`

  此设备的供应商和消费者的链接。

- `power`

  用于设备电源管理。有关详细信息，请参阅[设备电源管理基础知识](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/pm/devices.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp)。

- `pm_domain`

  提供在系统挂起、休眠、系统恢复和运行时 PM 转换期间执行的回调以及子系统级和驱动程序级回调。

- `em_pd`

  设备的能量模型性能域

- `pins`

  用于设备引脚管理。有关详细信息，请参阅[PINCTRL（PIN 控制）子系统](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/pin-control.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp)。

- `msi`

  MSI 相关数据

- `dma_ops`

  此设备的 DMA 映射操作。

- `dma_mask`

  Dma 掩码（如果 dma'ble 设备）。

- `coherent_dma_mask`

  与 dma_mask 类似，但对于 alloc_coherent 映射，并非所有硬件都支持 64 位地址以进行一致的分配，例如描述符。

- `bus_dma_limit`

  施加比设备本身支持的更小的 DMA 限制的上游桥接器或总线的限制。

- `dma_range_map`

  DMA 内存范围相对于 RAM 的映射

- `dma_parms`

  低级别的驱动程序可以设置这些来教 IOMMU 代码有关段限制的信息。

- `dma_pools`

  Dma 池（如果 dma'ble 设备）。

- `dma_mem`

  内部用于连贯的内存覆盖。

- `cma_area`

  用于 dma 分配的连续内存区域

- `dma_io_tlb_mem`

  指向使用的 swiotlb 池的指针。不供司机使用。

- `archdata`

  用于特定于拱门的添加。

- `of_node`

  关联的设备树节点。

- `fwnode`

  平台固件提供的关联设备节点。

- `numa_node`

  此设备接近的 NUMA 节点。

- `devt`

  用于创建 sysfs “dev”。

- `id`

  设备实例

- `devres_lock`

  自旋锁来保护设备的资源。

- `devres_head`

  设备的资源列表。

- `class`

  设备的类别。

- `groups`

  可选属性组。

- `release`

  在所有引用消失后回调以释放设备。这应该由设备的分配器（即发现设备的总线驱动程序）设置。

- `iommu_group`

  设备所属的 IOMMU 组。

- `iommu`

  每个设备的通用 IOMMU 运行时数据

- `removable`

  设备是否可以从系统中移除。这应该由发现设备的子系统/总线驱动程序设置。

- `offline_disabled`

  如果设置，设备将永久在线。

- `offline`

  在成功调用总线类型的 .offline() 后设置。

- `of_node_reused`

  如果设备树节点与祖先设备共享，则设置。

- `state_synced`

  此设备的硬件状态已通过调用驱动程序/总线 sync_state() 回调同步以匹配此设备的软件状态。

- `can_match`

  该设备至少与驱动程序匹配一次，或者它位于总线（如 AMBA）中，在其他设备成功探测之前无法检查匹配的驱动程序。

- `dma_coherent`

  这个特定的设备是 dma 相干的，即使架构支持非相干设备。

- `dma_ops_bypass`

  如果设置为，`true`则 dma_ops 被绕过用于流式 DMA 操作（->map_* / ->unmap_* / ->sync_*），并且可选（如果相干掩码足够大）也用于 dma 分配。此标志由 ->dma_supported 中的 dma ops 实例管理。

**例子**

- 对于定制板上的设备，作为典型的嵌入式

  和基于 SOC 的硬件，Linux 经常使用 platform_data 来指向特定于板的结构来描述设备及其连接方式。这可以包括哪些端口可用、芯片变体、哪些 GPIO 引脚充当哪些附加角色等等。这缩小了“板级支持包”（BSP）并最大限度地减少了驱动程序中特定于板的#ifdefs。

**描述**

在最低级别，Linux 系统中的每个设备都由. 设备结构包含设备模型核心对系统建模所需的信息。但是，大多数子系统会跟踪有关它们所托管设备的附加信息。因此，设备很少用裸设备结构来表示；相反，该结构，如 kobject 结构，通常嵌入在设备的更高级别表示中。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- 结构 `device_link`

  设备链接表示。

**定义**

```
struct device_link {
  struct device *supplier;
  struct list_head s_node;
  struct device *consumer;
  struct list_head c_node;
  struct device link_dev;
  enum device_link_state status;
  u32 flags;
  refcount_t rpm_active;
  struct kref kref;
  struct work_struct rm_work;
  bool supplier_preactivated;
};
```

**成员**

- `supplier`

  链路供应商端的设备。

- `s_node`

  连接到供应商设备的消费者链接列表。

- `consumer`

  链接消费者端的设备。

- `c_node`

  连接到消费设备的供应商链接列表。

- `link_dev`

  用于在 sysfs 中公开链接详细信息的设备

- `status`

  链接的状态（相对于驱动程序的存在）。

- `flags`

  链接标志。

- `rpm_active`

  消费设备是否运行时-PM-active。

- `kref`

  计算重复添加相同的链接。

- `rm_work`

  用于删除链接的工作结构。

- `supplier_preactivated`

  在消费者调查之前，供应商已被激活。

- bool `device_iommu_mapped`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  当设备 DMA 由 IOMMU 转换时返回 true

**参数**

- `struct device *dev`

  执行检查的设备

- const char * `dev_bus_name`( const struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  如果可能，返回设备的总线/类名

**参数**

- `const struct device *dev`

  [`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)获取总线/类名

**描述**

将返回设备所连接的总线/类的名称。如果它没有附加到总线/类，将返回一个空字符串。

## 设备驱动程序库

- 无效 `driver_init`（无效 ）

  初始化驱动模型。

**参数**

- `void`

  没有争论

**描述**

调用驱动模型初始化函数来初始化它们的子系统。从 init/main.c 提前调用。

- int `driver_for_each_device`( struct device_driver **drv* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start* , void **data* , int ( **fn* )(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *, void *) )

  绑定到驱动程序的设备的迭代器。

**参数**

- `struct device_driver *drv`

  我们正在迭代的驱动程序。

- `struct device *start`

  开始使用的设备

- `void *data`

  要传递给回调的数据。

- `int (*fn)(struct device *, void *)`

  调用每个设备的函数。

**描述**

遍历**drv**的设备列表，为每个设备调用**fn 。**

- struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) * `driver_find_device`( struct device_driver **drv* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start* , const void **data* , int ( **match* )(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *dev, const void *data) )

  用于定位特定设备的设备迭代器。

**参数**

- `struct device_driver *drv`

  设备的驱动程序

- `struct device *start`

  开始使用的设备

- `const void *data`

  传递给匹配函数的数据

- `int (*match)(struct device *dev, const void *data)`

  检查设备的回调函数

**描述**

这类似于[`driver_for_each_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.driver_for_each_device)上面的函数，但它返回对“找到”的设备的引用以供以后使用，由**匹配**回调确定。

如果设备不匹配，回调应该返回 0，如果匹配则返回非零。如果回调返回非零，此函数将返回给调用者并且不再迭代任何设备。

- int `driver_create_file`( struct device_driver **drv* , const struct driver_attribute **attr* )

  为驱动程序创建 sysfs 文件。

**参数**

- `struct device_driver *drv`

  司机。

- `const struct driver_attribute *attr`

  驱动程序属性描述符。

- void `driver_remove_file`( struct device_driver **drv* , const struct driver_attribute **attr* )

  删除驱动程序的 sysfs 文件。

**参数**

- `struct device_driver *drv`

  司机。

- `const struct driver_attribute *attr`

  驱动程序属性描述符。

- int `driver_register`( struct device_driver **drv* )

  用总线注册司机

**参数**

- `struct device_driver *drv`

  司机登记

**描述**

我们将大部分工作交给了 bus_add_driver() 调用，因为我们必须做的大部分事情都与总线结构有关。

- 无效 `driver_unregister`( struct device_driver **drv* )

  从系统中删除驱动程序。

**参数**

- `struct device_driver *drv`

  司机。

**描述**

同样，我们将大部分工作交给总线级调用。

- struct device_driver * `driver_find`( const char **name* , struct bus_type **bus* )

  通过它的名字在公共汽车上找到司机。

**参数**

- `const char *name`

  司机姓名。

- `struct bus_type *bus`

  公交车扫描司机。

**描述**

调用[`kset_find_obj()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/basics.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.kset_find_obj)以遍历公共汽车上的驱动程序列表以按名称查找驱动程序。如果找到返回驱动程序。

此例程不提供锁定以防止它返回的驱动程序在调用者使用它时被取消注册或卸载。调用者有责任防止这种情况发生。

- struct [device_link](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_link) * `device_link_add`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **consumer* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **supplier* , u32 *flags* )

  在两个设备之间创建链接。

**参数**

- `struct device *consumer`

  链接的消费者端。

- `struct device *supplier`

  链接的供应商端。

- `u32 flags`

  链接标志。

**描述**

The caller is responsible for the proper synchronization of the link creation with runtime PM. First, setting the DL_FLAG_PM_RUNTIME flag will cause the runtime PM framework to take the link into account. Second, if the DL_FLAG_RPM_ACTIVE flag is set in addition to it, the supplier devices will be forced into the active meta state and reference-counted upon the creation of the link. If DL_FLAG_PM_RUNTIME is not set, DL_FLAG_RPM_ACTIVE will be ignored.

If DL_FLAG_STATELESS is set in **flags**, the caller of this function is expected to release the link returned by it directly with the help of either [`device_link_del()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_link_del) or [`device_link_remove()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_link_remove).

If that flag is not set, however, the caller of this function is handing the management of the link over to the driver core entirely and its return value can only be used to check whether or not the link is present. In that case, the DL_FLAG_AUTOREMOVE_CONSUMER and DL_FLAG_AUTOREMOVE_SUPPLIER device link flags can be used to indicate to the driver core when the link can be safely deleted. Namely, setting one of them in **flags** indicates to the driver core that the link is not going to be used (by the given caller of this function) after unbinding the consumer or supplier driver, respectively, from its device, so the link can be deleted at that point. If none of them is set, the link will be maintained until one of the devices pointed to by it (either the consumer or the supplier) is unregistered.

Also, if DL_FLAG_STATELESS, DL_FLAG_AUTOREMOVE_CONSUMER and DL_FLAG_AUTOREMOVE_SUPPLIER are not set in **flags** (that is, a persistent managed device link is being added), the DL_FLAG_AUTOPROBE_CONSUMER flag can be used to request the driver core to automatically probe for a consumer driver after successfully binding a driver to the supplier device.

**flags**中同时设置的DL_FLAG_STATELESS和DL_FLAG_AUTOREMOVE_CONSUMER、DL_FLAG_AUTOREMOVE_SUPPLIER或DL_FLAG_AUTOPROBE_CONSUMER之一的组合是无效的，会导致提前返回NULL。但是，如果给定**消费者**和**供应商**对之间的设备链接在调用此函数时已经存在，则无论其当前类型和状态如何，都将返回现有链接（链接的标志可能会被修改）。然后，此函数的调用者应将链接视为刚刚创建，因此（特别是）如果 DL_FLAG_STATELESS 在**flags**中传递，则需要在不再需要时显式释放链接（如上所述）。

链接创建的副作用是重新排序 dpm_list 和 devices_kset 列表，方法是将消费者设备和依赖于它的所有设备移动到这些列表的末尾（这不会发生在此功能时尚未注册的设备上）称为）。

调用此函数时需要注册供应商设备，否则将返回 NULL。然而，消费者设备不需要注册。

- 无效 `device_link_del`（结构 [设备链接](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_link) **链接*）

  删除两个设备之间的无状态链接。

**参数**

- `struct device_link *link`

  要删除的设备链接。

**描述**

调用者必须确保此函数与运行时 PM 正确同步。如果链接被多次添加，则需要经常删除。热插拔设备需要小心：它们的链接在移除时被清除，[`device_link_del()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_link_del)然后不再允许调用。

- 无效 `device_link_remove`（无效 **消费者*，结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **供应商*）

  删除两个设备之间的无状态链接。

**参数**

- `void *consumer`

  链接的消费者端。

- `struct device *supplier`

  链接的供应商端。

**描述**

调用者必须确保此函数与运行时 PM 正确同步。

- const char * `dev_driver_string`( const struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  如果可能，返回设备的驱动程序名称

**参数**

- `const struct device *dev`

  [`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)得到名字

**描述**

如果绑定到设备，将返回设备的驱动程序名称。如果设备未绑定到驱动程序，它将返回它所连接的总线的名称。如果它也没有连接到总线，将返回一个空字符串。

- int `devm_device_add_group`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const struct attribute_group **grp* )

  给定一个设备，创建一个托管属性组

**参数**

- `struct device *dev`

  为其创建组的设备

- `const struct attribute_group *grp`

  要创建的属性组

**描述**

此功能首次创建组。如果正在创建的任何属性文件已经存在，它将显式地发出警告和错误。

成功时返回 0，失败时返回错误代码。

- 无效 `devm_device_remove_group`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，常量结构属性 *组\*grp* ）

  从设备中删除托管组

**参数**

- `struct device *dev`

  删除组的设备

- `const struct attribute_group *grp`

  要删除的组

**描述**

此函数从设备中删除一组属性。之前必须为该组创建属性，否则将失败。

- int `devm_device_add_groups`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const struct attribute_group ***groups* )

  创建一堆托管属性组

**参数**

- `struct device *dev`

  为其创建组的设备

- `const struct attribute_group **groups`

  要创建的属性组，NULL 终止

**描述**

此函数创建一堆托管属性组。如果在创建组时发生错误，所有以前创建的组都将被删除，将所有内容恢复到调用此函数时的原始状态。如果正在创建的任何属性文件已经存在，它将显式地发出警告和错误。

成功时返回 0，失败时从 sysfs_create_group 返回错误代码。

- 无效 `devm_device_remove_groups`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，常量结构属性 *组**组*）

  删除托管组列表

**参数**

- `struct device *dev`

  要从中删除组的设备

- `const struct attribute_group **groups`

  要删除的以 NULL 结尾的组列表

**描述**

如果 groups 不为 NULL，则从设备中删除指定的组。

- int `device_create_file`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , 常量 struct device_attribute **attr* )

  为设备创建 sysfs 属性文件。

**参数**

- `struct device *dev`

  设备。

- `const struct device_attribute *attr`

  设备属性描述符。

- 无效 `device_remove_file`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，常量结构 设备 *属性\*attr* ）

  删除 sysfs 属性文件。

**参数**

- `struct device *dev`

  设备。

- `const struct device_attribute *attr`

  设备属性描述符。

- bool `device_remove_file_self`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const struct device_attribute **attr* )

  从它自己的方法中删除 sysfs 属性文件。

**参数**

- `struct device *dev`

  设备。

- `const struct device_attribute *attr`

  设备属性描述符。

**描述**

有关详细信息，请参阅 kernfs_remove_self()。

- int `device_create_bin_file`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const struct bin_attribute **attr* )

  为设备创建 sysfs 二进制属性文件。

**参数**

- `struct device *dev`

  设备。

- `const struct bin_attribute *attr`

  设备二进制属性描述符。

- 无效 `device_remove_bin_file`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，常量结构bin_attribute **attr* ）

  删除 sysfs 二进制属性文件

**参数**

- `struct device *dev`

  设备。

- `const struct bin_attribute *attr`

  设备二进制属性描述符。

- 无效 `device_initialize`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  初始化设备结构。

**参数**

- `struct device *dev`

  设备。

**描述**

这通过初始化其字段来准备设备以供其他层使用。它是 的前半部分[`device_register()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_register)，如果由该函数调用，尽管它也可以单独调用，因此可以使用**dev**的字段。特别是[`get_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.get_device)/[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)可用于调用该函数后的**dev的引用计数。**

**dev**中的所有字段都必须由调用者初始化为 0，除了那些显式设置为其他值的字段。最简单的方法是使用[`kzalloc()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/mm-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.kzalloc)来分配包含**dev**的结构。

**笔记**

调用此函数后，用于放弃您的引用，而不是直接释放[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)dev **。**

- int `dev_set_name`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const char **fmt* , ... )

  设置设备名称

**参数**

- `struct device *dev`

  设备

- `const char *fmt`

  设备名称的格式字符串

- `...`

  可变参数

- int `device_add`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  将设备添加到设备层次结构。

**参数**

- `struct device *dev`

  设备。

**描述**

这是 的第 2 部分[`device_register()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_register)，虽然可以单独调用 _iff_[`device_initialize()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_initialize)已单独调用。

这会将**dev**添加到 kobject 层次结构中[`kobject_add()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/basics.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.kobject_add)，将其添加到设备的全局和兄弟列表中，然后将其添加到驱动程序模型的其他相关子系统中。

请勿[`device_register()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_register)对任何设备结构调用此例程或多次调用此例程。驱动程序模型核心并非旨在与未注册然后又恢复活力的设备一起使用。（除其他事项外，很难保证所有对**dev**前身的引用都已删除。）分配并注册一个新的代替。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

经验法则是：如果[`device_add()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_add)成功，您应该[`device_del()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_del)在想要摆脱它时调用它。如果[`device_add()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_add)没有成功，*仅**用于* [`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)删除引用计数。

**笔记**

_Never_ 调用此函数后直接释放**dev**，即使它返回错误！总是用[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)放弃你的参考来代替。

- int `device_register`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  向系统注册设备。

**参数**

- `struct device *dev`

  指向设备结构的指针

**描述**

这发生在两个干净的步骤中 - 初始化设备并将其添加到系统中。这两个步骤可以分别调用，但这是最简单和最常见的。即，如果在将设备添加到层次结构之前明确定义了使用和引用设备的需要，则只应分别调用这两个助手。

有关详细信息，请参阅 和 的[`device_initialize()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_initialize)kerneldoc [`device_add()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_add)。

**笔记**

_Never_ 调用此函数后直接释放**dev**，即使它返回错误！始终使用[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)放弃在此函数中初始化的引用。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `get_device`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  增加设备的引用计数。

**参数**

- `struct device *dev`

  设备。

**描述**

这只是将调用转发到[`kobject_get()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/basics.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.kobject_get)，尽管我们确实注意提供传入 NULL 指针的情况。

- 无效 `put_device`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  减少引用计数。

**参数**

- `struct device *dev`

  有问题的设备。

- 无效 `device_del`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  从系统中删除设备。

**参数**

- `struct device *dev`

  设备。

**描述**

这是设备注销序列的第一部分。这会将设备从我们从这里控制的列表中删除，将其从添加到 in 的其他驱动程序模型子系统中[`device_add()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_add)删除，并将其从 kobject 层次结构中删除。

**笔记**

这应该被手动调用_iff_[`device_add()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_add)也被手动调用。

- 无效 `device_unregister`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  从系统中注销设备。

**参数**

- `struct device *dev`

  设备消失。

**描述**

就像我们一样，我们分两部分执行此操作[`device_register()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_register)。首先，我们使用 将其从所有子系统中删除[`device_del()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_del)，然后通过 减少引用计数[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)。如果这是最终的引用计数，设备将通过上面的 device_release() 进行清理。否则，该结构将一直存在，直到对设备的最终引用被删除。

- int `device_for_each_child`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , void **data* , int ( **fn* )(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *dev, void *data) )

  设备子迭代器。

**参数**

- `struct device *parent`

  父母。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `void *data`

  回调的数据。

- `int (*fn)(struct device *dev, void *data)`

  为每个设备调用的函数。

**描述**

遍历**parent**的子设备，并为每个子设备调用**fn**，并传递给它**data**。

我们每次检查**fn的返回值。**如果它返回除 0 以外的任何值，我们将中断并返回该值。

- int `device_for_each_child_reverse`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , void **data* , int ( **fn* )(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *dev, void *data) )

  设备子迭代器以相反的顺序。

**参数**

- `struct device *parent`

  父母。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `void *data`

  回调的数据。

- `int (*fn)(struct device *dev, void *data)`

  为每个设备调用的函数。

**描述**

遍历**parent**的子设备，并为每个子设备调用**fn**，并传递给它**data**。

我们每次检查**fn的返回值。**如果它返回除 0 以外的任何值，我们将中断并返回该值。

- struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) * `device_find_child`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , void **data* , int ( **match* )(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *dev, void *data) )

  用于定位特定设备的设备迭代器。

**参数**

- `struct device *parent`

  父母[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `void *data`

  传递给匹配函数的数据

- `int (*match)(struct device *dev, void *data)`

  检查设备的回调函数

**描述**

这类似于[`device_for_each_child()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_for_each_child)上面的函数，但它返回对“找到”的设备的引用以供以后使用，由**匹配**回调确定。

如果设备不匹配，回调应该返回 0，如果匹配则返回非零。如果回调返回非零并且可以获得对当前设备的引用，则此函数将返回给调用者并且不再遍历任何设备。

**笔记**

您需要[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)在使用后删除参考。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `device_find_child_by_name`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , const char **name* )

  用于定位子设备的设备迭代器。

**参数**

- `struct device *parent`

  父母[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `const char *name`

  子设备的名称

**描述**

这与上面的函数类似，但它返回对名称**为 name**[`device_find_child()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_find_child)的设备的引用。

**笔记**

您需要[`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device)在使用后删除参考。

- struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) * `__root_device_register`( const char **name* , struct module **owner* )

  分配和注册根设备

**参数**

- `const char *name`

  根设备名称

- `struct module *owner`

  根设备的所有者模块，通常是 THIS_MODULE

**描述**

此函数分配一个根设备并使用[`device_register()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_register). 要释放退回的设备，请使用[`root_device_unregister()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.root_device_unregister).

根设备是虚拟设备，允许将其他设备分组在 /sys/devices 下。使用此函数分配根设备，然后将其用作应出现在 /sys/devices/{name} 下的任何设备的父设备

/sys/devices/{name} 目录还将包含一个“模块”符号链接，它指向 sysfs 中的**所有者**目录。

成功时返回指针，错误时返回 ERR_PTR()。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

**笔记**

您可能想要使用 root_device_register()。

- 无效 `root_device_unregister`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  注销并释放根设备

**参数**

- `struct device *dev`

  设备消失

**描述**

此函数取消注册并清理由 root_device_register() 创建的设备。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `device_create`(结构类 **class* , 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , dev_t *devt* , void **drvdata* , const char **fmt* , ... )

  创建一个设备并将其注册到 sysfs

**参数**

- `struct class *class`

  指向该设备应注册到的结构类的指针

- `struct device *parent`

  指向这个新设备的父设备的指针，如果有的话[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `dev_t devt`

  要添加的字符设备的 dev_t

- `void *drvdata`

  要添加到设备以进行回调的数据

- `const char *fmt`

  设备名称的字符串

- `...`

  可变参数

**描述**

char 设备类可以使用此函数。A将在 sysfs 中创建，注册到指定的类。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

如果 dev_t 不是 0,0，将创建一个“dev”文件，显示设备的 dev_t。如果传入了指向父设备的指针，则新创建的设备将是 sysfs 中该设备的子设备。指向 的指针将从调用中返回。可以使用此指针创建可能需要的任何其他 sysfs 文件。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

Returns [`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) pointer on success, or ERR_PTR() on error.

**Note**

the struct class passed to this function must have previously been created with a call to class_create().

- struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) * `device_create_with_groups`(struct class **class*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent*, dev_t *devt*, void **drvdata*, const struct attribute_group ***groups*, const char **fmt*, ... )

  creates a device and registers it with sysfs

**Parameters**

- `struct class *class`

  pointer to the struct class that this device should be registered to

- `struct device *parent`

  指向这个新设备的父设备的指针，如果有的话[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `dev_t devt`

  要添加的字符设备的 dev_t

- `void *drvdata`

  要添加到设备以进行回调的数据

- `const struct attribute_group **groups`

  要创建的以 NULL 结尾的属性组列表

- `const char *fmt`

  设备名称的字符串

- `...`

  可变参数

**描述**

char 设备类可以使用此函数。A将在 sysfs 中创建，注册到指定的类。组参数中指定的其他属性也将自动创建。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

如果 dev_t 不是 0,0，将创建一个“dev”文件，显示设备的 dev_t。如果传入了指向父设备的指针，则新创建的设备将是 sysfs 中该设备的子设备。指向 的指针将从调用中返回。可以使用此指针创建可能需要的任何其他 sysfs 文件。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

成功时返回指针，错误时返回 ERR_PTR()。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

**笔记**

传递给此函数的结构类必须先前已通过调用 class_create() 创建。

- 无效 `device_destroy`（结构类 **类*，dev_t *devt* ）

  删除创建的设备[`device_create()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_create)

**参数**

- `struct class *class`

  指向该设备注册的结构类的指针

- `dev_t devt`

  先前注册的设备的 dev_t

**描述**

此调用取消注册并清理通过调用创建的设备[`device_create()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_create)。

- int `device_rename`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const char **new_name* )

  重命名设备

**参数**

- `struct device *dev`

  指向要重命名的指针[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `const char *new_name`

  设备的新名称

**描述**

调用者有责任在同一设备上对 device_rename 的两次不同调用提供互斥，以确保 new_name 有效且不会与其他设备冲突。

Renaming devices is racy at many levels, symlinks and other stuff are not replaced atomically, and you get a “move” uevent, but it’s not easy to connect the event to the old and new device. Device nodes are not renamed at all, there isn’t even support for that in the kernel now.

In the meantime, during renaming, your target name might be taken by another driver, creating conflicts. Or the old name is taken directly after you renamed it – then you get events for the same DEVPATH, before you even see the “move” event. It’s just a mess, and nothing new should ever rely on kernel device renaming. Besides that, it’s not even implemented now for other things than (driver-core wise very simple) network devices.

我们目前即将更改 udev 中的网络重命名，以完全禁止重命名与内核使用的命名空间相同的设备，因为我们无法正确解决在没有竞争的情况下交换多个接口的名称所出现的问题。意味着，由于上述原因，仅允许将 eth[0-9]* 重命名为 eth[0-9]* 以外的其他名称。

在您注册任何东西之前在驱动程序中组成一个“真实”名称，或者为用户空间添加一些其他属性以查找设备，或者使用 udev 添加符号链接 - 但以后不要重命名内核设备，这完全是一团糟。我们甚至不想进入那个并尝试在核心中实现缺失的部分。在驱动程序核心混乱中，我们确实还有其他部分需要修复。:)

**笔记**

不要调用这个函数。目前，网络层调用了这个函数，但这会改变。Kay Sievers 的以下文字提供了一些见解：

- int `device_move`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **new_parent* , enum dpm_order *dpm_order* )

  将设备移至新父级

**参数**

- `struct device *dev`

  指向要移动的指针[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `struct device *new_parent`

  设备的新父级（可以为 NULL）

- `enum dpm_order dpm_order`

  如何重新排序 dpm_list

- int `device_change_owner`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , kuid_t *kuid* , kgid_t *kgid* )

  更改现有设备的所有者。

**参数**

- `struct device *dev`

  设备。

- `kuid_t kuid`

  新主人的 kuid

- `kgid_t kgid`

  新主人的kgid

**描述**

这会将**dev**的所有者及其相应的 sysfs 条目更改为**kuid** / **kgid**。此功能密切反映了**dev**是如何通过驱动程序核心添加的。

成功时返回 0，失败时返回错误代码。

- int `dev_err_probe`( const struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , int *err* , const char **fmt* , ... )

  探测错误检查和日志助手

**参数**

- `const struct device *dev`

  指向的指针[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `int err`

  要测试的错误值

- `const char *fmt`

  printf 样式的格式字符串

- `...`

  格式字符串中指定的参数

**描述**

该帮助程序实现了用于错误检查的探针函数中的常见模式：打印调试或错误消息，具体取决于错误值是否为 -EPROBE_DEFER 并向上传播错误。在 -EPROBE_DEFER 的情况下，它还设置延迟探测原因，稍后可以通过读取 devices_deferred debugfs 属性来检查。它替换了代码序列：

```
if (err != -EPROBE_DEFER)
        dev_err(dev, ...);
else
        dev_dbg(dev, ...);
return err;
```

和：

```
return dev_err_probe(dev, err, ...);
```

请注意，即使已知**错误永远不会是 -EPROBE_DEFER，在探测期间使用此函数打印错误也是可以接受的。**与普通 dev_err() 相比的好处是错误代码的标准化格式以及返回错误代码的事实。

返回**错误**。

- 无效 `set_primary_fwnode`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，结构fwnode_handle **fwnode* ）

  更改给定设备的主固件节点。

**参数**

- `struct device *dev`

  要处理的设备。

- `struct fwnode_handle *fwnode`

  设备的新主固件节点。

**描述**

将设备的固件节点指针设置为**fwnode**，但如果存在设备的辅助固件节点，则保留它。

- 有效的 fwnode 案例是：

  主要 -> 次要 -> -ENODEV主要的-> NULL次要 -> -ENODEV空值

- 无效 `set_secondary_fwnode`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，结构fwnode_handle **fwnode* ）

  更改给定设备的辅助固件节点。

**参数**

- `struct device *dev`

  要处理的设备。

- `struct fwnode_handle *fwnode`

  设备的新辅助固件节点。

**描述**

If a primary firmware node of the device is present, set its secondary pointer to **fwnode**. Otherwise, set the device’s firmware node pointer to **fwnode**.

- void `device_set_of_node_from_dev`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, const struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev2*)

  reuse device-tree node of another device

**Parameters**

- `struct device *dev`

  device whose device-tree node is being set

- `const struct device *dev2`

  device whose device-tree node is being reused

**Description**

Takes another reference to the new device-tree node after first dropping any reference held to the old node.

- void `register_syscore_ops`(struct syscore_ops **ops*)

  Register a set of system core operations.

**Parameters**

- `struct syscore_ops *ops`

  要注册的系统核心操作。

- 无效 `unregister_syscore_ops`（结构 syscore_ops **ops* ）

  注销一组系统核心操作。

**参数**

- `struct syscore_ops *ops`

  要注销的系统核心操作。

- 整数 `syscore_suspend`（无效 ）

  执行所有注册的系统核心挂起回调。

**参数**

- `void`

  没有争论

**描述**

该功能在一个 CPU 在线并禁用中断的情况下执行。

- 无效 `syscore_resume`（无效 ）

  执行所有注册的系统核心恢复回调。

**参数**

- `void`

  没有争论

**描述**

该功能在一个 CPU 在线并禁用中断的情况下执行。

- 结构类 * `__class_create`(结构模块 **owner* , const char **name* , struct lock_class_key **key* )

  创建一个struct类结构

**参数**

- `struct module *owner`

  指向要“拥有”这个结构类的模块的指针

- `const char *name`

  指向此类名称的字符串的指针。

- `struct lock_class_key *key`

  这个类的lock_class_key；互斥锁调试使用

**描述**

这用于创建一个结构类指针，然后可以在调用[`device_create()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_create).

成功时返回指针，错误时返回 ERR_PTR()。`struct class`

请注意，此处创建的指针将在完成时通过调用[`class_destroy()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.class_destroy).

- 无效 `class_destroy`（结构类 **cls* ）

  破坏结构类结构

**参数**

- `struct class *cls`

  指向要销毁的结构类的指针

**描述**

注意，要销毁的指针必须是通过调用 class_create() 创建的。

- void `class_dev_iter_init`( struct class_dev_iter **iter* , struct class **class* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start* , const struct device_type **type* )

  初始化类设备迭代器

**参数**

- `struct class_dev_iter *iter`

  类迭代器初始化

- `struct class *class`

  我们想要迭代的类

- `struct device *start`

  开始迭代的设备（如果有）

- `const struct device_type *type`

  要迭代的设备的 device_type，全部为 NULL

**描述**

初始化类迭代器**iter**以便它遍历**类**的设备。如果设置了**start**，则列表迭代将从那里开始，否则如果为 NULL，则迭代从列表的开头开始。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `class_dev_iter_next`(结构 class_dev_iter **iter* )

  迭代到下一个设备

**参数**

- `struct class_dev_iter *iter`

  类迭代器继续

**描述**

继续**迭代**到下一个设备并将其返回。如果迭代完成，则返回 NULL。

返回的设备被引用并且在迭代器继续到下一个设备或退出之前不会被释放。调用者可以自由地对设备做任何它想做的事情，包括回调类代码。

- 无效 `class_dev_iter_exit`（结构类dev_iter **iter* ）

  完成迭代

**参数**

- `struct class_dev_iter *iter`

  类迭代器完成

**描述**

完成一次迭代。无论迭代是否运行到结束，迭代完成后始终调用此函数。

- int `class_for_each_device`( struct class **class* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start* , void **data* , int ( **fn* )(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *, void *) )

  设备迭代器

**参数**

- `struct class *class`

  我们正在迭代的类

- `struct device *start`

  列表中开始使用的设备（如果有）。

- `void *data`

  回调的数据

- `int (*fn)(struct device *, void *)`

  为每个设备调用的函数

**描述**

遍历**class**的设备列表，并为每个设备调用**fn**，并传递给它**data**。如果设置了**start**，则列表迭代将从那里开始，否则如果为 NULL，则迭代从列表的开头开始。

我们每次检查**fn的返回值。**如果它返回除 0 以外的任何值，我们将中断并返回该值。

**fn**被允许做任何事情，包括回调类代码。没有锁定限制。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `class_find_device`（结构类 **类*，结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **开始*，常量无效 **数据*，int（ **匹配*）（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)*，常量无效*） ）

  用于定位特定设备的设备迭代器

**参数**

- `struct class *class`

  我们正在迭代的类

- `struct device *start`

  开始使用的设备

- `const void *data`

  匹配函数的数据

- `int (*match)(struct device *, const void *)`

  检查设备的功能

**Description**

This is similar to the class_for_each_dev() function above, but it returns a reference to a device that is ‘found’ for later use, as determined by the **match** callback.

The callback should return 0 if the device doesn’t match and non-zero if it does. If the callback returns non-zero, this function will return to the caller and not iterate over any more devices.

Note, you will need to drop the reference with [`put_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.put_device) after use.

**match** is allowed to do anything including calling back into class code. There’s no locking restriction.

- struct class_compat * `class_compat_register`(const char **name*)

  register a compatibility class

**Parameters**

- `const char *name`

  the name of the class

**Description**

兼容性类是在将一系列类设备转换为总线设备时作为临时用户空间兼容性解决方法。

- 无效 `class_compat_unregister`（结构类兼容性 **cls* ）

  注销兼容性类

**参数**

- `struct class_compat *cls`

  要注销的类

- int `class_compat_create_link`( struct class_compat **cls* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **device_link* )

  创建到总线设备的兼容性类设备链接

**参数**

- `struct class_compat *cls`

  兼容性等级

- `struct device *dev`

  目标总线设备

- `struct device *device_link`

  应创建“设备”链接的可选设备

- void `class_compat_remove_link`( struct class_compat **cls* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **device_link* )

  删除到总线设备的兼容性类设备链接

**参数**

- `struct class_compat *cls`

  兼容性等级

- `struct device *dev`

  目标总线设备

- `struct device *device_link`

  之前创建了“设备”链接的可选设备

- 结构 `node_access_nodes`

  访问类设备以保持用户与其他节点的可见关系。

**定义**

```
struct node_access_nodes {
  struct device           dev;
  struct list_head        list_node;
  unsigned int            access;
#ifdef CONFIG_HMEM_REPORTING;
  struct node_hmem_attrs  hmem_attrs;
#endif;
};
```

**成员**

- `dev`

  此内存访问类的设备

- `list_node`

  节点访问列表中的列表元素

- `access`

  访问等级等级

- `hmem_attrs`

  异构内存性能属性

- void `node_set_perf_attrs`(unsigned int *nid*, struct node_hmem_attrs **hmem_attrs*, unsigned int *access*)

  Set the performance values for given access class

**Parameters**

- `unsigned int nid`

  Node identifier to be set

- `struct node_hmem_attrs *hmem_attrs`

  Heterogeneous memory performance attributes

- `unsigned int access`

  The access class the for the given attributes

- struct `node_cache_info`

  Internal tracking for memory node caches

**Definition**

```
struct node_cache_info {
  struct device dev;
  struct list_head node;
  struct node_cache_attrs cache_attrs;
};
```

**Members**

- `dev`

  Device represeting the cache level

- `node`

  List element for tracking in the node

- `cache_attrs`

  Attributes for this cache level

- void `node_add_cache`(unsigned int *nid*, struct node_cache_attrs **cache_attrs*)

  add cache attribute to a memory node

**Parameters**

- `unsigned int nid`

  具有新缓存属性的节点标识符

- `struct node_cache_attrs *cache_attrs`

  正在添加的缓存的属性

- 无效 `unregister_node`（结构节点 **节点*）

  注销节点设备

**参数**

- `struct node *node`

  节点消失

**描述**

注销节点设备**node**。在调用该函数之前，必须注销节点上的所有设备。

- 整数 `register_memory_node_under_compute_node`（无符号整数 *mem_nid*，无符号整数 *cpu_nid*，无符号整数 *访问*）

  对于给定的访问类，将内存节点链接到其计算节点。

**参数**

- `unsigned int mem_nid`

  内存节点号

- `unsigned int cpu_nid`

  cpu节点数

- `unsigned int access`

  访问类注册

**描述**

> 用于可能具有单独内存和计算节点的平台。此函数将导出节点关系，链接哪些内存启动器节点可以访问给定排名访问类的内存目标。

- int `transport_class_register`( struct transport_class **tclass* )

  注册一个初始传输类

**参数**

- `struct transport_class *tclass`

  指向要初始化的传输类结构的指针

**描述**

传输类包含一个用于识别它的嵌入类。调用者应该用零初始化这个结构，然后必须用实际的传输类唯一名称初始化泛型类。有一个宏 DECLARE_TRANSPORT_CLASS() 可以做到这一点（声明的类仍然必须注册）。

成功返回 0，失败返回错误。

- void `transport_class_unregister`(struct transport_class **tclass*)

  unregister a previously registered class

**Parameters**

- `struct transport_class *tclass`

  The transport class to unregister

**Description**

Must be called prior to deallocating the memory for the transport class.

- int `anon_transport_class_register`(struct anon_transport_class **atc*)

  register an anonymous class

**Parameters**

- `struct anon_transport_class *atc`

  The anon transport class to register

**Description**

匿名传输类同时包含一个传输类和一个容器。匿名类的想法是它实际上从来没有与之关联的任何设备属性（因此节省了容器存储空间）。所以它只能用于触发事件。使用 prezero 然后使用 DECLARE_ANON_TRANSPORT_CLASS() 来初始化匿名传输类存储。

- 无效 `anon_transport_class_unregister`（结构 anon_transport_class **atc* ）

  注销一个匿名类

**参数**

- `struct anon_transport_class *atc`

  指向要注销的匿名传输类的指针

**描述**

必须在为匿名传输类释放内存之前调用。

- 无效 `transport_setup_device`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  为传输类关联声明一个新的开发，但还没有让它可见。

**参数**

- `struct device *dev`

  表示要添加的实体的通用设备

**描述**

通常，dev 代表 HBA 系统中的某些组件（HBA 本身或跨 HBA 总线的远程设备）。该例程只是一个触发点，用于查看是否有任何一组传输类希望与添加的设备相关联。这会为类设备分配存储并对其进行初始化，但尚未将其添加到系统或向其添加属性（您可以使用 transport_add_device 执行此操作）。如果您不需要单独的设置和添加操作，请使用 transport_register_device（请参阅 transport_class.h）。

- int `transport_add_device`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  为传输类关联声明一个新的开发者

**参数**

- `struct device *dev`

  表示要添加的实体的通用设备

**描述**

Usually, dev represents some component in the HBA system (either the HBA itself or a device remote across the HBA bus). This routine is simply a trigger point used to add the device to the system and register attributes for it.

- void `transport_configure_device`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  configure an already set up device

**Parameters**

- `struct device *dev`

  generic device representing device to be configured

**Description**

The idea of configure is simply to provide a point within the setup process to allow the transport class to extract information from a device after it has been setup. This is used in SCSI because we have to have a setup device to begin using the HBA, but after we send the initial inquiry, we use configure to extract the device parameters. The device need not have been added to be configured.

- void `transport_remove_device`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  移除设备的可见性

**参数**

- `struct device *dev`

  要移除的通用设备

**描述**

此调用删除设备的可见性（从 sysfs 对用户），但不会破坏它。要完全消除设备，您还必须调用 transport_destroy_device。如果您不需要将删除和销毁作为单独的操作进行，请使用 transport_unregister_device()（请参阅 transport_class.h），它将为您执行这两个调用。

- 无效 `transport_destroy_device`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  销毁已移除的设备

**参数**

- `struct device *dev`

  从传输类中删除的设备。

**描述**

此调用触发与传输类开发关联的存储的消除。注意：它真正做的只是放弃对 classdev 的引用。在最后一个引用变为零之前，不会释放内存。另请注意，classdev 在 dev 上保留了一个引用计数，因此只要传输类设备仍然存在，dev 也将保留。

- int `driver_deferred_probe_check_state`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  检查延迟探测状态

**参数**

- `struct device *dev`

  检查设备

**返回**

-ENODEV 如果 initcalls 已经完成并且模块被禁用。-ETIMEDOUT 如果设置了延迟探测超时并已过期

> 和模块已启用。

-EPROBE_DEFER 在其他情况下。

**描述**

驱动程序或子系统可以选择调用此函数，而不是直接返回 -EPROBE_DEFER。

- int `device_bind_driver`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  将驱动程序绑定到一个设备。

**参数**

- `struct device *dev`

  设备。

**描述**

允许将驱动程序手动连接到设备。调用者必须已经设置**了 dev->driver**。

请注意，这不会修改总线引用计数。请在调用此之前验证是否已解决。（可以通过驱动程序的 probe() 方法直接调用。）

必须在持有设备锁的情况下调用此函数。

来电者应该更喜欢使用[`device_driver_attach()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_driver_attach)。

- 无效 `wait_for_device_probe`（无效 ）

  

**参数**

- `void`

  没有争论

**描述**

等待设备探测完成。

- int `device_attach`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  尝试将设备附加到驱动程序。

**参数**

- `struct device *dev`

  设备。

**描述**

遍历总线具有的驱动程序列表并为每对调用 driver_probe_device()。如果找到了兼容的一对，请打破并返回。

如果设备绑定到驱动程序，则返回 1；0 如果没有找到匹配的驱动程序；-ENODEV 如果设备未注册。

当调用 USB 接口时，必须保持**dev->parent lock。**

- int `device_driver_attach`( struct device_driver **drv* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  将特定驱动程序附加到特定设备

**参数**

- `struct device_driver *drv`

  要连接的驱动程序

- `struct device *dev`

  将其连接到的设备

**描述**

手动将驱动程序附加到设备。如果需要，将获取**开发**锁和开发- **>父锁。**成功返回 0，失败返回 -ERR。

- int `driver_attach`(struct device_driver **drv*)

  try to bind driver to devices.

**Parameters**

- `struct device_driver *drv`

  driver.

**Description**

Walk the list of devices that the bus has on it and try to match the driver with each one. If driver_probe_device() returns 0 and the **dev->driver** is set, we’ve found a compatible pair.

- void `device_release_driver`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  manually detach device from driver.

**Parameters**

- `struct device *dev`

  device.

**Description**

Manually detach device from driver. When called for a USB interface, **dev->parent** lock must be held.

If this function is to be called with **dev->parent** lock held, ensure that the device’s consumers are unbound in advance or that their locks can be acquired under the **dev->parent** lock.

- struct platform_device * `platform_device_register_resndata`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent*, const char **name*, int *id*, const struct resource **res*, unsigned int *num*, const void **data*, size_t *size*)

  add a platform-level device with resources and platform-specific data

**Parameters**

- `struct device *parent`

  parent device for the device we’re adding

- `const char *name`

  我们要添加的设备的基本名称

- `int id`

  实例编号

- `const struct resource *res`

  需要为设备分配的资源集

- `unsigned int num`

  资源数量

- `const void *data`

  此平台设备的平台特定数据

- `size_t size`

  平台特定数据的大小

**描述**

成功时返回指针，错误时返回 ERR_PTR()。`struct platform_device`

- struct platform_device * `platform_device_register_simple`( const char **name* , int *id* , const struct resource **res* , unsigned int *num* )

  添加平台级设备及其资源

**参数**

- `const char *name`

  我们要添加的设备的基本名称

- `int id`

  实例编号

- `const struct resource *res`

  需要为设备分配的资源集

- `unsigned int num`

  资源数量

**描述**

该函数创建了一个需要最少资源和内存管理的简单平台设备。释放分配给设备的内存的罐装释放功能允许卸载使用此类设备的驱动程序，而无需等待对设备的最后一个引用被删除。

此接口主要用于与直接探测硬件的旧版驱动程序一起使用。由于此类驱动程序自己创建 sysfs 设备节点，而不是让系统基础架构处理此类设备枚举任务，因此它们并不完全符合 Linux 驱动程序模型。特别是，当这些驱动程序被构建为模块时，它们不能被“热插拔”。

成功时返回指针，错误时返回 ERR_PTR()。`struct platform_device`

- struct platform_device * `platform_device_register_data`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , const char **name* , int *id* , const void **data* , size_t *size* )

  添加具有平台特定数据的平台级设备

**参数**

- `struct device *parent`

  我们正在添加的设备的父设备

- `const char *name`

  我们要添加的设备的基本名称

- `int id`

  实例编号

- `const void *data`

  此平台设备的平台特定数据

- `size_t size`

  平台特定数据的大小

**描述**

This function creates a simple platform device that requires minimal resource and memory management. Canned release function freeing memory allocated for the device allows drivers using such devices to be unloaded without waiting for the last reference to the device to be dropped.

Returns `struct platform_device` pointer on success, or ERR_PTR() on error.

- struct resource * `platform_get_resource`(struct platform_device **dev*, unsigned int *type*, unsigned int *num*)

  get a resource for a device

**Parameters**

- `struct platform_device *dev`

  platform device

- `unsigned int type`

  resource type

- `unsigned int num`

  resource index

**Return**

a pointer to the resource or NULL on failure.

- void __iomem * `devm_platform_get_and_ioremap_resource`(struct platform_device **pdev*, unsigned int *index*, struct resource ***res*)

  call devm_ioremap_resource() for a platform device and get resource

**Parameters**

- `struct platform_device *pdev`

  platform device to use both for memory resource lookup as well as resource management

- `unsigned int index`

  resource index

- `struct resource **res`

  optional output parameter to store a pointer to the obtained resource.

**Return**

a pointer to the remapped memory or an ERR_PTR() encoded error code on failure.

- void __iomem * `devm_platform_ioremap_resource`(struct platform_device **pdev*, unsigned int *index*)

  call devm_ioremap_resource() for a platform device

**Parameters**

- `struct platform_device *pdev`

  platform device to use both for memory resource lookup as well as resource management

- `unsigned int index`

  resource index

**Return**

a pointer to the remapped memory or an ERR_PTR() encoded error code on failure.

- void __iomem * `devm_platform_ioremap_resource_byname`( struct platform_device **pdev* , const char **name* )

  为平台设备调用 devm_ioremap_resource，按名称检索资源

**参数**

- `struct platform_device *pdev`

  用于内存资源查找和资源管理的平台设备

- `const char *name`

  资源名称

**返回**

指向重新映射内存的指针或失败时的 ERR_PTR() 编码错误代码。

- int `platform_get_irq_optional`( struct platform_device **dev* , unsigned int *num* )

  获取设备的可选 IRQ

**参数**

- `struct platform_device *dev`

  平台设备

- `unsigned int num`

  IRQ 编号索引

**描述**

获取平台设备的 IRQ。设备驱动程序应检查返回值是否有错误，以免将负整数值传递给[`request_irq()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/genericirq.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.request_irq)API。这与 相同[`platform_get_irq()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_get_irq)，只是如果无法获得 IRQ，它不会打印错误消息。

例如：

```
int irq = platform_get_irq_optional(pdev, 0);
if (irq < 0)
        return irq;
```

**返回**

成功时非零 IRQ 编号，失败时为负错误编号。

- int `platform_get_irq`( struct platform_device **dev* , unsigned int *num* )

  获取设备的 IRQ

**参数**

- `struct platform_device *dev`

  平台设备

- `unsigned int num`

  IRQ 编号索引

**描述**

获取平台设备的 IRQ 并在查找 IRQ 失败时打印错误消息。设备驱动程序应检查返回值是否有错误，以免将负整数值传递给[`request_irq()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/genericirq.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.request_irq)API。

例如：

```
int irq = platform_get_irq(pdev, 0);
if (irq < 0)
        return irq;
```

**返回**

成功时非零 IRQ 编号，失败时为负错误编号。

- int `platform_irq_count`(struct platform_device **dev*)

  Count the number of IRQs a platform device uses

**Parameters**

- `struct platform_device *dev`

  platform device

**Return**

Number of IRQs a platform device uses or EPROBE_DEFER

- int `devm_platform_get_irqs_affinity`(struct platform_device **dev*, struct [irq_affinity](https://www-kernel-org.translate.goog/doc/html/latest/core-api/genericirq.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.irq_affinity) **affd*, unsigned int *minvec*, unsigned int *maxvec*, int ***irqs*)

  devm method to get a set of IRQs for a device using an interrupt affinity descriptor

**Parameters**

- `struct platform_device *dev`

  platform device pointer

- `struct irq_affinity *affd`

  affinity descriptor

- `unsigned int minvec`

  minimum count of interrupt vectors

- `unsigned int maxvec`

  maximum count of interrupt vectors

- `int **irqs`

  pointer holder for IRQ numbers

**Description**

获取平台设备的一组 IRQ，并根据传递的关联描述符更新 IRQ 关联

**返回**

成功时的向量数，失败时的负错误数。

- 结构资源* `platform_get_resource_byname`（结构平台设备 **dev*，无符号整数 *类型*，常量字符 **名称*）

  按名称获取设备的资源

**参数**

- `struct platform_device *dev`

  平台设备

- `unsigned int type`

  资源类型

- `const char *name`

  资源名称

- int `platform_get_irq_byname`( struct platform_device **dev* , const char **name* )

  按名称获取设备的 IRQ

**参数**

- `struct platform_device *dev`

  平台设备

- `const char *name`

  中断名称

**描述**

获取一个类似 的 IRQ [`platform_get_irq()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_get_irq)，但随后按名称而不是按索引。

**返回**

成功时非零 IRQ 编号，失败时为负错误编号。

- int `platform_get_irq_byname_optional`( struct platform_device **dev* , const char **name* )

  按名称获取设备的可选 IRQ

**参数**

- `struct platform_device *dev`

  平台设备

- `const char *name`

  中断名称

**描述**

按名称获取可选的 IRQ，例如[`platform_get_irq_byname()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_get_irq_byname). 除非如果无法获得 IRQ，它不会打印错误消息。

**返回**

成功时非零 IRQ 编号，失败时为负错误编号。

- int `platform_add_devices`( struct platform_device ***devs* , int *num* )

  添加多个平台设备

**参数**

- `struct platform_device **devs`

  要添加的平台设备数组

- `int num`

  阵列中的平台设备数

- void `platform_device_put`(struct platform_device **pdev*)

  destroy a platform device

**Parameters**

- `struct platform_device *pdev`

  platform device to free

**Description**

Free all memory associated with a platform device. This function must _only_ be externally called in error cases. All other usage is a bug.

- struct platform_device * `platform_device_alloc`(const char **name*, int *id*)

  create a platform device

**Parameters**

- `const char *name`

  base name of the device we’re adding

- `int id`

  instance id

**Description**

Create a platform device object which can have other objects attached to it, and which will have attached objects freed when it is released.

- int `platform_device_add_resources`( struct platform_device **pdev* , const struct resource **res* , unsigned int *num* )

  向平台设备添加资源

**参数**

- `struct platform_device *pdev`

  由 platform_device_alloc 分配的用于添加资源的平台设备

- `const struct resource *res`

  需要为设备分配的资源集

- `unsigned int num`

  资源数量

**描述**

将资源的副本添加到平台设备。当平台设备被释放时，与资源相关的内存将被释放。

- int `platform_device_add_data`( struct platform_device **pdev* , const void **data* , size_t *size* )

  将特定于平台的数据添加到平台设备

**参数**

- `struct platform_device *pdev`

  由 platform_device_alloc 分配的用于添加资源的平台设备

- `const void *data`

  此平台设备的平台特定数据

- `size_t size`

  平台特定数据的大小

**描述**

将平台特定数据的副本添加到平台设备的 platform_data 指针。当平台设备被释放时，与平台数据相关的内存将被释放。

- int `platform_device_add`( struct platform_device **pdev* )

  将平台设备添加到设备层次结构

**参数**

- `struct platform_device *pdev`

  我们正在添加的平台设备

**描述**

这是 的第 2 部分[`platform_device_register()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_device_register)，但可以单独调用 _iff_ pdev 是由[`platform_device_alloc()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_device_alloc).

- 无效 `platform_device_del`（结构平台 *设备\*pdev* ）

  移除平台级设备

**参数**

- `struct platform_device *pdev`

  我们正在移除的平台设备

**描述**

请注意，此函数还将释放设备拥有的所有基于内存和端口的资源（**dev->resource**）。此函数必须_only_ 在错误情况下被外部调用。所有其他用法都是错误。

- int `platform_device_register`( struct platform_device **pdev* )

  添加平台级设备

**参数**

- `struct platform_device *pdev`

  我们正在添加的平台设备

**笔记**

_Never_ 调用此函数后直接释放**pdev**，即使它返回错误！始终用于[`platform_device_put()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_device_put)放弃在此函数中初始化的引用。

- 无效 `platform_device_unregister`（结构平台 *设备\*pdev* ）

  注销平台级设备

**参数**

- `struct platform_device *pdev`

  我们正在注销的平台设备

**描述**

注销分两步完成。首先我们释放所有资源并将其从子系统中移除，然后我们通过调用[`platform_device_put()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_device_put).

- struct platform_device * `platform_device_register_full`( const struct platform_device_info **pdevinfo* )

  添加具有资源和平台特定数据的平台级设备

**参数**

- `const struct platform_device_info *pdevinfo`

  用于创建设备的数据

**描述**

成功时返回指针，错误时返回 ERR_PTR()。`struct platform_device`

- int `__platform_driver_register`( struct platform_driver **drv* , struct module **owner* )

  为平台级设备注册驱动程序

**参数**

- `struct platform_driver *drv`

  平台驱动结构

- `struct module *owner`

  拥有模块/驱动程序

- 无效 `platform_driver_unregister`（结构平台 *驱动程序 \*drv* ）

  注销平台级设备的驱动程序

**参数**

- `struct platform_driver *drv`

  平台驱动结构

- int `__platform_driver_probe`( struct platform_driver **drv* , int ( **probe* )(struct platform_device *), struct module **module* )

  为非热插拔设备注册驱动程序

**参数**

- `struct platform_driver *drv`

  平台驱动结构

- `int (*probe)(struct platform_device *)`

  驱动程序探测例程，可能来自 __init 部分

- `struct module *module`

  将成为驱动程序所有者的模块

**描述**

Use this instead of platform_driver_register() when you know the device is not hotpluggable and has already been registered, and you want to remove its run-once probe() infrastructure from memory after the driver has bound to the device.

One typical use for this would be with drivers for controllers integrated into system-on-chip processors, where the controller devices have been configured as part of board setup.

Note that this is incompatible with deferred probing.

Returns zero if the driver registered and bound to a device, else returns a negative error code and with the driver not registered.

- struct platform_device * `__platform_create_bundle`( struct platform_driver **driver* , int ( **probe* )(struct platform_device *), struct resource **res* , unsigned int *n_res* , const void **data* , size_t *size* , struct module **module* )

  注册驱动并创建对应的设备

**参数**

- `struct platform_driver *driver`

  平台驱动结构

- `int (*probe)(struct platform_device *)`

  驱动程序探测例程，可能来自 __init 部分

- `struct resource *res`

  需要为设备分配的资源集

- `unsigned int n_res`

  资源数量

- `const void *data`

  此平台设备的平台特定数据

- `size_t size`

  平台特定数据的大小

- `struct module *module`

  将成为驱动程序所有者的模块

**描述**

在直接探测硬件并注册单个平台设备和相应平台驱动程序的旧式模块中使用它。

成功时返回指针，错误时返回 ERR_PTR()。`struct platform_device`

- int `__platform_register_drivers`( struct platform_driver * const **drivers* , unsigned int *count* , struct module **owner* )

  注册一系列平台驱动程序

**参数**

- `struct platform_driver * const *drivers`

  要注册的驱动程序数组

- `unsigned int count`

  要注册的司机数量

- `struct module *owner`

  拥有驱动程序的模块

**描述**

注册由数组指定的平台驱动程序。如果未能注册驱动程序，所有先前注册的驱动程序都将被取消注册。此 API 的调用者应使用[`platform_unregister_drivers()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.platform_unregister_drivers)相反的顺序取消注册驱动程序。

**返回**

成功时为 0，失败时为负错误代码。

- void `platform_unregister_drivers`( struct platform_driver * const **drivers* , unsigned int *count* )

  注销一系列平台驱动程序

**参数**

- `struct platform_driver * const *drivers`

  要注销的驱动程序数组

- `unsigned int count`

  取消注册的司机数量

**描述**

取消注册由数组指定的平台驱动程序。这通常用于补充对 platform_register_drivers() 的早期调用。司机的注销顺序与注册时相反。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `platform_find_device_by_driver`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **开始*，常量结构设备驱动 程序 **drv* ）

  Find a platform device with a given driver.

**Parameters**

- `struct device *start`

  The device to start the search from.

- `const struct device_driver *drv`

  The device driver to look for.

- int `bus_for_each_dev`(struct bus_type **bus*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start*, void **data*, int ( **fn*)(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *, void *) )

  device iterator.

**Parameters**

- `struct bus_type *bus`

  bus type.

- `struct device *start`

  device to start iterating from.

- `void *data`

  data for the callback.

- `int (*fn)(struct device *, void *)`

  function to be called for each device.

**Description**

Iterate over **bus**’s list of devices, and call **fn** for each, passing it **data**. If **start** is not NULL, we use that device to begin iterating from.

We check the return of **fn** each time. If it returns anything other than 0, we break out and return that value.

**NOTE**

The device that returns a non-zero value is not retained in any way, nor is its refcount incremented. If the caller needs to retain this data, it should do so, and increment the reference count in the supplied callback.

- struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) * `bus_find_device`(struct bus_type **bus*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start*, const void **data*, int ( **match*)(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) *dev, const void *data) )

  device iterator for locating a particular device.

**参数**

- `struct bus_type *bus`

  巴士类型

- `struct device *start`

  开始使用的设备

- `const void *data`

  传递给匹配函数的数据

- `int (*match)(struct device *dev, const void *data)`

  检查设备的回调函数

**描述**

这类似于[`bus_for_each_dev()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.bus_for_each_dev)上面的函数，但它返回对“找到”的设备的引用以供以后使用，由**匹配**回调确定。

如果设备不匹配，回调应该返回 0，如果匹配则返回非零。如果回调返回非零，此函数将返回给调用者并且不再迭代任何设备。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `subsys_find_device_by_id`（结构总线类型 **subsys*，无符号整数 *ID*，结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **提示*）

  查找具有特定枚举编号的设备

**参数**

- `struct bus_type *subsys`

  子系统

- `unsigned int id`

  索引“id”在[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `struct device *hint`

  首先检查的设备

**描述**

检查提示的下一个对象，如果匹配则直接返回，否则返回完整列表搜索。无论哪种方式，都会获取返回对象的引用。

- int `bus_for_each_drv`( struct bus_type **bus* , struct device_driver **start* , void **data* , int ( **fn* )(struct device_driver *, void *) )

  驱动迭代器

**参数**

- `struct bus_type *bus`

  我们正在处理的公共汽车。

- `struct device_driver *start`

  驱动程序开始迭代。

- `void *data`

  传递给回调的数据。

- `int (*fn)(struct device_driver *, void *)`

  调用每个驱动程序的函数。

**描述**

这与上面的设备迭代器几乎相同。我们遍历属于**bus**的每个驱动程序，并为每个驱动程序调用**fn**。如果**fn**返回除 0 以外的任何值，我们就突破并返回它。如果**start**不为 NULL，我们将其用作列表的头部。

**笔记**

我们不会返回返回非零值的驱动程序，也不会为该驱动程序保留递增的引用计数。如果调用者需要知道该信息，它必须在回调中设置它。它还必须确保增加引用计数，以便它在返回给调用者之前不会消失。

- int `bus_rescan_devices`( struct bus_type **bus* )

  重新扫描总线上的设备以查找可能的驱动程序

**参数**

- `struct bus_type *bus`

  巴士扫描。

**描述**

此函数将在总线上查找未连接驱动程序的设备，并根据现有驱动程序重新扫描它，以通过调用[`device_attach()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device_attach)未绑定的设备来查看它是否与任何设备匹配。

- int `device_reprobe`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  删除设备的驱动程序并探测新驱动程序

**参数**

- `struct device *dev`

  重新探测的设备

**描述**

此函数分离给定设备的附加驱动程序（如果有）并重新启动驱动程序探测过程。如果探测标准在设备生命周期内发生更改并且驱动程序附件也应相应更改，则打算使用它。

- int `bus_register`( struct bus_type **bus* )

  注册一个驱动核心子系统

**参数**

- `struct bus_type *bus`

  巴士登记

**描述**

一旦我们有了它，我们就向 kobject 基础设施注册总线，然后注册它拥有的子子系统：属于子系统的设备和驱动程序。

- 无效 `bus_unregister`（结构 总线类型 **总线*）

  从系统中删除总线

**参数**

- `struct bus_type *bus`

  公共汽车。

**描述**

注销子子系统和总线本身。最后，我们调用 bus_put() 来释放 refcount

- void `subsys_dev_iter_init`( struct subsys_dev_iter **iter* , struct bus_type **subsys* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **start* , const struct device_type **type* )

  初始化子系统设备迭代器

**参数**

- `struct subsys_dev_iter *iter`

  子系统迭代器初始化

- `struct bus_type *subsys`

  我们想要迭代的子系统

- `struct device *start`

  开始迭代的设备（如果有）

- `const struct device_type *type`

  要迭代的设备的 device_type，全部为 NULL

**描述**

初始化 subsys 迭代器**iter**以便它遍历**subsys**的设备。如果设置了**start**，则列表迭代将从那里开始，否则如果为 NULL，则迭代从列表的开头开始。

- 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)* `subsys_dev_iter_next`(结构 subsys_dev_iter **iter* )

  迭代到下一个设备

**参数**

- `struct subsys_dev_iter *iter`

  子系统迭代器继续

**描述**

继续**迭代**到下一个设备并将其返回。如果迭代完成，则返回 NULL。

返回的设备被引用并且在迭代器继续到下一个设备或退出之前不会被释放。调用者可以自由地对设备做任何它想做的事情，包括回调子系统代码。

- 无效 `subsys_dev_iter_exit`（结构 subsys_dev_iter **iter* ）

  完成迭代

**参数**

- `struct subsys_dev_iter *iter`

  subsys 迭代器完成

**描述**

完成一次迭代。无论迭代是否运行到结束，迭代完成后始终调用此函数。

- int `subsys_system_register`( struct bus_type **subsys* , const struct attribute_group ***groups* )

  在 /sys/devices/system/ 注册一个子系统

**参数**

- `struct bus_type *subsys`

  系统子系统

- `const struct attribute_group **groups`

  根设备的默认属性

**描述**

所有“系统”子系统都有一个带有子系统名称的 /sys/devices/system/<name> 根设备。根设备可以携带子系统范围的属性。所有注册的设备都在这个单一的根设备之下，并以子系统命名，并附加一个简单的枚举编号。注册的设备没有明确命名；只需设置设备中的“id”。

不要将此界面用于任何新事物，它的存在只是为了与坏主意兼容。新子系统应该使用普通子系统；并添加子系统范围的属性应该添加到子系统目录本身，而不是一些创建假根设备放置在 /sys/devices/system/<name> 中。

- int `subsys_virtual_register`( struct bus_type **subsys* , const struct attribute_group ***groups* )

  在 /sys/devices/virtual/ 注册一个子系统

**参数**

- `struct bus_type *subsys`

  虚拟子系统

- `const struct attribute_group **groups`

  根设备的默认属性

**描述**

所有“虚拟”子系统都有一个带有子系统名称的 /sys/devices/system/<name> 根设备。根设备可以携带子系统范围的属性。所有注册的设备都在这个单一的根设备之下。设备命名没有限制。这适用于需要 sysfs 接口的内核软件结构。

## 设备驱动程序 DMA 管理

- 无效 `dmam_free_coherent`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，size_t *大小*，无效 **vaddr*，dma_addr_t *dma_handle* ）

  托管 dma_free_coherent()

**参数**

- `struct device *dev`

  释放相干内存的设备

- `size_t size`

  分配大小

- `void *vaddr`

  要释放的内存的虚拟地址

- `dma_addr_t dma_handle`

  要释放的内存的 DMA 句柄

**描述**

托管 dma_free_coherent()。

- void * `dmam_alloc_attrs`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , size_t *size* , dma_addr_t **dma_handle* , [gfp_t ](https://www-kernel-org.translate.goog/doc/html/latest/core-api/mm-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.gfp_t) *gfp* , unsigned long *attrs* )

  托管 dma_alloc_attrs()

**参数**

- `struct device *dev`

  分配 non_coherent 内存的设备

- `size_t size`

  分配大小

- `dma_addr_t *dma_handle`

  分配的 DMA 句柄的输出参数

- `gfp_t gfp`

  分配标志

- `unsigned long attrs`

  DMA_ATTR_* 命名空间中的标志。

**描述**

托管 dma_alloc_attrs()。使用此函数分配的内存将在驱动程序分离时自动释放。

**返回**

成功时指向分配内存的指针，失败时为 NULL。

- unsigned int `dma_map_sg_attrs`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct scatterlist **sg* , int *nents* , enum dma_data_direction *dir* , unsigned long *attrs* )

  为 DMA 映射给定的缓冲区

**参数**

- `struct device *dev`

  执行 DMA 操作的设备

- `struct scatterlist *sg`

  描述缓冲区的 sg_table 对象

- `int nents`

  要映射的条目数

- `enum dma_data_direction dir`

  DMA方向

- `unsigned long attrs`

  映射操作的可选 DMA 属性

**描述**

将 sg 参数中传递的 scatterlist 描述的缓冲区与**开发**设备的**dir** DMA 操作的nents 段进行映射。

成功时返回映射条目的数量（可以小于 nents）。任何错误都返回零。

应使用 dma_unmap_sg_attrs() 将缓冲区与原始 sg 和原始 nents（不是此函数返回的值）取消映射。

- int `dma_map_sgtable`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct sg_table *** sgt , enum dma_data_direction *dir* , unsigned long *attrs* )

  为 DMA 映射给定的缓冲区

**参数**

- `struct device *dev`

  执行 DMA 操作的设备

- `struct sg_table *sgt`

  描述缓冲区的 sg_table 对象

- `enum dma_data_direction dir`

  DMA方向

- `unsigned long attrs`

  映射操作的可选 DMA 属性

**描述**

映射由存储在给定 sg_table 对象中的 scatterlist 描述的缓冲区，用于**dev**设备的**dir** DMA 操作。成功后，缓冲区的所有权将转移到 DMA 域。在 CPU 接触缓冲区之前，必须调用 dma_sync_sgtable_for_cpu() 或 dma_unmap_sgtable() 将缓冲区的所有权移回 CPU 域。

成功时返回 0，错误时返回负错误代码。具有给定含义的以下错误代码受支持：

> - -EINVAL
>
>   无效的参数、未对齐的访问或其他使用错误。如果重试将不会成功。
>
> - -ENOMEM
>
>   完成映射的资源（如内存或 IOVA 空间）不足。如果稍后重试应该会成功。
>
> - -EIO
>
>   具有未知含义的旧错误代码。例如。如果较低级别的调用返回 DMA_MAPPING_ERROR，则返回此值。

- bool `dma_can_mmap`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  检查给定设备是否支持 dma_mmap_*

**参数**

- `struct device *dev`

  检查设备

**描述**

`true`如果**dev**支持 dma_mmap_coherent() 并将DMA[`dma_mmap_attrs()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.dma_mmap_attrs)分配映射到用户空间，则返回。

- int `dma_mmap_attrs`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct vm_area_struct **vma* , void **cpu_addr* , dma_addr_t *dma_addr* , size_t *size* , unsigned long *attrs* )

  将一致的 DMA 分配映射到用户空间

**参数**

- `struct device *dev`

  有效指针，对于 ISA 和类似 EISA 的设备为 NULL[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

- `struct vm_area_struct *vma`

  vm_area_struct 描述请求的用户映射

- `void *cpu_addr`

  从 dma_alloc_attrs 返回的内核 CPU 视图地址

- `dma_addr_t dma_addr`

  从 dma_alloc_attrs 返回的设备视图地址

- `size_t size`

  最初在 dma_alloc_attrs 中请求的内存大小

- `unsigned long attrs`

  dma_alloc_attrs 中请求的映射属性的属性

**描述**

将先前由 dma_alloc_attrs 分配的一致 DMA 缓冲区映射到用户空间。在释放用户空间映射之前，驱动程序不得释放一致的 DMA 缓冲区。

## 设备驱动程序 PnP 支持

- int `pnp_register_protocol`( struct pnp_protocol **protocol* )

  向 pnp 层添加 pnp 协议

**参数**

- `struct pnp_protocol *protocol`

  指向相应 pnp_protocol 结构的指针防爆协议：ISAPNP、PNPBIOS 等

- 无效 `pnp_unregister_protocol`（结构 pnp_protocol **protocol* ）

  从 pnp 层删除 pnp 协议

**参数**

- `struct pnp_protocol *protocol`

  指向相应 pnp_protocol 结构的指针

- struct pnp_dev * `pnp_request_card_device`( struct pnp_card_link **clink* , const char **id* , struct pnp_dev **from* )

  搜索指定卡下的 PnP 设备

**参数**

- `struct pnp_card_link *clink`

  指向卡链接的指针，不能为 NULL

- `const char *id`

  指向解释查找设备规则的 PnP ID 结构的指针

- `struct pnp_dev *from`

  搜索的起始位置。如果为 NULL，它将从头开始。

- 无效 `pnp_release_card_device`（结构 pnp_dev **dev* ）

  当驱动程序不再需要设备时调用它

**参数**

- `struct pnp_dev *dev`

  指向 PnP 设备结构的指针

- int `pnp_register_card_driver`( struct pnp_card_driver **drv* )

  向 PnP 层注册 PnP 卡驱动程序

**参数**

- `struct pnp_card_driver *drv`

  指向要注册的驱动程序的指针

- 无效 `pnp_unregister_card_driver`（结构 pnp_card_driver **drv* ）

  从 PnP 层注销 PnP 卡驱动程序

**参数**

- `struct pnp_card_driver *drv`

  指向要注销的驱动程序的指针

- struct pnp_id * `pnp_add_id`( struct pnp_dev **dev* , const char **id* )

  将 EISA id 添加到指定设备

**参数**

- `struct pnp_dev *dev`

  指向所需设备的指针

- `const char *id`

  指向 EISA id 字符串的指针

- 整数 `pnp_start_dev`（结构 pnp_dev **dev* ）

  PnP 设备的低级启动

**参数**

- `struct pnp_dev *dev`

  指向所需设备的指针

**描述**

假设资源已经分配

- 整数 `pnp_stop_dev`（结构 pnp_dev **dev* ）

  PnP 设备的低级禁用

**参数**

- `struct pnp_dev *dev`

  指向所需设备的指针

**描述**

不释放资源

- 整数 `pnp_activate_dev`（结构 pnp_dev **dev* ）

  激活 PnP 设备以供使用

**参数**

- `struct pnp_dev *dev`

  指向所需设备的指针

**描述**

不验证或设置资源，所以要小心。

- 整数 `pnp_disable_dev`（结构 pnp_dev **dev* ）

  禁用设备

**参数**

- `struct pnp_dev *dev`

  指向所需设备的指针

**描述**

通知正确的 pnp 协议，以便其他设备可以使用资源

- 整数 `pnp_is_active`（结构 pnp_dev **dev* ）

  根据当前资源确定设备是否处于活动状态

**参数**

- `struct pnp_dev *dev`

  指向所需 PnP 设备的指针

## 用户空间 IO 设备

- 无效 `uio_event_notify`（结构 [uio_info ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.uio_info) **info* ）

  触发中断事件

**参数**

- `struct uio_info *info`

  UIO 设备功能

- int `__uio_register_device`(结构模块 **owner* , 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , 结构 [uio_info ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.uio_info) **info* )

  注册一个新的用户空间 IO 设备

**参数**

- `struct module *owner`

  创建新设备的模块

- `struct device *parent`

  父设备

- `struct uio_info *info`

  UIO 设备功能

**描述**

成功或负错误代码返回零。

- int `__devm_uio_register_device`(结构模块 **owner* , 结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , 结构 [uio_info ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.uio_info) **info* )

  资源管理[`uio_register_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.uio_register_device)

**参数**

- `struct module *owner`

  创建新设备的模块

- `struct device *parent`

  父设备

- `struct uio_info *info`

  UIO 设备功能

**描述**

成功或负错误代码返回零。

- 无效 `uio_unregister_device`（结构 [uio_info ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.uio_info) **info* ）

  注销工业 IO 设备

**参数**

- `struct uio_info *info`

  UIO 设备功能

- 结构 `uio_mem`

  UIO 内存区域的描述

**定义**

```
struct uio_mem {
  const char              *name;
  phys_addr_t addr;
  unsigned long           offs;
  resource_size_t size;
  int memtype;
  void __iomem            *internal_addr;
  struct uio_map          *map;
};
```

**成员**

- `name`

  用于识别的内存区域的名称

- `addr`

  四舍五入到页面大小的设备内存地址（使用 phys_addr，因为 addr 可以是逻辑的、虚拟的或物理的 & phys_addr_t 应该总是足够大以处理任何地址类型）

- `offs`

  页内设备内存的偏移量

- `size`

  IO 大小（页面大小的倍数）

- `memtype`

  内存地址类型指向

- `internal_addr`

  ioremap-ped 版本的 addr，供驱动程序内部使用

- `map`

  仅供 UIO 内核使用。

- 结构 `uio_port`

  UIO 端口区域的描述

**定义**

```
struct uio_port {
  const char              *name;
  unsigned long           start;
  unsigned long           size;
  int porttype;
  struct uio_portio       *portio;
};
```

**成员**

- `name`

  用于识别的港口区域名称

- `start`

  港区起点

- `size`

  港区规模

- `porttype`

  端口类型（参见下面的 UIO_PORT_*）

- `portio`

  仅供 UIO 内核使用。

- 结构 `uio_info`

  UIO 设备功能

**定义**

```
struct uio_info {
  struct uio_device       *uio_dev;
  const char              *name;
  const char              *version;
  struct uio_mem          mem[MAX_UIO_MAPS];
  struct uio_port         port[MAX_UIO_PORT_REGIONS];
  long irq;
  unsigned long           irq_flags;
  void *priv;
  irqreturn_t (*handler)(int irq, struct uio_info *dev_info);
  int (*mmap)(struct uio_info *info, struct vm_area_struct *vma);
  int (*open)(struct uio_info *info, struct inode *inode);
  int (*release)(struct uio_info *info, struct inode *inode);
  int (*irqcontrol)(struct uio_info *info, s32 irq_on);
};
```

**成员**

- `uio_dev`

  此信息所属的 UIO 设备

- `name`

  设备名称

- `version`

  设备驱动程序版本

- `mem`

  可映射内存区域列表，size==0 表示列表末尾

- `port`

  端口区域列表，size==0 表示列表末尾

- `irq`

  中断号或 UIO_IRQ_CUSTOM

- `irq_flags`

  标志为[`request_irq()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/genericirq.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.request_irq)

- `priv`

  可选的私人数据

- `handler`

  设备的中断处理程序

- `mmap`

  此 uio 设备的 mmap 操作

- `open`

  此 uio 设备的打开操作

- `release`

  此 uio 设备的释放操作

- `irqcontrol`

  当 0/1 写入 /dev/uioX 时禁用/启用 irqs

- `uio_register_device`（*家长*， *信息*）

  注册一个新的用户空间 IO 设备

**参数**

- `parent`

  父设备

- `info`

  UIO 设备功能

**描述**

成功或负错误代码返回零。

- `devm_uio_register_device`（*家长*， *信息*）

  资源管理[`uio_register_device()`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.uio_register_device)

**参数**

- `parent`

  父设备

- `info`

  UIO 设备功能

**描述**

成功或负错误代码返回零。