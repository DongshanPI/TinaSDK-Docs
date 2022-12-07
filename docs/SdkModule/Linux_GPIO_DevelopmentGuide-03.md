# GPIO-接口说明
## 4 模块接口说明

### 4.1 pinctrl 接口说明

#### 4.1.1 pin4ctrl_get

*•* 函数原型：struct pinctrl *pinctrl_get(struct device *dev);

*•* 作用：获取设备的 pin 操作句柄，所有 pin 操作必须基于此 pinctrl 句柄。

*•* 参数：

* dev: 指向申请 pin 操作句柄的设备句柄。

*•* 返回：

- 成功，返回 pinctrl 句柄。

- 失败，返回 NULL。



#### 4.1.2 pinctrl_put

*•* 函数原型：void pinctrl_put(struct pinctrl *p)

*•* 作用：释放 pinctrl 句柄，必须与 pinctrl_get 配对使用。

*•* 参数：

​    	*•* p: 指向释放的 pinctrl 句柄。

*•* 返回：

​	    *•* 没有返回值。

**!** 警告

必须与 **pinctrl_get** 配对使用。



#### 4.1.3 devm_pinctrl_get

*•* 函数原型：struct pinctrl *devm_pinctrl_get(struct device *dev)

*•* 作用：根据设备获取 pin 操作句柄，所有 pin 操作必须基于此 pinctrl 句柄，与 pinctrl_get功能完全一样，只是 devm_pinctrl_get 会将申请到的 pinctrl 句柄做记录，绑定到设备句柄信息中。设备驱动申请 pin 资源，推荐优先使用 devm_pinctrl_get 接口。

*•* 参数：

​    	*•* dev: 指向申请 pin 操作句柄的设备句柄。

*•* 返回：

​		*•* 成功，返回 pinctrl 句柄。

​		*•* 失败，返回 NULL。



#### 4.1.4 devm_pinctrl_put

*•* 函数原型：void devm_pinctrl_put(struct pinctrl *p)

*•* 作用：释放 pinctrl 句柄，必须与 devm_pinctrl_get 配对使用。

*•* 参数：

​		*•* p: 指向释放的 pinctrl 句柄。

*•* 返回：

​		*•* 没有返回值。

**!** 警告

必须与 **devm_pinctrl_get** 配对使用，可以不显式的调用该接口。



#### 4.1.5 pinctrl_lookup_state

*•* 函数原型：struct pinctrl_state *pinctrl_lookup_state(struct pinctrl *p, const char *name)

*•* 作用：根据 pin 操作句柄，查找 state 状态句柄。

*•* 参数：

​		*•* p: 指向要操作的 pinctrl 句柄。

​		*•* name: 指向状态名称，如 “default”、“sleep” 等。

*•* 返回：

​		*•* 成功，返回执行 pin 状态的句柄 struct pinctrl_state *。 

​	    *•* 失败，返回 NULL。



#### 4.1.6 pinctrl_select_state

*•* 函数原型：int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s)

*•* 作用：将 pin 句柄对应的 pinctrl 设置为 state 句柄对应的状态。

*•* 参数：

​		*•* p: 指向要操作的 pinctrl 句柄。

​		*•* s: 指向 state 句柄。

*•* 返回：

​		*•* 成功，返回 0。 

​		*•* 失败，返回错误码。



#### 4.1.7 devm_pinctrl_get_select

*•* 函数原型：struct pinctrl *devm_pinctrl_get_select(struct device *dev, const char *name)

*•* 作用：获取设备的 pin 操作句柄，并将句柄设定为指定状态。

*•* 参数：

​		*•* dev: 指向管理 pin 操作句柄的设备句柄。

​		*•* name: 要设置的 state 名称，如 “default”、“sleep” 等。

*•* 返回：

​		*•* 成功，返回 pinctrl 句柄。

​		*•* 失败，返回 NULL。



#### 4.1.8 devm_pinctrl_get_select_default

*•* 函数原型：struct pinctrl *devm_pinctrl_get_select_default(struct device *dev)

*•* 作用：获取设备的 pin 操作句柄，并将句柄设定为默认状态。

*•* 参数：

​		*•* dev: 指向管理 pin 操作句柄的设备句柄。

*•* 返回：

​		*•* 成功，返回 pinctrl 句柄。

​		*•* 失败，返回 NULL。



#### 4.1.9 pin_config_get

