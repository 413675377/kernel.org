# DeviceTree 内核 API

## 核心功能

- struct device_node * `of_find_all_nodes`( struct device_node **prev* )

  获取全局列表中的下一个节点

**参数**

- `struct device_node *prev`

  [`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)将在其上调用上一个节点或开始迭代的 NULL

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- struct device_node * `of_get_cpu_node`( int *cpu* , unsigned int **thread* )

  获取与给定逻辑 CPU 关联的设备节点

**参数**

- `int cpu`

  需要设备节点的 CPU 编号（逻辑索引）

- `unsigned int *thread`

  如果不为 NULL，则返回物理内核中的本地线程号

**描述**

此函数的主要目的是检索给定逻辑 CPU 索引的设备节点。它应该用于初始化 cpu 设备中的 of_node。一旦 cpu 设备中的 of_node 被填充，所有进一步的引用都可以使用它。

CPU 逻辑到物理索引映射是特定于体系结构的，并且是在引导辅助内核之前构建的。此函数使用arch_match_cpu_phys_id，它可以被体系结构特定的实现覆盖。

**返回**

逻辑 cpu 的节点指针，引用计数递增，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。如果未找到，则返回 NULL。

- int `of_cpu_node_to_id`( struct device_node **cpu_node* )

  获取给定 device_node 的逻辑 CPU 编号

**参数**

- `struct device_node *cpu_node`

  指向 CPU 的 device_node 的指针。

**返回**

给定 CPU device_node 的逻辑 CPU 编号，如果未找到 CPU，则为 -ENODEV。

- struct device_node * `of_get_cpu_state_node`( struct device_node **cpu_node* , int *index* )

  在给定索引处获取 CPU 的空闲状态节点

**参数**

- `struct device_node *cpu_node`

  CPU 的设备节点

- `int index`

  空闲状态列表中的索引

**描述**

可以使用两种通用方法来描述 CPU 的空闲状态，通过“cpu-idle-states”绑定的扁平化描述或通过分层布局，使用“power-domains”和“domain-idle-states”绑定。此函数检查两者并返回请求索引的空闲状态节点。

**返回**

如果在**index**找到空闲状态节点。它的 refcount 会增加，所以[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)在完成时调用它。如果未找到，则返回 NULL。

- int `of_machine_is_compatible`( const char **compat* )

  测试给定兼容值的设备树根

**参数**

- `const char *compat`

  在根节点的 compatible 属性中查找的 compatible 字符串。

**返回**

如果根节点在其 compatible 属性中具有给定值，则为正整数。

- bool `of_device_is_available`( const struct device_node **device* )

  检查设备是否可用

**参数**

- `const struct device_node *device`

  检查可用性的节点

**返回**

- 如果 status 属性不存在或设置为“okay”或“ok”，则为真，

  否则为假

- bool `of_device_is_big_endian`( const struct device_node **device* )

  检查设备是否有 BE 寄存器

**参数**

- `const struct device_node *device`

  检查字节顺序的节点

**返回**

- 如果设备具有“big-endian”属性，或者如果内核

  是为 BE 编译的，*并且*设备具有“native-endian”属性。否则返回 false。[`of_device_is_big_endian()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_device_is_big_endian)如果== true，调用者名义上将使用 ioread32be/iowrite32be ，否则使用 readl/writel。

- struct device_node * `of_get_parent`( const struct device_node **node* )

  获取节点的父节点（如果有）

**参数**

- `const struct device_node *node`

  获取父节点的节点

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- 结构设备节点* `of_get_next_parent`（结构设备 *节点\*节点*）

  迭代到节点的父节点

**参数**

- `struct device_node *node`

  获取父节点的节点

**描述**

这就像[`of_get_parent()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_get_parent)它删除传递节点上的引用计数，使其适合遍历节点的父节点。

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- struct device_node * `of_get_next_child`( const struct device_node **node* , struct device_node **prev* )

  迭代一个节点子节点

**参数**

- `const struct device_node *node`

  父节点

- `struct device_node *prev`

  父节点的前一个子节点，或 NULL 获得第一个

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。当 prev 是最后一个孩子时返回 NULL。减少 prev 的引用计数。

- struct device_node * `of_get_next_available_child`( const struct device_node **node* , struct device_node **prev* )

  查找下一个可用的子节点

**参数**

- `const struct device_node *node`

  父节点

- `struct device_node *prev`

  父节点的前一个子节点，或 NULL 获得第一个

**描述**

这个函数类似于[`of_get_next_child()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_get_next_child)，除了它会自动跳过任何禁用的节点（即 status = “disabled”）。

- struct device_node * `of_get_next_cpu_node`( struct device_node **prev* )

  在 cpu 节点上迭代

**参数**

- `struct device_node *prev`

  /cpus 节点的前一个子节点，或 NULL 获得第一个

**描述**

不可用的 CPU（状态属性设置为“fail”或“fail-...”的 CPU）将被跳过。

**返回**

一个 refcount 递增的 cpu 节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。当 prev 是最后一个孩子时返回 NULL。减少 prev 的引用计数。

- struct device_node * `of_get_compatible_child`( const struct device_node **parent* , const char **compatible* )

  查找兼容的子节点

**参数**

- `const struct device_node *parent`

  父节点

- `const char *compatible`

  兼容字符串

**描述**

查找其 compatible 属性包含给定兼容字符串的子节点。

**返回**

一个 refcount 递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它；如果未找到，则为 NULL。

- struct device_node * `of_get_child_by_name`( const struct device_node **node* , const char **name* )

  按名称查找给定父节点的子节点

**参数**

- `const struct device_node *node`

  父节点

- `const char *name`

  要查找的子名。

**描述**

此函数查找给定匹配名称的子节点

**返回**

如果找到一个节点指针，并且 refcount 递增，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。如果未找到节点，则返回 NULL。

- struct device_node * `of_find_node_opts_by_path`( const char **path* , const char ***opts* )

  查找与完整 OF 路径匹配的节点

**参数**

- `const char *path`

  要匹配的完整路径，或者如果路径不以“/”开头，则为 /aliases 节点的属性名称（别名）。在别名的情况下，将返回与别名值匹配的节点。

- `const char **opts`

  一个指针的地址，用于存储附加到路径末尾的选项字符串的开头，并带有“：”分隔符。

**描述**

- 有效路径：

  /foo/bar 完整路径foo 有效别名foo/bar 有效别名 + 相对路径

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- struct device_node * `of_find_node_by_name`( struct device_node **from* , const char **name* )

  通过其“名称”属性查找节点

**参数**

- `struct device_node *from`

  开始搜索的节点或NULL；你通过的节点不会被搜索，只会搜索下一个。通常，您传递上一个调用返回的内容。[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)将从调用**。**

- `const char *name`

  要匹配的名称字符串

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- struct device_node * `of_find_node_by_type`( struct device_node **from* , const char **type* )

  通过其“device_type”属性查找节点

**参数**

