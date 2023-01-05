## 5 使用示例

### 5.1 使用 pin 的驱动 dts 配置示例

对于使用 pin 的驱动来说，驱动主要设置 pin 的常用的几种功能，列举如下：

*•* 驱动使用者只配置通用 GPIO, 即用来做输入、输出和中断的

*•* 驱动使用者设置 pin 的 pin mux，如 uart 设备的 pin,lcd 设备的 pin 等，用于特殊功能

*•* 驱动使用者既要配置 pin 的通用功能，也要配置 pin 的特性

下面对常见使用场景进行分别介绍。



#### 5.1.1 配置通用 GPIO 功能/中断功能

用法一：配置 GPIO，中断，device tree 配置 demo 如下所示：

```
soc{
    ...
    gpiokey {
        device_type = "gpiokey"; 
        compatible = "gpio-keys";
        
        ok_key {
            device_type = "ok_key";
            label = "ok_key";
            gpios = <&r_pio PL 0x4 0x0 0x1 0x0 0x1>; //如果是linux-5.4，则应该为gpios = <&r_pio 0 4 GPIO_ACTIVE_HIGH>;
            linux,input-type = "1>";
            linux,code = <0x1c>;
            wakeup-source = <0x1>;
            };
        };
    ...
};
```

说明

```
说明：gpio in/gpio out/ interrupt采用dts的配置方法，配置参数解释如下：
对于linux-4.9:
gpios = <&r_pio PL 0x4 0x0 0x1 0x0 0x1>;
            |    |  |   |   |   |   `---输出电平，只有output才有效
            |    |  |   |   |   `-------驱动能力，值为0x0时采用默认值
            |    |  |   |   `-----------上下拉，值为0x1时采用默认值
            |    |  |   `---------------复用类型
            |    |  `-------------------当前bank中哪个引脚
            |    `-----------------------哪个bank
            `---------------------------指向哪个pio，属于cpus要用&r_pio
使用上述方式配置gpio时，需要驱动调用以下接口解析dts的配置参数：
int of_get_named_gpio_flags(struct device_node *np, const char *list_name, int index,
enum of_gpio_flags *flags)
拿到gpio的配置信息后(保存在flags参数中，见4.2.8.小节)，在根据需要调用相应的标准接口实现自己的功能
对于linux-5.4:
gpios = <&r_pio 0 4 GPIO_ACTIVE_HIGH>;
            |   |      |
            |   |      `-------------------gpio active时状态，如果需要上下拉，还可以或上
            GPIO_PULL_UP、GPIO_PULL_DOWN标志
            |   `-----------------------哪个bank
            `---------------------------指向哪个pio，属于cpus要用&r_pio
```



#### 5.1.2 用法二

用法二：配置设备引脚，device tree 配置 demo 如下所示：

```
device tree对应配置
soc{
    pio: pinctrl@0300b000 {
        ...
        uart0_ph_pins_a: uart0-ph-pins-a {
            allwinner,pins = "PH7", "PH8"; 
            allwinner,function = "uart0"; 
            allwinner,muxsel = <3>;
            allwinner,drive = <0x1>;
            allwinner,pull = <0x1>;
        };
        /* 对于linux-5.4 请使用下面这种方式配置 */
        mmc2_ds_pin: mmc2-ds-pin {
            pins = "PC1";
            function = "mmc2";
            drive-strength = <30>;
            bias-pull-up;
        };
        ...
    }；
    ...
    uart0: uart@05000000 {
        compatible = "allwinner,sun8i-uart";
        device_type = "uart0";
        reg = <0x0 0x05000000 0x0 0x400>;
        interrupts = <GIC_SPI 49 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&clk_uart0>;
        pinctrl-names = "default", "sleep";
        pinctrl-0 = <&uart0_pins_a>;
        pinctrl-1 = <&uart0_pins_b>;
        uart0_regulator = "vcc-io";
        uart0_port = <0>;
        uart0_type = <2>;
    };
    ...
};
```

其中：

*•* pinctrl-0 对应 pinctrl-names 中的 default，即模块正常工作模式下对应的 pin 配置

