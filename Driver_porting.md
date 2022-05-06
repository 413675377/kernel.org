# 7步移植Driver

[TOC]

The generic objects must be registered with the driver model core. By doing so, they will exported via the sysfs filesystem. sysfs can be mounted by doing:
```
mount -t sysfs sysfs /sys
```

### 0. Read include/linux/device.h for object and function definitions.
### 1. Registering the bus driver
```
#定义一个bus类型
struct bus_type pci_bus_type = {
      .name           = "pci",
};
EXPORT_SYMBOL(pci_bus_type);

#头文件中声明bus类型
extern struct bus_type pci_bus_type;

#注册和注销bus
static int __init pci_driver_init(void)
{
        return bus_register(&pci_bus_type);
}
static int __exit pci_driver_exit(void)
{
        return bus_unregister(&pci_bus_type);
}

subsys_initcall(pci_driver_init);
```
#做完以上后，Sysfs中会出现
```
# tree -d /sys/bus/pci/
/sys/bus/pci/
|-- devices
`-- drivers
```
### 2. Registering Devices
```
struct pci_dev {
       ...
       struct  device  dev;            /* Generic device interface */
       ...
};
initialize the struct device...
device_register(&dev->dev);
```
#做完以上后，Sysfs中会出现
```
/sys/devices/pci0/
|-- 00:00.0
|-- 00:01.0
|   `-- 01:00.0
|-- 00:02.0
|   `-- 02:1f.0
|       `-- 03:00.0

/sys/bus/pci/devices/
|-- 00:00.0 -> ../../../devices/pci0/00:00.0
|-- 00:01.0 -> ../../../devices/pci0/00:01.0
|-- 00:02.0 -> ../../../devices/pci0/00:02.0
|-- 00:1e.0 -> ../../../devices/pci0/00:1e.0
|-- 00:1f.0 -> ../../../devices/pci0/00:1f.0
|-- 00:1f.1 -> ../../../devices/pci0/00:1f.1
```
### 3. Registering Drivers
```
struct pci_driver {
       ...
       struct device_driver    driver;
};
initializing the struct pci_driver ...
driver_register(&drv->driver);
```
#做完以上后，Sysfs中会出现
```
/sys/bus/pci/drivers/
|-- 3c59x
|-- Ensoniq AudioPCI
|-- agpgart-amdk7
|-- e100
`-- serial
```
### 4. Define Generic Methods for Drivers
```
static int pci_device_remove(struct device * dev)
{
        struct pci_dev * pci_dev = to_pci_dev(dev);
        struct pci_driver * drv = pci_dev->driver;

        if (drv) {
                if (drv->remove)
                        drv->remove(pci_dev);
                pci_dev->driver = NULL;
        }
        return 0;
}
```
### 5. Support generic driver binding
The model assumes that a device or driver can be **dynamically registered** with the bus at any time. When registration happens, devices **must** be bound to a driver, or drivers must be bound to all devices that it supports.
```
int (*match)(struct device * dev, struct device_driver * drv);
```
#做完以上后，Sysfs中会出现
```
/sys/bus/pci/drivers/
|-- 3c59x
|   `-- 00:0b.0 -> ../../../../devices/pci0/00:0b.0
|-- Ensoniq AudioPCI
|-- agpgart-amdk7
|   `-- 00:00.0 -> ../../../../devices/pci0/00:00.0
|-- e100
|   `-- 00:0c.0 -> ../../../../devices/pci0/00:0c.0
`-- serial
```
### 6. Supply a hotplug callback
Whenever a device is registered with the driver model core, the userspace program `/sbin/hotplug` is called to **notify userspace**. Users can define actions to perform when a device is inserted or removed.  
The driver model core passes several arguments to userspace via environment variables, including  
ACTION: set to 'add' or 'remove'  
DEVPATH: set to the device's physical path in sysfs.  

```
int (*hotplug) (struct device *dev, char **envp,
                int num_envp, char *buffer, int buffer_size);
```
### 7. Cleaning up the bus driver
```
int bus_for_each_dev(struct bus_type * bus, struct device * start,
                     void * data, int (*fn)(struct device *, void *));

int bus_for_each_drv(struct bus_type * bus, struct device_driver 
        * start,void * data, int (*fn)(struct device_driver *, void *));
#Please see drivers/base/bus.c for more information.
```

Reference: https://01.org/linuxgraphics/gfx-docs/drm/driver-api/driver-model/porting.html