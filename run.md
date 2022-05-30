```
#!/bin/sh

#Download resource
# https://ftp.denx.de/pub/u-boot/
# https://kernel.org
# https://busybox.net/

jobnum=32

#Reference
#Configure: Documentation/admin-guide/README.rst
#

#Configure platform and compiler tool
export ARCH=arm CROSS_COMPILE=arm-linux-gnueabi-  

#Configure and compile U-boot
make vexpress_ca9x4_defconfig # from configs/vexpress_ca9x4_defconfig
make -j$jobnum                # output: u-boot, u-boot.bin

#Configure and compile kernel + dtb
make vexpress_defconfig      # load board CONFIG, refer to arch/arm/configs/
make zImage dtbs -j$jobnum   # Output: arch/arm/boot/zImage, arch/arm/boot/dts/xxx.dtb                      

#Configure and compile and install busybox
make menuconfig     #设置 Build Options 开启静态编译(提示: 可使用 / 搜索 `static`)
make -j$jobnum
make install        #output: _install

#Create rootfs or system
dd if=/dev/zero of=rootfs.img bs=1024 count=102400  # 创建 100MB 虚拟磁盘
mkfs.ext2 rootfs.img                          # 格式化成 ext2 格式文件系统, fat do not support soft link
sudo mount -o loop rootfs.img /mnt            # 将镜像文件和块设备关联并挂载设备到 /mnt
sudo cp -r _install/* /mnt                    # 将 BUsybox 所有生成的程序拷贝到根目录
sudo mkdir -p /mnt/dev       # 创建4个tty设备（c代表字符设备，4是主设备号，1~4分别是次设备号）
sudo mknod /mnt/dev/tty1 c 4 1 
sudo mknod /mnt/dev/tty2 c 4 2
sudo mknod /mnt/dev/tty3 c 4 3
sudo mknod /mnt/dev/tty4 c 4 4
sudo mknod -m 666 /mnt/dev/console c 5 1  # 创建终端和回收站
sudo mknod -m 666 /mnt/dev/null c 1 3
sudo mkdir -p /etc/init.d/     # Creat first shell /etc/init.d/rcS
echo "echo \"hello world\""/etc/init.d/rcS
sudo cp zImage vexpress-v2p-ca9.dtb /mnt #cp the kernel+dtb in rootfs, to make a system
sudo umount /mnt 

sudo apt-get install qemu-system-arm
#Run uboot only
#qemu-system-arm -M vexpress-a9 -m 256M -nographic -kernel u-boot

#Run kernel only
qemu-system-arm -M vexpress-a9 -m 256M -nographic -kernel zImage  -dtb vexpress-v2p-ca9.dtb

#Run system=dtb + kernel + rootfs(busybox,)
qemu-system-arm -M vexpress-a9 -m 256M -nographic -kernel zImage  -dtb vexpress-v2p-ca9.dtb -sd rootfs.img -append "root=/dev/mmcblk0 rw console=ttyAMA0"

#Run uboot + system
qemu-system-arm -M vexpress-a9 -m 256M -kernel u-boot -sd system.img -nographic
#Manually setting to boot the dtb then load the kernel 
:<<Manual_Boot
=> mmcinfo
=> ls mmc 0:0
=> load mmc 0:0 0x60008000 zImage
=> load mmc 0:0 0x61000000 vexpress-v2p-ca9.dtb
=> setenv bootargs "root=/dev/mmcblk0 rw console=ttyAMA0"
=> bootz 0x60008000 - 0x61000000
Manual_Boot

#Netboot through TFTP
:<<TFTP_Setup
apt-get install tftp-hpa tftpd-hpa #tftp client and server
mkdir ~/tftpboot
chmod 777 ~/tftpboot
cat /etc/default/tftpd-hpa #changes:  TFTP_DIRECTORY=~/tftpboot
cp zImage vexpress-v2p-ca9.dtb ~/tftpboot
touch ~/tftpboot/a.txt
tftp 127.0.0.1
tftp> get a.txt
ifconfig
TFTP_Setup

:<<TFTP_Boot
=> setenv ipaddr 192.168.1.21
=> setenv serverip 192.168.1.20
=> tftp 0x60008000 zImage
=> tftp 0x61000000 vexpress-v2p-ca9.dtb
=> setenv bootargs "root=/dev/mmcblk0 rw console=ttyAMA0"
=> bootz 0x60008000 - 0x61000000
TFTP_Boot


#Run commands in U-boot, please refer to https://www.jianshu.com/p/f7d5b6ad0710
:<<Half_AutoBoot
$ cat boot.cmd
    load mmc 0:0 0x60008000 zImage
    load mmc 0:0 0x61000000 vexpress-v2p-ca9.dtb
    setenv bootargs "root=/dev/mmcblk0 rw console=ttyAMA0"
    bootz 0x60008000 - 0x61000000

mkimage -C none -A arm -T script -d boot.cmd boot.scr
sudo cp boot.scr /mnt

=> load mmc 0:0 0x62000000 boot.scr
=> source 0x62000000   #triger the boot.scr excution
Half_AutoBoot

:<<AutoBoot
# cat boot.cmd
    load mmc 0:1 0x60008000 zImage
    load mmc 0:1 0x61000000 vexpress-v2p-ca9.dtb
    setenv bootargs "root=/dev/mmcblk0p1 rw console=ttyAMA0"
    bootz 0x60008000 - 0x61000000

mkimage -C none -A arm -T script -d boot.cmd boot.scr

dd if=/dev/zero of=sdcard.img bs=1024 count=102400
fdisk sdcard.img                                        # 简单起见这里只分一个区
losetup -f                                              # 查询可用的块设备地址, 记下来如 /dev/loop16
losetup -P /dev/loop16 sdcard.img                       # 关联块设备   
lsblk                                                   # 可以看到块设备已经关联, 并且看到分区信息   
mkfs.ext2  /dev/loop16p1                                 # 格式化第一分区                       
mosudo unt /dev/loop16p1 /mnt                                # 挂载块设备的第一分区
sudo cp boot.scr zImage vexpress-v2p-ca9.dtb /mnt            # 复制到 第一分区 
sudo umount /mnt   
AutoBoot
```