*•* pinctrl-1 对应 pinctrl-names 中的 sleep，即模块休眠模式下对应的 pin 配置



### 5.2 接口使用示例

#### 5.2.1 配置设备引脚

一般设备驱动只需要使用一个接口 devm_pinctrl_get_select_default 就可以申请到设备所有pin 资源。

```c
static int sunxi_pin_req_demo(struct platform_device *pdev)
{ 
	struct pinctrl *pinctrl;
	/* request device pinctrl, set as default state */
	pinctrl = devm_pinctrl_get_select_default(&pdev->dev);
	if (IS_ERR_OR_NULL(pinctrl))
		return -EINVAL;

	return 0;
}
```



#### 5.2.2 获取 GPIO 号 

```
static int sunxi_pin_req_demo(struct platform_device *pdev)
{
    struct device *dev = &pdev->dev;
    struct device_node *np = dev->of_node;
    unsigned int gpio;
    
    #get gpio config in device node.
    gpio = of_get_named_gpio(np, "vdevice_3", 0);
    if (!gpio_is_valid(gpio)) {
    	if (gpio != -EPROBE_DEFER)
    		dev_err(dev, "Error getting vdevice_3\n");
		return gpio;
    }
}
```



#### 5.2.3 GPIO 属性配置

通过 pin_config_set/pin_config_get/pin_config_group_set/pin_config_group_get 接口单独控制指定 pin 或 group 的相关属性。

```
static int pctrltest_request_all_resource(void)
{
    struct device *dev;
    struct device_node *node;
    struct pinctrl *pinctrl;
    struct sunxi_gpio_config *gpio_list = NULL;
    struct sunxi_gpio_config *gpio_cfg;
    unsigned gpio_count = 0;
    unsigned gpio_index;
    unsigned long config;
    int ret;

    dev = bus_find_device_by_name(&platform_bus_type, NULL, sunxi_ptest_data->dev_name);
    if (!dev) {
        pr_warn("find device [%s] failed...\n", sunxi_ptest_data->dev_name);
        return -EINVAL;
    }

    node = of_find_node_by_type(NULL, dev_name(dev));
    if (!node) {
        pr_warn("find node for device [%s] failed...\n", dev_name(dev));
        return -EINVAL;
    }
    dev->of_node = node;

    pr_warn("++++++++++++++++++++++++++++%s++++++++++++++++++++++++++++\n", __func__);
    pr_warn("device[%s] all pin resource we want to request\n", dev_name(dev));
    pr_warn("-----------------------------------------------\n");

    pr_warn("step1: request pin all resource.\n");
    pinctrl = devm_pinctrl_get_select_default(dev);
    if (IS_ERR_OR_NULL(pinctrl)) {
        pr_warn("request pinctrl handle for device [%s] failed...\n", dev_name(dev));
        return -EINVAL;
    }

    pr_warn("step2: get device[%s] pin count.\n", dev_name(dev));
    ret = dt_get_gpio_list(node, &gpio_list, &gpio_count);
    if (ret < 0 || gpio_count == 0) {
        pr_warn(" devices own 0 pin resource or look for main key failed!\n");
        return -EINVAL;
    }

    pr_warn("step3: get device[%s] pin configure and check.\n", dev_name(dev));
    for (gpio_index = 0; gpio_index < gpio_count; gpio_index++) {
        gpio_cfg = &gpio_list[gpio_index];

        /*check function config */
        config = SUNXI_PINCFG_PACK(SUNXI_PINCFG_TYPE_FUNC, 0xFFFF);
        pin_config_get(SUNXI_PINCTRL, gpio_cfg->name, &config);
        if (gpio_cfg->mulsel != SUNXI_PINCFG_UNPACK_VALUE(config)) {
            pr_warn("failed! mul value isn't equal as dt.\n");
            return -EINVAL;
        }

        /*check pull config */
        if (gpio_cfg->pull != GPIO_PULL_DEFAULT) {
            config = SUNXI_PINCFG_PACK(SUNXI_PINCFG_TYPE_PUD, 0xFFFF);
            pin_config_get(SUNXI_PINCTRL, gpio_cfg->name, &config);
            if (gpio_cfg->pull != SUNXI_PINCFG_UNPACK_VALUE(config)) {
                pr_warn("failed! pull value isn't equal as dt.\n");
                return -EINVAL;
            }
        }

        /*check dlevel config */
        if (gpio_cfg->drive != GPIO_DRVLVL_DEFAULT) {
            config = SUNXI_PINCFG_PACK(SUNXI_PINCFG_TYPE_DRV, 0XFFFF);
            pin_config_get(SUNXI_PINCTRL, gpio_cfg->name, &config);
            if (gpio_cfg->drive != SUNXI_PINCFG_UNPACK_VALUE(config)) {
                pr_warn("failed! dlevel value isn't equal as dt.\n");
                return -EINVAL;
            }
        }

        /*check data config */
        if (gpio_cfg->data != GPIO_DATA_DEFAULT) {
            config = SUNXI_PINCFG_PACK(SUNXI_PINCFG_TYPE_DAT, 0XFFFF);
            pin_config_get(SUNXI_PINCTRL, gpio_cfg->name, &config);
            if (gpio_cfg->data != SUNXI_PINCFG_UNPACK_VALUE(config)) {
                pr_warn("failed! pin data value isn't equal as dt.\n");
                return -EINVAL;
            }
        }
    }

    pr_warn("-----------------------------------------------\n");
    pr_warn("test pinctrl request all resource success!\n");
    pr_warn("++++++++++++++++++++++++++++end++++++++++++++++++++++++++++\n\n");
    return 0;
}
注：需要注意，存在SUNXI_PINCTRL和SUNXI_R_PINCTRL两个pinctrl设备，cpus域的pin需要使用
SUNXI_R_PINCTRL
```

