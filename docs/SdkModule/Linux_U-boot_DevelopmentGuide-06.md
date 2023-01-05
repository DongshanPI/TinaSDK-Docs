## 8 常用接口函数

### 8.1 fdt 相关接口

1. const void \*fdt_getprop(const void \*fdt, int nodeoffset, const char \*name, int \*lenp)

*•* 作用：检索指定属性的值

*•* 参数：

​	*•* fdt: 工作 flattened device tree

​	*•* nodeoffset: 待修改节点的偏移

​	*•* name: 待检索的属性名

​	*•* lenp: 检索属性值的长度（会被覆盖）或者为 NULL

*•* 返回：

​	*•* 非空（属性值的指针）：成功

​	*•* NULL（lenp 为空）：失败

​	*•* 失败代码（lenp 非空）：失败



2. int fdt_set_node_status(void \*fdt, int nodeoffset, enum fdt_status status, unsigned int error_code)

*•* 作用：设置节点状态

*•* 参数：

​	*•* fdt: 工作 flattened device tree

​	*•* nodeoffset: 待修改节点的偏移

​	*•* status:FDT_STATUS_OKAY, FDT_STATUS_DISABLED, FDT_STATUS_FAIL, FDT_STATUS_FAIL_ERROR_CODE

​	*•* error_code:optional, only used if status is FDT_STATUS_FAIL_ERROR_CODE

*•* 返回：

​	*•* 0: 成功

​	*•* 非 0: 失败



3. int fdt_path_offset(const void \*fdt, const char \*path)

*•* 作用：通过全路径查找节点的偏移量

*•* 参数：

​	*•* fdt: 工作 fdt

​	*•* path: 全路径名称

*•* 返回：

​	*•* >=0(节点的偏移量): 成功

​	*•* <0: 失败代码



4. static inline int fdt_setprop_u32(void \*fdt, int nodeoffset, const char \*name, uint32_t val)

*•* 作用：将属性值设置为一个 32 位整型数值，如果属性值不存在，则新建该属性

*•* 参数：

​	*•* fdt: 工作 flattened device tree

​	*•* nodeoffset: 待修改节点的偏移

​	*•* name: 待修改的属性名

​	*•* val:32 位目标值

*•* 返回：

​	*•* 0: 成功

​	*•* <0: 失败代码



5. static inline int fdt_setprop_u64(void \*fdt, int nodeoffset, const char \*name, uint64_t val)

*•* 作用：与fdt_setprop_u32类似，将属性值设置为一个 64 位整型数值，如果属性值不存在，则新建该属性

*•* 参数：

​	*•* fdt: 工作 flattened device tree

​	*•* nodeoffset: 待修改节点的偏移

​	*•* name: 待修改的属性名

​	*•* val:64 位目标值

*•* 返回：

​	*•* 0: 成功

​	*•* <0: 失败代码



6. \#define fdt_setprop_string(fdt, nodeoffset, name, str) fdt_setprop((fdt), (nodeoffset), (name), (str), strlen(str)+1)

*•* 作用：将属性值设置为一个字符串，如果属性值不存在，则新建该属性

*•* 参数：

​	*•* fdt: 工作 flattened device tree

​	*•* nodeoffset: 待修改节点的偏移

​	*•* name: 待修改的属性名

​	*•* str: 目标值

*•* 返回：

​	*•* 0: 成功

​	*•* <0: 失败代码

注意：在sys_config.fex的配置中，节点的启用状态为 0 或 1。转换到 fdt 中对应的 status 属性为disable或okay。



7. int save_fdt_to_flash(void \*fdt_buf, size_t fdt_size)

*•* 作用：保存修改到 flash

*•* 参数：

​	*•* fdt_buf: 当前工作 flattened device tree

​	*•* fdt_size: 当前工作 flattened device tree 的大小，可以通过fdt_totalsize(fdt_buf )获取

*•* 返回：

​	*•* 0: 成功

​	*•* <0: 失败



8. 应用参考

U-Boot 中 fdt 命令行的实现：cmd/fdt.c



### 8.2 env 相关接口函数

1. int env_set(const char \*varname, const char \*varvalue)

*•* 作用：将环境变量 varname 的值设置为 varvalue，重启失效

*•* 参数：

​	*•* varname: 待设置环境变量的名称		

​	*•* varvalue: 将指定的环境变量修改为该值

*•* 返回：

​	*•* 0: 成功

​	*•* 非 0: 失败



2. char \*env_get(const char \*name)

*•* 作用：获取指定环境变量的值

*•* 参数：

​	*•* name: 变量名称

*•* 返回：

​	*•* NULL: 失败

​	*•* 非空（环境变量的值）：成功



3. int env_save(void)

*•* 作用：保存环境变量，重启仍保存

*•* 参数: 无 

*•* 返回：

​	*•* 0: 成功

​	*•* 非 0: 失败



4. 应用参考