- `struct device_node *from`

  开始搜索的节点，或 NULL 开始搜索整个设备树。你经过的节点不会被搜索，只会搜索下一个；通常，您传递上一个调用返回的内容。[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)将为您调用。

- `const char *type`

  要匹配的类型字符串

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- struct device_node * `of_find_compatible_node`( struct device_node **from* , const char **type* , const char **compatible* )

  根据类型和其“兼容”属性中的令牌之一查找节点

**参数**

- `struct device_node *from`

  开始搜索的节点或NULL，不搜索你传递的节点，只搜索下一个；通常，您传递上一个调用返回的内容。[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)将被调用

- `const char *type`

  匹配“device_type”的类型字符串或忽略 NULL

- `const char *compatible`

  与设备“兼容”列表中的令牌之一匹配的字符串。

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- struct device_node * `of_find_node_with_property`( struct device_node **from* , const char **prop_name* )

  查找具有给定名称的属性的节点。

**参数**

- `struct device_node *from`

  开始搜索的节点或NULL，不搜索你传递的节点，只搜索下一个；通常，您传递上一个调用返回的内容。[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)将被调用

- `const char *prop_name`

  要查找的属性的名称。

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- const struct of_device_id * `of_match_node`( const struct of_device_id **matches* , const struct device_node **node* )

  判断一个 device_node 是否有匹配的 of_match 结构

**参数**

- `const struct of_device_id *matches`

  要搜索的设备匹配结构数组

- `const struct device_node *node`

  要匹配的设备结构

**描述**

设备匹配使用的低级实用功能。

- struct device_node * `of_find_matching_node_and_match`( struct device_node **from* , const struct of_device_id **matches* , const struct of_device_id ***match* )

  根据 of_device_id 匹配表查找节点。

**参数**

- `struct device_node *from`

  开始搜索的节点或NULL，不搜索你传递的节点，只搜索下一个；通常，您传递上一个调用返回的内容。[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)将被调用

- `const struct of_device_id *matches`

  要搜索的设备匹配结构数组

- `const struct of_device_id **match`

  更新为指向匹配的匹配项

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- int `of_modalias_node`( struct device_node **node* , char **modalias* , int *len* )

  为设备节点查找适当的模态

**参数**

- `struct device_node *node`

  指向设备树节点的指针

- `char *modalias`

  指向缓冲区的指针，模态变量值将被复制到

- `int len`

  模态值的长度

**描述**

根据 compatible 属性的值，该例程将尝试为特定设备树节点选择适当的模态变量值。它通过从兼容列表属性的第一个条目中去除制造商前缀（由“，”分隔）来实现这一点。

**返回**

此例程在成功时返回 0，在失败时返回 <0。

- struct device_node * `of_find_node_by_phandle`( phandle *句柄*)

  查找给定 phandle 的节点

**参数**

- `phandle handle`

  要查找的节点的phandle

**返回**

引用计数递增的节点指针，[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)完成后使用它。

- int `of_parse_phandle_with_args_map`( const struct device_node **np* , const char **list_name* , const char **stem_name* , int *index* , struct of_phandle_args **out_args* )

  在列表中找到 phandle 指向的节点并重新映射它

**参数**

- `const struct device_node *np`

  指向包含列表的设备树节点的指针

- `const char *list_name`

  包含列表的属性名称

- `const char *stem_name`

  指定 phandles 参数计数的属性名称的词干

- `int index`

  要解析的 phandle 的索引

- `struct of_phandle_args *out_args`

  指向输出参数结构的可选指针（将被填充）

**描述**

