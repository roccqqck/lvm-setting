 # LVM extend
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
fdisk /dev/sda
```

# 2.創建pv
```
pvcreate /dev/sda2
```

# 3. vgextend   
查詢root的VG Name：rhel

將sda2擴充到VG Name：rhel下

```
vgextend rhel /dev/sda2
```
查看Free  PE / Size 
```
vgdisplay 
```
 

# 4. lvextend     

將lv name: /dev/rhel/root 增加50G
```
lvextend –L +50G /dev/rhel/root
```
查看LV Size
```
lvdisplay
```
 


# 5.resize
若是xfs
```
xfs_growfs /dev/mapper/rhel-root
```

若是ext
對lv1進行線上(動態)擴容 resize2fs是ext2檔案系統大小的調整工具，ext3只是多了journal的ext2，也可以用。
```
resize2fs /dev/datavg/lv1     
```    


查詢root是否被放大
```
df -h
```