**!** 警告

**linux5.4** 中 使 用 **pinctrl_gpio_set_config** 配 置 **gpio** 属 性， 对 应 使 用**pinconf_to_config_pack** 生成 **config** 参数：

*•* **SUNXI_PINCFG_TYPE_FUNC** 已不再生效，暂未支持 **FUNC** 配置（建议使用 **pinctrl_select_state**接口代替）

*•* **SUNXI_PINCFG_TYPE_PUD** 更新为内核标准定义（**PIN_CONFIG_BIAS_PULL_UP/PIN_CONFIG_BIAS_PULL_DOWN**） 

*•* **SUNXI_PINCFG_TYPE_DRV** 更新为内核标准定义（**PIN_CONFIG_DRIVE_STRENGTH**），相应的 **val** 对应关系为（**4.9->5.4: 0->10, 1->20…**） 

*•* **SUNXI_PINCFG_TYPE_DAT** 已不再生效，暂未支持 **DAT** 配置（建议使用 **gpio_direction_output**或者 **__gpio_set_value** 设置电平值）



### 5.3 设备驱动使用 GPIO 中断功能

方式一：通过 gpio_to_irq 获取虚拟中断号，然后调用申请中断函数即可目前 sunxi-pinctrl 使用 irq-domain 为 gpio 中断实现虚拟 irq 的功能，使用 gpio 中断功能时，设备驱动只需要通过 gpio_to_irq 获取虚拟中断号后，其他均可以按标准 irq 接口操作。

```
static int sunxi_gpio_eint_demo(struct platform_device *pdev)
{ 
    struct device *dev = &pdev->dev;
    int virq;
    int ret;
    /* map the virq of gpio */
    virq = gpio_to_irq(GPIOA(0));
    if (IS_ERR_VALUE(virq)) {
	    pr_warn("map gpio [%d] to virq failed, errno = %d\n",
    											GPIOA(0), virq);
        return -EINVAL;
    }
    pr_debug("gpio [%d] map to virq [%d] ok\n", GPIOA(0), virq);
    /* request virq, set virq type to high level trigger */
    ret = devm_request_irq(dev, virq, sunxi_gpio_irq_test_handler,
                                IRQF_TRIGGER_HIGH, "PA0_EINT", NULL);
    if (IS_ERR_VALUE(ret)) {
        pr_warn("request virq %d failed, errno = %d\n", virq, ret);
        return -EINVAL;
    }
    
	return 0;
}
```