board/sunxi/sunxi_bootargs.c update_bootargs通过 cmdline 向 kernel 提供信息，主要是通过更新bootargs变量实现env_set(\"bootargs\", cmdline)。



### 8.3 调用 U-Boot 命令行

1. int run_command_list(const char \*cmd, int len, int flag)

*•* 作用：执行 U-Boot 命令行

*•* 参数：

​	*•* cmd: 命令字符指针

​	*•* len: 命令行长度，设置为-1 则自动获取

​	*•* flag: 任意，因为 sunxi 中没有用到

*•* 返回：

​	*•* 0: 成功

​	*•* 非 0: 失败



2. 应用参考：

common/autoboot.c autoboot_command实现了 U-Boot 的自动启动命令

s = env_get(\"bootcmd\");

run_command_list(s, -1, 0)。



### 8.4 Flash 的读写

1. int sunxi_flash_read(uint start_block, uint nblock, void \*buffer)

*•* 作用：将指定起始位置start_block的nblock读取到buffer

*•* 参数：

​	*•* start_block: 起始地址

​	*•* nblock:block 个数

​	*•* buffer: 内存地址

*•* 返回：

​	*•* 0: 成功

​	*•* 非 0: 失败



2. int sunxi_flash_write(uint start_block, uint nblock, void \*buffer)

*•* 作用：将buffer写入指定起始位置start_block的nblock中 

*•* 参数：

​	*•* start_block: 起始地址

​	*•* nblock:block 个数

​	*•* buffer: 内存地址

*•* 返回：

​	*•* 0: 成功

​	*•* 非 0: 失败



3. int sunxi_sprite_read(uint start_block, uint nblock, void \*buffer)

*•* 作用与sunxi_flash_read相似



4. int sunxi_sprite_write(uint start_block, uint nblock, void \*buffer)

*•* 作用与sunxi_flash_write相似



5. 应用参考

common/sunxi/board_helper.c sunxi_set_bootcmd_from_mis实现了对 misc 分区的读写操作





### 8.5 获取分区信息

1. int sunxi_partition_get_partno_byname(const char \*part_name)

*•* 作用：根据分区名称获取分区号

*•* 参数：

​	*•* part_name: 分区名称

*•* 返回：

​	*•* <0: 失败

​	*•* >0（分区号）：成功



2. int sunxi_partition_get_info_byname(const char \*part_name, uint \*part_offset, uint \*part_size) 

*•* 作用：根据分区名称获取分区的偏移量和大小

*•* 参数：

​	*•* part_name: 分区名称

​	*•* part_offset: 分区的偏移量

​	*•* part_size: 分区的大小

*•* 返回：

​	*•* 0: 成功

​	*•* -1: 失败



3. uint sunxi_partition_get_offset_byname(const char \*part_name)

*•* 作用：根据分区名称获取偏移量

*•* 参数：

​	*•* part_name: 分区名称

*•* 返回：

​	*•* <=0 : 失败

​	*•* >0 : 成功



4. int sunxi_partition_get_info(const char \*part_name, disk_partition_t \*info)

*•* 作用：根据part_name获取分区信息

*•* 参数：

​	*•* part_name: 分区名称

​	*•* info: 分区信息

*•* 返回：

​	*•* 非 0: 失败

​	*•* 0: 成功



5. lbaint_t sunxi_partition_get_offset(int part_index)

*•* 作用：card sprite 模式下获取分区的偏移量

*•* 参数：

​	*•* part_index: 分区号

*•* 返回：

​	*•* >=0（偏移量）：成功

​	*•* -1: 失败



6. 应用参考

启动时加载图片：drivers/video/sunxi/logo_display/sunxi_load_bmp.c



### 8.6 GPIO 相关操作

1. int fdt_get_one_gpio(const char\* node_path, const char\* prop_name,user_gpio_set_t\* gpio_list)

*•* 作用：根据路径node_path和 gpio 名称prop_name获取 gpio 配置

*•* 参数：

​	*•* node_path：fdt 路径

​	*•* prop_name：gpio 名称

​	*•* gpio_list：待获取的 gpio 信息

*•* 返回：

​	*•* 0：成功

​	*•* -1：失败



2. ulong sunxi_gpio_request(user_gpio_set_t *gpio_list, __u32 group_count_max)

*•* 作用：根据 gpio 配置获取 gpio 操作句柄

*•* 参数：

​	*•* gpio_list：gpio 配置列表，可以由fdt_get_one_gpio获得

​	*•* group_count_max: gpio_list中最大的 gpio 配置个数

*•* 返回：

​	*•* 0：失败

​	*•* >0（gpio 操作句柄）：成功



3. \__s32 gpio_write_one_pin_value(ulong p_handler, __u32 value_to_gpio, const char *gpio_name)

*•* 作用：根据 gpio 操作句柄写数据

*•* 参数：

​	*•* p_handler：gpio 操作句柄，可由sunxi_gpio_request获取

​	*•* value_to_gpio：待写入数据，0 或 1 

​	*•* gpio_name：gpio 名称

*•* 返回：

​	*•* EGPIO_SUCCESS：成功

​	*•* EGPIO_FAIL：失败



4. 应用参考

操作 led 状态：

```
ssprite/sprite_led.c

user_gpio_set_t gpio_init;

fdt_get_one_gpio("/soc/card_boot", "sprite_gpio0", &gpio_init); //获取/soc/card_boot中sprite_gpio0的gpio配置

sprite_led_hd = sunxi_gpio_request(&gpio_init, 1); //获取gpio操作句柄

gpio_write_one_pin_value(sprite_led_hd, sprite_led_status, "sprite_gpio0"); //操作led状态
```



## 9 常用资源的初始化阶段

*•* env ：环境变量初始化后可以访问

*•* fdt ：在 U-Boot 运行开始即可访问

*•* malloc ：在重定位后才能访问