此函数可用于解析 phandle 列表及其参数。成功返回 0 并填写 out_args，错误返回适当的 errno 值。此函数与此函数之间的区别在于[`of_parse_phandle_with_args()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_parse_phandle_with_args)，如果 phandle 指向的节点具有 < **stem_name** >-map 属性，则此 API 会重新映射 phandle。

调用者负责调用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)返回的 out_args->np 指针。

例子：

```
phandle1: node1 {
    #list-cells = <2>;
};

phandle2: node2 {
    #list-cells = <1>;
};

phandle3: node3 {
    #list-cells = <1>;
    list-map = <0 &phandle2 3>,
               <1 &phandle2 2>,
               <2 &phandle1 5 1>;
    list-map-mask = <0x3>;
};

node4 {
    list = <&phandle1 1 2 &phandle3 0>;
};
```

要获取`node2`节点的 device_node，您可以调用： of_parse_phandle_with_args(node4, “list”, “list”, 1, `args`);

- int `of_count_phandle_with_args`( const struct device_node **np* , const char **list_name* , const char **cells_name* )

  查找属性中的 phandles 引用数

**参数**

- `const struct device_node *np`

  指向包含列表的设备树节点的指针

- `const char *list_name`

  包含列表的属性名称

- `const char *cells_name`

  指定 phandles 参数计数的属性名称

**返回**

属性中的 phandle + 参数元组的数量。将 phandle 和变量参数列表编码为单个属性是一种典型的模式。参数的数量由 phandle-target 节点中的属性编码。例如，一个 gpios 属性将包含一个 GPIO 指定列表，该列表由一个 phandle 和 1 个或多个参数组成。参数的数量由 phandle 指向的节点中的#gpio-cells 属性确定。

- int `of_add_property`( struct device_node **np* , struct property **prop* )

  向节点添加属性

**参数**

- `struct device_node *np`

  调用方设备节点

- `struct property *prop`

  要添加的属性

- int `of_remove_property`( struct device_node **np* , struct property **prop* )

  从节点中删除属性。

**参数**

- `struct device_node *np`

  调用方设备节点

- `struct property *prop`

  要移除的属性

**描述**

请注意，我们实际上并没有删除它，因为我们已经使用 get-property 给出了指向数据的 who-knows-how-many 指针。相反，我们只是将属性移动到“死属性”列表中，这样就不会再找到它了。

- int `of_alias_get_id`( struct device_node **np* , const char **stem* )

  获取给定 device_node 的别名 id

**参数**

- `struct device_node *np`

  指向给定 device_node 的指针

- `const char *stem`

  给定 device_node 的别名词干

**描述**

该函数遍历查找表以获取给定 device_node 和别名 stem 的别名 id。

**返回**

别名 ID（如果找到）。

- int `of_alias_get_highest_id`( const char **stem* )

  获取给定词干的最高别名 id

**参数**

- `const char *stem`

  要检查的别名茎

**描述**

该函数遍历查找表以获取给定别名词干的最高别名 ID。如果找到，它会返回别名 id。

- bool `of_console_check`( struct device_node **dn* , char **name* , int *index* )

  用于 DT 设置的测试和设置控制台

**参数**

- `struct device_node *dn`

  指向设备节点的指针

- `char *name`

  用于不带索引的首选控制台的名称。前任。“tty”

- `int index`

  用于首选控制台的索引。

**描述**

检查给定的设备节点是否与 /chosen 节点中的 stdout-path 属性匹配。如果是，则将其注册为首选控制台。

**返回**

如果控制台成功设置，则为 TRUE。否则返回 FALSE。

- int `of_map_id`( struct device_node **np* , u32 *id* , const char **map_name* , const char **map_mask_name* , struct device_node ***target* , *u32 \*id_out* )

  通过下游映射转换 ID。

**参数**

- `struct device_node *np`

  根复杂设备节点。

- `u32 id`

  要映射的设备 ID。

- `const char *map_name`

  要使用的地图的属性名称。

- `const char *map_mask_name`

  要使用的掩码的可选属性名称。

- `struct device_node **target`

  指向目标设备节点的可选指针。

- `u32 *id_out`

  optional pointer to receive the translated ID.

**Description**

Given a device ID, look up the appropriate implementation-defined platform ID and/or the target device which receives transactions on that ID, as per the “iommu-map” and “msi-map” bindings. Either of **target** or **id_out** may be NULL if only the other is required. If **target** points to a non-NULL device node pointer, only entries targeting that node will be matched; if it points to a NULL value, it will receive the device node of the first matching target phandle, with a reference held.

**Return**

0 on success or a standard error code on failure.

- struct device_node * `of_parse_phandle`( const struct device_node **np* , const char **phandle_name* , int *index* )

  将 phandle 属性解析为 device_node 指针

**参数**

- `const struct device_node *np`

  指向持有 phandle 属性的设备节点的指针

- `const char *phandle_name`

  持有 phandle 值的属性名称

- `int index`

  对于持有 phandles 表的属性，这是表中的索引

**返回**

引用计数递增的 device_node 指针。完成后使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)它。

- int `of_parse_phandle_with_args`( const struct device_node **np* , const char **list_name* , const char **cells_name* , int *index* , struct of_phandle_args **out_args* )

  在列表中查找 phandle 指向的节点

**参数**

- `const struct device_node *np`

  指向包含列表的设备树节点的指针

- `const char *list_name`

  包含列表的属性名称

- `const char *cells_name`

  指定 phandles 参数计数的属性名称

- `int index`

  要解析的 phandle 的索引

- `struct of_phandle_args *out_args`

  指向输出参数结构的可选指针（将被填充）

**描述**

此函数可用于解析 phandle 列表及其参数。成功返回 0 并填写 out_args，错误返回适当的 errno 值。

调用者负责调用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)返回的 out_args->np 指针。

例子：

```
phandle1: node1 {
    #list-cells = <2>;
};

phandle2: node2 {
    #list-cells = <1>;
};

node3 {
    list = <&phandle1 1 2 &phandle2 3>;
};
```

要获取`node2`节点的 device_node，您可以调用： of_parse_phandle_with_args(node3, “list”, “#list-cells”, 1, `args`);

- int `of_parse_phandle_with_fixed_args`( const struct device_node **np* , const char **list_name* , int *cell_count* , int *index* , struct of_phandle_args **out_args* )

  在列表中查找 phandle 指向的节点

**参数**

- `const struct device_node *np`

  指向包含列表的设备树节点的指针

- `const char *list_name`

  包含列表的属性名称

- `int cell_count`

  phandle 之后的参数单元的数量

- `int index`

  要解析的 phandle 的索引

- `struct of_phandle_args *out_args`

  指向输出参数结构的可选指针（将被填充）

**描述**

此函数可用于解析 phandle 列表及其参数。成功返回 0 并填写 out_args，错误返回适当的 errno 值。

调用者负责调用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)返回的 out_args->np 指针。

例子：

```
phandle1: node1 {
};

phandle2: node2 {
};

node3 {
    list = <&phandle1 0 2 &phandle2 2 3>;
};
```

要获取`node2`节点的 device_node，您可以调用： of_parse_phandle_with_fixed_args(node3, “list”, 2, 1, `args`);

- int `of_property_count_u8_elems`( const struct device_node **np* , const char **propname* )

  计算属性中 u8 元素的数量

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

**描述**

在设备节点中搜索属性并计算其中的 u8 元素的数量。

**返回**

成功时的元素数，如果属性不存在或其长度不匹配 u8 的倍数，则为 -EINVAL，如果属性没有值，则为 -ENODATA。

- int `of_property_count_u16_elems`( const struct device_node **np* , const char **propname* )

  计算属性中 u16 元素的数量

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

**描述**

在设备节点中搜索一个属性并计算其中的 u16 元素的数量。

**返回**

成功时的元素数，如果属性不存在或其长度不匹配 u16 的倍数，则为 -EINVAL，如果属性没有值，则为 -ENODATA。

- int `of_property_count_u32_elems`( const struct device_node **np* , const char **propname* )

  计算属性中 u32 元素的数量

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

**描述**

在设备节点中搜索一个属性并计算其中的 u32 元素的数量。

**返回**

成功时的元素数，如果属性不存在或其长度不匹配 u32 的倍数，则为 -EINVAL，如果属性没有值，则为 -ENODATA。

- int `of_property_count_u64_elems`( const struct device_node **np* , const char **propname* )

  计算属性中 u64 元素的数量

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

**描述**

在设备节点中搜索一个属性并计算其中的 u64 元素的数量。

**返回**

The number of elements on sucess, -EINVAL if the property does not exist or its length does not match a multiple of u64 and -ENODATA if the property does not have a value.

- int `of_property_read_string_array`(const struct device_node **np*, const char **propname*, const char ***out_strs*, size_t *sz*)

  Read an array of strings from a multiple strings property.

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  name of the property to be searched.

- `const char **out_strs`

  output array of string pointers.

- `size_t sz`

  number of array elements to read.

**Description**

Search for a property in a device tree node and retrieve a list of terminated string values (pointer to data, not a copy) in that property.

**Return**

If **out_strs** is NULL, the number of strings in the property is returned.

- int `of_property_count_strings`(const struct device_node **np*, const char **propname*)

  Find and return the number of strings from a multiple strings property.

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  name of the property to be searched.

**Description**

Search for a property in a device tree node and retrieve the number of null terminated string contain in it.

**Return**

The number of strings on success, -EINVAL if the property does not exist, -ENODATA if property does not have a value, and -EILSEQ if the string is not null-terminated within the length of the property data.

- int `of_property_read_string_index`(const struct device_node **np*, const char **propname*, int *index*, const char ***output*)

  Find and read a string from a multiple strings property.

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  name of the property to be searched.

- `int index`

  index of the string in the list of strings

- `const char **output`

  pointer to null terminated return string, modified only if return value is 0.

**Description**

Search for a property in a device tree node and retrieve a null terminated string value (pointer to data, not a copy) in the list of strings contained in that property.

The out_string pointer is modified only if a valid string can be decoded.

**Return**

0 表示成功，-EINVAL 如果属性不存在，-ENODATA 如果属性没有值，-EILSEQ 如果字符串在属性数据的长度内不是以空值结尾的。

- bool `of_property_read_bool`( const struct device_node **np* , const char **propname* )

  寻找房产

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

**描述**

在设备节点中搜索属性。

**返回**

如果属性存在则为 true，否则为 false。

- int `of_property_read_u8_array`( const struct device_node **np* , const char **propname* , u8 **out_values* , size_t *sz* )

  从属性中查找并读取 u8 数组。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u8 *out_values`

  指向返回值的指针，仅在返回值为 0 时修改。

- `size_t sz`

  要读取的数组元素数

**描述**

在设备节点中搜索属性并从中读取 8 位值。

- 数组的 dts 条目应如下所示：

  `property = /bits/ 8 <0x50 0x60 0x70>;`

仅当可以解码有效的 u8 值时，才会修改 out_values。

**返回**

0 表示成功，-EINVAL 如果属性不存在，-ENODATA 如果属性没有值，-EOVERFLOW 如果属性数据不够大。

- int `of_property_read_u16_array`( const struct device_node **np* , const char **propname* , u16 **out_values* , size_t *sz* )

  从属性中查找并读取 u16 数组。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u16 *out_values`

  指向返回值的指针，仅在返回值为 0 时修改。

- `size_t sz`

  要读取的数组元素数

**描述**

在设备节点中搜索属性并从中读取 16 位值。

- 数组的 dts 条目应如下所示：

  `property = /bits/ 16 <0x5000 0x6000 0x7000>;`

The out_values is modified only if a valid u16 value can be decoded.

**Return**

0 on success, -EINVAL if the property does not exist, -ENODATA if property does not have a value, and -EOVERFLOW if the property data isn’t large enough.

- int `of_property_read_u32_array`(const struct device_node **np*, const char **propname*, u32 **out_values*, size_t *sz*)

  Find and read an array of 32 bit integers from a property.

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  name of the property to be searched.

- `u32 *out_values`

  pointer to return value, modified only if return value is 0.

- `size_t sz`

  number of array elements to read

**Description**

Search for a property in a device node and read 32-bit value(s) from it.

仅当可以解码有效的 u32 值时，才会修改 out_values。

**返回**

0 表示成功，-EINVAL 如果属性不存在，-ENODATA 如果属性没有值，-EOVERFLOW 如果属性数据不够大。

- int `of_property_read_u64_array`( const struct device_node **np* , const char **propname* , u64 **out_values* , size_t *sz* )

  从属性中查找并读取 64 位整数数组。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u64 *out_values`

  指向返回值的指针，仅在返回值为 0 时修改。

- `size_t sz`

  要读取的数组元素数

**描述**

在设备节点中搜索属性并从中读取 64 位值。

仅当可以解码有效的 u64 值时，才会修改 out_values。

**返回**

0 表示成功，-EINVAL 如果属性不存在，-ENODATA 如果属性没有值，-EOVERFLOW 如果属性数据不够大。

- 结构 `of_changeset_entry`

  持有变更集条目

**定义**

```
struct of_changeset_entry {
  struct list_head node;
  unsigned long action;
  struct device_node *np;
  struct property *prop;
  struct property *old_prop;
};
```

**成员**

- `node`

  日志列表的 list_head

- `action`

  通知人行动

- `np`

  指向受影响设备节点的指针

- `prop`

  指向受影响属性的指针

- `old_prop`

  持有指向原始属性的指针

**描述**

在变更集期间对设备树的每次修改都保存在 of_changeset_entry 结构列表中。这样我们可以从部分应用程序中恢复，或者我们可以恢复变更集

- 结构 `of_changeset`

  变更集跟踪器结构

**定义**

```
struct of_changeset {
  struct list_head entries;
};
```

**成员**

- `entries`

  变更集条目的 list_head

**描述**

变更集是将批量更改应用于活动树的便捷方式。如果出现错误，更改会回滚。变更集在初始应用后继续存在，如果在使用后没有销毁，它们可以在一次调用中恢复。

- bool `of_device_is_system_power_controller`( const struct device_node **np* )

  判断是否为 device_node 找到了 system-power-controller

**参数**

- `const struct device_node *np`

  指向给定 device_node 的指针

**返回**

如果存在则为真，否则为假

- bool `of_graph_is_present`( const struct device_node **node* )

  检查图表的存在

**参数**

- `const struct device_node *node`

  指向包含图形端口的 device_node 的指针

**返回**

如果**节点**有一个或多个端口（带有端口）子节点，则为真，否则为假。

- int `of_property_count_elems_of_size`( const struct device_node **np* , const char **propname* , int *elem_size* )

  计算属性中的元素数量

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `int elem_size`

  单个元素的大小

**描述**

在设备节点中搜索一个属性并计算其中大小为 elem_size 的元素的数量。

**返回**

成功时的元素数，如果属性不存在或其长度不匹配 elem_size 的倍数，则为 -EINVAL，如果属性没有值，则为 -ENODATA。

- int `of_property_read_u32_index`( const struct device_node **np* , const char **propname* , u32 *index* , *u32 \*out_value* )

  从多值属性中查找并读取 u32。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u32 index`

  值列表中 u32 的索引

- `u32 *out_value`

  指向返回值的指针，仅在没有错误时修改。

**描述**

在设备节点中搜索属性并从中读取第 n 个 32 位值。

仅当可以解码有效的 u32 值时，才会修改 out_value。

**返回**

0 表示成功，-EINVAL 如果属性不存在，-ENODATA 如果属性没有值，-EOVERFLOW 如果属性数据不够大。

- int `of_property_read_u64_index`(const struct device_node **np*, const char **propname*, u32 *index*, u64 **out_value*)

  Find and read a u64 from a multi-value property.

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  name of the property to be searched.

- `u32 index`

  index of the u64 in the list of values

- `u64 *out_value`

  pointer to return value, modified only if no error.

**Description**

Search for a property in a device node and read nth 64-bit value from it.

The out_value is modified only if a valid u64 value can be decoded.

**Return**

0 on success, -EINVAL if the property does not exist, -ENODATA if property does not have a value, and -EOVERFLOW if the property data isn’t large enough.

- int `of_property_read_variable_u8_array`( const struct device_node **np* , const char **propname* , u8 **out_values* , size_t *sz_min* , size_t *sz_max* )

  从属性中查找并读取 u8 数组，其中包含最小和最大数组大小的界限。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u8 *out_values`

  指向找到的值的指针。

- `size_t sz_min`

  要读取的最小数组元素数

- `size_t sz_max`

  要读取的数组元素的最大数量，如果为零，则 dts 条目中的元素数量没有上限，但只会读取 sz_min。

**描述**

在设备节点中搜索属性并从中读取 8 位值。

- 数组的 dts 条目应如下所示：

  `property = /bits/ 8 <0x50 0x60 0x70>;`

仅当可以解码有效的 u8 值时，才会修改 out_values。

**返回**

成功时读取的元素数，如果属性不存在则为 -EINVAL，如果属性没有值则为 -ENODATA，如果属性数据小于 sz_min 或长于 sz_max，则为 -EOVERFLOW。

- int `of_property_read_variable_u16_array`( const struct device_node **np* , const char **propname* , u16 **out_values* , size_t *sz_min* , size_t *sz_max* )

  从属性中查找并读取一个 u16 数组，其中包含最小和最大数组大小的界限。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  name of the property to be searched.

- `u16 *out_values`

  pointer to found values.

- `size_t sz_min`

  minimum number of array elements to read

- `size_t sz_max`

  maximum number of array elements to read, if zero there is no upper limit on the number of elements in the dts entry but only sz_min will be read.

**Description**

Search for a property in a device node and read 16-bit value(s) from it.

- dts entry of array should be like:

  `property = /bits/ 16 <0x5000 0x6000 0x7000>;`

The out_values is modified only if a valid u16 value can be decoded.

**Return**

The number of elements read on success, -EINVAL if the property does not exist, -ENODATA if property does not have a value, and -EOVERFLOW if the property data is smaller than sz_min or longer than sz_max.

- int `of_property_read_variable_u32_array`( const struct device_node **np* , const char **propname* , u32 **out_values* , size_t *sz_min* , size_t *sz_max* )

  从属性中查找并读取一个 32 位整数数组，其中包含最小和最大数组大小的界限。

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u32 *out_values`

  返回找到值的指针。

- `size_t sz_min`

  要读取的最小数组元素数

- `size_t sz_max`

  要读取的数组元素的最大数量，如果为零，则 dts 条目中的元素数量没有上限，但只会读取 sz_min。

**描述**

在设备节点中搜索属性并从中读取 32 位值。

仅当可以解码有效的 u32 值时，才会修改 out_values。

**返回**

成功时读取的元素数，如果属性不存在则为 -EINVAL，如果属性没有值则为 -ENODATA，如果属性数据小于 sz_min 或长于 sz_max，则为 -EOVERFLOW。

- int `of_property_read_u64`( const struct device_node **np* , const char **propname* , u64 **out_value* )

  从属性中查找并读取 64 位整数

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `u64 *out_value`

  指向返回值的指针，仅在返回值为 0 时修改。

**描述**

在设备节点中搜索属性并从中读取 64 位值。

The out_value is modified only if a valid u64 value can be decoded.

**Return**

0 on success, -EINVAL if the property does not exist, -ENODATA if property does not have a value, and -EOVERFLOW if the property data isn’t large enough.

- int `of_property_read_variable_u64_array`(const struct device_node **np*, const char **propname*, u64 **out_values*, size_t *sz_min*, size_t *sz_max*)

  Find and read an array of 64 bit integers from a property, with bounds on the minimum and maximum array size.

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  name of the property to be searched.

- `u64 *out_values`

  pointer to found values.

- `size_t sz_min`

  minimum number of array elements to read

- `size_t sz_max`

  maximum number of array elements to read, if zero there is no upper limit on the number of elements in the dts entry but only sz_min will be read.

**Description**

Search for a property in a device node and read 64-bit value(s) from it.

The out_values is modified only if a valid u64 value can be decoded.

**Return**

The number of elements read on success, -EINVAL if the property does not exist, -ENODATA if property does not have a value, and -EOVERFLOW if the property data is smaller than sz_min or longer than sz_max.

- int `of_property_read_string`(const struct device_node **np*, const char **propname*, const char ***out_string*)

  Find and read a string from a property

**Parameters**

- `const struct device_node *np`

  device node from which the property value is to be read.

- `const char *propname`

  要搜索的属性的名称。

- `const char **out_string`

  指向以 null 结尾的返回字符串的指针，仅当返回值为 0 时才修改。

**描述**

在设备树节点中搜索属性并检索以空结尾的字符串值（指向数据的指针，而不是副本）。

仅当可以解码有效字符串时，才会修改 out_string 指针。

**返回**

0 表示成功，-EINVAL 如果属性不存在，-ENODATA 如果属性没有值，-EILSEQ 如果字符串在属性数据的长度内不是以空值结尾的。

- int `of_property_match_string`( const struct device_node **np* , const char **propname* , const char **string* )

  在列表中查找字符串并返回索引

**参数**

- `const struct device_node *np`

  指向包含字符串列表属性的节点的指针

- `const char *propname`

  字符串列表属性名称

- `const char *string`

  指向要在字符串列表中搜索的字符串的指针

**描述**

此函数搜索字符串列表属性并返回特定字符串值的索引。

- int `of_property_read_string_helper`( const struct device_node **np* , const char **propname* , const char ***out_strs* , size_t *sz* , int *skip* )

  用于解析字符串属性的实用程序助手

**参数**

- `const struct device_node *np`

  要从中读取属性值的设备节点。

- `const char *propname`

  要搜索的属性的名称。

- `const char **out_strs`

  字符串指针的输出数组。

- `size_t sz`

  要读取的数组元素的数量。

- `int skip`

  要在列表开头跳过的字符串数。

**描述**

不要直接调用这个函数。它是 of_property_read_string*() 系列函数的实用助手。

- int `of_graph_parse_endpoint`( const struct device_node **node* , struct [of_endpoint ](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_endpoint) **endpoint* )

  解析公共端点节点属性

**参数**

- `const struct device_node *node`

  指向端点 device_node 的指针

- `struct of_endpoint *endpoint`

  指向 OF 端点数据结构的指针

**描述**

调用者应该持有对**node**的引用。

- struct device_node * `of_graph_get_port_by_id`( struct device_node **parent* , u32 *id* )

  获取与给定 id 匹配的端口

**参数**

- `struct device_node *parent`

  指向父设备节点的指针

- `u32 id`

  端口号

**返回**

引用计数递增的“端口”节点指针。调用者必须[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)在完成后使用它。

- struct device_node * `of_graph_get_next_endpoint`( const struct device_node **parent* , struct device_node **prev* )

  获取下一个端点节点

**参数**

- `const struct device_node *parent`

  指向父设备节点的指针

- `struct device_node *prev`

  前一个端点节点，或 NULL 获得第一

**返回**

引用计数递增的“端点”节点指针。**传递的prev**节点的Refcount递减。

- struct device_node * `of_graph_get_endpoint_by_regs`( const struct device_node **parent* , int *port_reg* , int *reg* )

  获取特定标识符的端点节点

**参数**

- `const struct device_node *parent`

  指向父设备节点的指针

- `int port_reg`

  父端口节点的标识符（reg 属性的值）

- `int reg`

  端点节点的标识符（reg 属性的值）

**返回**

由 reg 标识的“端点”节点指针同时是由 port_reg 标识的端口节点的子节点。reg 和 port_reg 为 -1 时将被忽略。完成后在指针上使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)。

- struct device_node * `of_graph_get_remote_endpoint`( const struct device_node **node* )

  获取远程端点节点

**参数**

- `const struct device_node *node`

  指向本地端点 device_node 的指针

**返回**

- 与链接的远程端点节点关联的远程端点节点

  到**节点**。完成后使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)它。

- 结构设备节点* `of_graph_get_port_parent`（结构设备 *节点\*节点*）

  获取端口的父节点

**参数**

- `struct device_node *node`

  指向本地端点 device_node 的指针

**返回**

- 与链接的端点节点关联的设备节点

  到**节点**。完成后使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)它。

- struct device_node * `of_graph_get_remote_port_parent`( const struct device_node **node* )

  获取远程端口的父节点

**参数**

- `const struct device_node *node`

  指向本地端点 device_node 的指针

**返回**

- 与链接的远程端点节点关联的远程设备节点

  到**节点**。完成后使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)它。

- struct device_node * `of_graph_get_remote_port`( const struct device_node **node* )

  获取远程端口节点

**参数**

- `const struct device_node *node`

  指向本地端点 device_node 的指针

**返回**

与链接到 node的远程端点节点关联的远程端口**节点**。完成后使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)它。

- struct device_node * `of_graph_get_remote_node`( const struct device_node **node* , u32 *端口*, u32 *端点*)

  获取给定端口/端点的远程父设备节点

**参数**

- `const struct device_node *node`

  指向包含图形端口/端点的父设备节点的指针

- `u32 port`

  父端口节点的标识符（reg 属性的值）

- `u32 endpoint`

  端点节点的标识符（reg 属性的值）

**返回**

与链接到 node的远程端点节点关联的远程设备**节点**。完成后使用[`of_node_put()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_node_put)它。

