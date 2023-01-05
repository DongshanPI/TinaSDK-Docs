## 3 接口描述

### 3.1 驱动物理层接口

#### 3.1.1 spi_nor_erase

```
static int spi_nor_erase(struct mtd_info *mtd, struct erase_info *instr)
```

description：mtd erase interface
@mtd： MTD device structure
@instr： erase operation descrition structure
return：success return 0，fail return fail code**

#### 3.1.2 spi_nor_read

```
static int spi_nor_read(struct mtd_info *mtd, loff_t from, size_t len,
size_t *retlen, u_char *buf)
```

description：mtd read interface
@mtd：MTD device structure
@from： offset to read from MTD device
@len： data len
@retlen： had read data len
@buf： data buffer
return：success return max_bitflips，fail return fail code**

#### 3.1.3 spi_nor_write

```
static int spi_nor_write(struct mtd_info *mtd, loff_t to, size_t len,
size_t *retlen, const u_char *buf)
```

description：mtd write data interface
@to： offset to MTD device
@len： want write data len
@retlen：return the writen len
@buf： data buffer
return： success return 0, fail return code fail

#### 3.1.4 spi_nor_lock

```
static int spi_nor_lock(struct mtd_info *mtd, loff_t ofs, uint64_t len)
description：check block is badblock or not
```

@mtd：MTD device structure
@ofs: offset the mtd device start (align to simu block size)
@len：The length of the operating
return： success return 0, fail return code fail

#### 3.1.5 spi_nor_unlock

```
static int spi_nor_unlock(struct mtd_info *mtd, loff_t ofs, uint64_t len)
description：check block is badblock or not
```

@mtd：MTD device structure
@ofs: offset the mtd device start (align to simu block size)
@len：The length of the operating
return： success return 0, fail return code fail

#### 3.1.6 spi_nor_is_locked

```
static int spi_nor_is_locked(struct mtd_info *mtd, loff_t ofs, uint64_t len)
description：check block is badblock or not
```

@mtd：MTD device structure
@ofs: offset the mtd device start (align to simu block size)
@len：The length of the operating
return： Is lock return 1, else return 0

#### 3.1.7 spi_nor_has_lock_erase

```
static int spi_nor_has_lock_erase(struct mtd_info *mtd, struct erase_info *instr)
```

description：mtd has lock erase interface，First unlock to operate space, after the
completion of the flash lock up
@mtd： MTD device structure
@instr： erase operation descrition structure
return：success return 0，fail return fail code

#### 3.1.8 spi_nor_has_lock_write

```
static int spi_nor_has_lock_write(struct mtd_info *mtd, loff_t to, size_t len,
size_t *retlen, const u_char *buf)
```

description：mtd has lock write data interface，First unlock to operate space, after
the completion of the flash lock up
@to： offset to MTD device
@len： want write data len
@retlen：return the writen len
@buf： data buffer
return： success return 0, fail return code fail

### 3.2 Uboot 应用接口

#### 3.2.1 sunxi_flash_spinor_probe

```
static int sunxi_flash_spinor_probe(void)
```

description：SPINOR initialization，Set the storage type。
return：zero on success, else a negative error code.

#### 3.2.2 sunxi_flash_spinor_init

```
static int sunxi_flash_spinor_init(int boot_mode, int res)
```


description：SPINOR initialization。
@boot_mode：Working mode
@res：The default is 0
return：zero on success, else a negative error code.

#### 3.2.3 sunxi_flash_spinor_exit

```
int sunxi_flash_spinor_exit(void)
```


description：Release registration is a resource for applications.
return：zero on success, else a negative error code.

#### 3.2.4 sunxi_flash_spinor_write

```
static int sunxi_flash_spinor_write(uint start_block, uint nblock, void *buffer)
```

description：mtd write data interface.
@start_block：want write start sector
@nblock：want write sectorcount
@buffer：data buffer

return：zero on success, else a negative error code.

#### 3.2.5 sunxi_flash_spinor_write

```
static int sunxi_flash_spinor_write(uint start_block, uint nblock, void *buffer)
```


description：mtd readdata interface.
@start_block：want read start sector
@nblock：want read sector count
@buffer：data buffer
return：zero on success, else a negative error code.

#### 3.2.6 sunxi_flash_spinor_erase

```
static int sunxi_flash_spinor_erase(int erase, void *mbr_buffer)
```

description：erase boot || partition data.
@erase：erase flag
@buffer：The default is NULL
return：zero on success, else a negative error code.

#### 3.2.7 sunxi_flash_spinor_force_erase

```
int sunxi_flash_spinor_force_erase(void)
```


description：erase boot & partition data.
return：zero on success, else a negative error code.

#### 3.2.8 sunxi_flash_spinor_flush

```
int sunxi_flash_spinor_flush(void)
```

description：Flush physical cache data to flash.

return：zero on success, else a negative error code.

#### 3.2.9 sunxi_flash_spinor_download_spl

```
static int sunxi_flash_spinor_download_spl(unsigned char *buf, int len, unsigned int ext)
```

description：write boot0.
@buf：boot0 data buffer
@len：boot0 data len
@ext：storage type
return：zero on success, else a negative error code.

#### 3.2.10 sunxi_flash_spinor_download_toc

```
static int sunxi_flash_spinor_download_toc(unsigned char *buf, int len, unsigned int ext)
```

description：write uboot.
@buf：uboot data buffer
@len：uboot data len
@ext：storage type
return：zero on success, else a negative error code.