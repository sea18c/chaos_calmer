hi3518 smartfrog mtd parts old/new


do not use minicom in screen-Session - you'll need CTRL-Z for minicom commands :D

############################################
Everything beyond this line @ your own risk
############################################


# Set kernel param's
setenv 'bootargs mem=64M sensor=ov9712 console=ttyAMA0,115200 root=/dev/mtdblock3 rootfstype=squashfs,jffs2 mtdparts=hi_sfc:512K(boot),64k(env),3M(kernel),5M(rootfs),-(rootfs_data)'
# Set boot command
setenv 'bootcmd sf probe 0;sf read 0x82000000 0x90000 0x300000;bootm 0x82000000'
#Save settings
saveenv
#hisilicon # saveenv
#Saving Environment to SPI Flash...
#Erasing SPI flash, offset 0x00060000 size 64K ...done
#Writing to SPI flash, offset 0x00060000 size 64K ...done
# Prepare flash
sf probe 0
# Unlock flash
sf lock 0

# just for fresh setup for reinstall kernel and rootfs 
# erase all partitions behind "env"
#sf erase 0x90000 0xf70000
#
# Download kernel via console to RAM
loady 0x82000000 
# in minicom start ymodem upload of openwrt-hisilicon-armv5tej-3.0.8-uImage with CTRL-A Z S
## Ready for binary (ymodem) download to 0x82000000 at 115200 bps...
#Cmode, 12830(SOH)/0(STX)/0(CAN) packets, 4 retries
## Total Size      = 0x00190da4 = 1641892 Bytes
# erase env - if needed
# sf erase 0x80000 0x10000
# Erase part of FLASH
# 512K+64k Offset instead of 256K
sf erase 0x90000 0x300000
# Write kernel
sf write 0x82000000 0x90000 0x00190da4 # last param=filesize of upload in hex
# Download rootfs via console to RAM 
loady 0x82000000
# in minicom start ymodem upload of openwrt-hisilicon-armv5tej-3.0.8-root.squashfs with CTRL-A Z S
## Ready for binary (ymodem) download to 0x82000000 at 115200 bps...
#C910(SOH)/0(STX)/0(CAN) packets, 5 retries
## Total Size      = 0x003a69de = 3828190 Bytes

# Erase part of FLASH
# offset for 512K Boot instead of 256K
sf erase 0x390000 0x500000
# Write rootfs to FLASH
# also here - 512K offset instead of 256K
sf write 0x82000000 0x390000 0x003a69de # last param=filesize of upload in hex
# backup copy of env to new partition
#sf erase 0x80000 0x10000
#sf read 0x82000000 0x60000 0x10000
#sf write 0x82000000 0x80000 0x10000
# erase rootfs_data part
sf erase 0x000000890000 0x870000
# Restart
reset


after boot:
flash_eraseall -j /dev/mtd4 # formatting overlayfs


part-table-new:
4 cmdlinepart partitions found on MTD device hi_sfc
Creating 4 MTD partitions on "hi_sfc":
0x000000000000-0x000000080000 : "boot" # 512K 
0x0x0000080000-0x000000090000 :	"env" #64 K
0x000000090000-0x000000390000 : "kernel" #3MB
0x000000390000-0x000000890000 : "rootfs" # 5MB
0x000000890000-				  : "overlay"

bootlog:
[    1.054862] 5 cmdlinepart partitions found on MTD device hi_sfc
[    1.060832] Creating 5 MTD partitions on "hi_sfc":
[    1.065651] 0x000000000000-0x000000080000 : "boot"
[    1.074769] 0x000000080000-0x000000090000 : "env"
[    1.083910] 0x000000090000-0x000000390000 : "kernel"
[    1.093391] 0x000000390000-0x000000890000 : "rootfs"
[    1.102893] 0x000000890000-0x000001000000 : "rootfs_data"


orig smartfrog:
4 cmdlinepart partitions found on MTD device hi_sfc
Creating 4 MTD partitions on "hi_sfc":
0x000000000000-0x000000080000 : "boot"
0x000000080000-0x000000580000 : "rootfs"
0x000000580000-0x000000e80000 : "app"
0x000000e80000-0x000001000000 : "config"