- 结构 `of_endpoint`

  OF图端点数据结构

**定义**

```
struct of_endpoint {
  unsigned int port;
  unsigned int id;
  const struct device_node *local_node;
};
```

**成员**

- `port`

  此端点所属端口的标识符（reg 属性的值）

- `id`

  此端点的标识符（reg 属性的值）

- `local_node`

  指向此端点的 device_node 的指针

- `for_each_endpoint_of_node`（*父母*， *孩子*）

  遍历设备节点中的每个端点

**参数**

- `parent`

  包含端口和端点的父设备节点

- `child`

  指向当前端点节点的循环变量

**描述**

跳出循环时，必须手动调用 of_node_put(child)。

- int `of_address_to_resource`( struct device_node **dev* , int *index* , struct resource **r* )

  翻译设备树地址并作为资源返回

**参数**

- `struct device_node *dev`

  调用方设备节点

- `int index`

  索引到数组中

- `struct resource *r`

  指向资源数组的指针

**描述**

请注意，如果您的地址是 PIO 地址，如果物理地址无法在内部使用 pci_address_to_pio() 转换为 IO 令牌，则转换将失败，这是因为它要么调用得太早，要么无法匹配到任何主机桥IO空间

- void __iomem * `of_iomap`( struct device_node **np* , int *index* )

  映射给定 device_node 的内存映射 IO

