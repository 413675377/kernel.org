# ALSA 驱动程序 API

## 卡和设备的管理

### 卡管理

- 无效 `snd_device_initialize`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，结构snd_card **卡*）

  为声音设备初始化[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

**参数**

- `struct device *dev`

  初始化设备

- `struct snd_card *card`

  要分配的卡，可选

- int `snd_card_new`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , int *idx* , const char **xid* , struct module **module* , int *extra_size* , struct snd_card ***card_ret* )

  创建并初始化声卡结构

**参数**

- `struct device *parent`

  父设备对象

- `int idx`

  卡索引（地址）[0 … (SNDRV_CARDS-1)]

- `const char *xid`

  卡标识（ASCII 字符串）

- `struct module *module`

  用于锁定的顶级模块

- `int extra_size`

  在主声卡结构之后分配这个额外的大小

- `struct snd_card **card_ret`

  存储创建的卡片实例的指针该函数通过 kzalloc 分配 snd_card 实例和给定的空间供驱动程序自由使用。分配的结构存储在给定的 card_ret 指针中。

**返回**

如果成功则为零或负错误代码。

- int `snd_devm_card_new`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* , int *idx* , const char **xid* , struct module **module* , size_t *extra_size* , struct snd_card ***card_ret* )

  托管 snd_card 对象创建

**参数**

- `struct device *parent`

  父设备对象

- `int idx`

  卡索引（地址）[0 … (SNDRV_CARDS-1)]

- `const char *xid`

  卡标识（ASCII 字符串）

- `struct module *module`

  用于锁定的顶级模块

- `size_t extra_size`

  在主声卡结构之后分配这个额外的大小

- `struct snd_card **card_ret`

  存储创建的卡片实例的指针

**描述**

这个函数的工作方式类似于[`snd_card_new()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_new)但通过 devres 管理分配的资源，即您不需要显式释放。

当使用此函数创建 snd_card 对象并通过 注册时[`snd_card_register()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_register)，会自动添加第一个要调用的 devres 操作[`snd_card_free()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_free)。这样，首先保证资源断开，然后按预期顺序释放。

如果在调用之前的探针发生错误[`snd_card_register()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_register)并且有其他开发资源，您需要通过[`snd_card_free()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_free)错误调用手动释放卡；否则可能会因 devres call 命令而导致 UAF。您可以使用[`snd_card_free_on_error()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_free_on_error)helper 来更轻松地处理它。

- int `snd_card_free_on_error`(结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , int *ret* )

  处理 devm 探针错误的小帮手

**参数**

- `struct device *dev`

  受管设备对象

- `int ret`

  探测回调的返回码

**描述**

此函数处理[`snd_card_free()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_free)来自探测回调的错误的显式调用。它只是简化受管设备错误处理的小帮手。

- 结构 snd_card * `snd_card_ref`( int *idx* )

  从索引中获取卡片对象

**参数**

- `int idx`

  卡片索引

**描述**

返回与给定索引对应的卡片对象，如果未找到则返回 NULL。通过 释放对象[`snd_card_unref()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_unref)。

- int `snd_card_disconnect`( struct snd_card **card* )

  断开所有 API 与文件操作（用户空间）的连接

**参数**

- `struct snd_card *card`

  声卡结构断开所有 API 与文件操作（用户空间）的连接。

**返回**

零，否则为负错误代码。

**笔记**

- 当前实现将所有活动文件->f_op 替换为特殊的

  虚拟文件操作（它们除了释放之外什么都不做）。

- 无效 `snd_card_disconnect_sync`（结构 snd_card **卡*）

  断开卡并等待文件关闭

**参数**

- `struct snd_card *card`

  要断开的卡对象

**描述**

这要求[`snd_card_disconnect()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_disconnect)断开所有所属组件并等待所有待处理文件关闭。它确保来自用户空间的所有访问都完成，以便驱动程序可以优雅地释放其资源。

- int `snd_card_free_when_closed`( struct snd_card **card* )

  断开卡，稍后释放它

**参数**

- `struct snd_card *card`

  声卡结构

**描述**

与 不同[`snd_card_free()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_free)的是，该函数不会立即尝试释放卡片资源，而是首先尝试断开连接。当卡仍在使用中时，该函数在释放资源之前返回。当 refcount 变为零时，卡片资源将被释放。

- int `snd_card_free`( struct snd_card **card* )

  释放给定的声卡结构

**参数**

- `struct snd_card *card`

  声卡结构

**描述**

此功能自动释放声卡结构和所有分配的设备。也就是说，您不必自己释放设备。

该函数等待所有资源被正确释放。

**返回**

零。释放所有关联设备并释放与给定声卡关联的控制接口。

- void `snd_card_set_id`( struct snd_card **card* , const char **nid* )

  设置卡识别名称

**参数**

- `struct snd_card *card`

  声卡结构

- `const char *nid`

  新的标识字符串此函数设置卡标识并检查名称冲突。

- int `snd_card_add_dev_attr`( struct snd_card **card* , const struct attribute_group **group* )

  将新的 sysfs 属性组附加到卡

**参数**

- `struct snd_card *card`

  卡片实例

- `const struct attribute_group *group`

  要附加的属性组

- int `snd_card_register`( struct snd_card **card* )

  注册声卡

**参数**

- `struct snd_card *card`

  声卡结构此函数注册分配给声卡的所有设备。在调用它之前，ALSA 控制接口被阻止外部访问。因此，您应该在卡的初始化结束时调用此函数。

**返回**

如果注册失败，则为零，否则为负错误代码。

- int `snd_component_add`( struct snd_card **card* , const char **component* )

  添加组件字符串

**参数**

- `struct snd_card *card`

  声卡结构

- `const char *component`

  组件 ID 字符串此函数将组件 ID 字符串添加到支持的列表中。该组件可以从 alsa-lib 中引用。

**返回**

零否则为负错误代码。

- int `snd_card_file_add`( struct snd_card **card* , struct file **file* )

  将文件添加到卡的文件列表中

**参数**

- `struct snd_card *card`

  声卡结构

- `struct file *file`

  文件指针该函数将文件添加到卡片的文件链表中。该链表用于跟踪连接状态，避免热插拔释放繁忙资源。

**返回**

零或负错误代码。

- int `snd_card_file_remove`( struct snd_card **card* , struct file **file* )

  从文件列表中删除文件

**参数**

- `struct snd_card *card`

  声卡结构

- `struct file *file`

  文件指针此函数删除以前通过函数添加到卡中的文件[`snd_card_file_add()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_file_add)。如果所有文件都被删除[`snd_card_free_when_closed()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_free_when_closed)并被事先调用，它会处理等待释放的资源。

**返回**

零或负错误代码。

- int `snd_power_ref_and_wait`( struct snd_card **card* )

  等到卡通电

**参数**

- `struct snd_card *card`

  声卡结构

**描述**

获取给定卡的 power_ref 引用计数，并等待卡上电到 SNDRV_CTL_POWER_D0 状态。refcount 在休眠时再次下降，直到上电，因此此函数可用于同步浮动控制操作访问，通常围绕调用控制操作。

[`snd_power_unref()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_power_unref)无论此函数是否返回错误，调用者都需要通过 later 拉下 refcount 。

**返回**

如果成功则为零，或负错误代码。

- int `snd_power_wait`( struct snd_card **card* )

  等到卡通电（旧形式）

**参数**

- `struct snd_card *card`

  声卡结构

**描述**

等到卡上电到 SNDRV_CTL_POWER_D0 状态。

**返回**

如果成功则为零，或负错误代码。

### 设备组件

- int `snd_device_new`( struct snd_card **card* , enum snd_device_type *类型*, void *** device_data , const struct snd_device_ops **ops* )

  创建 ALSA 设备组件

**参数**

- `struct snd_card *card`

  卡片实例

- `enum snd_device_type type`

  the device type, SNDRV_DEV_XXX

- `void *device_data`

  the data pointer of this device

- `const struct snd_device_ops *ops`

  the operator table

**Description**

Creates a new device component for the given data pointer. The device will be assigned to the card and managed together by the card.

The data pointer plays a role as the identifier, too, so the pointer address must be unique and unchanged.

**Return**

Zero if successful, or a negative error code on failure.

- void `snd_device_disconnect`(struct snd_card **card*, void **device_data*)

  disconnect the device

**Parameters**

- `struct snd_card *card`

  the card instance

- `void *device_data`

  the data pointer to disconnect

**Description**

Turns the device into the disconnection state, invoking dev_disconnect callback, if the device was already registered.

Usually called from [`snd_card_disconnect()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_disconnect).

**Return**

Zero if successful, or a negative error code on failure or if the device not found.

- void `snd_device_free`(struct snd_card **card*, void **device_data*)

  release the device from the card

**Parameters**

- `struct snd_card *card`

  the card instance

- `void *device_data`

  the data pointer to release

**Description**

Removes the device from the list on the card and invokes the callbacks, dev_disconnect and dev_free, corresponding to the state. Then release the device.

- int `snd_device_register`(struct snd_card **card*, void **device_data*)

  register the device

**Parameters**

- `struct snd_card *card`

  the card instance

- `void *device_data`

  the data pointer to register

**Description**

Registers the device which was already created via [`snd_device_new()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_device_new). Usually this is called from [`snd_card_register()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_register), but it can be called later if any new devices are created after invocation of [`snd_card_register()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_register).

**Return**

Zero if successful, or a negative error code on failure or if the device not found.

- int `snd_device_get_state`(struct snd_card **card*, void **device_data*)

  Get the current state of the given device

**Parameters**

- `struct snd_card *card`

  the card instance

- `void *device_data`

  the data pointer to release

**Description**

Returns the current state of the given device object. For the valid device, either **SNDRV_DEV_BUILD**, **SNDRV_DEV_REGISTERED** or **SNDRV_DEV_DISCONNECTED** is returned. Or for a non-existing device, -1 is returned as an error.

### Module requests and Device File Entries

- void `snd_request_card`(int *card*)

  try to load the card module

**Parameters**

- `int card`

  the card number

**Description**

Tries to load the module “snd-card-X” for the given card number via request_module. Returns immediately if already loaded.

- void * `snd_lookup_minor_data`(unsigned int *minor*, int *type*)

  get user data of a registered device

**Parameters**

- `unsigned int minor`

  the minor number

- `int type`

  device type (SNDRV_DEVICE_TYPE_XXX)

**Description**

Checks that a minor device with the specified type is registered, and returns its user data pointer.

This function increments the reference counter of the card instance if an associated instance with the given minor number and type is found. The caller must call [`snd_card_unref()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_unref) appropriately later.

**Return**

The user data pointer if the specified device is found. `NULL` otherwise.

- int `snd_register_device`(int *type*, struct snd_card **card*, int *dev*, const struct file_operations **f_ops*, void **private_data*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **device*)

  Register the ALSA device file for the card

**Parameters**

- `int type`

  the device type, SNDRV_DEVICE_TYPE_XXX

- `struct snd_card *card`

  the card instance

- `int dev`

  the device index

- `const struct file_operations *f_ops`

  the file operations

- `void *private_data`

  user pointer for f_ops->open()

- `struct device *device`

  the device to register

**Description**

Registers an ALSA device file for the given card. The operators have to be set in reg parameter.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_unregister_device`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  unregister the device on the given card

**Parameters**

- `struct device *dev`

  the device instance

**Description**

Unregisters the device file already registered via [`snd_register_device()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_register_device).

**Return**

Zero if successful, or a negative error code on failure.

### Memory Management Helpers

- int `copy_to_user_fromio`(void __user **dst*, const volatile void __iomem **src*, size_t *count*)

  copy data from mmio-space to user-space

**Parameters**

- `void __user *dst`

  the destination pointer on user-space

- `const volatile void __iomem *src`

  the source pointer on mmio

- `size_t count`

  the data size to copy in bytes

**Description**

Copies the data from mmio-space to user-space.

**Return**

Zero if successful, or non-zero on failure.

- int `copy_from_user_toio`( volatile void __iomem **dst* , const void __user **src* , size_t *count* )

  将数据从用户空间复制到 mmio 空间

**参数**

- `volatile void __iomem *dst`

  mmio-space 上的目标指针

- `const void __user *src`

  用户空间上的源指针

- `size_t count`

  要复制的数据大小（以字节为单位）

**描述**

将数据从用户空间复制到 mmio 空间。

**返回**

成功时为零，失败时为非零。

- int `snd_dma_alloc_dir_pages`( int *类型*, struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **device* , enum dma_data_direction *dir* , size_t *size* , struct snd_dma_buffer **dmab* )

  根据给定的类型和方向分配缓冲区

**参数**

- `int type`

  DMA 缓冲区类型

- `struct device *device`

  设备指针

- `enum dma_data_direction dir`

  DMA方向

- `size_t size`

  要分配的缓冲区大小

- `struct snd_dma_buffer *dmab`

  缓冲区分配记录，用于存储分配的数据

**描述**

调用相应缓冲区类型的内存分配器函数。

**返回**

如果成功分配给定大小的缓冲区，则为零，否则为负值。

- int `snd_dma_alloc_pages_fallback`( int *类型*, struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **device* , size_t *size* , struct snd_dma_buffer **dmab* )

  根据给定类型分配缓冲区并回退

**参数**

- `int type`

  DMA 缓冲区类型

- `struct device *device`

  设备指针

- `size_t size`

  要分配的缓冲区大小

- `struct snd_dma_buffer *dmab`

  缓冲区分配记录，用于存储分配的数据

**描述**

Calls the memory-allocator function for the corresponding buffer type. When no space is left, this function reduces the size and tries to allocate again. The size actually allocated is stored in res_size argument.

**Return**

Zero if the buffer with the given size is allocated successfully, otherwise a negative value on error.

- void `snd_dma_free_pages`(struct snd_dma_buffer **dmab*)

  release the allocated buffer

**Parameters**

- `struct snd_dma_buffer *dmab`

  the buffer allocation record to release

**Description**

Releases the allocated buffer via snd_dma_alloc_pages().

- struct snd_dma_buffer * `snd_devm_alloc_dir_pages`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, int *type*, enum dma_data_direction *dir*, size_t *size*)

  allocate the buffer and manage with devres

**Parameters**

- `struct device *dev`

  the device pointer

- `int type`

  the DMA buffer type

- `enum dma_data_direction dir`

  DMA direction

- `size_t size`

  the buffer size to allocate

**Description**

Allocate buffer pages depending on the given type and manage using devres. The pages will be released automatically at the device removal.

Unlike snd_dma_alloc_pages(), this function requires the real device pointer, hence it can’t work with SNDRV_DMA_TYPE_CONTINUOUS or SNDRV_DMA_TYPE_VMALLOC type.

The function returns the snd_dma_buffer object at success, or NULL if failed.

- int `snd_dma_buffer_mmap`(struct snd_dma_buffer **dmab*, struct vm_area_struct **area*)

  perform mmap of the given DMA buffer

**Parameters**

- `struct snd_dma_buffer *dmab`

  buffer allocation information

- `struct vm_area_struct *area`

  VM area information

- void `snd_dma_buffer_sync`(struct snd_dma_buffer **dmab*, enum snd_dma_sync_mode *mode*)

  sync DMA buffer between CPU and device

**Parameters**

- `struct snd_dma_buffer *dmab`

  buffer allocation information

- `enum snd_dma_sync_mode mode`

  sync mode

- dma_addr_t `snd_sgbuf_get_addr`(struct snd_dma_buffer **dmab*, size_t *offset*)

  return the physical address at the corresponding offset

**Parameters**

- `struct snd_dma_buffer *dmab`

  buffer allocation information

- `size_t offset`

  offset in the ring buffer

- struct page * `snd_sgbuf_get_page`(struct snd_dma_buffer **dmab*, size_t *offset*)

  return the physical page at the corresponding offset

**Parameters**

- `struct snd_dma_buffer *dmab`

  buffer allocation information

- `size_t offset`

  offset in the ring buffer

- unsigned int `snd_sgbuf_get_chunk_size`(struct snd_dma_buffer **dmab*, unsigned int *ofs*, unsigned int *size*)

  compute the max chunk size with continuous pages on sg-buffer

**Parameters**

- `struct snd_dma_buffer *dmab`

  buffer allocation information

- `unsigned int ofs`

  offset in the ring buffer

- `unsigned int size`

  the requested size

## PCM API

### PCM Core

- const char * `snd_pcm_format_name`(snd_pcm_format_t *format*)

  Return a name string for the given PCM format

**Parameters**

- `snd_pcm_format_t format`

  PCM format

- int `snd_pcm_new_stream`(struct snd_pcm **pcm*, int *stream*, int *substream_count*)

  create a new PCM stream

**Parameters**

- `struct snd_pcm *pcm`

  the pcm instance

- `int stream`

  the stream direction, SNDRV_PCM_STREAM_XXX

- `int substream_count`

  the number of substreams

**Description**