方式二：通过 dts 配置 gpio 中断，通过 dts 解析函数获取虚拟中断号，最后调用申请中断函数即可，demo 如下所示：

```
dts配置如下：
soc{
	...
    Vdevice: vdevice@0 {
        compatible = "allwinner,sun8i-vdevice";
        device_type = "Vdevice";
        interrupt-parent = <&pio>; /*依赖的中断控制器(带interrupt-controller属性的结 点)*/
        interrupts = < PD 3 IRQ_TYPE_LEVEL_HIGH>;
                        | |   `------------------中断触发条件、类型
                        | `-------------------------pin bank内偏移
                        `---------------------------哪个bank
        pinctrl-names = "default";
        pinctrl-0 = <&vdevice_pins_a>;
        test-gpios = <&pio PC 3 1 2 2 1>;
        status = "okay";
	};
	...
};
```

在驱动中，通过 platform_get_irq() 标准接口获取虚拟中断号，如下所示：

```
static int sunxi_pctrltest_probe(struct platform_device *pdev)
{ 
    struct device_node *np = pdev->dev.of_node;
    struct gpio_config config;
    int gpio, irq;
    int ret;

    if (np == NULL) {
        pr_err("Vdevice failed to get of_node\n");
        return -ENODEV;
    }
    ....
    irq = platform_get_irq(pdev, 0);
    if (irq < 0) {
        printk("Get irq error!\n");
        return -EBUSY;
    }
	.....
	sunxi_ptest_data->irq = irq;
	......
	return ret;
}

//申请中断：
static int pctrltest_request_irq(void)
{
    int ret;
    int virq = sunxi_ptest_data->irq;
    int trigger = IRQF_TRIGGER_HIGH;

    reinit_completion(&sunxi_ptest_data->done);

    pr_warn("step1: request irq(%s level) for irq:%d.\n",
	    trigger == IRQF_TRIGGER_HIGH ? "high" : "low", virq);
	ret = request_irq(virq, sunxi_pinctrl_irq_handler_demo1,
			trigger, "PIN_EINT", NULL);
    if (IS_ERR_VALUE(ret)) {
        pr_warn("request irq failed !\n");
        return -EINVAL;
    }

    pr_warn("step2: wait for irq.\n");
    ret = wait_for_completion_timeout(&sunxi_ptest_data->done, HZ);
    
    if (ret == 0) {
        pr_warn("wait for irq timeout!\n");
        free_irq(virq, NULL);
        return -EINVAL;
    }

    free_irq(virq, NULL);

    pr_warn("-----------------------------------------------\n");
    pr_warn("test pin eint success !\n");
    pr_warn("+++++++++++++++++++++++++++end++++++++++++++++++++++++++++\n\n\n");

    return 0;
}
```



### 5.4 设备驱动设置中断 debounce 功能

方式一：通过 dts 配置每个中断 bank 的 debounce，以 pio 设备为例，如下所示：

```
&pio {
    /* takes the debounce time in usec as argument */
    input-debounce = <0 0 0 0 0 0 0>;
                      | | | | | | `----------PA bank
                      | | | | | `------------PC bank
                      | | | | `--------------PD bank
                      | | | `----------------PF bank
                      | | `------------------PG bank
                      | `--------------------PH bank
                      `----------------------PI bank
};
```

注意：input-debounce 的属性值中需把 pio 设备支持中断的 bank 都配上，如果缺少，会以bank 的顺序设置相应的属性值到 debounce 寄存器，缺少的 bank 对应的 debounce 应该是默认值（启动时没修改的情况）。sunxi linux-4.9 平台，中断采样频率最大是 24M, 最小 32k，debounce 的属性值只能为 0 或 1。对于 linux-5.4，debounce 取值范围是 0~1000000（单位 usec）。

方式二：驱动模块调用 gpio 相关接口设置中断 debounce 

```
static inline int gpio_set_debounce(unsigned gpio, unsigned debounce);
int gpiod_set_debounce(struct gpio_desc *desc, unsigned debounce);
```

在驱动中，调用上面两个接口即可设置 gpio 对应的中断 debounce 寄存器，注意，debounce 是以 ms 为单位的 (linux-5.4 已经移除这个接口)。