**参数**

- `struct device_node *np`

  将映射其 io 范围的设备

- `int index`

  io范围的索引

**描述**

返回指向映射内存的指针

- bool `of_dma_is_coherent`( struct device_node **np* )

  检查设备是否一致

**参数**

- `struct device_node *np`

  设备节点

**描述**

如果在 DT 中找到此设备的“dma-coherent”属性，或者默认情况下对于当前平台上的 OF 设备，DMA 是一致的，则返回 true。

- unsigned int `irq_of_parse_and_map`( struct device_node **dev* , int *index* )

  解析并将中断映射到 linux virq 空间

**参数**

- `struct device_node *dev`

  待映射中断的设备的设备节点

- `int index`

  要映射的中断索引

**描述**

这个函数是一个包装器，它链接[`of_irq_parse_one()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_irq_parse_one)和 irq_create_of_mapping() 以使调用者更容易

- struct device_node * `of_irq_find_parent`( struct device_node **child* )

  给定一个设备节点，找到它的中断父节点

**参数**

- `struct device_node *child`

  指向设备节点的指针

**返回**

指向中断父节点的指针，如果无法确定中断父节点，则为 NULL。

- int `of_irq_parse_raw`( const __be32 **addr* , struct of_phandle_args **out_irq* )

  低级中断树解析

**参数**

- `const __be32 *addr`

  地址说明符（设备“reg”属性的开始），be32 格式

- `struct of_phandle_args *out_irq`

  由该函数更新的结构 of_phandle_args

**描述**

该函数是一个低级中断树遍历函数。它可用于使用合成的 reg 和中断属性进行部分遍历，例如在父节点不存在设备节点时解决 PCI 中断时。它将中断说明符结构作为输入，遍历树以查找任何中断映射属性，翻译每个映射的说明符，然后返回翻译后的映射。

**返回**

0 表示成功，负数表示错误

- int `of_irq_parse_one`( struct device_node **device* , int *index* , struct of_phandle_args **out_irq* )

  解决设备的中断

**参数**

- `struct device_node *device`

  要解决其中断的设备

- `int index`

  要解决的中断索引

- `struct of_phandle_args *out_irq`

  此函数填充的结构 of_phandle_args

**描述**

此函数通过遍历中断树、查找它连接到哪个中断控制器节点并返回可用于检索 Linux IRQ 号的中断说明符来解决节点的中断。

- int `of_irq_to_resource`( struct device_node **dev* , int *index* , struct resource **r* )

  解码节点的 IRQ 并将其作为资源返回

**参数**

- `struct device_node *dev`

  指向设备树节点的指针

- `int index`

  irq 的从零开始的索引

- `struct resource *r`

  指向要返回结果的资源结构的指针。

- int `of_irq_get`( struct device_node **dev* , int *index* )

  Decode a node’s IRQ and return it as a Linux IRQ number

**Parameters**

- `struct device_node *dev`

  pointer to device tree node

- `int index`

  zero-based index of the IRQ

**Return**

Linux IRQ number on success, or 0 on the IRQ mapping failure, or -EPROBE_DEFER if the IRQ domain is not yet created, or error code in case of any other failure.

- int `of_irq_get_byname`(struct device_node **dev*, const char **name*)

  Decode a node’s IRQ and return it as a Linux IRQ number

**Parameters**

- `struct device_node *dev`

  pointer to device tree node

- `const char *name`

  IRQ name

**Return**

Linux IRQ number on success, or 0 on the IRQ mapping failure, or -EPROBE_DEFER if the IRQ domain is not yet created, or error code in case of any other failure.

- int `of_irq_to_resource_table`(struct device_node **dev*, struct resource **res*, int *nr_irqs*)

  Fill in resource table with node’s IRQ info

**Parameters**

- `struct device_node *dev`

  pointer to device tree node

- `struct resource *res`

  array of resources to fill in

- `int nr_irqs`

  **IRQ 的数量（以及res**元素数量的上限）

**返回**

填写表格的大小（最多**nr_irqs**）。

- 无效 `of_msi_configure`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*， 结构设备节点 **np* ）

  设置设备的 msi_domain 字段

**参数**

- `struct device *dev`

  与 MSI irq 域关联的设备结构

- `struct device_node *np`

  该设备的设备节点

- void * `of_fdt_unflatten_tree`( const unsigned long **blob* , struct device_node **dad* , struct device_node ***mynodes* )

  从平面 blob 创建 device_nodes 树

**参数**

- `const unsigned long *blob`

  平面设备树 blob

- `struct device_node *dad`

  父设备节点

- `struct device_node **mynodes`

  调用创建的设备树

**描述**

展开固件传递的设备树，创建 struct device_node 的树。它还填充节点的“名称”和“类型”指针，因此可以使用正常的设备树遍历功能。

**返回**

失败时为 NULL 或成功时包含未展平设备树的内存块。

## 驱动模型函数

- int `of_driver_match_device`（结构 [设备](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*，常量结构设备驱动 程序 **drv* ）

  判断驱动程序的 of_match_table 是否与设备匹配。

**参数**

- `struct device *dev`

  要匹配的设备结构

- `const struct device_driver *drv`

  要测试的 device_driver 结构

- const struct of_device_id * `of_match_device`( const struct of_device_id **matches* , const struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* )

  判断 a 是否匹配 of_device_id 列表[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

**参数**

- `const struct of_device_id *matches`

  要搜索的设备匹配结构数组

- `const struct device *dev`

  要匹配的设备结构

**描述**

驱动程序使用它来检查系统中存在的 platform_device 是否在其支持的设备列表中。

- int `of_dma_configure_id`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , struct device_node **np* , bool *force_dma* , const u32 **id* )

  设置 DMA 配置

**参数**

- `struct device *dev`

  应用 DMA 配置的设备

- `struct device_node *np`

  指向具有 DMA 配置的 OF 节点的指针

- `bool force_dma`

  即使固件没有明确描述 DMA 功能，是否由 of_dma_configure() 设置设备。

- `const u32 *id`

  可选的 const 指针值输入 id

**描述**

尝试从 DT 获取设备的 DMA 配置并进行相应的更新。

如果平台代码需要使用自己特殊的 DMA 配置，它可以使用平台总线通知器并处理 BUS_NOTIFY_ADD_DEVICE 事件来修复 DMA 配置。

- ssize_t `of_device_modalias`( struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev* , char **str* , ssize_t *len* )

  用换行符终止的模态字符串填充缓冲区

**参数**

- `struct device *dev`

  呼叫设备

- `char *str`

  模态字符串

- `ssize_t len`

  **str**的大小

- 结构 `of_dev_auxdata`

  设备名称和平台数据的查找表条目

**定义**

```
struct of_dev_auxdata {
  char *compatible;
  resource_size_t phys_addr;
  char *name;
  void *platform_data;
};
```

**成员**

- `compatible`

  与节点匹配的节点的兼容值

- `phys_addr`

  与节点匹配的寄存器起始地址

- `name`

  为匹配节点分配的名称

- `platform_data`

  为匹配节点分配的平台数据

**描述**

此查找表允许调用者在[`of_platform_populate()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_platform_populate)从设备树创建设备时覆盖设备的名称。该表应以空条目终止。它还允许设置 platform_data 指针。