*•* 作用：获取指定 pin 的属性。

*•* 参数：

​		*•* dev_name: 指向 pinctrl 设备。

​		*•* name: 指向 pin 名称。

​		*•* config: 保存 pin 的配置信息。

*•* 返回：

​		*•* 成功，返回 pin 编号。

​		*•* 失败，返回错误码。

**!** 警告

该接口在 **linux-5.4** 已经移除。



#### 4.1.10 pin_config_set

*•* 作用：设置指定 pin 的属性。

*•* 参数：

​		*•* dev_name: 指向 pinctrl 设备。

​		*•* name: 指向 pin 名称。

​		*•* config:pin 的配置信息。

*•* 返回：

​		*•* 成功，返回 0。 

​		*•* 失败，返回错误码。

**!** 警告

该接口在 **linux-5.4** 已经移除。



### 4.2 gpio 接口说明

#### 4.2.1 gpio_request

*•* 函数原型：int gpio_request(unsigned gpio, const char *label)

*•* 作用：申请 gpio，获取 gpio 的访问权。

*•* 参数：

​		*•* gpio:gpio 编号。

​		*•* label:gpio 名称，可以为 NULL。 

*•* 返回：

​		*•* 成功，返回 0。 

​		*•* 失败，返回错误码。



#### 4.2.2 gpio_free

*•* 函数原型：void gpio_free(unsigned gpio)

*•* 作用：释放 gpio。 

*•* 参数：

​		*•* gpio:gpio 编号。

*•* 返回：

​		*•* 无返回值。



#### 4.2.3 gpio_direction_input

*•* 函数原型：int gpio_direction_input(unsigned gpio)

*•* 作用：设置 gpio 为 input。 

*•* 参数：

​		*•* gpio:gpio 编号。

*•* 返回：

​		*•* 成功，返回 0。 

​		*•* 失败，返回错误码。



#### 4.2.5 __gpio_get_value

*•* 函数原型：int __gpio_get_value(unsigned gpio)

*•* 作用：获取 gpio 电平值 (gpio 已为 input/output 状态)。 

*•* 参数：

​		*•* gpio:gpio 编号。

*•* 返回：

​		*•* 返回 gpio 对应的电平逻辑，1 表示高, 0 表示低。



#### 4.2.6 __gpio_set_value

*•* 函数原型：void __gpio_set_value(unsigned gpio, int value)

*•* 作用：设置 gpio 电平值 (gpio 已为 input/output 状态)。 

*•* 参数：

​		*•* gpio:gpio 编号。

​		*•* value: 期望设置的 gpio 电平值，非 0 表示高, 0 表示低。

*•* 返回：

​		*•* 无返回值



#### 4.2.7 of_get_named_gpio

*•* 函数原型：int of_get_named_gpio(struct device_node *np, const char *propname, int index)

*•* 作用：通过名称从 dts 解析 gpio 属性并返回 gpio 编号。

*•* 参数：

​		*•* np: 指向使用 gpio 的设备结点。

​		*•* propname:dts 中属性的名称。

​		*•* index:dts 中属性的索引值。

*•* 返回：

​		*•* 成功，返回 gpio 编号。

​		*•* 失败，返回错误码。



#### 4.2.8 of_get_named_gpio_flags

*•* 函数原型：int of_get_named_gpio_flags(struct device_node *np, const char *list_name, int index,

enum of_gpio_flags *flags)

*•* 作用：通过名称从 dts 解析 gpio 属性并返回 gpio 编号。

*•* 参数：

​		*•* np: 指向使用 gpio 的设备结点。

​		*•* propname:dts 中属性的名称。

​		*•* index:dts 中属性的索引值

​		*•* flags: 在 sunxi 平台上，必须定义为 struct gpio_config * 类型变量，因为 sunxi pinctrl的 pin 支持上下拉，					 驱动能力等信息，而内核 enum of_gpio_flags * 类型变量只能包含输入、输出信息，后续 sunxi 平台					 需要标准化该接口。

*•* 返回：

​		*•* 成功，返回 gpio 编号。

​		*•* 失败，返回错误码。

**!** 警告

该接口的 **flags** 参数，在 **sunxi linux-4.9** 及以前的平台上，必须定义为 **struct gpio_config** 类型变量。**linux-5.4** 已经标准化该接口，直接采用 **enum of_gpio_flags** 的定义。