Creates a new stream for the pcm. The corresponding stream on the pcm must have been empty before calling this, i.e. zero must be given to the argument of [`snd_pcm_new()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_new).

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_new`(struct snd_card **card*, const char **id*, int *device*, int *playback_count*, int *capture_count*, struct snd_pcm ***rpcm*)

  create a new PCM instance

**Parameters**

- `struct snd_card *card`

  the card instance

- `const char *id`

  the id string

- `int device`

  the device index (zero based)

- `int playback_count`

  the number of substreams for playback

- `int capture_count`

  the number of substreams for capture

- `struct snd_pcm **rpcm`

  the pointer to store the new pcm instance

**Description**

Creates a new PCM instance.

The pcm operators have to be set afterwards to the new instance via [`snd_pcm_set_ops()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_set_ops).

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_new_internal`(struct snd_card **card*, const char **id*, int *device*, int *playback_count*, int *capture_count*, struct snd_pcm ***rpcm*)

  create a new internal PCM instance

**Parameters**

- `struct snd_card *card`

  the card instance

- `const char *id`

  the id string

- `int device`

  the device index (zero based - shared with normal PCMs)

- `int playback_count`

  the number of substreams for playback

- `int capture_count`

  the number of substreams for capture

- `struct snd_pcm **rpcm`

  the pointer to store the new pcm instance

**Description**

Creates a new internal PCM instance with no userspace device or procfs entries. This is used by ASoC Back End PCMs in order to create a PCM that will only be used internally by kernel drivers. i.e. it cannot be opened by userspace. It provides existing ASoC components drivers with a substream and access to any private data.

The pcm operators have to be set afterwards to the new instance via [`snd_pcm_set_ops()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_set_ops).

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_notify`(struct [snd_pcm_notify](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_notify) **notify*, int *nfree*)

  Add/remove the notify list

**Parameters**

- `struct snd_pcm_notify *notify`

  PCM notify list

- `int nfree`

  0 = register, 1 = unregister

**Description**

This adds the given notifier to the global list so that the callback is called for each registered PCM devices. This exists only for PCM OSS emulation, so far.

- void `snd_pcm_set_ops`(struct snd_pcm **pcm*, int *direction*, const struct snd_pcm_ops **ops*)

  set the PCM operators

**Parameters**

- `struct snd_pcm *pcm`

  the pcm instance

- `int direction`

  stream direction, SNDRV_PCM_STREAM_XXX

- `const struct snd_pcm_ops *ops`

  the operator table

**Description**

Sets the given PCM operators to the pcm instance.

- void `snd_pcm_set_sync`(struct snd_pcm_substream **substream*)

  set the PCM sync id

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream

**Description**

Sets the PCM sync identifier for the card.

- int `snd_interval_refine`(struct snd_interval **i*, const struct snd_interval **v*)

  refine the interval value of configurator

**Parameters**

- `struct snd_interval *i`

  the interval value to refine

- `const struct snd_interval *v`

  the interval value to refer to

**Description**

Refines the interval value with the reference value. The interval is changed to the range satisfying both intervals. The interval status (min, max, integer, etc.) are evaluated.

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- void `snd_interval_div`(const struct snd_interval **a*, const struct snd_interval **b*, struct snd_interval **c*)

  refine the interval value with division

**Parameters**

- `const struct snd_interval *a`

  dividend

- `const struct snd_interval *b`

  divisor

- `struct snd_interval *c`

  quotient

**Description**

c = a / b

Returns non-zero if the value is changed, zero if not changed.

- void `snd_interval_muldivk`(const struct snd_interval **a*, const struct snd_interval **b*, unsigned int *k*, struct snd_interval **c*)

  refine the interval value

**Parameters**

- `const struct snd_interval *a`

  dividend 1

- `const struct snd_interval *b`

  dividend 2

- `unsigned int k`

  divisor (as integer)

- `struct snd_interval *c`

  result

**Description**

c = a * b / k

Returns non-zero if the value is changed, zero if not changed.

- void `snd_interval_mulkdiv`(const struct snd_interval **a*, unsigned int *k*, const struct snd_interval **b*, struct snd_interval **c*)

  refine the interval value

**Parameters**

- `const struct snd_interval *a`

  dividend 1

- `unsigned int k`

  dividend 2 (as integer)

- `const struct snd_interval *b`

  divisor

- `struct snd_interval *c`

  result

**Description**

c = a * k / b

Returns non-zero if the value is changed, zero if not changed.

- int `snd_interval_ratnum`(struct snd_interval **i*, unsigned int *rats_count*, const struct snd_ratnum **rats*, unsigned int **nump*, unsigned int **denp*)

  refine the interval value

**Parameters**

- `struct snd_interval *i`

  interval to refine

- `unsigned int rats_count`

  number of ratnum_t

- `const struct snd_ratnum *rats`

  ratnum_t array

- `unsigned int *nump`

  pointer to store the resultant numerator

- `unsigned int *denp`

  pointer to store the resultant denominator

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_interval_ratden`(struct snd_interval **i*, unsigned int *rats_count*, const struct snd_ratden **rats*, unsigned int **nump*, unsigned int **denp*)

  refine the interval value

**Parameters**

- `struct snd_interval *i`

  interval to refine

- `unsigned int rats_count`

  number of struct ratden

- `const struct snd_ratden *rats`

  struct ratden array

- `unsigned int *nump`

  pointer to store the resultant numerator

- `unsigned int *denp`

  pointer to store the resultant denominator

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_interval_list`(struct snd_interval **i*, unsigned int *count*, const unsigned int **list*, unsigned int *mask*)

  refine the interval value from the list

**Parameters**

- `struct snd_interval *i`

  the interval value to refine

- `unsigned int count`

  the number of elements in the list

- `const unsigned int *list`

  the value list

- `unsigned int mask`

  the bit-mask to evaluate

**Description**

Refines the interval value from the list. When mask is non-zero, only the elements corresponding to bit 1 are evaluated.

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_interval_ranges`(struct snd_interval **i*, unsigned int *count*, const struct snd_interval **ranges*, unsigned int *mask*)

  refine the interval value from the list of ranges

**Parameters**

- `struct snd_interval *i`

  the interval value to refine

- `unsigned int count`

  the number of elements in the list of ranges

- `const struct snd_interval *ranges`

  the ranges list

- `unsigned int mask`

  the bit-mask to evaluate

**Description**

Refines the interval value from the list of ranges. When mask is non-zero, only the elements corresponding to bit 1 are evaluated.

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_pcm_hw_rule_add`(struct snd_pcm_runtime **runtime*, unsigned int *cond*, int *var*, snd_pcm_hw_rule_func_t *func*, void **private*, int *dep*, ... )

  add the hw-constraint rule

**Parameters**

- `struct snd_pcm_runtime *runtime`

  the pcm runtime instance

- `unsigned int cond`

  condition bits

- `int var`

  the variable to evaluate

- `snd_pcm_hw_rule_func_t func`

  the evaluation function

- `void *private`

  the private data pointer passed to function

- `int dep`

  the dependent variables

- `...`

  variable arguments

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_mask`(struct snd_pcm_runtime **runtime*, snd_pcm_hw_param_t *var*, u_int32_t *mask*)

  apply the given bitmap mask constraint

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the mask

- `u_int32_t mask`

  the bitmap mask

**Description**

Apply the constraint of the given bitmap mask to a 32-bit mask parameter.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_mask64`(struct snd_pcm_runtime **runtime*, snd_pcm_hw_param_t *var*, u_int64_t *mask*)

  apply the given bitmap mask constraint

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the mask

- `u_int64_t mask`

  the 64bit bitmap mask

**Description**

Apply the constraint of the given bitmap mask to a 64-bit mask parameter.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_integer`(struct snd_pcm_runtime **runtime*, snd_pcm_hw_param_t *var*)

  apply an integer constraint to an interval

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the integer constraint

**Description**

Apply the constraint of integer to an interval parameter.

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_pcm_hw_constraint_minmax`(struct snd_pcm_runtime **runtime*, snd_pcm_hw_param_t *var*, unsigned int *min*, unsigned int *max*)

  apply a min/max range constraint to an interval

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the range

- `unsigned int min`

  the minimal value

- `unsigned int max`

  the maximal value

**Description**

Apply the min/max range constraint to an interval parameter.

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_pcm_hw_constraint_list`(struct snd_pcm_runtime **runtime*, unsigned int *cond*, snd_pcm_hw_param_t *var*, const struct [snd_pcm_hw_constraint_list](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_hw_constraint_list) **l*)

  apply a list of constraints to a parameter

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `unsigned int cond`

  condition bits

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the list constraint

- `const struct snd_pcm_hw_constraint_list *l`

  list

**Description**

Apply the list of constraints to an interval parameter.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_ranges`(struct snd_pcm_runtime **runtime*, unsigned int *cond*, snd_pcm_hw_param_t *var*, const struct [snd_pcm_hw_constraint_ranges](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_hw_constraint_ranges) **r*)

  apply list of range constraints to a parameter

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `unsigned int cond`

  condition bits

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the list of range constraints

- `const struct snd_pcm_hw_constraint_ranges *r`

  ranges

**Description**

Apply the list of range constraints to an interval parameter.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_ratnums`(struct snd_pcm_runtime **runtime*, unsigned int *cond*, snd_pcm_hw_param_t *var*, const struct [snd_pcm_hw_constraint_ratnums](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_hw_constraint_ratnums) **r*)

  apply ratnums constraint to a parameter

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `unsigned int cond`

  condition bits

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the ratnums constraint

- `const struct snd_pcm_hw_constraint_ratnums *r`

  struct snd_ratnums constriants

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_ratdens`( struct snd_pcm_runtime **runtime* , unsigned int *cond* , snd_pcm_hw_param_t *var* , const struct [snd_pcm_hw_constraint_ratdens ](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_hw_constraint_ratdens) **r* )

  将ratdens约束应用于参数

**参数**

- `struct snd_pcm_runtime *runtime`

  PCM 运行时实例

- `unsigned int cond`

  条件位

- `snd_pcm_hw_param_t var`

  hw_params 变量应用 ratdens 约束

- `const struct snd_pcm_hw_constraint_ratdens *r`

  struct snd_ratdens 约束

**返回**

成功则为零，失败则为负错误代码。

- int `snd_pcm_hw_constraint_msbits`( struct snd_pcm_runtime **runtime* , unsigned int *cond* , unsigned int *width* , unsigned int *msbits* )

  添加硬件约束 msbits 规则

**参数**

- `struct snd_pcm_runtime *runtime`

  PCM 运行时实例

- `unsigned int cond`

  条件位

- `unsigned int width`

  样本位宽度

- `unsigned int msbits`

  毫秒位宽度

**描述**

如果选择了具有指定宽度的样本格式，则此约束将设置最高有效位 (msbits) 的数量。如果宽度设置为 0，则将为宽度大于指定 msbits 的任何样本格式设置 msbits。

**返回**

成功则为零，失败则为负错误代码。

- int `snd_pcm_hw_constraint_step`(struct snd_pcm_runtime **runtime*, unsigned int *cond*, snd_pcm_hw_param_t *var*, unsigned long *step*)

  add a hw constraint step rule

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `unsigned int cond`

  condition bits

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the step constraint

- `unsigned long step`

  step size

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_constraint_pow2`(struct snd_pcm_runtime **runtime*, unsigned int *cond*, snd_pcm_hw_param_t *var*)

  add a hw constraint power-of-2 rule

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `unsigned int cond`

  condition bits

- `snd_pcm_hw_param_t var`

  hw_params variable to apply the power-of-2 constraint

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_rule_noresample`(struct snd_pcm_runtime **runtime*, unsigned int *base_rate*)

  add a rule to allow disabling hw resampling

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `unsigned int base_rate`

  the rate at which the hardware does not resample

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_param_value`(const struct snd_pcm_hw_params **params*, snd_pcm_hw_param_t *var*, int **dir*)

  return **params** field **var** value

**Parameters**

- `const struct snd_pcm_hw_params *params`

  the hw_params instance

- `snd_pcm_hw_param_t var`

  parameter to retrieve

- `int *dir`

  pointer to the direction (-1,0,1) or `NULL`

**Return**

The value for field **var** if it’s fixed in configuration space defined by **params**. -`EINVAL` otherwise.

- int `snd_pcm_hw_param_first`(struct snd_pcm_substream **pcm*, struct snd_pcm_hw_params **params*, snd_pcm_hw_param_t *var*, int **dir*)

  refine config space and return minimum value

**Parameters**

- `struct snd_pcm_substream *pcm`

  PCM instance

- `struct snd_pcm_hw_params *params`

  the hw_params instance

- `snd_pcm_hw_param_t var`

  parameter to retrieve

- `int *dir`

  pointer to the direction (-1,0,1) or `NULL`

**Description**

Inside configuration space defined by **params** remove from **var** all values > minimum. Reduce configuration space accordingly.

**Return**

The minimum, or a negative error code on failure.

- int `snd_pcm_hw_param_last`(struct snd_pcm_substream **pcm*, struct snd_pcm_hw_params **params*, snd_pcm_hw_param_t *var*, int **dir*)

  refine config space and return maximum value

**Parameters**

- `struct snd_pcm_substream *pcm`

  PCM instance

- `struct snd_pcm_hw_params *params`

  the hw_params instance

- `snd_pcm_hw_param_t var`

  parameter to retrieve

- `int *dir`

  pointer to the direction (-1,0,1) or `NULL`

**Description**

Inside configuration space defined by **params** remove from **var** all values < maximum. Reduce configuration space accordingly.

**Return**

The maximum, or a negative error code on failure.

- int `snd_pcm_lib_ioctl`(struct snd_pcm_substream **substream*, unsigned int *cmd*, void **arg*)

  a generic PCM ioctl callback

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

- `unsigned int cmd`

  ioctl command

- `void *arg`

  ioctl argument

**Description**

Processes the generic ioctl commands for PCM. Can be passed as the ioctl callback for PCM ops.

**Return**

Zero if successful, or a negative error code on failure.

- void `snd_pcm_period_elapsed_under_stream_lock`(struct snd_pcm_substream **substream*)

  update the status of runtime for the next period under acquired lock of PCM substream.

**Parameters**

- `struct snd_pcm_substream *substream`

  the instance of pcm substream.

**Description**

This function is called when the batch of audio data frames as the same size as the period of buffer is already processed in audio data transmission.

The call of function updates the status of runtime with the latest position of audio data transmission, checks overrun and underrun over buffer, awaken user processes from waiting for available audio data frames, sampling audio timestamp, and performs stop or drain the PCM substream according to configured threshold.

The function is intended to use for the case that PCM driver operates audio data frames under acquired lock of PCM substream; e.g. in callback of any operation of `snd_pcm_ops` in process context. In any interrupt context, it’s preferrable to use `snd_pcm_period_elapsed()` instead since lock of PCM substream should be acquired in advance.

Developer should pay enough attention that some callbacks in `snd_pcm_ops` are done by the call of function:

- .pointer - to retrieve current position of audio data transmission by frame count or XRUN state.
- .trigger - with SNDRV_PCM_TRIGGER_STOP at XRUN or DRAINING state.
- .get_time_info - to retrieve audio time stamp if needed.

Even if more than one periods have elapsed since the last call, you have to call this only once.

- void `snd_pcm_period_elapsed`(struct snd_pcm_substream **substream*)

  update the status of runtime for the next period by acquiring lock of PCM substream.

**Parameters**

- `struct snd_pcm_substream *substream`

  the instance of PCM substream.

**Description**

This function is mostly similar to `snd_pcm_period_elapsed_under_stream_lock()` except for acquiring lock of PCM substream voluntarily.

It’s typically called by any type of IRQ handler when hardware IRQ occurs to notify event that the batch of audio data frames as the same size as the period of buffer is already processed in audio data transmission.

- int `snd_pcm_add_chmap_ctls`(struct snd_pcm **pcm*, int *stream*, const struct snd_pcm_chmap_elem **chmap*, int *max_channels*, unsigned long *private_value*, struct snd_pcm_chmap ***info_ret*)

  create channel-mapping control elements

**Parameters**

- `struct snd_pcm *pcm`

  the assigned PCM instance

- `int stream`

  stream direction

- `const struct snd_pcm_chmap_elem *chmap`

  channel map elements (for query)

- `int max_channels`

  the max number of channels for the stream

- `unsigned long private_value`

  the value passed to each kcontrol’s private_value field

- `struct snd_pcm_chmap **info_ret`

  store struct snd_pcm_chmap instance if non-NULL

**Description**

Create channel-mapping control elements assigned to the given PCM stream(s).

**Return**

Zero if successful, or a negative error value.

- void `snd_pcm_stream_lock`(struct snd_pcm_substream **substream*)

  Lock the PCM stream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

This locks the PCM stream’s spinlock or mutex depending on the nonatomic flag of the given substream. This also takes the global link rw lock (or rw sem), too, for avoiding the race with linked streams.

- void `snd_pcm_stream_unlock`(struct snd_pcm_substream **substream*)

  Unlock the PCM stream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

This unlocks the PCM stream that has been locked via [`snd_pcm_stream_lock()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock).

- void `snd_pcm_stream_lock_irq`(struct snd_pcm_substream **substream*)

  Lock the PCM stream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

This locks the PCM stream like [`snd_pcm_stream_lock()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock) and disables the local IRQ (only when nonatomic is false). In nonatomic case, this is identical as [`snd_pcm_stream_lock()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock).

- void `snd_pcm_stream_unlock_irq`(struct snd_pcm_substream **substream*)

  Unlock the PCM stream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

This is a counter-part of [`snd_pcm_stream_lock_irq()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock_irq).

- void `snd_pcm_stream_unlock_irqrestore`(struct snd_pcm_substream **substream*, unsigned long *flags*)

  Unlock the PCM stream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `unsigned long flags`

  irq flags

**Description**

This is a counter-part of [`snd_pcm_stream_lock_irqsave()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock_irqsave).

- int `snd_pcm_hw_params_choose`(struct snd_pcm_substream **pcm*, struct snd_pcm_hw_params **params*)

  choose a configuration defined by **params**

**Parameters**

- `struct snd_pcm_substream *pcm`

  PCM instance

- `struct snd_pcm_hw_params *params`

  the hw_params instance

**Description**

Choose one configuration from configuration space defined by **params**. The configuration chosen is that obtained fixing in this order: first access, first format, first subformat, min channels, min rate, min period time, max buffer size, min tick time

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_start`(struct snd_pcm_substream **substream*)

  start all linked streams

**Parameters**

- `struct snd_pcm_substream *substream`

  the PCM substream instance

**Return**

Zero if successful, or a negative error code. The stream lock must be acquired before calling this function.

- int `snd_pcm_stop`(struct snd_pcm_substream **substream*, snd_pcm_state_t *state*)

  try to stop all running streams in the substream group

**Parameters**

- `struct snd_pcm_substream *substream`

  the PCM substream instance

- `snd_pcm_state_t state`

  PCM state after stopping the stream

**Description**

The state of each stream is then changed to the given state unconditionally.

**Return**

Zero if successful, or a negative error code.

- int `snd_pcm_drain_done`(struct snd_pcm_substream **substream*)

  stop the DMA only when the given stream is playback

**Parameters**

- `struct snd_pcm_substream *substream`

  the PCM substream

**Description**

After stopping, the state is changed to SETUP. Unlike [`snd_pcm_stop()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stop), this affects only the given stream.

**Return**

Zero if successful, or a negative error code.

- int `snd_pcm_stop_xrun`(struct snd_pcm_substream **substream*)

  stop the running streams as XRUN

**Parameters**

- `struct snd_pcm_substream *substream`

  the PCM substream instance

**Description**

This stops the given running substream (and all linked substreams) as XRUN. Unlike [`snd_pcm_stop()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stop), this function takes the substream lock by itself.

**Return**

Zero if successful, or a negative error code.

- int `snd_pcm_suspend_all`(struct snd_pcm **pcm*)

  trigger SUSPEND to all substreams in the given pcm

**Parameters**

- `struct snd_pcm *pcm`

  the PCM instance

**Description**

After this call, all streams are changed to SUSPENDED state.

**Return**

Zero if successful (or **pcm** is `NULL`), or a negative error code.

- int `snd_pcm_prepare`(struct snd_pcm_substream **substream*, struct file **file*)

  prepare the PCM substream to be triggerable

**Parameters**

- `struct snd_pcm_substream *substream`

  the PCM substream instance

- `struct file *file`

  file to refer f_flags

**Return**

Zero if successful, or a negative error code.

- int `snd_pcm_kernel_ioctl`(struct snd_pcm_substream **substream*, unsigned int *cmd*, void **arg*)

  Execute PCM ioctl in the kernel-space

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `unsigned int cmd`

  IOCTL cmd

- `void *arg`

  IOCTL argument

**Description**

The function is provided primarily for OSS layer and USB gadget drivers, and it allows only the limited set of ioctls (hw_params, sw_params, prepare, start, drain, drop, forward).

- int `snd_pcm_lib_default_mmap`(struct snd_pcm_substream **substream*, struct vm_area_struct **area*)

  Default PCM data mmap function

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `struct vm_area_struct *area`

  VMA

**Description**

This is the default mmap handler for PCM data. When mmap pcm_ops is NULL, this function is invoked implicitly.

- int `snd_pcm_lib_mmap_iomem`(struct snd_pcm_substream **substream*, struct vm_area_struct **area*)

  Default PCM data mmap function for I/O mem

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `struct vm_area_struct *area`

  VMA

**Description**

When your hardware uses the iomapped pages as the hardware buffer and wants to mmap it, pass this function as mmap pcm_ops. Note that this is supposed to work only on limited architectures.

- int `snd_pcm_stream_linked`(struct snd_pcm_substream **substream*)

  Check whether the substream is linked with others

**Parameters**

- `struct snd_pcm_substream *substream`

  substream to check

**Description**

Returns true if the given substream is being linked with others.

- `snd_pcm_stream_lock_irqsave`(*substream*, *flags*)

  Lock the PCM stream

**Parameters**

- `substream`

  PCM substream

- `flags`

  irq flags

**Description**

This locks the PCM stream like [`snd_pcm_stream_lock()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock) but with the local IRQ (only when nonatomic is false). In nonatomic case, this is identical as [`snd_pcm_stream_lock()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock).

- `snd_pcm_stream_lock_irqsave_nested`(*substream*, *flags*)

  Single-nested PCM stream locking

**Parameters**

- `substream`

  PCM substream

- `flags`

  irq flags

**Description**

This locks the PCM stream like [`snd_pcm_stream_lock_irqsave()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_stream_lock_irqsave) but with the single-depth lockdep subclass.

- `snd_pcm_group_for_each_entry`(*s*, *substream*)

  iterate over the linked substreams

**Parameters**

- `s`

  the iterator

- `substream`

  the substream

**Description**

Iterate over the all linked substreams to the given **substream**. When **substream** isn’t linked with any others, this gives returns **substream** itself once.

- int `snd_pcm_running`(struct snd_pcm_substream **substream*)

  Check whether the substream is in a running state

**Parameters**

- `struct snd_pcm_substream *substream`

  substream to check

**Description**

Returns true if the given substream is in the state RUNNING, or in the state DRAINING for playback.

- ssize_t `bytes_to_samples`(struct snd_pcm_runtime **runtime*, ssize_t *size*)

  Unit conversion of the size from bytes to samples

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `ssize_t size`

  size in bytes

- snd_pcm_sframes_t `bytes_to_frames`(struct snd_pcm_runtime **runtime*, ssize_t *size*)

  Unit conversion of the size from bytes to frames

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `ssize_t size`

  size in bytes

- ssize_t `samples_to_bytes`(struct snd_pcm_runtime **runtime*, ssize_t *size*)

  Unit conversion of the size from samples to bytes

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `ssize_t size`

  size in samples

- ssize_t `frames_to_bytes`(struct snd_pcm_runtime **runtime*, snd_pcm_sframes_t *size*)

  Unit conversion of the size from frames to bytes

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `snd_pcm_sframes_t size`

  size in frames

- int `frame_aligned`(struct snd_pcm_runtime **runtime*, ssize_t *bytes*)

  Check whether the byte size is aligned to frames

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `ssize_t bytes`

  size in bytes

- size_t `snd_pcm_lib_buffer_bytes`(struct snd_pcm_substream **substream*)

  Get the buffer size of the current PCM in bytes

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- size_t `snd_pcm_lib_period_bytes`(struct snd_pcm_substream **substream*)

  Get the period size of the current PCM in bytes

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- snd_pcm_uframes_t `snd_pcm_playback_avail`(struct snd_pcm_runtime **runtime*)

  Get the available (writable) space for playback

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

**Description**

Result is between 0 … (boundary - 1)

- snd_pcm_uframes_t `snd_pcm_capture_avail`(struct snd_pcm_runtime **runtime*)

  Get the available (readable) space for capture

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

**Description**

Result is between 0 … (boundary - 1)

- snd_pcm_sframes_t `snd_pcm_playback_hw_avail`(struct snd_pcm_runtime **runtime*)

  Get the queued space for playback

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- snd_pcm_sframes_t `snd_pcm_capture_hw_avail`(struct snd_pcm_runtime **runtime*)

  Get the free space for capture

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- int `snd_pcm_playback_ready`(struct snd_pcm_substream **substream*)

  check whether the playback buffer is available

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Description**

Checks whether enough free space is available on the playback buffer.

**Return**

Non-zero if available, or zero if not.

- int `snd_pcm_capture_ready`(struct snd_pcm_substream **substream*)

  check whether the capture buffer is available

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Description**

Checks whether enough capture data is available on the capture buffer.

**Return**

Non-zero if available, or zero if not.

- int `snd_pcm_playback_data`(struct snd_pcm_substream **substream*)

  check whether any data exists on the playback buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Description**

Checks whether any data exists on the playback buffer.

**Return**

Non-zero if any data exists, or zero if not. If stop_threshold is bigger or equal to boundary, then this function returns always non-zero.

- int `snd_pcm_playback_empty`(struct snd_pcm_substream **substream*)

  check whether the playback buffer is empty

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Description**

Checks whether the playback buffer is empty.

**Return**

Non-zero if empty, or zero if not.

- int `snd_pcm_capture_empty`(struct snd_pcm_substream **substream*)

  check whether the capture buffer is empty

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Description**

Checks whether the capture buffer is empty.

**Return**

Non-zero if empty, or zero if not.

- void `snd_pcm_trigger_done`(struct snd_pcm_substream **substream*, struct snd_pcm_substream **master*)

  Mark the master substream

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

- `struct snd_pcm_substream *master`

  the linked master substream

**Description**

When multiple substreams of the same card are linked and the hardware supports the single-shot operation, the driver calls this in the loop in [`snd_pcm_group_for_each_entry()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_group_for_each_entry) for marking the substream as “done”. Then most of trigger operations are performed only to the given master substream.

The trigger_master mark is cleared at timestamp updates at the end of trigger operations.

- unsigned int `params_channels`(const struct snd_pcm_hw_params **p*)

  Get the number of channels from the hw params

**Parameters**

- `const struct snd_pcm_hw_params *p`

  hw params

- unsigned int `params_rate`(const struct snd_pcm_hw_params **p*)

  Get the sample rate from the hw params

**Parameters**

- `const struct snd_pcm_hw_params *p`

  hw params

- unsigned int `params_period_size`(const struct snd_pcm_hw_params **p*)

  Get the period size (in frames) from the hw params

**Parameters**

- `const struct snd_pcm_hw_params *p`

  hw params

- unsigned int `params_periods`(const struct snd_pcm_hw_params **p*)

  Get the number of periods from the hw params

**Parameters**

- `const struct snd_pcm_hw_params *p`

  hw params

- unsigned int `params_buffer_size`(const struct snd_pcm_hw_params **p*)

  Get the buffer size (in frames) from the hw params

**Parameters**

- `const struct snd_pcm_hw_params *p`

  hw params

- unsigned int `params_buffer_bytes`(const struct snd_pcm_hw_params **p*)

  Get the buffer size (in bytes) from the hw params

**Parameters**

- `const struct snd_pcm_hw_params *p`

  hw params

- int `snd_pcm_hw_constraint_single`(struct snd_pcm_runtime **runtime*, snd_pcm_hw_param_t *var*, unsigned int *val*)

  Constrain parameter to a single value

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `snd_pcm_hw_param_t var`

  The hw_params variable to constrain

- `unsigned int val`

  The value to constrain to

**Return**

Positive if the value is changed, zero if it’s not changed, or a negative error code.

- int `snd_pcm_format_cpu_endian`(snd_pcm_format_t *format*)

  Check the PCM format is CPU-endian

**Parameters**

- `snd_pcm_format_t format`

  the format to check

**Return**

1 if the given PCM format is CPU-endian, 0 if opposite, or a negative error code if endian not specified.

- void `snd_pcm_set_runtime_buffer`(struct snd_pcm_substream **substream*, struct snd_dma_buffer **bufp*)

  Set the PCM runtime buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream to set

- `struct snd_dma_buffer *bufp`

  the buffer information, NULL to clear

**Description**

Copy the buffer information to runtime->dma_buffer when **bufp** is non-NULL. Otherwise it clears the current buffer information.

- void `snd_pcm_gettime`(struct snd_pcm_runtime **runtime*, struct timespec64 **tv*)

  Fill the timespec64 depending on the timestamp mode

**Parameters**

- `struct snd_pcm_runtime *runtime`

  PCM runtime instance

- `struct timespec64 *tv`

  timespec64 to fill

- int `snd_pcm_set_fixed_buffer`(struct snd_pcm_substream **substream*, int *type*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **data*, size_t *size*)

  Preallocate and set up the fixed size PCM buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

- `int type`

  DMA type (SNDRV_DMA_TYPE_*)

- `struct device *data`

  DMA type dependent data

- `size_t size`

  the requested pre-allocation size in bytes

**Description**

This is a variant of [`snd_pcm_set_managed_buffer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_set_managed_buffer), but this pre-allocates only the given sized buffer and doesn’t allow re-allocation nor dynamic allocation of a larger buffer unlike the standard one. The function may return -ENOMEM error, hence the caller must check it.

- int `snd_pcm_set_fixed_buffer_all`(struct snd_pcm **pcm*, int *type*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **data*, size_t *size*)

  Preallocate and set up the fixed size PCM buffer

**Parameters**

- `struct snd_pcm *pcm`

  the pcm instance

- `int type`

  DMA type (SNDRV_DMA_TYPE_*)

- `struct device *data`

  DMA type dependent data

- `size_t size`

  the requested pre-allocation size in bytes

**Description**

Apply the set up of the fixed buffer via [`snd_pcm_set_fixed_buffer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_set_fixed_buffer) for all substream. If any of allocation fails, it returns -ENOMEM, hence the caller must check the return value.

- int `snd_pcm_lib_alloc_vmalloc_buffer`(struct snd_pcm_substream **substream*, size_t *size*)

  allocate virtual DMA buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  the substream to allocate the buffer to

- `size_t size`

  the requested buffer size, in bytes

**Description**

Allocates the PCM substream buffer using [`vmalloc()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/mm-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.vmalloc), i.e., the memory is contiguous in kernel virtual space, but not in physical memory. Use this if the buffer is accessed by kernel code but not by device DMA.

**Return**

1 if the buffer was changed, 0 if not changed, or a negative error code.

- int `snd_pcm_lib_alloc_vmalloc_32_buffer`(struct snd_pcm_substream **substream*, size_t *size*)

  allocate 32-bit-addressable buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  the substream to allocate the buffer to

- `size_t size`

  the requested buffer size, in bytes

**Description**

This function works like [`snd_pcm_lib_alloc_vmalloc_buffer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_lib_alloc_vmalloc_buffer), but uses [`vmalloc_32()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/mm-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.vmalloc_32), i.e., the pages are allocated from 32-bit-addressable memory.

**Return**

1 if the buffer was changed, 0 if not changed, or a negative error code.

- dma_addr_t `snd_pcm_sgbuf_get_addr`(struct snd_pcm_substream **substream*, unsigned int *ofs*)

  Get the DMA address at the corresponding offset

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `unsigned int ofs`

  byte offset

- unsigned int `snd_pcm_sgbuf_get_chunk_size`(struct snd_pcm_substream **substream*, unsigned int *ofs*, unsigned int *size*)

  Compute the max size that fits within the contig. page from the given size

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `unsigned int ofs`

  byte offset

- `unsigned int size`

  byte size to examine

- void `snd_pcm_mmap_data_open`(struct vm_area_struct **area*)

  increase the mmap counter

**Parameters**

- `struct vm_area_struct *area`

  VMA

**Description**

PCM mmap callback should handle this counter properly

- void `snd_pcm_mmap_data_close`(struct vm_area_struct **area*)

  decrease the mmap counter

**Parameters**

- `struct vm_area_struct *area`

  VMA

**Description**

PCM mmap callback should handle this counter properly

- void `snd_pcm_limit_isa_dma_size`(int *dma*, size_t **max*)

  Get the max size fitting with ISA DMA transfer

**Parameters**

- `int dma`

  DMA number

- `size_t *max`

  pointer to store the max size

- const char * `snd_pcm_stream_str`(struct snd_pcm_substream **substream*)

  Get a string naming the direction of a stream

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Return**

A string naming the direction of the stream.

- struct snd_pcm_substream * `snd_pcm_chmap_substream`(struct snd_pcm_chmap **info*, unsigned int *idx*)

  get the PCM substream assigned to the given chmap info

**Parameters**

- `struct snd_pcm_chmap *info`

  chmap information

- `unsigned int idx`

  the substream number index

- u64 `pcm_format_to_bits`(snd_pcm_format_t *pcm_format*)

  Strong-typed conversion of pcm_format to bitwise

**Parameters**

- `snd_pcm_format_t pcm_format`

  PCM format

- `pcm_for_each_format`(*f*)

  helper to iterate for each format type

**Parameters**

- `f`

  the iterator variable in snd_pcm_format_t type

### PCM Format Helpers

- int `snd_pcm_format_signed`(snd_pcm_format_t *format*)

  Check the PCM format is signed linear

**Parameters**

- `snd_pcm_format_t format`

  the format to check

**Return**

如果给定的 PCM 格式是有符号线性的，则为 1，如果无符号线性，则为 0，对于非线性格式，则为负错误代码。

- int `snd_pcm_format_unsigned`( snd_pcm_format_t *格式*)

  检查 PCM 格式是否为无符号线性

**参数**

- `snd_pcm_format_t format`

  要检查的格式

**返回**

如果给定的 PCM 格式是无符号线性的，则为 1，如果有符号线性，则为 0，对于非线性格式，则为负错误代码。

- int `snd_pcm_format_linear`( snd_pcm_format_t *格式*)

  检查 PCM 格式是否为线性

**参数**

- `snd_pcm_format_t format`

  要检查的格式

**返回**

如果给定的 PCM 格式是线性的，则为 1，否则为 0。

- int `snd_pcm_format_little_endian`( snd_pcm_format_t *格式*)

  检查 PCM 格式是 little-endian

**参数**

- `snd_pcm_format_t format`

  要检查的格式

**返回**

如果给定的 PCM 格式是 little-endian，则为 1，如果 big-endian，则为 0，如果未指定 endian，则为负错误代码。

- int `snd_pcm_format_big_endian`( snd_pcm_format_t *格式*)

  检查 PCM 格式是否为大端

**参数**

- `snd_pcm_format_t format`

  要检查的格式

**返回**

如果给定的 PCM 格式是 big-endian，则为 1，如果为 little-endian，则为 0，如果未指定 endian，则为负错误代码。

- int `snd_pcm_format_width`( snd_pcm_format_t *格式*)

  返回格式的位宽

**参数**

- `snd_pcm_format_t format`

  要检查的格式

**返回**

格式的位宽，如果格式未知，则为负错误代码。

- int `snd_pcm_format_physical_width`( snd_pcm_format_t *格式*)

  返回格式的物理位宽

**参数**

- `snd_pcm_format_t format`

  要检查的格式

**返回**

格式的物理位宽，如果格式未知，则为负错误代码。

- ssize_t `snd_pcm_format_size`（ snd_pcm_format_t *格式*，size_t *样本*）

  返回给定格式的样本的字节大小

**参数**

- `snd_pcm_format_t format`

  要检查的格式

- `size_t samples`

  采样率

**返回**

格式的给定样本的字节大小，如果格式未知，则为负错误代码。

- const unsigned char * `snd_pcm_format_silence_64`( snd_pcm_format_t *格式*)

  return the silent data in 8 bytes array

**Parameters**

- `snd_pcm_format_t format`

  the format to check

**Return**

The format pattern to fill or `NULL` if error.

- int `snd_pcm_format_set_silence`(snd_pcm_format_t *format*, void **data*, unsigned int *samples*)

  set the silence data on the buffer

**Parameters**

- `snd_pcm_format_t format`

  the PCM format

- `void *data`

  the buffer pointer

- `unsigned int samples`

  the number of samples to set silence

**Description**

Sets the silence data on the buffer for the given samples.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_hw_limit_rates`(struct snd_pcm_hardware **hw*)

  determine rate_min/rate_max fields

**Parameters**

- `struct snd_pcm_hardware *hw`

  the pcm hw instance

**Description**

Determines the rate_min and rate_max fields from the rates bits of the given hw.

**Return**

Zero if successful.

- unsigned int `snd_pcm_rate_to_rate_bit`(unsigned int *rate*)

  converts sample rate to SNDRV_PCM_RATE_xxx bit

**Parameters**

- `unsigned int rate`

  the sample rate to convert

**Return**

The SNDRV_PCM_RATE_xxx flag that corresponds to the given rate, or SNDRV_PCM_RATE_KNOT for an unknown rate.

- unsigned int `snd_pcm_rate_bit_to_rate`(unsigned int *rate_bit*)

  converts SNDRV_PCM_RATE_xxx bit to sample rate

**Parameters**

- `unsigned int rate_bit`

  the rate bit to convert

**Return**

The sample rate that corresponds to the given SNDRV_PCM_RATE_xxx flag or 0 for an unknown rate bit.

- unsigned int `snd_pcm_rate_mask_intersect`(unsigned int *rates_a*, unsigned int *rates_b*)

  computes the intersection between two rate masks

**Parameters**

- `unsigned int rates_a`

  The first rate mask

- `unsigned int rates_b`

  The second rate mask

**Description**

This function computes the rates that are supported by both rate masks passed to the function. It will take care of the special handling of SNDRV_PCM_RATE_CONTINUOUS and SNDRV_PCM_RATE_KNOT.

**Return**

A rate mask containing the rates that are supported by both rates_a and rates_b.

- unsigned int `snd_pcm_rate_range_to_bits`(unsigned int *rate_min*, unsigned int *rate_max*)

  converts rate range to SNDRV_PCM_RATE_xxx bit

**Parameters**

- `unsigned int rate_min`

  the minimum sample rate

- `unsigned int rate_max`

  the maximum sample rate

**Description**

This function has an implicit assumption: the rates in the given range have only the pre-defined rates like 44100 or 16000.

**Return**

The SNDRV_PCM_RATE_xxx flag that corresponds to the given rate range, or SNDRV_PCM_RATE_KNOT for an unknown range.

### PCM Memory Management

- void `snd_pcm_lib_preallocate_free`(struct snd_pcm_substream **substream*)

  release the preallocated buffer of the specified substream.

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

**Description**

Releases the pre-allocated buffer of the given substream.

- void `snd_pcm_lib_preallocate_free_for_all`(struct snd_pcm **pcm*)

  release all pre-allocated buffers on the pcm

**Parameters**

- `struct snd_pcm *pcm`

  the pcm instance

**Description**

Releases all the pre-allocated buffers on the given pcm.

- void `snd_pcm_lib_preallocate_pages`(struct snd_pcm_substream **substream*, int *type*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **data*, size_t *size*, size_t *max*)

  pre-allocation for the given DMA type

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

- `int type`

  DMA type (SNDRV_DMA_TYPE_*)

- `struct device *data`

  DMA type dependent data

- `size_t size`

  the requested pre-allocation size in bytes

- `size_t max`

  the max. allowed pre-allocation size

**Description**

Do pre-allocation for the given DMA buffer type.

- void `snd_pcm_lib_preallocate_pages_for_all`(struct snd_pcm **pcm*, int *type*, void **data*, size_t *size*, size_t *max*)

  pre-allocation for continuous memory type (all substreams)

**Parameters**

- `struct snd_pcm *pcm`

  the pcm instance

- `int type`

  DMA type (SNDRV_DMA_TYPE_*)

- `void *data`

  DMA type dependent data

- `size_t size`

  the requested pre-allocation size in bytes

- `size_t max`

  the max. allowed pre-allocation size

**Description**

Do pre-allocation to all substreams of the given pcm for the specified DMA type.

- int `snd_pcm_set_managed_buffer`(struct snd_pcm_substream **substream*, int *type*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **data*, size_t *size*, size_t *max*)

  set up buffer management for a substream

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream instance

- `int type`

  DMA type (SNDRV_DMA_TYPE_*)

- `struct device *data`

  DMA type dependent data

- `size_t size`

  the requested pre-allocation size in bytes

- `size_t max`

  the max. allowed pre-allocation size

**Description**

Do pre-allocation for the given DMA buffer type, and set the managed buffer allocation mode to the given substream. In this mode, PCM core will allocate a buffer automatically before PCM hw_params ops call, and release the buffer after PCM hw_free ops call as well, so that the driver doesn’t need to invoke the allocation and the release explicitly in its callback. When a buffer is actually allocated before the PCM hw_params call, it turns on the runtime buffer_changed flag for drivers changing their h/w parameters accordingly.

When **size** is non-zero and **max** is zero, this tries to allocate for only the exact buffer size without fallback, and may return -ENOMEM. Otherwise, the function tries to allocate smaller chunks if the allocation fails. This is the behavior of [`snd_pcm_set_fixed_buffer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_set_fixed_buffer).