这个功能的原因是一些 Linux 基础设施使用设备名称来查找特定的设备，但是 Linux 特定的名称没有编码到设备树中，因此内核需要提供特定的值。

**笔记**

在将平台转换为使用 DT 时，应将使用 auxdata 查找表视为最后的手段。通常自动生成的设备名称无关紧要，驱动程序应该从设备节点获取数据，而不是从匿名的 platform_data 指针获取数据。

- 结构平台设备* `of_find_device_by_node`（结构设备节点 **np* ）

  查找与节点关联的 platform_device

**参数**

- `struct device_node *np`

  指向设备树节点的指针

**描述**

引用使用后需要删除的嵌入。[`struct device`](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device)

**返回**

platform_device 指针，如果未找到，则为 NULL

- struct platform_device * `of_device_alloc`( struct device_node **np* , const char **bus_id* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* )

  分配并初始化一个 of_device

**参数**

- `struct device_node *np`

  分配给设备的设备节点

- `const char *bus_id`

  分配给设备的名称。可以为 null 以使用默认名称。

- `struct device *parent`

  父设备。

- struct platform_device * `of_platform_device_create`( struct device_node **np* , const char **bus_id* , struct [device ](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent* )

  分配、初始化和注册一个of_device

**参数**

- `struct device_node *np`

  指向要为其创建设备的节点的指针

- `const char *bus_id`

  分配设备的名称

- `struct device *parent`

  Linux 设备模型父设备。

**返回**

指向已创建平台设备的指针，如果设备未注册，则为 NULL。不可用的设备将不会被注册。

- int `of_platform_bus_probe`(struct device_node **root*, const struct of_device_id **matches*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent*)

  Probe the device-tree for platform buses

**Parameters**

- `struct device_node *root`

  parent of the first level to probe or NULL for the root of the tree

- `const struct of_device_id *matches`

  match table for bus nodes

- `struct device *parent`

  parent to hook devices from, NULL for toplevel

**Description**

Note that children of the provided root are not instantiated as devices unless the specified root itself matches the bus list and is not NULL.

- int `of_platform_populate`(struct device_node **root*, const struct of_device_id **matches*, const struct [of_dev_auxdata](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_dev_auxdata) **lookup*, struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent*)

  Populate platform_devices from device tree data

**Parameters**

- `struct device_node *root`

  parent of the first level to probe or NULL for the root of the tree

- `const struct of_device_id *matches`

  match table, NULL to use the default

- `const struct of_dev_auxdata *lookup`

  auxdata table for matching id and platform_data with device nodes

- `struct device *parent`

  parent to hook devices from, NULL for toplevel

**Description**

Similar to [`of_platform_bus_probe()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_platform_bus_probe), this function walks the device tree and creates devices from nodes. It differs in that it follows the modern convention of requiring all device nodes to have a ‘compatible’ property, and it is suitable for creating devices which are children of the root node (of_platform_bus_probe will only create children of the root which are selected by the **matches** argument).

New board support should be using this function instead of [`of_platform_bus_probe()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_platform_bus_probe).

**Return**

0 on success, < 0 on failure.

- void `of_platform_depopulate`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **parent*)

  Remove devices populated from device tree

