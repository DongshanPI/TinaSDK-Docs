# 7 打包流程总结

(1) 最终打包生成固件的工具是dragon。

(2)dragon 工具需要2 个配置文件image.cfg，sys_partition.fex。

(3)dragon 工具就是根据image.cfg 和sys_partition.fex 描述进行固件文件的打包。

(4) 整个打包流程实质上就是在处理image.cfg 和sys_partition.fex 里描述的文件。

(5) 整个打包流程可以简单理解为下面3 个步骤：

​	• 生成或拷贝image.cfg 和sys_partition.fex 描述的文件。

​	• 对描述的文件进行一些中间处理，例如更新一些配置到文件里面等。

​	• 用dragon 工具生成最终固件。