When both **size** and **max** are zero, the function only sets up the buffer for later dynamic allocations. It’s used typically for buffers with SNDRV_DMA_TYPE_VMALLOC type.

Upon successful buffer allocation and setup, the function returns 0.

- int `snd_pcm_set_managed_buffer_all`(struct snd_pcm **pcm*, int *type*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **data*, size_t *size*, size_t *max*)

  set up buffer management for all substreams for all substreams

**Parameters**

- `struct snd_pcm *pcm`

  the pcm instance

- `int type`

  DMA type (SNDRV_DMA_TYPE_*)

- `struct device *data`

  DMA type dependent data

- `size_t size`

  the requested pre-allocation size in bytes

- `size_t max`

  the max. allowed pre-allocation size

**Description**

Do pre-allocation to all substreams of the given pcm for the specified DMA type and size, and set the managed_buffer_alloc flag to each substream.

- int `snd_pcm_lib_malloc_pages`(struct snd_pcm_substream **substream*, size_t *size*)

  allocate the DMA buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  the substream to allocate the DMA buffer to

- `size_t size`

  the requested buffer size in bytes

**Description**

Allocates the DMA buffer on the BUS type given earlier to snd_pcm_lib_preallocate_xxx_pages().

**Return**

1 if the buffer is changed, 0 if not changed, or a negative code on failure.

- int `snd_pcm_lib_free_pages`(struct snd_pcm_substream **substream*)

  release the allocated DMA buffer.

**Parameters**

- `struct snd_pcm_substream *substream`

  the substream to release the DMA buffer

**Description**