**Parameters**

- `struct device *parent`

  device which children will be removed

**Description**

Complementary to [`of_platform_populate()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_platform_populate), this function removes children of the given device (and, recurrently, their children) that have been created from their respective device tree nodes (and only those, leaving others - eg. manually created - unharmed).

- int `devm_of_platform_populate`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  Populate platform_devices from device tree data

**Parameters**

- `struct device *dev`

  device that requested to populate from device tree data

**Description**

Similar to [`of_platform_populate()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_platform_populate), but will automatically call [`of_platform_depopulate()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_platform_depopulate) when the device is unbound from the bus.

**Return**

0 on success, < 0 on failure.

- void `devm_of_platform_depopulate`(struct [device](https://www-kernel-org.translate.goog/doc/html/latest/driver-api/infrastructure.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.device) **dev*)

  Remove devices populated from device tree

**Parameters**

- `struct device *dev`

  请求从设备树数据中删除的设备

**描述**

作为 的补充[`devm_of_platform_populate()`](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.devm_of_platform_populate)，此函数删除已从它们各自的设备树节点创建的给定设备的子设备（以及它们的子设备）（并且仅删除那些设备，而其他设备 - 例如手动创建 - 不会受到伤害）。

## 叠加和动态 DT 功能

- int `of_resolve_phandles`( struct device_node **overlay* )

  重新定位并解决针对活树的覆盖

**参数**

- `struct device_node *overlay`

  指向设备树覆盖以重定位和解析的指针

**描述**

**修改（重定位）覆盖**中本地 phandle 的值到不与实时扩展设备树冲突的范围。**在overlay**中更新对本地 phandles 的引用。更新（解析）**覆盖**中引用实时扩展设备树的 phandle 引用。

活树中的 Phandle 值在 1 .. live_tree_max_phandle() 的范围内。叠加层中的 phandle 值范围也从 1 开始。将叠加层中的 phandle 值调整为从 live_tree_max_phandle() + 1 开始。将对 phandle 的引用更新为调整后的 phandle 值。

叠加层中“__fixups__”节点中每个属性的名称与活动树中符号（标签）的名称相匹配。“__fixups__”节点中每个属性的值是覆盖层中需要更新的属性值列表，以包含与活动树中该符号对应的 phandle 引用。使用活动树中的 phandle 值更新覆盖中的引用。

**覆盖层**必须分离。

解析和应用**覆盖**到实时扩展的设备树必须由一种机制保护，以确保以单线程方式处理多个覆盖，以便多个覆盖不会将 phandle 重新定位到重叠范围。执行此操作的机制尚未实施。

**返回**

`0`成功或错误的负错误值。

- 结构设备节点* `of_node_get`（结构设备 *节点\*节点*）

  增加节点的引用计数

**参数**

- `struct device_node *node`

  Node to inc refcount，支持NULL以简化调用者的编写

**返回**

引用计数增加的节点。

- 无效 `of_node_put`（结构设备 *节点\*节点*）

  减少节点的引用计数

**参数**

- `struct device_node *node`

  Node to dec refcount, NULL 支持简化调用者的编写

- int `of_detach_node`(结构 device_node **np* )

  从设备树中“拔出”一个节点。

**参数**

- `struct device_node *np`

  指向调用者设备节点的指针

- 无效 `of_changeset_init`（结构 [of_changeset ](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_changeset) **ocs* ）

  初始化变更集以供使用

**参数**

- `struct of_changeset *ocs`

  变更集指针

**描述**

初始化变更集结构

- 无效 `of_changeset_destroy`（结构 [of_changeset ](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_changeset) **ocs* ）

  销毁变更集

**参数**

- `struct of_changeset *ocs`

  变更集指针

**描述**

销毁变更集。请注意，如果应用了变更集，则无法恢复其对树的更改。

- int `of_changeset_apply`( struct [of_changeset ](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_changeset) **ocs* )

  应用变更集

**参数**

- `struct of_changeset *ocs`

  变更集指针

**描述**

将变更集应用于活动树。实时树状态更改的任何副作用都会在此处应用成功，例如创建/销毁设备以及创建 sysfs 属性和目录等副作用。

**返回**

成功时为 0，如果发生错误，则为负错误值。出错时，部分应用的效果将被恢复。

- int `of_changeset_revert`( struct [of_changeset ](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_changeset) **ocs* )

  恢复应用的变更集

**参数**

- `struct of_changeset *ocs`

  变更集指针

**描述**

恢复一个变更集，将树的状态返回到应用程序之前的状态。应用任何副作用，例如创建/销毁设备以及删除 sysfs 属性和目录。

**返回**

成功时为 0，如果发生错误，则为负错误值。

- int `of_changeset_action`( struct [of_changeset ](https://www-kernel-org.translate.goog/doc/html/latest/devicetree/kernel-api.html?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp#c.of_changeset) **ocs* , unsigned long *action* , struct device_node **np* , struct property **prop* )

  在变更集列表的尾部添加一个操作

**参数**

- `struct of_changeset *ocs`

  变更集指针

- `unsigned long action`

  执行的动作

- `struct device_node *np`

  指向设备节点的指针

- `struct property *prop`

  指向属性的指针

**描述**

动作为以下之一：+ OF_RECONFIG_ATTACH_NODE + OF_RECONFIG_DETACH_NODE，+ OF_RECONFIG_ADD_PROPERTY + OF_RECONFIG_REMOVE_PROPERTY，+ OF_RECONFIG_UPDATE_PROPERTY

**返回**

成功时为 0，如果发生错误，则为负错误值。

- int `of_overlay_notifier_register`( struct notifier_block **nb* )

  为覆盖操作注册通知程序

**参数**

- `struct notifier_block *nb`

  通知程序块注册

**描述**

注册设备树节点上的覆盖操作通知。**由of_reconfig_change 定义**的报告操作。通知回调还接收指向受影响设备树节点的指针。

Note that a notifier callback is not supposed to store pointers to a device tree node or its content beyond **OF_OVERLAY_POST_REMOVE** corresponding to the respective node it received.

- int `of_overlay_notifier_unregister`(struct notifier_block **nb*)

  Unregister notifier for overlay operations

**Parameters**

- `struct notifier_block *nb`

  Notifier block to unregister

- int `of_overlay_remove`(int **ovcs_id*)

  Revert and free an overlay changeset

**Parameters**

- `int *ovcs_id`

  Pointer to overlay changeset id

**Description**

Removes an overlay if it is permissible. **ovcs_id** was previously returned by of_overlay_fdt_apply().

如果在尝试还原覆盖变更集时发生错误，则尝试重新应用已还原的任何变更集条目。如果重新应用时发生错误，则无法确定设备树的状态，并且任何后续尝试应用或删除覆盖变更集都将被拒绝。

- 如果错误来自：非零返回值将不会恢复变更集：

  参数检查覆盖变更集预删除通知程序覆盖变更集条目还原

如果覆盖变更集预删除通知程序返回错误，则不会调用进一步的覆盖变更集预删除通知程序。

如果多个通知器返回错误，则返回最后发生的通知器错误。

- 如果错误来自：非零返回值将恢复变更集：

  覆盖变更集条目通知器覆盖变更集删除后通知程序

如果覆盖变更集删除后通知程序返回错误，则不会调用进一步的覆盖变更集删除后通知程序。

**返回**

0 表示成功，或负错误号。***ovcs_id**在还原变更集后设置为零，即使发生后续错误。

- 整数 `of_overlay_remove_all`（无效 ）

  恢复并释放所有覆盖变更集

**参数**

- `void`

  没有争论

**描述**

以正确的顺序从系统中删除所有覆盖。

**返回**

0 表示成功，或负错误号