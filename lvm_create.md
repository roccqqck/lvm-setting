# LVM create

參考
http://linux.vbird.org/linux_basic/0420quota/0420quota-centos5.php#lvm

https://linoxide.com/how-extend-resize-lvm-partition-linux/

https://www.itread01.com/content/1549887872.html



 ```
 df -h 
 
 lsblk

 fdisk -l 
 fdisk -l /dev/sda
```

root
```
 pvdisplay
 vgdisplay
 lvdispplay
```




# 1.創建Create a physical partition

```
fdisk /dev/sdb
```
```
Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```
```
Command (m for help): n
```

* e: extend, p: 
* primary partition
```
Partition type: p 
```
if create sdb1: 1 , if create sdb2: 2
```
Selected partition：1
```
First cylinder 硬碟起始點
```
First cylinder：不用輸，直接enter (default)
```
Last cylinder 硬碟終點
```
Last cylinder, +cylinders or +size{K,M,G}：+20G
```
存檔w或不存q
```
Command (m for help): w
```



# 2.創建pv

確認你要設定的硬碟分割區 例如 /dev/sdb1
```
fdisk -l /dev/sdb1
```
```
pvcreate /dev/sdb1
```
```
pvdisplay
```
```
  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               datavg
  PV Size               <150.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              38399
  Free PE               0
  Allocated PE          38399
  PV UUID               cLrxE7-8n17-jVpH-N0WF-C8sG-is4C-xRF2tC
```






# 3.創建vg

vgcreate [-s N[mgt]] VG名稱 PV名稱
```
vgcreate -s 100G datavg /dev/sdb1
```
全部空間
```
vgcreate -s 100G datavg /dev/sdb1
```
```
vgdisplay
```
```
  --- Volume group ---
  VG Name               datavg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <150.00 GiB
  PE Size               4.00 MiB
  Total PE              38399
  Alloc PE / Size       38399 / <150.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               cwLBGc-6LrX-H0Xw-Vadh-Ggvt-0By4-aqP00c

```



# 4.創建lv

lvcreate [-L N[mgt]] [-n LV名稱] VG名稱
```
lvcreate -L 100G -n datalv datavg
```
```
lvdisplay
```
```
  --- Logical volume ---
  LV Path                /dev/datavg/datalv
  LV Name                datalv
  VG Name                datavg
  LV UUID                na8skp-seoQ-7eA0-P3bP-N3r6-YXqt-jA3eb3
  LV Write Access        read/write
  LV Creation host, time TL-NEXUS01-AP, 2021-08-18 16:01:50 +0800
  LV Status              available
  # open                 1
  LV Size                <150.00 GiB
  Current LE             38399
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2

```





格式化lv
```
mkfs.xfs /dev/datavg/datalv
```

創你要mount的資料夾
```
mkdir /data
```

# 5.掛載
```
mount /dev/datavg/datalv /data
```

確定有掛載成功
```
df -T
```
```
Filesystem                Type     1K-blocks    Used Available Use% Mounted on
devtmpfs                  devtmpfs   8116256       0   8116256   0% /dev
tmpfs                     tmpfs      8133256       0   8133256   0% /dev/shm
tmpfs                     tmpfs      8133256   10492   8122764   1% /run
tmpfs                     tmpfs      8133256       0   8133256   0% /sys/fs/cgroup
/dev/mapper/rhel-root     xfs       95369732 6811292  88558440   8% /
/dev/sda1                 xfs        1038336  188580    849756  19% /boot
/dev/mapper/datavg-datalv xfs      157205508 1676284 155529224   2% /data
```




再來寫到/etc/fstab 每次開機自動掛載
```
vim /etc/fstab
```
```
#
# /etc/fstab
# Created by anaconda on Mon Aug 16 13:45:45 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#

/dev/datavg/datalv  /data                   xfs     defaults        0 0
```

重開機再看掛載有沒有成功
```
reboot
```
```
df -T
```
```
Filesystem                Type     1K-blocks    Used Available Use% Mounted on
devtmpfs                  devtmpfs   8116256       0   8116256   0% /dev
tmpfs                     tmpfs      8133256       0   8133256   0% /dev/shm
tmpfs                     tmpfs      8133256   10492   8122764   1% /run
tmpfs                     tmpfs      8133256       0   8133256   0% /sys/fs/cgroup
/dev/mapper/rhel-root     xfs       95369732 6811292  88558440   8% /
/dev/sda1                 xfs        1038336  188580    849756  19% /boot
/dev/mapper/datavg-datalv xfs      157205508 1676284 155529224   2% /data
```