Releases the DMA buffer allocated via [`snd_pcm_lib_malloc_pages()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_lib_malloc_pages).

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_pcm_lib_free_vmalloc_buffer`(struct snd_pcm_substream **substream*)

  free vmalloc buffer

**Parameters**

- `struct snd_pcm_substream *substream`

  the substream with a buffer allocated by [`snd_pcm_lib_alloc_vmalloc_buffer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_lib_alloc_vmalloc_buffer)

**Return**

Zero if successful, or a negative error code on failure.

- struct page * `snd_pcm_lib_get_vmalloc_page`(struct snd_pcm_substream **substream*, unsigned long *offset*)

  map vmalloc buffer offset to page struct

**Parameters**

- `struct snd_pcm_substream *substream`

  the substream with a buffer allocated by [`snd_pcm_lib_alloc_vmalloc_buffer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_pcm_lib_alloc_vmalloc_buffer)

- `unsigned long offset`

  offset in the buffer

**Description**

This function is to be used as the page callback in the PCM ops.

**Return**

The page struct, or `NULL` on failure.

### PCM DMA Engine API

- int `snd_hwparams_to_dma_slave_config`(const struct snd_pcm_substream **substream*, const struct snd_pcm_hw_params **params*, struct dma_slave_config **slave_config*)

  Convert hw_params to dma_slave_config

**Parameters**

- `const struct snd_pcm_substream *substream`

  PCM substream

- `const struct snd_pcm_hw_params *params`

  hw_params

- `struct dma_slave_config *slave_config`

  DMA slave config

**Description**

This function can be used to initialize a dma_slave_config from a substream and hw_params in a dmaengine based PCM driver implementation.

- void `snd_dmaengine_pcm_set_config_from_dai_data`(const struct snd_pcm_substream **substream*, const struct [snd_dmaengine_dai_dma_data](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_dmaengine_dai_dma_data) **dma_data*, struct dma_slave_config **slave_config*)

  Initializes a dma slave config using DAI DMA data.

**Parameters**

- `const struct snd_pcm_substream *substream`

  PCM substream

- `const struct snd_dmaengine_dai_dma_data *dma_data`

  DAI DMA data

- `struct dma_slave_config *slave_config`

  DMA slave configuration

**Description**

Initializes the {dst,src}_addr, {dst,src}_maxburst, {dst,src}_addr_width fields of the DMA slave config from the same fields of the DAI DMA data struct. The src and dst fields will be initialized depending on the direction of the substream. If the substream is a playback stream the dst fields will be initialized, if it is a capture stream the src fields will be initialized. The {dst,src}_addr_width field will only be initialized if the SND_DMAENGINE_PCM_DAI_FLAG_PACK flag is set or if the addr_width field of the DAI DMA data struct is not equal to DMA_SLAVE_BUSWIDTH_UNDEFINED. If both conditions are met the latter takes priority.

- int `snd_dmaengine_pcm_trigger`(struct snd_pcm_substream **substream*, int *cmd*)

  dmaengine based PCM trigger implementation

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `int cmd`

  Trigger command

**Description**

Returns 0 on success, a negative error code otherwise.

This function can be used as the PCM trigger callback for dmaengine based PCM driver implementations.

- snd_pcm_uframes_t `snd_dmaengine_pcm_pointer_no_residue`(struct snd_pcm_substream **substream*)

  dmaengine based PCM pointer implementation

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

This function is deprecated and should not be used by new drivers, as its results may be unreliable.

- snd_pcm_uframes_t `snd_dmaengine_pcm_pointer`(struct snd_pcm_substream **substream*)

  dmaengine based PCM pointer implementation

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

This function can be used as the PCM pointer callback for dmaengine based PCM driver implementations.

- struct dma_chan * `snd_dmaengine_pcm_request_channel`(dma_filter_fn *filter_fn*, void **filter_data*)

  Request channel for the dmaengine PCM

**Parameters**

- `dma_filter_fn filter_fn`

  Filter function used to request the DMA channel

- `void *filter_data`

  Data passed to the DMA filter function

**Description**

Returns NULL or the requested DMA channel.

This function request a DMA channel for usage with dmaengine PCM.

- int `snd_dmaengine_pcm_open`(struct snd_pcm_substream **substream*, struct dma_chan **chan*)

  Open a dmaengine based PCM substream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `struct dma_chan *chan`

  DMA channel to use for data transfers

**Description**

Returns 0 on success, a negative error code otherwise.

The function should usually be called from the pcm open callback. Note that this function will use private_data field of the substream’s runtime. So it is not available to your pcm driver implementation.

- int `snd_dmaengine_pcm_open_request_chan`(struct snd_pcm_substream **substream*, dma_filter_fn *filter_fn*, void **filter_data*)

  Open a dmaengine based PCM substream and request channel

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `dma_filter_fn filter_fn`

  Filter function used to request the DMA channel

- `void *filter_data`

  Data passed to the DMA filter function

**Description**

Returns 0 on success, a negative error code otherwise.

This function will request a DMA channel using the passed filter function and data. The function should usually be called from the pcm open callback. Note that this function will use private_data field of the substream’s runtime. So it is not available to your pcm driver implementation.

- int `snd_dmaengine_pcm_close`(struct snd_pcm_substream **substream*)

  Close a dmaengine based PCM substream

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- int `snd_dmaengine_pcm_close_release_chan`(struct snd_pcm_substream **substream*)

  Close a dmaengine based PCM substream and release channel

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

**Description**

Releases the DMA channel associated with the PCM substream.

- int `snd_dmaengine_pcm_refine_runtime_hwparams`(struct snd_pcm_substream **substream*, struct [snd_dmaengine_dai_dma_data](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_dmaengine_dai_dma_data) **dma_data*, struct snd_pcm_hardware **hw*, struct dma_chan **chan*)

  Refine runtime hw params

**Parameters**

- `struct snd_pcm_substream *substream`

  PCM substream

- `struct snd_dmaengine_dai_dma_data *dma_data`

  DAI DMA data

- `struct snd_pcm_hardware *hw`

  PCM hw params

- `struct dma_chan *chan`

  DMA channel to use for data transfers

**Description**

Returns 0 on success, a negative error code otherwise.

This function will query DMA capability, then refine the pcm hardware parameters.

- enum dma_transfer_direction `snd_pcm_substream_to_dma_direction`(const struct snd_pcm_substream **substream*)

  Get dma_transfer_direction for a PCM substream

**Parameters**

- `const struct snd_pcm_substream *substream`

  PCM substream

- struct `snd_dmaengine_dai_dma_data`

  DAI DMA configuration data

**Definition**

```
struct snd_dmaengine_dai_dma_data {
  dma_addr_t addr;
  enum dma_slave_buswidth addr_width;
  u32 maxburst;
  void *filter_data;
  const char *chan_name;
  unsigned int fifo_size;
  unsigned int flags;
  void *peripheral_config;
  size_t peripheral_size;
};
```

**Members**

- `addr`

  Address of the DAI data source or destination register.

- `addr_width`

  Width of the DAI data source or destination register.

- `maxburst`

  Maximum number of words(note: words, as in units of the src_addr_width member, not bytes) that can be send to or received from the DAI in one burst.

- `filter_data`

  Custom DMA channel filter data, this will usually be used when requesting the DMA channel.

- `chan_name`

  Custom channel name to use when requesting DMA channel.

- `fifo_size`

  FIFO size of the DAI controller in bytes

- `flags`

  PCM_DAI flags, only SND_DMAENGINE_PCM_DAI_FLAG_PACK for now

- `peripheral_config`

  peripheral configuration for programming peripheral for dmaengine transfer

- `peripheral_size`

  peripheral configuration buffer size

- struct `snd_dmaengine_pcm_config`

  Configuration data for dmaengine based PCM

**Definition**

```
struct snd_dmaengine_pcm_config {
  int (*prepare_slave_config)(struct snd_pcm_substream *substream,struct snd_pcm_hw_params *params, struct dma_slave_config *slave_config);
  struct dma_chan *(*compat_request_channel)(struct snd_soc_pcm_runtime *rtd, struct snd_pcm_substream *substream);
  int (*process)(struct snd_pcm_substream *substream,int channel, unsigned long hwoff, void *buf, unsigned long bytes);
  dma_filter_fn compat_filter_fn;
  struct device *dma_dev;
  const char *chan_names[SNDRV_PCM_STREAM_LAST + 1];
  const struct snd_pcm_hardware *pcm_hardware;
  unsigned int prealloc_buffer_size;
};
```

**Members**

- `prepare_slave_config`

  Callback used to fill in the DMA slave_config for a PCM substream. Will be called from the PCM drivers hwparams callback.

- `compat_request_channel`

  Callback to request a DMA channel for platforms which do not use devicetree.

- `process`

  Callback used to apply processing on samples transferred from/to user space.

- `compat_filter_fn`

  Will be used as the filter function when requesting a channel for platforms which do not use devicetree. The filter parameter will be the DAI’s DMA data.

- `dma_dev`

  If set, request DMA channel on this device rather than the DAI device.

- `chan_names`

  If set, these custom DMA channel names will be requested at registration time.

- `pcm_hardware`

  snd_pcm_hardware struct to be used for the PCM.

- `prealloc_buffer_size`

  Size of the preallocated audio buffer.

**Note**

If both compat_request_channel and compat_filter_fn are set compat_request_channel will be used to request the channel and compat_filter_fn will be ignored. Otherwise the channel will be requested using dma_request_channel with compat_filter_fn as the filter function.

## Control/Mixer API

### General Control Interface

- void `snd_ctl_notify`(struct snd_card **card*, unsigned int *mask*, struct snd_ctl_elem_id **id*)

  Send notification to user-space for a control change

**Parameters**

- `struct snd_card *card`

  the card to send notification

- `unsigned int mask`

  the event mask, SNDRV_CTL_EVENT_*

- `struct snd_ctl_elem_id *id`

  the ctl element id to send notification

**Description**

This function adds an event record with the given id and mask, appends to the list and wakes up the user-space for notification. This can be called in the atomic context.

- void `snd_ctl_notify_one`(struct snd_card **card*, unsigned int *mask*, struct snd_kcontrol **kctl*, unsigned int *ioff*)

  Send notification to user-space for a control change

**Parameters**

- `struct snd_card *card`

  the card to send notification

- `unsigned int mask`

  the event mask, SNDRV_CTL_EVENT_*

- `struct snd_kcontrol *kctl`

  the pointer with the control instance

- `unsigned int ioff`

  the additional offset to the control index

**Description**

This function calls [`snd_ctl_notify()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_notify) and does additional jobs like LED state changes.

- int `snd_ctl_new`(struct snd_kcontrol ***kctl*, unsigned int *count*, unsigned int *access*, struct snd_ctl_file **file*)

  create a new control instance with some elements

**Parameters**

- `struct snd_kcontrol **kctl`

  the pointer to store new control instance

- `unsigned int count`

  the number of elements in this control

- `unsigned int access`

  the default access flags for elements in this control

- `struct snd_ctl_file *file`

  given when locking these elements

**Description**

Allocates a memory object for a new control instance. The instance has elements as many as the given number (**count**). Each element has given access permissions (**access**). Each element is locked when **file** is given.

**Return**

0 on success, error code on failure

- struct snd_kcontrol * `snd_ctl_new1`(const struct snd_kcontrol_new **ncontrol*, void **private_data*)

  create a control instance from the template

**Parameters**

- `const struct snd_kcontrol_new *ncontrol`

  the initialization record

- `void *private_data`

  the private data to set

**Description**

Allocates a new struct snd_kcontrol instance and initialize from the given template. When the access field of ncontrol is 0, it’s assumed as READWRITE access. When the count field is 0, it’s assumes as one.

**Return**

The pointer of the newly generated instance, or `NULL` on failure.

- void `snd_ctl_free_one`(struct snd_kcontrol **kcontrol*)

  release the control instance

**Parameters**

- `struct snd_kcontrol *kcontrol`

  the control instance

**Description**

Releases the control instance created via [`snd_ctl_new()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_new) or [`snd_ctl_new1()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_new1). Don’t call this after the control was added to the card.

- int `snd_ctl_add`(struct snd_card **card*, struct snd_kcontrol **kcontrol*)

  add the control instance to the card

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_kcontrol *kcontrol`

  the control instance to add

**Description**

Adds the control instance created via [`snd_ctl_new()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_new) or [`snd_ctl_new1()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_new1) to the given card. Assigns also an unique numid used for fast search.

It frees automatically the control which cannot be added.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ctl_replace`(struct snd_card **card*, struct snd_kcontrol **kcontrol*, bool *add_on_replace*)

  replace the control instance of the card

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_kcontrol *kcontrol`

  the control instance to replace

- `bool add_on_replace`

  add the control if not already added

**Description**

Replaces the given control. If the given control does not exist and the add_on_replace flag is set, the control is added. If the control exists, it is destroyed first.

It frees automatically the control which cannot be added or replaced.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ctl_remove`(struct snd_card **card*, struct snd_kcontrol **kcontrol*)

  remove the control from the card and release it

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_kcontrol *kcontrol`

  the control instance to remove

**Description**

Removes the control from the card and then releases the instance. You don’t need to call [`snd_ctl_free_one()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_free_one). You must be in the write lock - down_write(`card->controls_rwsem`).

**Return**

0 if successful, or a negative error code on failure.

- int `snd_ctl_remove_id`(struct snd_card **card*, struct snd_ctl_elem_id **id*)

  remove the control of the given id and release it

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_ctl_elem_id *id`

  the control id to remove

**Description**

Finds the control instance with the given id, removes it from the card list and releases it.

**Return**

0 if successful, or a negative error code on failure.

- int `snd_ctl_remove_user_ctl`(struct snd_ctl_file * *file*, struct snd_ctl_elem_id **id*)

  remove and release the unlocked user control

**Parameters**

- `struct snd_ctl_file * file`

  active control handle

- `struct snd_ctl_elem_id *id`

  the control id to remove

**Description**

Finds the control instance with the given id, removes it from the card list and releases it.

**Return**

0 if successful, or a negative error code on failure.

- int `snd_ctl_activate_id`(struct snd_card **card*, struct snd_ctl_elem_id **id*, int *active*)

  activate/inactivate the control of the given id

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_ctl_elem_id *id`

  the control id to activate/inactivate

- `int active`

  non-zero to activate

**Description**

Finds the control instance with the given id, and activate or inactivate the control together with notification, if changed. The given ID data is filled with full information.

**Return**

0 if unchanged, 1 if changed, or a negative error code on failure.

- int `snd_ctl_rename_id`(struct snd_card **card*, struct snd_ctl_elem_id **src_id*, struct snd_ctl_elem_id **dst_id*)

  replace the id of a control on the card

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_ctl_elem_id *src_id`

  the old id

- `struct snd_ctl_elem_id *dst_id`

  the new id

**Description**

Finds the control with the old id from the card, and replaces the id with the new one.

**Return**

Zero if successful, or a negative error code on failure.

- struct snd_kcontrol * `snd_ctl_find_numid`(struct snd_card **card*, unsigned int *numid*)

  find the control instance with the given number-id

**Parameters**

- `struct snd_card *card`

  the card instance

- `unsigned int numid`

  the number-id to search

**Description**

Finds the control instance with the given number-id from the card.

The caller must down card->controls_rwsem before calling this function (if the race condition can happen).

**Return**

The pointer of the instance if found, or `NULL` if not.

- struct snd_kcontrol * `snd_ctl_find_id`(struct snd_card **card*, struct snd_ctl_elem_id **id*)

  find the control instance with the given id

**Parameters**

- `struct snd_card *card`

  the card instance

- `struct snd_ctl_elem_id *id`

  the id to search

**Description**

Finds the control instance with the given id from the card.

The caller must down card->controls_rwsem before calling this function (if the race condition can happen).

**Return**

The pointer of the instance if found, or `NULL` if not.

- int `snd_ctl_register_ioctl`(snd_kctl_ioctl_func_t *fcn*)

  register the device-specific control-ioctls

**Parameters**

- `snd_kctl_ioctl_func_t fcn`

  ioctl callback function

**Description**

called from each device manager like pcm.c, hwdep.c, etc.

- int `snd_ctl_register_ioctl_compat`(snd_kctl_ioctl_func_t *fcn*)

  register the device-specific 32bit compat control-ioctls

**Parameters**

- `snd_kctl_ioctl_func_t fcn`

  ioctl callback function

- int `snd_ctl_unregister_ioctl`(snd_kctl_ioctl_func_t *fcn*)

  de-register the device-specific control-ioctls

**Parameters**

- `snd_kctl_ioctl_func_t fcn`

  ioctl callback function to unregister

- int `snd_ctl_unregister_ioctl_compat`(snd_kctl_ioctl_func_t *fcn*)

  de-register the device-specific compat 32bit control-ioctls

**Parameters**

- `snd_kctl_ioctl_func_t fcn`

  ioctl callback function to unregister

- int `snd_ctl_request_layer`(const char **module_name*)

  request to use the layer

**Parameters**

- `const char *module_name`

  Name of the kernel module (NULL == build-in)

**Description**

Return an error code when the module cannot be loaded.

- void `snd_ctl_register_layer`(struct snd_ctl_layer_ops **lops*)

  register new control layer

**Parameters**

- `struct snd_ctl_layer_ops *lops`

  operation structure

**Description**

The new layer can track all control elements and do additional operations on top (like audio LED handling).

- void `snd_ctl_disconnect_layer`(struct snd_ctl_layer_ops **lops*)

  disconnect control layer

**Parameters**

- `struct snd_ctl_layer_ops *lops`

  operation structure

**Description**

It is expected that the information about tracked cards is freed before this call (the disconnect callback is not called here).

- int `snd_ctl_boolean_mono_info`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  Helper function for a standard boolean info callback with a mono channel

**Parameters**

- `struct snd_kcontrol *kcontrol`

  the kcontrol instance

- `struct snd_ctl_elem_info *uinfo`

  info to store

**Description**

This is a function that can be used as info callback for a standard boolean control with a single mono channel.

- int `snd_ctl_boolean_stereo_info`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  Helper function for a standard boolean info callback with stereo two channels

**Parameters**

- `struct snd_kcontrol *kcontrol`

  the kcontrol instance

- `struct snd_ctl_elem_info *uinfo`

  info to store

**Description**

This is a function that can be used as info callback for a standard boolean control with stereo two channels.

- int `snd_ctl_enum_info`(struct snd_ctl_elem_info **info*, unsigned int *channels*, unsigned int *items*, const char *const *names[]*)

  fills the info structure for an enumerated control

**Parameters**

- `struct snd_ctl_elem_info *info`

  the structure to be filled

- `unsigned int channels`

  the number of the control’s channels; often one

- `unsigned int items`

  the number of control values; also the size of **names**

- `const char *const names[]`

  an array containing the names of all control values

**Description**

Sets all required fields in **info** to their appropriate values. If the control’s accessibility is not the default (readable and writable), the caller has to fill **info->access**.

**Return**

Zero.

### AC97 Codec API

- void `snd_ac97_write`(struct snd_ac97 **ac97*, unsigned short *reg*, unsigned short *value*)

  write a value on the given register

**Parameters**

- `struct snd_ac97 *ac97`

  the ac97 instance

- `unsigned short reg`

  the register to change

- `unsigned short value`

  the value to set

**Description**

Writes a value on the given register. This will invoke the write callback directly after the register check. This function doesn’t change the register cache unlike #snd_ca97_write_cache(), so use this only when you don’t want to reflect the change to the suspend/resume state.

- unsigned short `snd_ac97_read`(struct snd_ac97 **ac97*, unsigned short *reg*)

  read a value from the given register

**Parameters**

- `struct snd_ac97 *ac97`

  the ac97 instance

- `unsigned short reg`

  the register to read

**Description**

Reads a value from the given register. This will invoke the read callback directly after the register check.

**Return**

The read value.

- void `snd_ac97_write_cache`(struct snd_ac97 **ac97*, unsigned short *reg*, unsigned short *value*)

  write a value on the given register and update the cache

**Parameters**

- `struct snd_ac97 *ac97`

  the ac97 instance

- `unsigned short reg`

  the register to change

- `unsigned short value`

  the value to set

**Description**

Writes a value on the given register and updates the register cache. The cached values are used for the cached-read and the suspend/resume.

- int `snd_ac97_update`(struct snd_ac97 **ac97*, unsigned short *reg*, unsigned short *value*)

  update the value on the given register

**Parameters**

- `struct snd_ac97 *ac97`

  the ac97 instance

- `unsigned short reg`

  the register to change

- `unsigned short value`

  the value to set

**Description**

Compares the value with the register cache and updates the value only when the value is changed.

**Return**

1 if the value is changed, 0 if no change, or a negative code on failure.

- int `snd_ac97_update_bits`(struct snd_ac97 **ac97*, unsigned short *reg*, unsigned short *mask*, unsigned short *value*)

  update the bits on the given register

**Parameters**

- `struct snd_ac97 *ac97`

  the ac97 instance

- `unsigned short reg`

  the register to change

- `unsigned short mask`

  the bit-mask to change

- `unsigned short value`

  the value to set

**Description**

Updates the masked-bits on the given register only when the value is changed.

**Return**

1 if the bits are changed, 0 if no change, or a negative code on failure.

- const char * `snd_ac97_get_short_name`(struct snd_ac97 **ac97*)

  retrieve codec name

**Parameters**

- `struct snd_ac97 *ac97`

  the codec instance

**Return**

The short identifying name of the codec.

- int `snd_ac97_bus`(struct snd_card **card*, int *num*, const struct snd_ac97_bus_ops **ops*, void **private_data*, struct [snd_ac97_bus](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ac97_bus) ***rbus*)

  create an AC97 bus component

**Parameters**

- `struct snd_card *card`

  the card instance

- `int num`

  the bus number

- `const struct snd_ac97_bus_ops *ops`

  the bus callbacks table

- `void *private_data`

  private data pointer for the new instance

- `struct snd_ac97_bus **rbus`

  the pointer to store the new AC97 bus instance.

**Description**

Creates an AC97 bus component. An [`struct snd_ac97_bus`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ac97_bus) instance is newly allocated and initialized.

The ops table must include valid callbacks (at least read and write). The other callbacks, wait and reset, are not mandatory.

The clock is set to 48000. If another clock is needed, set `(*rbus)->clock` manually.

The AC97 bus instance is registered as a low-level device, so you don’t have to release it manually.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ac97_mixer`(struct [snd_ac97_bus](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ac97_bus) **bus*, struct snd_ac97_template **template*, struct snd_ac97 ***rac97*)

  create an Codec97 component

**Parameters**

- `struct snd_ac97_bus *bus`

  the AC97 bus which codec is attached to

- `struct snd_ac97_template *template`

  the template of ac97, including index, callbacks and the private data.

- `struct snd_ac97 **rac97`

  the pointer to store the new ac97 instance.

**Description**

Creates an Codec97 component. An struct snd_ac97 instance is newly allocated and initialized from the template. The codec is then initialized by the standard procedure.

The template must include the codec number (num) and address (addr), and the private data (private_data).

The ac97 instance is registered as a low-level device, so you don’t have to release it manually.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ac97_update_power`(struct snd_ac97 **ac97*, int *reg*, int *powerup*)

  update the powerdown register

**Parameters**

- `struct snd_ac97 *ac97`

  the codec instance

- `int reg`

  the rate register, e.g. AC97_PCM_FRONT_DAC_RATE

- `int powerup`

  non-zero when power up the part

**Description**

Update the AC97 powerdown register bits of the given part.

**Return**

Zero.

- 无效 `snd_ac97_suspend`（结构 snd_ac97 **ac97* ）

  AC97 编解码器的通用挂起功能

**参数**

- `struct snd_ac97 *ac97`

  ac97 实例

**描述**

暂停编解码器，关闭芯片。

- 无效 `snd_ac97_resume`（结构 snd_ac97 **ac97* ）

  AC97 编解码器的通用恢复功能

**参数**

- `struct snd_ac97 *ac97`

  ac97 实例

**描述**

执行标准恢复程序，上电并恢复旧的寄存器值。

- int `snd_ac97_tune_hardware`( struct snd_ac97 **ac97* , const struct ac97_quirk *** quirk , const char **override* )

  调整硬件

**参数**

- `struct snd_ac97 *ac97`

  ac97 实例

- `const struct ac97_quirk *quirk`

  怪癖清单

- `const char *override`

  显式的怪癖值（如果非 NULL，则覆盖列表）

**描述**

对每个 pci 设备执行一些解决方法，例如将耳机（真正的线路输出）控件重命名为“Master”。quirk-list 必须以一个零填充条目终止。

**返回**

成功则为零，失败则为负错误代码。

- int `snd_ac97_set_rate`( struct snd_ac97 **ac97* , int *reg* , unsigned int *rate* )

  改变给定输入/输出的速率。

**参数**

- `struct snd_ac97 *ac97`

  ac97 实例

- `int reg`

  要更改的寄存器

- `unsigned int rate`

  要设置的采样率

**描述**

更改编解码器上给定输入/输出的速率。如果编解码器不支持 VAR，则速率必须为 48000（SPDIF 除外）。

The valid registers are AC97_PCM_MIC_ADC_RATE, AC97_PCM_FRONT_DAC_RATE, AC97_PCM_LR_ADC_RATE. AC97_PCM_SURR_DAC_RATE and AC97_PCM_LFE_DAC_RATE are accepted if the codec supports them. AC97_SPDIF is accepted as a pseudo register to modify the SPDIF status bits.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ac97_pcm_assign`(struct [snd_ac97_bus](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ac97_bus) **bus*, unsigned short *pcms_count*, const struct ac97_pcm **pcms*)

  assign AC97 slots to given PCM streams

**Parameters**

- `struct snd_ac97_bus *bus`

  the ac97 bus instance

- `unsigned short pcms_count`

  count of PCMs to be assigned

- `const struct ac97_pcm *pcms`

  PCMs to be assigned

**Description**

It assigns available AC97 slots for given PCMs. If none or only some slots are available, pcm->xxx.slots and pcm->xxx.rslots[] members are reduced and might be zero.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ac97_pcm_open`(struct ac97_pcm **pcm*, unsigned int *rate*, enum ac97_pcm_cfg *cfg*, unsigned short *slots*)

  opens the given AC97 pcm

**Parameters**

- `struct ac97_pcm *pcm`

  the ac97 pcm instance

- `unsigned int rate`

  rate in Hz, if codec does not support VRA, this value must be 48000Hz

- `enum ac97_pcm_cfg cfg`

  output stream characteristics

- `unsigned short slots`

  a subset of allocated slots (snd_ac97_pcm_assign) for this pcm

**Description**

It locks the specified slots and sets the given rate to AC97 registers.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_ac97_pcm_close`(struct ac97_pcm **pcm*)

  closes the given AC97 pcm

**Parameters**

- `struct ac97_pcm *pcm`

  the ac97 pcm instance

**Description**

It frees the locked AC97 slots.

**Return**

Zero.

- int `snd_ac97_pcm_double_rate_rules`(struct snd_pcm_runtime **runtime*)

  set double rate constraints

**Parameters**

- `struct snd_pcm_runtime *runtime`

  the runtime of the ac97 front playback pcm

**Description**

Installs the hardware constraint rules to prevent using double rates and more than two channels at the same time.

**Return**

Zero if successful, or a negative error code on failure.

### Virtual Master Control API

- struct snd_kcontrol * `snd_ctl_make_virtual_master`(char **name*, const unsigned int **tlv*)

  Create a virtual master control

**Parameters**

- `char *name`

  name string of the control element to create

- `const unsigned int *tlv`

  optional TLV int array for dB information

**Description**

Creates a virtual master control with the given name string.

After creating a vmaster element, you can add the follower controls via [`snd_ctl_add_follower()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_add_follower) or [`snd_ctl_add_follower_uncached()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_add_follower_uncached).

The optional argument **tlv** can be used to specify the TLV information for dB scale of the master control. It should be a single element with #SNDRV_CTL_TLVT_DB_SCALE, #SNDRV_CTL_TLV_DB_MINMAX or #SNDRV_CTL_TLVT_DB_MINMAX_MUTE type, and should be the max 0dB.

**Return**

The created control element, or `NULL` for errors (ENOMEM).

- int `snd_ctl_add_vmaster_hook`(struct snd_kcontrol **kcontrol*, void ( **hook*)(void *private_data, int), void **private_data*)

  Add a hook to a vmaster control

**Parameters**

- `struct snd_kcontrol *kcontrol`

  vmaster kctl element

- `void (*hook)(void *private_data, int)`

  the hook function

- `void *private_data`

  the private_data pointer to be saved

**Description**

Adds the given hook to the vmaster control element so that it’s called at each time when the value is changed.

**Return**

Zero.

- void `snd_ctl_sync_vmaster`(struct snd_kcontrol **kcontrol*, bool *hook_only*)

  Sync the vmaster followers and hook

**Parameters**

- `struct snd_kcontrol *kcontrol`

  vmaster kctl element

- `bool hook_only`

  sync only the hook

**Description**

Forcibly call the put callback of each follower and call the hook function to synchronize with the current value of the given vmaster element. NOP when NULL is passed to **kcontrol**.

- int `snd_ctl_apply_vmaster_followers`(struct snd_kcontrol **kctl*, int ( **func*)(struct snd_kcontrol *vfollower, struct snd_kcontrol *follower, void *arg), void **arg*)

  Apply function to each vmaster follower

**Parameters**

- `struct snd_kcontrol *kctl`

  vmaster kctl element

- `int (*func)(struct snd_kcontrol *vfollower, struct snd_kcontrol *follower, void *arg)`

  function to apply

- `void *arg`

  optional function argument

**Description**

Apply the function **func** to each follower kctl of the given vmaster kctl. Returns 0 if successful, or a negative error code.

- int `snd_ctl_add_follower`(struct snd_kcontrol **master*, struct snd_kcontrol **follower*)

  Add a virtual follower control

**Parameters**

- `struct snd_kcontrol *master`

  vmaster element

- `struct snd_kcontrol *follower`

  follower element to add

**Description**

Add a virtual follower control to the given master element created via snd_ctl_create_virtual_master() beforehand.

All followers must be the same type (returning the same information via info callback). The function doesn’t check it, so it’s your responsibility.

Also, some additional limitations: at most two channels, logarithmic volume control (dB level) thus no linear volume, master can only attenuate the volume without gain

**Return**

Zero if successful or a negative error code.

- int `snd_ctl_add_follower_uncached`(struct snd_kcontrol **master*, struct snd_kcontrol **follower*)

  Add a virtual follower control

**Parameters**

- `struct snd_kcontrol *master`

  vmaster element

- `struct snd_kcontrol *follower`

  follower element to add

**Description**

Add a virtual follower control to the given master. Unlike [`snd_ctl_add_follower()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_ctl_add_follower), the element added via this function is supposed to have volatile values, and get callback is called at each time queried from the master.

When the control peeks the hardware values directly and the value can be changed by other means than the put callback of the element, this function should be used to keep the value always up-to-date.

**Return**

Zero if successful or a negative error code.

## MIDI API

### Raw MIDI API

- int `snd_rawmidi_receive`(struct snd_rawmidi_substream **substream*, const unsigned char **buffer*, int *count*)

  receive the input data from the device

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

- `const unsigned char *buffer`

  the buffer pointer

- `int count`

  the data size to read

**Description**

Reads the data from the internal buffer.

**Return**

The size of read data, or a negative error code on failure.

- int `snd_rawmidi_transmit_empty`(struct snd_rawmidi_substream **substream*)

  check whether the output buffer is empty

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

**Return**

1 if the internal output buffer is empty, 0 if not.

- int `__snd_rawmidi_transmit_peek`(struct snd_rawmidi_substream **substream*, unsigned char **buffer*, int *count*)

  copy data from the internal buffer

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

- `unsigned char *buffer`

  the buffer pointer

- `int count`

  data size to transfer

**Description**

This is a variant of [`snd_rawmidi_transmit_peek()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_rawmidi_transmit_peek) without spinlock.

- int `snd_rawmidi_transmit_peek`(struct snd_rawmidi_substream **substream*, unsigned char **buffer*, int *count*)

  copy data from the internal buffer

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

- `unsigned char *buffer`

  the buffer pointer

- `int count`

  data size to transfer

**Description**

Copies data from the internal output buffer to the given buffer.

Call this in the interrupt handler when the midi output is ready, and call [`snd_rawmidi_transmit_ack()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_rawmidi_transmit_ack) after the transmission is finished.

**Return**

The size of copied data, or a negative error code on failure.

- int `__snd_rawmidi_transmit_ack`(struct snd_rawmidi_substream **substream*, int *count*)

  acknowledge the transmission

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

- `int count`

  the transferred count

**Description**

This is a variant of [`__snd_rawmidi_transmit_ack()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.__snd_rawmidi_transmit_ack) without spinlock.

- int `snd_rawmidi_transmit_ack`(struct snd_rawmidi_substream **substream*, int *count*)

  acknowledge the transmission

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

- `int count`

  the transferred count

**Description**

Advances the hardware pointer for the internal output buffer with the given size and updates the condition. Call after the transmission is finished.

**Return**

The advanced size if successful, or a negative error code on failure.

- int `snd_rawmidi_transmit`(struct snd_rawmidi_substream **substream*, unsigned char **buffer*, int *count*)

  copy from the buffer to the device

**Parameters**

- `struct snd_rawmidi_substream *substream`

  the rawmidi substream

- `unsigned char *buffer`

  the buffer pointer

- `int count`

  the data size to transfer

**Description**

Copies data from the buffer to the device and advances the pointer.

**Return**

The copied size if successful, or a negative error code on failure.

- int `snd_rawmidi_proceed`(struct snd_rawmidi_substream **substream*)

  Discard the all pending bytes and proceed

**Parameters**

- `struct snd_rawmidi_substream *substream`

  rawmidi substream

**Return**

the number of discarded bytes

- int `snd_rawmidi_new`(struct snd_card **card*, char **id*, int *device*, int *output_count*, int *input_count*, struct snd_rawmidi ***rrawmidi*)

  create a rawmidi instance

**Parameters**

- `struct snd_card *card`

  the card instance

- `char *id`

  the id string

- `int device`

  the device index

- `int output_count`

  the number of output streams

- `int input_count`

  the number of input streams

- `struct snd_rawmidi **rrawmidi`

  the pointer to store the new rawmidi instance

**Description**

Creates a new rawmidi instance. Use [`snd_rawmidi_set_ops()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_rawmidi_set_ops) to set the operators to the new instance.

**Return**

Zero if successful, or a negative error code on failure.

- void `snd_rawmidi_set_ops`(struct snd_rawmidi **rmidi*, int *stream*, const struct snd_rawmidi_ops **ops*)

  set the rawmidi operators

**Parameters**

- `struct snd_rawmidi *rmidi`

  the rawmidi instance

- `int stream`

  the stream direction, SNDRV_RAWMIDI_STREAM_XXX

- `const struct snd_rawmidi_ops *ops`

  the operator table

**Description**

Sets the rawmidi operators for the given stream direction.

### MPU401-UART API

- irqreturn_t `snd_mpu401_uart_interrupt`(int *irq*, void **dev_id*)

  generic MPU401-UART interrupt handler

**Parameters**

- `int irq`

  the irq number

- `void *dev_id`

  mpu401 instance

**Description**

Processes the interrupt for MPU401-UART i/o.

**Return**

`IRQ_HANDLED` if the interrupt was handled. `IRQ_NONE` otherwise.

- irqreturn_t `snd_mpu401_uart_interrupt_tx`(int *irq*, void **dev_id*)

  generic MPU401-UART transmit irq handler

**Parameters**

- `int irq`

  the irq number

- `void *dev_id`

  mpu401 instance

**Description**

Processes the interrupt for MPU401-UART output.

**Return**

`IRQ_HANDLED` if the interrupt was handled. `IRQ_NONE` otherwise.

- int `snd_mpu401_uart_new`(struct snd_card **card*, int *device*, unsigned short *hardware*, unsigned long *port*, unsigned int *info_flags*, int *irq*, struct snd_rawmidi ** *rrawmidi*)

  create an MPU401-UART instance

**Parameters**

- `struct snd_card *card`

  the card instance

- `int device`

  the device index, zero-based

- `unsigned short hardware`

  the hardware type, MPU401_HW_XXXX

- `unsigned long port`

  the base address of MPU401 port

- `unsigned int info_flags`

  bitflags MPU401_INFO_XXX

- `int irq`

  the ISA irq number, -1 if not to be allocated

- `struct snd_rawmidi ** rrawmidi`

  the pointer to store the new rawmidi instance

**Description**

Creates a new MPU-401 instance.

Note that the rawmidi instance is returned on the rrawmidi argument, not the mpu401 instance itself. To access to the mpu401 instance, cast from rawmidi->private_data (with struct snd_mpu401 magic-cast).

**Return**

Zero if successful, or a negative error code.

## Proc Info API

### Proc Info Interface

- int `snd_info_get_line`(struct snd_info_buffer **buffer*, char **line*, int *len*)

  read one line from the procfs buffer

**Parameters**

- `struct snd_info_buffer *buffer`

  the procfs buffer

- `char *line`

  the buffer to store

- `int len`

  the max. buffer size

**Description**

Reads one line from the buffer and stores the string.

**Return**

Zero if successful, or 1 if error or EOF.

- const char * `snd_info_get_str`(char **dest*, const char **src*, int *len*)

  parse a string token

**Parameters**

- `char *dest`

  the buffer to store the string token

- `const char *src`

  the original string

- `int len`

  the max. length of token - 1

**Description**

Parses the original string and copy a token to the given string buffer.

**Return**

The updated pointer of the original string so that it can be used for the next call.

- struct snd_info_entry * `snd_info_create_module_entry`(struct module * *module*, const char **name*, struct snd_info_entry **parent*)

  create an info entry for the given module

**Parameters**

- `struct module * module`

  the module pointer

- `const char *name`

  the file name

- `struct snd_info_entry *parent`

  the parent directory

**Description**

Creates a new info entry and assigns it to the given module.

**Return**

The pointer of the new instance, or `NULL` on failure.

- struct snd_info_entry * `snd_info_create_card_entry`(struct snd_card **card*, const char **name*, struct snd_info_entry * *parent*)

  create an info entry for the given card

**Parameters**

- `struct snd_card *card`

  the card instance

- `const char *name`

  the file name

- `struct snd_info_entry * parent`

  the parent directory

**Description**

Creates a new info entry and assigns it to the given card.

**Return**

The pointer of the new instance, or `NULL` on failure.

- void `snd_info_free_entry`(struct snd_info_entry * *entry*)

  release the info entry

**Parameters**

- `struct snd_info_entry * entry`

  the info entry

**Description**

Releases the info entry.

- int `snd_info_register`(struct snd_info_entry **entry*)

  register the info entry

**Parameters**

- `struct snd_info_entry *entry`

  the info entry

**Description**

Registers the proc info entry. The all children entries are registered recursively.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_card_rw_proc_new`(struct snd_card **card*, const char **name*, void **private_data*, void ( **read*)(struct snd_info_entry *, struct snd_info_buffer *), void (*write)(struct snd_info_entry *entry, struct snd_info_buffer *buffer) )

  Create a read/write text proc file entry for the card

**Parameters**

- `struct snd_card *card`

  the card instance

- `const char *name`

  the file name

- `void *private_data`

  the arbitrary private data

- `void (*read)(struct snd_info_entry *, struct snd_info_buffer *)`

  the read callback

- `void (*write)(struct snd_info_entry *entry, struct snd_info_buffer *buffer)`

  the write callback, NULL for read-only

**Description**

This proc file entry will be registered via [`snd_card_register()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_register) call, and it will be removed automatically at the card removal, too.

## Compress Offload

### Compress Offload API

- struct `snd_compressed_buffer`

  compressed buffer

**Definition**

```
struct snd_compressed_buffer {
  __u32 fragment_size;
  __u32 fragments;
};
```

**Members**

- `fragment_size`

  size of buffer fragment in bytes

- `fragments`

  number of such fragments

- struct `snd_compr_params`

  compressed stream params

**Definition**

```
struct snd_compr_params {
  struct snd_compressed_buffer buffer;
  struct snd_codec codec;
  __u8 no_wake_mode;
};
```

**Members**

- `buffer`

  buffer description

- `codec`

  codec parameters

- `no_wake_mode`

  dont wake on fragment elapsed

- struct `snd_compr_tstamp`

  timestamp descriptor

**Definition**

```
struct snd_compr_tstamp {
  __u32 byte_offset;
  __u32 copied_total;
  __u32 pcm_frames;
  __u32 pcm_io_frames;
  __u32 sampling_rate;
};
```

**Members**

- `byte_offset`

  Byte offset in ring buffer to DSP

- `copied_total`

  Total number of bytes copied from/to ring buffer to/by DSP

- `pcm_frames`

  Frames decoded or encoded by DSP. This field will evolve by large steps and should only be used to monitor encoding/decoding progress. It shall not be used for timing estimates.

- `pcm_io_frames`

  Frames rendered or received by DSP into a mixer or an audio output/input. This field should be used for A/V sync or time estimates.

- `sampling_rate`

  sampling rate of audio

- struct `snd_compr_avail`

  avail descriptor

**Definition**

```
struct snd_compr_avail {
  __u64 avail;
  struct snd_compr_tstamp tstamp;
};
```

**Members**

- `avail`

  Number of bytes available in ring buffer for writing/reading

- `tstamp`

  timestamp information

- struct `snd_compr_caps`

  caps descriptor

**Definition**

```
struct snd_compr_caps {
  __u32 num_codecs;
  __u32 direction;
  __u32 min_fragment_size;
  __u32 max_fragment_size;
  __u32 min_fragments;
  __u32 max_fragments;
  __u32 codecs[MAX_NUM_CODECS];
  __u32 reserved[11];
};
```

**Members**

- `num_codecs`

  number of codecs supported

- `direction`

  direction supported. Of type snd_compr_direction

- `min_fragment_size`

  minimum fragment supported by DSP

- `max_fragment_size`

  maximum fragment supported by DSP

- `min_fragments`

  min fragments supported by DSP

- `max_fragments`

  max fragments supported by DSP

- `codecs`

  pointer to array of codecs

- `reserved`

  reserved field

- struct `snd_compr_codec_caps`

  query capability of codec

**Definition**

```
struct snd_compr_codec_caps {
  __u32 codec;
  __u32 num_descriptors;
  struct snd_codec_desc descriptor[MAX_NUM_CODEC_DESCRIPTORS];
};
```

**Members**

- `codec`

  codec for which capability is queried

- `num_descriptors`

  number of codec descriptors

- `descriptor`

  array of codec capability descriptor

- enum `sndrv_compress_encoder`

  

**Constants**

- `SNDRV_COMPRESS_ENCODER_PADDING`

  no of samples appended by the encoder at the end of the track

- `SNDRV_COMPRESS_ENCODER_DELAY`

  no of samples inserted by the encoder at the beginning of the track

- struct `snd_compr_metadata`

  compressed stream metadata

**Definition**

```
struct snd_compr_metadata {
  __u32 key;
  __u32 value[8];
};
```

**Members**

- `key`

  key id

- `value`

  key value

- struct `snd_enc_vorbis`

  

**Definition**

```
struct snd_enc_vorbis {
  __s32 quality;
  __u32 managed;
  __u32 max_bit_rate;
  __u32 min_bit_rate;
  __u32 downmix;
};
```

**Members**

- `quality`

  Sets encoding quality to n, between -1 (low) and 10 (high). In the default mode of operation, the quality level is 3. Normal quality range is 0 - 10.

- `managed`

  Boolean. Set bitrate management mode. This turns off the normal VBR encoding, but allows hard or soft bitrate constraints to be enforced by the encoder. This mode can be slower, and may also be lower quality. It is primarily useful for streaming.

- `max_bit_rate`

  Enabled only if managed is TRUE

- `min_bit_rate`

  Enabled only if managed is TRUE

- `downmix`

  Boolean. Downmix input from stereo to mono (has no effect on non-stereo streams). Useful for lower-bitrate encoding.

**Description**

These options were extracted from the OpenMAX IL spec and Gstreamer vorbisenc properties

For best quality users should specify VBR mode and set quality levels.

- struct `snd_enc_real`

  

**Definition**

```
struct snd_enc_real {
  __u32 quant_bits;
  __u32 start_region;
  __u32 num_regions;
};
```

**Members**

- `quant_bits`

  number of coupling quantization bits in the stream

- `start_region`

  coupling start region in the stream

- `num_regions`

  number of regions value

**Description**

These options were extracted from the OpenMAX IL spec

- struct `snd_enc_flac`

  

**Definition**

```
struct snd_enc_flac {
  __u32 num;
  __u32 gain;
};
```

**Members**

- `num`

  serial number, valid only for OGG formats needs to be set by application

- `gain`

  Add replay gain tags

**Description**

These options were extracted from the FLAC online documentation at [http://flac.sourceforge.net/documentation_tools_flac.html](https://translate.google.com/website?sl=auto&tl=zh-CN&hl=zh-CN&client=webapp&u=http://flac.sourceforge.net/documentation_tools_flac.html)

To make the API simpler, it is assumed that the user will select quality profiles. Additional options that affect encoding quality and speed can be added at a later stage if needed.

By default the Subset format is used by encoders.

TAGS such as pictures, etc, cannot be handled by an offloaded encoder and are not supported in this API.

- struct `snd_compr_runtime`

  runtime stream description

**Definition**

```
struct snd_compr_runtime {
  snd_pcm_state_t state;
  struct snd_compr_ops *ops;
  void *buffer;
  u64 buffer_size;
  u32 fragment_size;
  u32 fragments;
  u64 total_bytes_available;
  u64 total_bytes_transferred;
  wait_queue_head_t sleep;
  void *private_data;
  unsigned char *dma_area;
  dma_addr_t dma_addr;
  size_t dma_bytes;
  struct snd_dma_buffer *dma_buffer_p;
};
```

**Members**

- `state`

  stream state

- `ops`

  pointer to DSP callbacks

- `buffer`

  pointer to kernel buffer, valid only when not in mmap mode or DSP doesn’t implement copy

- `buffer_size`

  size of the above buffer

- `fragment_size`

  size of buffer fragment in bytes

- `fragments`

  number of such fragments

- `total_bytes_available`

  cumulative number of bytes made available in the ring buffer

- `total_bytes_transferred`

  cumulative bytes transferred by offload DSP

- `sleep`

  poll sleep

- `private_data`

  driver private data pointer

- `dma_area`

  virtual buffer address

- `dma_addr`

  physical buffer address (not accessible from main CPU)

- `dma_bytes`

  size of DMA area

- `dma_buffer_p`

  runtime dma buffer pointer

- struct `snd_compr_stream`

  compressed stream

**Definition**

```
struct snd_compr_stream {
  const char *name;
  struct snd_compr_ops *ops;
  struct snd_compr_runtime *runtime;
  struct snd_compr *device;
  struct delayed_work error_work;
  enum snd_compr_direction direction;
  bool metadata_set;
  bool next_track;
  bool partial_drain;
  bool pause_in_draining;
  void *private_data;
  struct snd_dma_buffer dma_buffer;
};
```

**Members**

- `name`

  device name

- `ops`

  pointer to DSP callbacks

- `runtime`

  pointer to runtime structure

- `device`

  device pointer

- `error_work`

  delayed work used when closing the stream due to an error

- `direction`

  stream direction, playback/recording

- `metadata_set`

  metadata set flag, true when set

- `next_track`

  has userspace signal next track transition, true when set

- `partial_drain`

  undergoing partial_drain for stream, true when set

- `pause_in_draining`

  paused during draining state, true when set

- `private_data`

  pointer to DSP private data

- `dma_buffer`

  allocated buffer if any

- struct `snd_compr_ops`

  compressed path DSP operations

**Definition**

```
struct snd_compr_ops {
  int (*open)(struct snd_compr_stream *stream);
  int (*free)(struct snd_compr_stream *stream);
  int (*set_params)(struct snd_compr_stream *stream, struct snd_compr_params *params);
  int (*get_params)(struct snd_compr_stream *stream, struct snd_codec *params);
  int (*set_metadata)(struct snd_compr_stream *stream, struct snd_compr_metadata *metadata);
  int (*get_metadata)(struct snd_compr_stream *stream, struct snd_compr_metadata *metadata);
  int (*trigger)(struct snd_compr_stream *stream, int cmd);
  int (*pointer)(struct snd_compr_stream *stream, struct snd_compr_tstamp *tstamp);
  int (*copy)(struct snd_compr_stream *stream, char __user *buf, size_t count);
  int (*mmap)(struct snd_compr_stream *stream, struct vm_area_struct *vma);
  int (*ack)(struct snd_compr_stream *stream, size_t bytes);
  int (*get_caps) (struct snd_compr_stream *stream, struct snd_compr_caps *caps);
  int (*get_codec_caps) (struct snd_compr_stream *stream, struct snd_compr_codec_caps *codec);
};
```

**Members**

- `open`

  Open the compressed stream This callback is mandatory and shall keep dsp ready to receive the stream parameter

- `free`

  Close the compressed stream, mandatory

- `set_params`

  Sets the compressed stream parameters, mandatory This can be called in during stream creation only to set codec params and the stream properties

- `get_params`

  retrieve the codec parameters, mandatory

- `set_metadata`

  Set the metadata values for a stream

- `get_metadata`

  retrieves the requested metadata values from stream

- `trigger`

  Trigger operations like start, pause, resume, drain, stop. This callback is mandatory

- `pointer`

  Retrieve current h/w pointer information. Mandatory

- `copy`

  Copy the compressed data to/from userspace, Optional Can’t be implemented if DSP supports mmap

- `mmap`

  DSP mmap method to mmap DSP memory

- `ack`

  Ack for DSP when data is written to audio buffer, Optional Not valid if copy is implemented

- `get_caps`

  Retrieve DSP capabilities, mandatory

- `get_codec_caps`

  Retrieve capabilities for a specific codec, mandatory

- struct `snd_compr`

  Compressed device

**Definition**

```
struct snd_compr {
  const char *name;
  struct device dev;
  struct snd_compr_ops *ops;
  void *private_data;
  struct snd_card *card;
  unsigned int direction;
  struct mutex lock;
  int device;
  bool use_pause_in_draining;
#ifdef CONFIG_SND_VERBOSE_PROCFS;
};
```

**Members**

- `name`

  DSP device name

- `dev`

  associated device instance

- `ops`

  pointer to DSP callbacks

- `private_data`

  pointer to DSP pvt data

- `card`

  sound card pointer

- `direction`

  Playback or capture direction

- `lock`

  device lock

- `device`

  device id

- `use_pause_in_draining`

  allow pause in draining, true when set

- void `snd_compr_use_pause_in_draining`(struct [snd_compr_stream](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_compr_stream) **substream*)

  Allow pause and resume in draining state

**Parameters**

- `struct snd_compr_stream *substream`

  compress substream to set

**Description**

Allow pause and resume in draining state. Only HW driver supports this transition can call this API.

- 无效 `snd_compr_set_runtime_buffer`（结构 [snd_compr_stream ](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_compr_stream) **stream*，结构 snd_dma_buffer **bufp* ）

  设置压缩运行时缓冲区

**参数**

- `struct snd_compr_stream *stream`

  压缩流设置

- `struct snd_dma_buffer *bufp`

  缓冲区信息，NULL清空

**描述**

**当bufp**为非 NULL时，将缓冲区信息复制到运行时缓冲区。否则，它会清除当前缓冲区信息。

## 系统级芯片

### ASoC 核心 API

- 结构 snd_soc_component * `snd_soc_kcontrol_component`(结构 snd_kcontrol **kcontrol* )

  返回注册控件的组件

**参数**

- `struct snd_kcontrol *kcontrol`

  获取组件的控件

**笔记**

如果控件已为组件注册，此功能将正常工作。使用 snd_soc_add_codec_controls() 或通过基于表的设置为 CODEC 或组件驱动程序。否则行为未定义。

- 结构 snd_soc_dai * `snd_soc_find_dai`( const 结构 snd_soc_dai_link_component **dlc* )

  查找已注册的 DAI

**参数**

- `const struct snd_soc_dai_link_component *dlc`

  DAI 或 DAI 驱动程序的名称以及要匹配的可选组件信息

**描述**

此函数将搜索所有注册的组件及其 DAI，以找到同名的 DAI。如果指定，组件的 of_node 和名称也应该匹配。

**返回**

DAI 的指针，如果未找到，则为 NULL。

- 无效 `snd_soc_remove_pcm_runtime`（结构 snd_soc_card **card*，结构 snd_soc_pcm_runtime **rtd* ）

  从卡中删除 pcm_runtime

**参数**

- `struct snd_soc_card *card`

  pcm_runtime 所在的 ASoC 卡

- `struct snd_soc_pcm_runtime *rtd`

  要删除的 pcm_runtime

**描述**

此函数从 ASoC 卡中删除 pcm_runtime。

- int `snd_soc_add_pcm_runtime`( struct snd_soc_card **card* , struct snd_soc_dai_link **dai_link* )

  通过 dai_link 动态添加 pcm_runtime

**参数**

- `struct snd_soc_card *card`

  添加了 pcm_runtime 的 ASoC 卡

- `struct snd_soc_dai_link *dai_link`

  找到 pcm_runtime 的 DAI 链接

**描述**

此函数通过使用 dai_link 添加一个 pcm_runtime ASoC 卡。

**笔记**

Topology can use this API to add pcm_runtime when probing the topology component. And machine drivers can still define static DAI links in dai_link array.

- int `snd_soc_runtime_set_dai_fmt`(struct snd_soc_pcm_runtime **rtd*, unsigned int *dai_fmt*)

  Change DAI link format for a ASoC runtime

**Parameters**

- `struct snd_soc_pcm_runtime *rtd`

  The runtime for which the DAI link format should be changed

- `unsigned int dai_fmt`

  The new DAI link format

**Description**

This function updates the DAI link format for all DAIs connected to the DAI link for the specified runtime.

Returns 0 on success, otherwise a negative error code.

**Note**

For setups with a static format set the dai_fmt field in the corresponding snd_dai_link struct instead of using this function.

- int `snd_soc_set_dmi_name`(struct snd_soc_card **card*, const char **flavour*)

  Register DMI names to card

**Parameters**

- `struct snd_soc_card *card`

  The card to register DMI names

- `const char *flavour`

  The flavour “differentiator” for the card amongst its peers.

**Description**

An Intel machine driver may be used by many different devices but are difficult for userspace to differentiate, since machine drivers ususally use their own name as the card short name and leave the card long name blank. To differentiate such devices and fix bugs due to lack of device-specific configurations, this function allows DMI info to be used as the sound card long name, in the format of “vendor-product-version-board” (Character ‘-‘ is used to separate different DMI fields here). This will help the user space to load the device-specific Use Case Manager (UCM) configurations for the card.

Possible card long names may be: DellInc.-XPS139343-01-0310JH ASUSTeKCOMPUTERINC.-T100TA-1.0-T100TA Circuitco-MinnowboardMaxD0PLATFORM-D0-MinnowBoardMAX

This function also supports flavoring the card longname to provide the extra differentiation, like “vendor-product-version-board-flavor”.

We only keep number and alphabet characters and a few separator characters in the card long name since UCM in the user space uses the card long names as card configuration directory names and AudoConf cannot support special charactors like SPACE.

Returns 0 on success, otherwise a negative error code.

- struct snd_kcontrol * `snd_soc_cnew`(const struct snd_kcontrol_new **_template*, void **data*, const char **long_name*, const char **prefix*)

  create new control

**Parameters**

- `const struct snd_kcontrol_new *_template`

  control template

- `void *data`

  control private data

- `const char *long_name`

  control long name

- `const char *prefix`

  control name prefix

**Description**

Create a new mixer control from a template control.

Returns 0 for success, else error.

- int `snd_soc_add_component_controls`(struct snd_soc_component **component*, const struct snd_kcontrol_new **controls*, unsigned int *num_controls*)

  Add an array of controls to a component.

**Parameters**

- `struct snd_soc_component *component`

  Component to add controls to

- `const struct snd_kcontrol_new *controls`

  Array of controls to add

- `unsigned int num_controls`

  Number of elements in the array

**Return**

0 for success, else error.

- int `snd_soc_add_card_controls`(struct snd_soc_card **soc_card*, const struct snd_kcontrol_new **controls*, int *num_controls*)

  add an array of controls to a SoC card. Convenience function to add a list of controls.

**Parameters**

- `struct snd_soc_card *soc_card`

  SoC card to add controls to

- `const struct snd_kcontrol_new *controls`

  array of controls to add

- `int num_controls`

  number of elements in the array

**Description**

Return 0 for success, else error.

- int `snd_soc_add_dai_controls`(struct snd_soc_dai **dai*, const struct snd_kcontrol_new **controls*, int *num_controls*)

  add an array of controls to a DAI. Convienience function to add a list of controls.

**Parameters**

- `struct snd_soc_dai *dai`

  DAI to add controls to

- `const struct snd_kcontrol_new *controls`

  array of controls to add

- `int num_controls`

  number of elements in the array

**Description**

Return 0 for success, else error.

- int `snd_soc_register_card`(struct snd_soc_card **card*)

  Register a card with the ASoC core

**Parameters**

- `struct snd_soc_card *card`

  Card to register

- int `snd_soc_unregister_card`(struct snd_soc_card **card*)

  Unregister a card with the ASoC core

**Parameters**

- `struct snd_soc_card *card`

  Card to unregister

- struct snd_soc_dai * `snd_soc_register_dai`(struct snd_soc_component **component*, struct snd_soc_dai_driver **dai_drv*, bool *legacy_dai_naming*)

  Register a DAI dynamically & create its widgets

**Parameters**

- `struct snd_soc_component *component`

  The component the DAIs are registered for

- `struct snd_soc_dai_driver *dai_drv`

  DAI driver to use for the DAI

- `bool legacy_dai_naming`

  if `true`, use legacy single-name format; if `false`, use multiple-name format;

**Description**

Topology can use this API to register DAIs when probing a component. These DAIs’s widgets will be freed in the card cleanup and the DAIs will be freed in the component cleanup.

- void `snd_soc_unregister_dais`(struct snd_soc_component **component*)

  Unregister DAIs from the ASoC core

**Parameters**

- `struct snd_soc_component *component`

  The component for which the DAIs should be unregistered

- int `snd_soc_register_dais`(struct snd_soc_component **component*, struct snd_soc_dai_driver **dai_drv*, size_t *count*)

  Register a DAI with the ASoC core

**Parameters**

- `struct snd_soc_component *component`

  The component the DAIs are registered for

- `struct snd_soc_dai_driver *dai_drv`

  DAI driver to use for the DAIs

- `size_t count`

  Number of DAIs

- void `snd_soc_unregister_component_by_driver`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, const struct snd_soc_component_driver **component_driver*)

  Unregister component using a given driver from the ASoC core

**Parameters**

- `struct device *dev`

  The device to unregister

- `const struct snd_soc_component_driver *component_driver`

  The component driver to unregister

- void `snd_soc_unregister_component`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  Unregister all related component from the ASoC core

**Parameters**

- `struct device *dev`

  The device to unregister

- struct snd_soc_dai * `devm_snd_soc_register_dai`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, struct snd_soc_component **component*, struct snd_soc_dai_driver **dai_drv*, bool *legacy_dai_naming*)

  resource-managed dai registration

