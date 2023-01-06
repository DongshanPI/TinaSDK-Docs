## 4 附录

### 4.1 Linux-4.x/Linux-5.4 Gadget 配置示例

#### 4.1.1 小机做 mass storage

```
dd if=/dev/zero of=/dev/a.bin bs=1M count=100
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x18d1" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x0001" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0
echo /dev/a.bin > /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/lun.0/file
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/ /sys/kernel/config/
usb_gadget/g1/configs/c.1/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```

说明

**如果需要增加 lun，在 functions/mass_storage.usb0 下：**

**mkdir lun.1**

**mkdir lun.2**



#### 4.1.2 小机做 cdrom

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1f3a" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0xa4ac" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0
echo 1 > /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/lun.0/cdrom
echo /tmp/phoenixcard.iso > /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/
lun.0/file
ln -s /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/ /sys/kernel/config/
usb_gadget/g1/configs/c.1/mass_storage.usb0
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```

说明

**/tmp/phoenixcard.iso  根据实际情况更改。**



#### 4.1.4 小机做 UAC2

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1d61" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x0101" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/uac2.usb0
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/uac2.usb0/ /sys/kernel/config/usb_gadget/
g1/configs/c.1/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.5 小机做 UVC

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1f3a" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x100d" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0
mkdir -p /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p
echo 1280 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/wWidth
echo 720 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/
wHeight
echo 333333 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/
dwFrameInterval
echo 333333 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/
dwDefaultFrameInterval
echo 442368000 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p
/dwMinBitRate
echo 442368000 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p
/dwMaxBitRate
echo 1843200 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/
dwMaxVideoFrameBufferSize
mkdir /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h/
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/streaming/class/fs
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/streaming/class/hs
mkdir /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/control/header/h
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/control/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/control/class/fs/
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/control/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/control/class/ss/
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/ /sys/kernel/config/usb_gadget/g1
/configs/c.1/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.6 小机做 HID

```
mount -t configfs none /sys/kernel/config/
mkdir /sys/kernel/config/usb_gadget/g1
echo 0x0525 >/sys/kernel/config/usb_gadget/g1/idVendor
echo 0xa4ac >/sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/hid.usb0
echo 512 >/sys/kernel/config/usb_gadget/g1/functions/hid.usb0/report_length
echo -ne <report_desc> >/sys/kernel/config/usb_gadget/g1/functions/hid.usb0/report_desc
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 >/sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 >/sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/hid.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```

说明

**report_desc 根据需求自定义。**



#### 4.1.7 小机做 rndis

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1f3a" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x200a" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/rndis.usb0
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/rndis.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/rndis.usb0
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.8 小机做 acm

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1f3a" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x0007" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/acm.usb0
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/acm.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/acm.usb0
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.9 小机做 adb

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x18d1" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x0002" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
echo "20080411" > /sys/kernel/config/usb_gadget/g1/strings/0x409/serialnumber
echo "Android" > /sys/kernel/config/usb_gadget/g1/strings/0x409/manufacturer
mkdir /sys/kernel/config/usb_gadget/g1/functions/ffs.adb
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/ffs.adb/ /sys/kernel/config/usb_gadget/g1/
configs/c.1/ffs.adb
mkdir /dev/usb-ffs
mkdir /dev/usb-ffs/adb
mount -o uid=2000,gid=2000 -t functionfs adb /dev/usb-ffs/adb/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.10 小机做 mass storage+adb

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x18d1" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x0003" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
echo "20080411" > /sys/kernel/config/usb_gadget/g1/strings/0x409/serialnumber
echo "Android" > /sys/kernel/config/usb_gadget/g1/strings/0x409/manufacturer
mkdir /sys/kernel/config/usb_gadget/g1/functions/ffs.adb
mkdir /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0
echo ${BLOCK_PATH} > /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/lun.0/file
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/ /sys/kernel/config/
usb_gadget/g1/configs/c.1/mass_storage.usb0
ln -s /sys/kernel/config/usb_gadget/g1/functions/ffs.adb/ /sys/kernel/config/usb_gadget/g1/
configs/c.1/ffs.adb
mkdir /dev/usb-ffs
mkdir /dev/usb-ffs/adb
mount -o uid=2000,gid=2000 -t functionfs adb /dev/usb-ffs/adb/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.11 小机做 uvc+uac1

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1f3a" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x100d" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/uac1.usb0
mkdir /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0
mkdir -p /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p
echo 1280 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/wWidth
echo 720 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/wHeight
echo 333333 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/dwFrameInterval
echo 333333 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/dwDefaultFrameInterval
echo 442368000 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/dwMinBitRate
echo 442368000 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/dwMaxBitRate
echo 1843200 > /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/720p/dwMaxVideoFrameBufferSize
mkdir /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/mjpeg/m/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h/
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/streaming/class/fs
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/streaming/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/streaming/class/hs
mkdir /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/control/header/h
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/control/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/control/class/fs/
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/control/header/h/ /sys/kernel/
config/usb_gadget/g1/functions/uvc.usb0/control/class/ss/
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/uvc.usb0/ /sys/kernel/config/usb_gadget/g1
/configs/c.1/
ln -s /sys/kernel/config/usb_gadget/g1/functions/uac1.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.12 小机做 hid+cdrom

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x1f3a" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0xa4ac" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 > /sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 > /sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/hid.usb0
mkdir /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0
echo 512 >/sys/kernel/config/usb_gadget/g1/functions/hid.usb0/report_length
echo -ne <report_desc> >/sys/kernel/config/usb_gadget/g1/functions/hid.usb0/report_desc
ln -s /sys/kernel/config/usb_gadget/g1/functions/hid.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/hid.usb0
echo 1 > /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/lun.0/cdrom
echo /tmp/phoenixcard.iso > /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/lun.0/file
ln -s /sys/kernel/config/usb_gadget/g1/functions/mass_storage.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/mass_storage.usb0
ls /sys/class/udc/ | xargs echo > /sys/kernel/config/usb_gadget/g1/UDC
```



#### 4.1.13 小机做 rndis+adb

```
mount -t configfs none /sys/kernel/config
mkdir /sys/kernel/config/usb_gadget/g1
echo "0x18d1" > /sys/kernel/config/usb_gadget/g1/idVendor
echo "0x0010" > /sys/kernel/config/usb_gadget/g1/idProduct
mkdir /sys/kernel/config/usb_gadget/g1/strings/0x409
mkdir /sys/kernel/config/usb_gadget/g1/functions/ffs.adb
mkdir /sys/kernel/config/usb_gadget/g1/functions/rndis.usb0
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1
echo 0xc0 >/sys/kernel/config/usb_gadget/g1/configs/c.1/bmAttributes
echo 500 >/sys/kernel/config/usb_gadget/g1/configs/c.1/MaxPower
mkdir /sys/kernel/config/usb_gadget/g1/configs/c.1/strings/0x409
ln -s /sys/kernel/config/usb_gadget/g1/functions/rndis.usb0/ /sys/kernel/config/usb_gadget/g1/configs/c.1/rndis.usb0
ln -s /sys/kernel/config/usb_gadget/g1/functions/ffs.adb/ /sys/kernel/config/usb_gadget/g1/configs/c.1/ffs.adb
mkdir /dev/usb-ffs
mkdir /dev/usb-ffs/adb
mount -o uid=2000,gid=2000 -t functionfs adb /dev/usb-ffs/adb/
ls/sys/class/udc/|xargs echo>/sys/kernel/config/usb_gadget/g1/UDC
```



