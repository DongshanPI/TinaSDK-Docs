# 3 测试环境搭建

1、研发人员打开dragonMAT 目录下的global.ini 文件，根据《dragonMAT 使用说明文档》中2.1 节，结合测试需求对dragonMAT 进行配置，修改后保存。

2、研发人员配置好Tina & tinatest 后，编译出固件，并烧写到TF 卡。

3、研发人员将tina/out/< 方案名称，如: tulip-noma>/staging_dir/target/rootfs/etc/tinatest.json放到PC 端指定目录，方便工人加载使用。