**Parameters**

- `struct device *dev`

  Device used to manage component

- `struct snd_soc_component *component`

  The component the DAIs are registered for

- `struct snd_soc_dai_driver *dai_drv`

  DAI driver to use for the DAI

- `bool legacy_dai_naming`

  if `true`, use legacy single-name format; if `false`, use multiple-name format;

- int `devm_snd_soc_register_component`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, const struct snd_soc_component_driver **cmpnt_drv*, struct snd_soc_dai_driver **dai_drv*, int *num_dai*)

  resource managed component registration

**Parameters**

- `struct device *dev`

  Device used to manage component

- `const struct snd_soc_component_driver *cmpnt_drv`

  Component driver

- `struct snd_soc_dai_driver *dai_drv`

  DAI driver

- `int num_dai`

  Number of DAIs to register

**Description**

Register a component with automatic unregistration when the device is unregistered.

- int `devm_snd_soc_register_card`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, struct snd_soc_card **card*)

  resource managed card registration

**Parameters**

- `struct device *dev`

  Device used to manage card

- `struct snd_soc_card *card`

  Card to register

**Description**

Register a card with automatic unregistration when the device is unregistered.

- int `devm_snd_dmaengine_pcm_register`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*, const struct [snd_dmaengine_pcm_config](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_dmaengine_pcm_config) **config*, unsigned int *flags*)

  resource managed dmaengine PCM registration

