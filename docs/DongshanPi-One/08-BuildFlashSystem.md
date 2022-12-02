# 编译烧写系统
## 编译整个系统

### 检查编译环境

### 指定配置文件
> ./setup_config.sh ./configs/nvr/i2m/8.2.1/spinand.glibc.011a.128

### 生产系统
Release kernel相关资源 （非必须动作，只有当改动到kernel部分config的时候需要执行此步）Nand Flash：
> ./release.sh -k  ${kernel_path} -b 011A -p nvr -f spinand -c i2m -l glibc -v 8.2.1；

执行: make image （project工程不支持多线程编译）

生成image，目录：project/image/output/images 拿到生成的image后，参照烧录文档/母片制作文档烧录到板子即可。

## 烧写镜像

### uboot内烧写