**Parameters**

- `struct device *dev`

  The parent device for the PCM device

- `const struct snd_dmaengine_pcm_config *config`

  Platform specific PCM configuration

- `unsigned int flags`

  Platform specific quirks

**Description**

Register a dmaengine based PCM device with automatic unregistration when the device is unregistered.

- int `snd_soc_component_set_sysclk`(struct snd_soc_component **component*, int *clk_id*, int *source*, unsigned int *freq*, int *dir*)

  configure COMPONENT system or master clock.

**Parameters**

- `struct snd_soc_component *component`

  COMPONENT

- `int clk_id`

  DAI specific clock ID

- `int source`

  Source for the clock

- `unsigned int freq`

  new clock frequency in Hz

- `int dir`

  new clock direction - input/output.

**Description**

Configures the CODEC master (MCLK) or system (SYSCLK) clocking.

- int `snd_soc_component_set_jack`(struct snd_soc_component **component*, struct snd_soc_jack **jack*, void **data*)

  configure component jack.

**Parameters**

- `struct snd_soc_component *component`

  COMPONENTs

- `struct snd_soc_jack *jack`

  structure to use for the jack

- `void *data`

  can be used if codec driver need extra data for configuring jack

**Description**

Configures and enables jack detection function.

- void `snd_soc_component_init_regmap`(struct snd_soc_component **component*, struct regmap **regmap*)

  Initialize regmap instance for the component

**Parameters**

- `struct snd_soc_component *component`

  The component for which to initialize the regmap instance

- `struct regmap *regmap`

  The regmap instance that should be used by the component

**Description**

This function allows deferred assignment of the regmap instance that is associated with the component. Only use this if the regmap instance is not yet ready when the component is registered. The function must also be called before the first IO attempt of the component.

- void `snd_soc_component_exit_regmap`(struct snd_soc_component **component*)

  De-initialize regmap instance for the component

**Parameters**

- `struct snd_soc_component *component`

  The component for which to de-initialize the regmap instance

**Description**

Calls regmap_exit() on the regmap instance associated to the component and removes the regmap instance from the component.

This function should only be used if [`snd_soc_component_init_regmap()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_component_init_regmap) was used to initialize the regmap instance.

- unsigned int `snd_soc_component_read`(struct snd_soc_component **component*, unsigned int *reg*)

  Read register value

**Parameters**

- `struct snd_soc_component *component`

  Component to read from

- `unsigned int reg`

  Register to read

**Return**

read value

- int `snd_soc_component_write`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *val*)

  Write register value

**Parameters**

- `struct snd_soc_component *component`

  Component to write to

- `unsigned int reg`

  Register to write

- `unsigned int val`

  Value to write to the register

**Return**

0 on success, a negative error code otherwise.

- int `snd_soc_component_update_bits`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *mask*, unsigned int *val*)

  Perform read/modify/write cycle

**Parameters**

- `struct snd_soc_component *component`

  Component to update

- `unsigned int reg`

  Register to update

- `unsigned int mask`

  Mask that specifies which bits to update

- `unsigned int val`

  New value for the bits specified by mask

**Return**

1 if the operation was successful and the value of the register changed, 0 if the operation was successful, but the value did not change. Returns a negative error code otherwise.

- int `snd_soc_component_update_bits_async`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *mask*, unsigned int *val*)

  Perform asynchronous read/modify/write cycle

**Parameters**

- `struct snd_soc_component *component`

  Component to update

- `unsigned int reg`

  Register to update

- `unsigned int mask`

  Mask that specifies which bits to update

- `unsigned int val`

  New value for the bits specified by mask

**Description**

This function is similar to [`snd_soc_component_update_bits()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_component_update_bits), but the update operation is scheduled asynchronously. This means it may not be completed when the function returns. To make sure that all scheduled updates have been completed [`snd_soc_component_async_complete()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_component_async_complete) must be called.

**Return**

1 if the operation was successful and the value of the register changed, 0 if the operation was successful, but the value did not change. Returns a negative error code otherwise.

- unsigned int `snd_soc_component_read_field`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *mask*)

  Read register field value

**Parameters**

- `struct snd_soc_component *component`

  Component to read from

- `unsigned int reg`

  Register to read

- `unsigned int mask`

  mask of the register field

**Return**

read value of register field.

- int `snd_soc_component_write_field`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *mask*, unsigned int *val*)

  write to register field

**Parameters**

- `struct snd_soc_component *component`

  Component to write to

- `unsigned int reg`

  Register to write

- `unsigned int mask`

  mask of the register field to update

- `unsigned int val`

  value of the field to write

**Return**

1 for change, otherwise 0.

- void `snd_soc_component_async_complete`(struct snd_soc_component **component*)

  Ensure asynchronous I/O has completed

**Parameters**

- `struct snd_soc_component *component`

  Component for which to wait

**Description**

This function blocks until all asynchronous I/O which has previously been scheduled using [`snd_soc_component_update_bits_async()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_component_update_bits_async) has completed.

- int `snd_soc_component_test_bits`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *mask*, unsigned int *value*)

  Test register for change

**Parameters**

- `struct snd_soc_component *component`

  component

- `unsigned int reg`

  Register to test

- `unsigned int mask`

  Mask that specifies which bits to test

- `unsigned int value`

  Value to test against

**Description**

Tests a register with a new value and checks if the new value is different from the old value.

**Return**

1 for change, otherwise 0.

- void `snd_soc_runtime_action`(struct snd_soc_pcm_runtime **rtd*, int *stream*, int *action*)

  Increment/Decrement active count for PCM runtime components

**Parameters**

- `struct snd_soc_pcm_runtime *rtd`

  ASoC PCM runtime that is activated

- `int stream`

  Direction of the PCM stream

- `int action`

  Activate stream if 1. Deactivate if -1.

**Description**

Increments/Decrements the active count for all the DAIs and components attached to a PCM runtime. Should typically be called when a stream is opened.

Must be called with the rtd->card->pcm_mutex being held

- bool `snd_soc_runtime_ignore_pmdown_time`(struct snd_soc_pcm_runtime **rtd*)

  Check whether to ignore the power down delay

**Parameters**

- `struct snd_soc_pcm_runtime *rtd`

  The ASoC PCM runtime that should be checked.

**Description**

This function checks whether the power down delay should be ignored for a specific PCM runtime. Returns true if the delay is 0, if it the DAI link has been configured to ignore the delay, or if none of the components benefits from having the delay.

- int `snd_soc_set_runtime_hwparams`(struct snd_pcm_substream **substream*, const struct snd_pcm_hardware **hw*)

  set the runtime hardware parameters

**Parameters**

- `struct snd_pcm_substream *substream`

  the pcm substream

- `const struct snd_pcm_hardware *hw`

  the hardware parameters

**Description**

Sets the substream runtime hardware parameters.

- int `snd_soc_runtime_calc_hw`(struct snd_soc_pcm_runtime **rtd*, struct snd_pcm_hardware **hw*, int *stream*)

  Calculate hw limits for a PCM stream

**Parameters**

- `struct snd_soc_pcm_runtime *rtd`

  ASoC PCM runtime

- `struct snd_pcm_hardware *hw`

  PCM hardware parameters (output)

- `int stream`

  Direction of the PCM stream

**Description**

Calculates the subset of stream parameters supported by all DAIs associated with the PCM stream.

- int `snd_soc_info_enum_double`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  enumerated double mixer info callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_info *uinfo`

  control element information

**Description**

Callback to provide information about a double enumerated mixer control.

Returns 0 for success.

- int `snd_soc_get_enum_double`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  enumerated double mixer get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value of a double enumerated mixer.

Returns 0 for success.

- int `snd_soc_put_enum_double`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  enumerated double mixer put callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value of a double enumerated mixer.

Returns 0 for success.

- int `snd_soc_read_signed`(struct snd_soc_component **component*, unsigned int *reg*, unsigned int *mask*, unsigned int *shift*, unsigned int *sign_bit*, int **signed_val*)

  Read a codec register and interpret as signed value

**Parameters**

- `struct snd_soc_component *component`

  component

- `unsigned int reg`

  Register to read

- `unsigned int mask`

  Mask to use after shifting the register value

- `unsigned int shift`

  Right shift of register value

- `unsigned int sign_bit`

  Bit that describes if a number is negative or not.

- `int *signed_val`

  Pointer to where the read value should be stored

**Description**

This functions reads a codec register. The register value is shifted right by ‘shift’ bits and masked with the given ‘mask’. Afterwards it translates the given registervalue into a signed integer if sign_bit is non-zero.

Returns 0 on sucess, otherwise an error value

- int `snd_soc_info_volsw`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  single mixer info callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_info *uinfo`

  control element information

**Description**

Callback to provide information about a single mixer control, or a double mixer control that spans 2 registers.

Returns 0 for success.

- int `snd_soc_info_volsw_sx`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  Mixer info callback for SX TLV controls

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_info *uinfo`

  control element information

**Description**

Callback to provide information about a single mixer control, or a double mixer control that spans 2 registers of the SX TLV type. SX TLV controls have a range that represents both positive and negative values either side of zero but without a sign bit.

Returns 0 for success.

- int `snd_soc_get_volsw`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  single mixer get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value of a single mixer control, or a double mixer control that spans 2 registers.

Returns 0 for success.

- int `snd_soc_put_volsw`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  single mixer put callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value of a single mixer control, or a double mixer control that spans 2 registers.

Returns 0 for success.

- int `snd_soc_get_volsw_sx`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  single mixer get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value of a single mixer control, or a double mixer control that spans 2 registers.

Returns 0 for success.

- int `snd_soc_put_volsw_sx`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  double mixer set callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value of a double mixer control that spans 2 registers.

Returns 0 for success.

- int `snd_soc_info_volsw_range`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  single mixer info callback with range.

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_info *uinfo`

  control element information

**Description**

Callback to provide information, within a range, about a single mixer control.

returns 0 for success.

- int `snd_soc_put_volsw_range`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  single mixer put value callback with range.

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value, within a range, for a single mixer control.

Returns 0 for success.

- int `snd_soc_get_volsw_range`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  single mixer get callback with range

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value, within a range, of a single mixer control.

Returns 0 for success.

- int `snd_soc_limit_volume`(struct snd_soc_card **card*, const char **name*, int *max*)

  Set new limit to an existing volume control.

**Parameters**

- `struct snd_soc_card *card`

  where to look for the control

- `const char *name`

  Name of the control

- `int max`

  new maximum limit

**Description**

Return 0 for success, else error.

- int `snd_soc_info_xr_sx`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  signed multi register info callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mreg control

- `struct snd_ctl_elem_info *uinfo`

  control element information

**Description**

Callback to provide information of a control that can span multiple codec registers which together forms a single signed value in a MSB/LSB manner.

Returns 0 for success.

- int `snd_soc_get_xr_sx`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  signed multi register get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mreg control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value of a control that can span multiple codec registers which together forms a single signed value in a MSB/LSB manner. The control supports specifying total no of bits used to allow for bitfields across the multiple codec registers.

Returns 0 for success.

- int `snd_soc_put_xr_sx`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  signed multi register get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mreg control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value of a control that can span multiple codec registers which together forms a single signed value in a MSB/LSB manner. The control supports specifying total no of bits used to allow for bitfields across the multiple codec registers.

Returns 0 for success.

- int `snd_soc_get_strobe`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  strobe get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback get the value of a strobe mixer control.

Returns 0 for success.

- int `snd_soc_put_strobe`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  strobe put callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback strobe a register bit to high then low (or the inverse) in one pass of a single mixer enum control.

Returns 1 for success.

- int `snd_soc_new_compress`(struct snd_soc_pcm_runtime **rtd*, int *num*)

  create a new compress.

**Parameters**

- `struct snd_soc_pcm_runtime *rtd`

  The runtime for which we will create compress

- `int num`

  the device index number (zero based - shared with normal PCMs)

**Return**

0 for success, else error.

### ASoC DAPM API

- struct snd_soc_dapm_widget * `snd_soc_dapm_kcontrol_widget`(struct snd_kcontrol **kcontrol*)

  Returns the widget associated to a kcontrol

**Parameters**

- `struct snd_kcontrol *kcontrol`

  The kcontrol

- struct snd_soc_dapm_context * `snd_soc_dapm_kcontrol_dapm`(struct snd_kcontrol **kcontrol*)

  Returns the dapm context associated to a kcontrol

**Parameters**

- `struct snd_kcontrol *kcontrol`

  The kcontrol

**Note**

This function must only be used on kcontrols that are known to have been registered for a CODEC. Otherwise the behaviour is undefined.

- int `snd_soc_dapm_force_bias_level`(struct snd_soc_dapm_context **dapm*, enum snd_soc_bias_level *level*)

  Sets the DAPM bias level

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  The DAPM context for which to set the level

- `enum snd_soc_bias_level level`

  The level to set

**Description**

Forces the DAPM bias level to a specific state. It will call the bias level callback of DAPM context with the specified level. This will even happen if the context is already at the same level. Furthermore it will not go through the normal bias level sequencing, meaning any intermediate states between the current and the target state will not be entered.

Note that the change in bias level is only temporary and the next time [`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) is called the state will be set to the level as determined by the DAPM core. The function is mainly intended to be used to used during probe or resume from suspend to power up the device so initialization can be done, before the DAPM core takes over.

- int `snd_soc_dapm_set_bias_level`(struct snd_soc_dapm_context **dapm*, enum snd_soc_bias_level *level*)

  set the bias level for the system

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `enum snd_soc_bias_level level`

  level to configure

**Description**

Configure the bias (power) levels for the SoC audio device.

Returns 0 for success else error.

- int `snd_soc_dapm_dai_get_connected_widgets`(struct snd_soc_dai **dai*, int *stream*, struct snd_soc_dapm_widget_list ***list*, bool ( **custom_stop_condition*)(struct snd_soc_dapm_widget *, enum snd_soc_dapm_direction) )

  query audio path and it’s widgets.

**Parameters**

- `struct snd_soc_dai *dai`

  the soc DAI.

- `int stream`

  stream direction.

- `struct snd_soc_dapm_widget_list **list`

  list of active widgets for this stream.

- `bool (*custom_stop_condition)(struct snd_soc_dapm_widget *, enum snd_soc_dapm_direction)`

  (optional) a function meant to stop the widget graph walk based on custom logic.

**Description**

Queries DAPM graph as to whether a valid audio stream path exists for the initial stream specified by name. This takes into account current mixer and mux kcontrol settings. Creates list of valid widgets.

可选地，可以提供作为停止条件的功能。此函数将当前正在检查的 dapm 小部件和步行方向作为参数，如果应该停止步行，它应该返回 true，否则返回 false。

返回有效路径的数量或负错误。

- 无效 `snd_soc_dapm_free_widget`（结构 snd_soc_dapm_widget **w* ）

  免费指定小部件

**参数**

- `struct snd_soc_dapm_widget *w`

  小部件免费

**描述**

从所有路径中删除小部件并释放它占用的内存。

- 整数 `snd_soc_dapm_sync_unlocked`（结构 snd_soc_dapm_context **dapm* ）

  扫描和电源 dapm 路径

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

**描述**

遍历所有 dapm 音频路径并根据其流或路径使用情况为小部件供电。

需要外部锁定。

返回 0 表示成功。

- 整数 `snd_soc_dapm_sync`（结构 snd_soc_dapm_context **dapm* ）

  扫描和电源 dapm 路径

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

**描述**

遍历所有 dapm 音频路径并根据其流或路径使用情况为小部件供电。

返回 0 表示成功。

- int `snd_soc_dapm_add_routes`( struct snd_soc_dapm_context **dapm* , const struct snd_soc_dapm_route **route* , int *num* )

  在 DAPM 小部件之间添加路由

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const struct snd_soc_dapm_route *route`

  音频路由

- `int num`

  路线数

**描述**

通过命名音频路径将 2 个 dapm 小部件连接在一起。接收器是接收音频信号的小部件，而源是音频信号的发送器。

返回 0 表示成功，否则返回错误。出错时可以调用 snd_soc_card_free() 释放所有资源。

- int `snd_soc_dapm_del_routes`( struct snd_soc_dapm_context **dapm* , const struct snd_soc_dapm_route **route* , int *num* )

  删除 DAPM 小部件之间的路由

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const struct snd_soc_dapm_route *route`

  音频路由

- `int num`

  路线数

**描述**

从 DAPM 上下文中删除路由。

- int `snd_soc_dapm_weak_routes`( struct snd_soc_dapm_context **dapm* , const struct snd_soc_dapm_route **route* , int *num* )

  将 DAPM 小部件之间的路由标记为弱

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const struct snd_soc_dapm_route *route`

  音频路由

- `int num`

  路线数

**描述**

Mark existing routes matching those specified in the passed array as being weak, meaning that they are ignored for the purpose of power decisions. The main intended use case is for sidetone paths which couple audio between other independent paths if they are both active in order to make the combination work better at the user level but which aren’t intended to be “used”.

Note that CODEC drivers should not use this as sidetone type paths can frequently also be used as bypass paths.

- int `snd_soc_dapm_new_widgets`(struct snd_soc_card **card*)

  add new dapm widgets

**Parameters**

- `struct snd_soc_card *card`

  card to be checked for new dapm widgets

**Description**

Checks the codec for any new dapm widgets and creates them if found.

Returns 0 for success.

- int `snd_soc_dapm_get_volsw`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  dapm mixer get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value of a dapm mixer control.

Returns 0 for success.

- int `snd_soc_dapm_put_volsw`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  dapm mixer set callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value of a dapm mixer control.

Returns 0 for success.

- int `snd_soc_dapm_get_enum_double`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  dapm enumerated double mixer get callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to get the value of a dapm enumerated double mixer control.

Returns 0 for success.

- int `snd_soc_dapm_put_enum_double`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_value **ucontrol*)

  dapm enumerated double mixer set callback

**Parameters**

- `struct snd_kcontrol *kcontrol`

  mixer control

- `struct snd_ctl_elem_value *ucontrol`

  control element information

**Description**

Callback to set the value of a dapm enumerated double mixer control.

Returns 0 for success.

- int `snd_soc_dapm_info_pin_switch`(struct snd_kcontrol **kcontrol*, struct snd_ctl_elem_info **uinfo*)

  引脚开关的信息

**参数**

- `struct snd_kcontrol *kcontrol`

  混音器控制

- `struct snd_ctl_elem_info *uinfo`

  控制元件信息

**描述**

回调以提供有关引脚开关控制的信息。

- int `snd_soc_dapm_get_pin_switch`( struct snd_kcontrol **kcontrol* , struct snd_ctl_elem_value **ucontrol* )

  获取引脚开关的信息

**参数**

- `struct snd_kcontrol *kcontrol`

  混音器控制

- `struct snd_ctl_elem_value *ucontrol`

  价值

- int `snd_soc_dapm_put_pin_switch`( struct snd_kcontrol **kcontrol* , struct snd_ctl_elem_value **ucontrol* )

  设置引脚开关的信息

**参数**

- `struct snd_kcontrol *kcontrol`

  混音器控制

- `struct snd_ctl_elem_value *ucontrol`

  价值

- struct snd_soc_dapm_widget * `snd_soc_dapm_new_control`( struct snd_soc_dapm_context **dapm* , const struct snd_soc_dapm_widget **widget* )

  创建新的 dapm 控件

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const struct snd_soc_dapm_widget *widget`

  小部件模板

**描述**

基于模板创建新的 DAPM 控件。

返回成功时的小部件指针或失败时的错误指针

- int `snd_soc_dapm_new_controls`( struct snd_soc_dapm_context **dapm* , const struct snd_soc_dapm_widget **widget* , int *num* )

  创建新的 dapm 控件

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const struct snd_soc_dapm_widget *widget`

  小部件数组

- `int num`

  小部件数量

**描述**

基于模板创建新的 DAPM 控件。

返回 0 表示成功，否则返回错误。

- 整数 `snd_soc_dapm_new_dai_widgets`（结构 snd_soc_dapm_context **dapm*，结构 snd_soc_dai **dai* ）

  创建新的 DAPM 小部件

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `struct snd_soc_dai *dai`

  父 DAI

**描述**

成功返回 0，否则返回错误代码。

- void `snd_soc_dapm_stream_event`(struct snd_soc_pcm_runtime **rtd*, int *stream*, int *event*)

  send a stream event to the dapm core

**Parameters**

- `struct snd_soc_pcm_runtime *rtd`

  PCM runtime data

- `int stream`

  stream name

- `int event`

  stream event

**Description**

Sends a stream event to the dapm core. The core then makes any necessary widget power changes.

Returns 0 for success else error.

- int `snd_soc_dapm_enable_pin_unlocked`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  enable pin.

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

Enables input/output pin and its parents or children widgets iff there is a valid audio route and active audio stream.

Requires external locking.

**NOTE**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) needs to be called after this for DAPM to do any widget power switching.

- int `snd_soc_dapm_enable_pin`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  enable pin.

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

Enables input/output pin and its parents or children widgets iff there is a valid audio route and active audio stream.

**NOTE**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) needs to be called after this for DAPM to do any widget power switching.

- int `snd_soc_dapm_force_enable_pin_unlocked`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  force a pin to be enabled

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

Enables input/output pin regardless of any other state. This is intended for use with microphone bias supplies used in microphone jack detection.

Requires external locking.

**NOTE**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) needs to be called after this for DAPM to do any widget power switching.

- int `snd_soc_dapm_force_enable_pin`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  force a pin to be enabled

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

Enables input/output pin regardless of any other state. This is intended for use with microphone bias supplies used in microphone jack detection.

**NOTE**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) needs to be called after this for DAPM to do any widget power switching.

- int `snd_soc_dapm_disable_pin_unlocked`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  disable pin.

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

Disables input/output pin and its parents or children widgets.

Requires external locking.

**NOTE**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) needs to be called after this for DAPM to do any widget power switching.

- int `snd_soc_dapm_disable_pin`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  disable pin.

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

Disables input/output pin and its parents or children widgets.

**NOTE**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync) needs to be called after this for DAPM to do any widget power switching.

- int `snd_soc_dapm_nc_pin_unlocked`(struct snd_soc_dapm_context **dapm*, const char **pin*)

  permanently disable pin.

**Parameters**

- `struct snd_soc_dapm_context *dapm`

  DAPM context

- `const char *pin`

  pin name

**Description**

将指定的引脚标记为未连接，沿任何父或子小部件禁用它。目前这与它相同，[`snd_soc_dapm_disable_pin()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_disable_pin)但将来它将被扩展以执行其他操作，例如禁用仅影响通过引脚的路径的控件。

需要外部锁定。

**笔记**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync)在此之后需要调用 DAPM 以执行任何小部件电源切换。

- int `snd_soc_dapm_nc_pin`( struct snd_soc_dapm_context **dapm* , const char **pin* )

  永久禁用引脚。

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const char *pin`

  引脚名称

**描述**

将指定的引脚标记为未连接，沿任何父或子小部件禁用它。目前这与它相同，[`snd_soc_dapm_disable_pin()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_disable_pin)但将来它将被扩展以执行其他操作，例如禁用仅影响通过引脚的路径的控件。

**笔记**

[`snd_soc_dapm_sync()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_dapm_sync)在此之后需要调用 DAPM 以执行任何小部件电源切换。

- int `snd_soc_dapm_get_pin_status`( struct snd_soc_dapm_context **dapm* , const char **pin* )

  获取音频引脚状态

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const char *pin`

  音频信号引脚端点（或起点）

**描述**

获取音频引脚状态 - 连接或断开连接。

连接时返回 1，否则返回 0。

- int `snd_soc_dapm_ignore_suspend`( struct snd_soc_dapm_context **dapm* , const char **pin* )

  忽略 DAPM 端点的挂起状态

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

- `const char *pin`

  音频信号引脚端点（或起点）

**描述**

将给定的端点或引脚标记为忽略挂起。当系统被禁用时，标记为忽略挂起的两个端点之间的路径将不会被禁用。该路径必须已在挂起时通过正常方式启用，如果尚未启用，则不会打开。

- 无效 `snd_soc_dapm_free`（结构 snd_soc_dapm_context **dapm* ）

  免费的 dapm 资源

**参数**

- `struct snd_soc_dapm_context *dapm`

  DAPM 上下文

**描述**

释放所有 dapm 小部件和资源。

### ASoC DMA 引擎 API

- int `snd_dmaengine_pcm_prepare_slave_config`( struct snd_pcm_substream **substream* , struct snd_pcm_hw_params **params* , struct dma_slave_config **slave_config* )

  通用 prepare_slave_config 回调

**参数**

- `struct snd_pcm_substream *substream`

  PCM 子流

- `struct snd_pcm_hw_params *params`

  hw_params

- `struct dma_slave_config *slave_config`

  准备 DMA 从站配置

**描述**

此函数可用作平台的通用 prepare_slave_config 回调，这些平台使用 snd_dmaengine_dai_dma_data 结构来存储其 DAI DMA 数据。该函数在内部将首先调用 snd_hwparams_to_dma_slave_config 根据 hw_params 填写从属配置，然后调用 snd_dmaengine_set_config_from_dai_data 根据 DAI DMA 数据填写剩余字段。

- int `snd_dmaengine_pcm_register`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , const struct [snd_dmaengine_pcm_config ](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_dmaengine_pcm_config) **config* , unsigned int *flags* )

  注册一个基于 dmaengine 的 PCM 设备

**参数**

- `struct device *dev`

  PCM 设备的父设备

- `const struct snd_dmaengine_pcm_config *config`

  平台特定的 PCM 配置

- `unsigned int flags`

  平台特定的怪癖

- 无效 `snd_dmaengine_pcm_unregister`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* ）

  删除基于 dmaengine 的 PCM 设备

**参数**

- `struct device *dev`

  PCM 注册的父设备

**描述**

删除先前使用 snd_dmaengine_pcm_register 注册的基于 dmaengine 的 PCM 设备。

## 杂项功能

### 硬件相关设备 API

- int `snd_hwdep_new`( struct snd_card **card* , char **id* , int *device* , struct snd_hwdep ***rhwdep* )

  创建一个新的 hwdep 实例

**参数**

- `struct snd_card *card`

  卡片实例

- `char *id`

  id 字符串

- `int device`

  设备索引（从零开始）

- `struct snd_hwdep **rhwdep`

  存储新 hwdep 实例的指针

**描述**

使用卡片上的给定索引创建一个新的 hwdep 实例。回调（hwdep->ops）必须由调用者手动设置在返回的实例上。

**返回**

成功则为零，失败则为负错误代码。

### Jack 抽象层 API

- 枚举 `snd_jack_types`

  可报告的插孔类型

**常数**

- `SND_JACK_HEADPHONE`

  耳机

- `SND_JACK_MICROPHONE`

  麦克风

- `SND_JACK_HEADSET`

  耳机

- `SND_JACK_LINEOUT`

  线路输出

- `SND_JACK_MECHANICAL`

  机械开关

- `SND_JACK_VIDEOOUT`

  影像输出

- `SND_JACK_AVOUT`

  AV（音频视频）输出

- `SND_JACK_LINEIN`

  输入

- `SND_JACK_BTN_0`

  按钮 0

- `SND_JACK_BTN_1`

  按钮 1

- `SND_JACK_BTN_2`

  按钮 2

- `SND_JACK_BTN_3`

  按钮 3

- `SND_JACK_BTN_4`

  按钮 4

- `SND_JACK_BTN_5`

  按钮 5

**描述**

这些值用作位掩码。

请注意，这必须与 sound/core/jack.c 中的查找表保持同步。

- int `snd_jack_add_new_kctl`( struct snd_jack **jack* , const char * *name* , int *mask* )

  Create a new snd_jack_kctl and add it to jack

**Parameters**

- `struct snd_jack *jack`

  the jack instance which the kctl will attaching to

- `const char * name`

  the name for the snd_kcontrol object

- `int mask`

  a bitmask of enum snd_jack_type values that can be detected by this snd_jack_kctl object.

**Description**

Creates a new snd_kcontrol object and adds it to the jack kctl_list.

**Return**

Zero if successful, or a negative error code on failure.

- int `snd_jack_new`(struct snd_card **card*, const char **id*, int *type*, struct snd_jack ***jjack*, bool *initial_kctl*, bool *phantom_jack*)

  Create a new jack

**Parameters**

- `struct snd_card *card`

  the card instance

- `const char *id`

  an identifying string for this jack

- `int type`

  此插孔可以检测到的枚举 snd_jack_type 值的位掩码

- `struct snd_jack **jjack`

  用于向调用者提供分配的插孔对象。

- `bool initial_kctl`

  如果为真，则创建一个 kcontrol 并将其添加到插孔列表中。

- `bool phantom_jack`

  不要为幻象插孔创建输入设备。

**描述**

创建一个新的插孔对象。

**返回**

成功则为零，失败则为负错误代码。成功**时 jjack**将被初始化。

- void `snd_jack_set_parent`( struct snd_jack **jack* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* )

  为插孔设置父设备

**参数**

- `struct snd_jack *jack`

  要配置的插孔

- `struct device *parent`

  设置为插孔父设备的设备。

**描述**

在设备树中设置插孔设备的父级。此功能仅在插孔注册前有效。如果没有配置父设备，则父设备将是声卡。

- int `snd_jack_set_key`( struct snd_jack **jack* , enum [snd_jack_types ](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_jack_types) *type* , int *keytype* )

  在插孔上设置键映射

**参数**

- `struct snd_jack *jack`

  要配置的插孔

- `enum snd_jack_types type`

  此密钥的 Jack 报告类型

- `int keytype`

  要报告的输入层键类型

**描述**

将 SND_JACK_BTN_* 按钮类型映射到输入层键，允许通过插孔抽象报告附件上的键。如果未提供映射但在插孔类型中启用了按键，则将报告 BTN_n 数字按钮。

If jacks are not reporting via the input API this call will have no effect.

Note that this is intended to be use by simple devices with small numbers of keys that can be reported. It is also possible to access the input device directly - devices with complex input capabilities on accessories should consider doing this rather than using this abstraction.

This function may only be called prior to registration of the jack.

**Return**

Zero if successful, or a negative error code on failure.

- void `snd_jack_report`(struct snd_jack **jack*, int *status*)

  Report the current status of a jack

**Parameters**

- `struct snd_jack *jack`

  The jack to report status for

- `int status`

  The current status of the jack

- void `snd_soc_jack_report`(struct snd_soc_jack **jack*, int *status*, int *mask*)

  Report the current status for a jack

**Parameters**

- `struct snd_soc_jack *jack`

  the jack

- `int status`

  a bitmask of enum snd_jack_type values that are currently detected.

- `int mask`

  a bitmask of enum snd_jack_type values that being reported.

**Description**

If configured using [`snd_soc_jack_add_pins()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_soc_jack_add_pins) then the associated DAPM pins will be enabled or disabled as appropriate and DAPM synchronised.

**Note**

This function uses mutexes and should be called from a context which can sleep (such as a workqueue).

- int `snd_soc_jack_add_zones`( struct snd_soc_jack **jack* , int *count* , struct snd_soc_jack_zone **zones* )

  将电压区域与插孔关联

**参数**

- `struct snd_soc_jack *jack`

  ASoC 插孔

- `int count`

  区域数

- `struct snd_soc_jack_zone *zones`

  区域数组

**描述**

调用此函数后，数组中指定的区域将与插孔相关联。

- int `snd_soc_jack_get_type`（结构 snd_soc_jack **jack*，int *micbias_voltage* ）

  根据麦克风偏置值，此函数从插孔类型中声明的区域返回插孔类型

**参数**

- `struct snd_soc_jack *jack`

  ASoC 插孔

- `int micbias_voltage`

  插入插孔时，ADC 通道的麦克风偏置电压

**描述**

根据传递的麦克风偏置值，此功能有助于从已声明的插孔区域中识别插孔类型

- int `snd_soc_jack_add_pins`( struct snd_soc_jack **jack* , int *count* , struct snd_soc_jack_pin **pins* )

  将 DAPM 引脚与 ASoC 插孔相关联

**参数**

- `struct snd_soc_jack *jack`

  ASoC 插孔

- `int count`

  引脚数

- `struct snd_soc_jack_pin *pins`

  引脚阵列

**描述**

调用此函数后，引脚数组中指定的 DAPM 引脚将更新其状态，以反映插孔状态更新时的当前状态。

- 无效 `snd_soc_jack_notifier_register`（结构 snd_soc_jack **jack*，结构 notifier_block **nb* ）

  注册插孔状态通知器

**参数**

- `struct snd_soc_jack *jack`

  ASoC 插孔

- `struct notifier_block *nb`

  Notifier block to register

**Description**

Register for notification of the current status of the jack. Note that it is not possible to report additional jack events in the callback from the notifier, this is intended to support applications such as enabling electrical detection only when a mechanical detection event has occurred.

- void `snd_soc_jack_notifier_unregister`(struct snd_soc_jack **jack*, struct notifier_block **nb*)

  Unregister a notifier for jack status

**Parameters**

- `struct snd_soc_jack *jack`

  ASoC jack

- `struct notifier_block *nb`

  Notifier block to unregister

**Description**

Stop notifying for status changes.

- int `snd_soc_jack_add_gpios`(struct snd_soc_jack **jack*, int *count*, struct snd_soc_jack_gpio **gpios*)

  Associate GPIO pins with an ASoC jack

**Parameters**

- `struct snd_soc_jack *jack`

  ASoC jack

- `int count`

  number of pins

- `struct snd_soc_jack_gpio *gpios`

  array of gpio pins

**Description**

This function will request gpio, set data direction and request irq for each gpio in the array.

- int `snd_soc_jack_add_gpiods`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **gpiod_dev*, struct snd_soc_jack **jack*, int *count*, struct snd_soc_jack_gpio **gpios*)

  Associate GPIO descriptor pins with an ASoC jack

**Parameters**

- `struct device *gpiod_dev`

  GPIO consumer device

- `struct snd_soc_jack *jack`

  ASoC jack

- `int count`

  number of pins

- `struct snd_soc_jack_gpio *gpios`

  array of gpio pins

**Description**

This function will request gpio, set data direction and request irq for each gpio in the array.

- 无效 `snd_soc_jack_free_gpios`（结构 snd_soc_jack **jack*，int *计数*，结构 snd_soc_jack_gpio **gpios* ）

  释放 ASoC 插孔的 GPIO 管脚资源

**参数**

- `struct snd_soc_jack *jack`

  ASoC 插孔

- `int count`

  引脚数

- `struct snd_soc_jack_gpio *gpios`

  gpio 引脚阵列

**描述**

释放与 ASoC 插孔关联的 gpio 引脚的 gpio 和 irq 资源。

### ISA DMA 助手

- 无效 `snd_dma_program`（无符号长 *dma*、无符号长 *地址*、无符号整数 *大小*、无符号短 *模式*）

  编程 ISA DMA 传输

**参数**

- `unsigned long dma`

  dma 号码

- `unsigned long addr`

  缓冲区的物理地址

- `unsigned int size`

  DMA 传输大小

- `unsigned short mode`

  DMA 传输模式，DMA_MODE_XXX

**描述**

为给定缓冲区编程 ISA DMA 传输。

- 无效 `snd_dma_disable`（无符号长 *dma* ）

  停止 ISA DMA 传输

**参数**

- `unsigned long dma`

  dma 号码

**描述**

停止 ISA DMA 传输。

- unsigned int `snd_dma_pointer`( unsigned long *dma* , unsigned int *size* )

  以字节为单位返回指向 DMA 传输缓冲区的当前指针

**参数**

- `unsigned long dma`

  dma 号码

- `unsigned int size`

  dma 传输大小

**返回**

DMA 传输缓冲区中的当前指针（以字节为单位）。

- int `snd_devm_request_dma`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , int *dma* , const char **name* )

  的托管版本[`request_dma()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.request_dma)

**参数**

- `struct device *dev`

  设备指针

- `int dma`

  dma 号码

- `const char *name`

  请求者的姓名字符串

**描述**

成功返回零，或负错误代码。请求的 DMA 将在通过 devres 解除绑定时自动释放。

### 其他辅助宏

- 无效 `snd_power_ref`（结构 snd_card **卡*）

  取功率控制的参考计数

**参数**

- `struct snd_card *card`

  声卡对象

**描述**

卡的 power_ref 引用用于管理阻止[`snd_power_sync_ref()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_power_sync_ref)操作。此函数增加引用。[`snd_power_unref()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_power_unref)稍后必须适当地调用对方。

- 无效 `snd_power_unref`（结构 snd_card **卡*）

  释放电源控制的参考计数

**参数**

- `struct snd_card *card`

  声卡对象

- 无效 `snd_power_sync_ref`（结构 snd_card **卡*）

  等到卡 power_ref 被释放

**参数**

- `struct snd_card *card`

  声卡对象

**描述**

此函数用于与正在释放的挂起的 power_ref 同步。

- 无效 `snd_card_unref`（结构 snd_card **卡*）

  取消引用卡片对象

**参数**

- `struct snd_card *card`

  要取消引用的卡片对象

**描述**

为通过[`snd_card_ref()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_card_ref)或获取的卡片对象调用此函数[`snd_lookup_minor_data()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_lookup_minor_data)。

- `snd_printk`( *fmt* , *...* )

  printk 包装器

**参数**

- `fmt`

  格式化字符串

- `...`

  可变参数

**描述**

当使用 CONFIG_SND_VERBOSE_PRINTK 配置时，类似[`printk()`](https://www-kernel-org.translate.goog/doc/html/latest/core-api/printk-basics.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.printk)但打印调用者的文件和行。

- `snd_printd`( *fmt* , *...* )

  调试 printk

**参数**

- `fmt`

  格式化字符串

- `...`

  可变参数

**描述**

用于[`snd_printk()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_printk)调试目的。未设置 CONFIG_SND_DEBUG 时忽略。

- `snd_BUG`( )

  给出一个 BUG 警告信息和堆栈跟踪

**参数**

**描述**

如果设置了 CONFIG_SND_DEBUG，则调用 WARN()。未设置 CONFIG_SND_DEBUG 时忽略。

- `snd_printd_ratelimit`( )

  启用 CONFIG_SND_DEBUG 时抑制高输出速率。

**参数**

- `snd_BUG_ON`（*条件*）

  调试检查宏

**参数**

- `cond`

  评估条件

**描述**

Has the same behavior as WARN_ON when CONFIG_SND_DEBUG is set, otherwise just evaluates the conditional and returns the value.

- `snd_printdd`(*format*, *…*)

  debug printk

**Parameters**

- `format`

  format string

- `...`

  variable arguments

**Description**

Works like [`snd_printk()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.snd_printk) for debugging purposes. Ignored when CONFIG_SND_DEBUG_VERBOSE is not set.

- int `register_sound_special_device`(const struct file_operations **fops*, int *unit*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  register a special sound node

**Parameters**

- `const struct file_operations *fops`

  File operations for the driver

- `int unit`

  Unit number to allocate

- `struct device *dev`

  device pointerAllocate a special sound device by minor number from the sound subsystem.

**Return**

- The allocated number is returned on success. On failure,

  a negative error code is returned.

- int `register_sound_mixer`(const struct file_operations **fops*, int *dev*)

  register a mixer device

**Parameters**

- `const struct file_operations *fops`

  File operations for the driver

- `int dev`

  Unit number to allocateAllocate a mixer device. Unit is the number of the mixer requested. Pass -1 to request the next free mixer unit.

**Return**

- On success, the allocated number is returned. On failure,

  a negative error code is returned.

- int `register_sound_dsp`(const struct file_operations **fops*, int *dev*)

  register a DSP device

**Parameters**

- `const struct file_operations *fops`

  File operations for the driver

- `int dev`

  Unit number to allocateAllocate a DSP device. Unit is the number of the DSP requested. Pass -1 to request the next free DSP unit.This function allocates both the audio and dsp device entries together and will always allocate them as a matching pair - eg dsp3/audio3

**Return**

- On success, the allocated number is returned. On failure,

  a negative error code is returned.

- 无效 `unregister_sound_special`（整数 *单位*）

  注销一个特殊的声音设备

**参数**

- `int unit`

  分配的单元号释放使用 register_sound_special() 分配的声音设备。传递的单位是寄存器函数的返回值。

- 无效 `unregister_sound_mixer`（整数 *单位*）

  注销混音器

**参数**

- `int unit`

  分配的单元号释放分配给 的声音设备[`register_sound_mixer()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.register_sound_mixer)。传递的单位是寄存器函数的返回值。

- 无效 `unregister_sound_dsp`（整数 *单位*）

  注销 DSP 设备

**参数**

- `int unit`

  分配的单元号释放分配给 的声音设备[`register_sound_dsp()`](https://www-kernel-org.translate.goog/doc/html/latest/sound/kernel-api/alsa-driver-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.register_sound_dsp)。传递的单位是寄存器函数的返回值。两个分配的单元会自动一起释放。