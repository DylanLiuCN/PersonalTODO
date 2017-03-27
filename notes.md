## 零碎笔记    
### 1. Linux命令 
* 批量删除指定文件    
  * 命令： find / -name XXX -exec rm -v {} \\;

### 2. Calcite 查看RelNode    
RelOptUtil.toString(RelNode)

### 3. 挂载逻辑卷     
* fdisk -l 查看要挂载的卷名称      
* vgdisplay查看需要挂载的卷的大小     
* 然后 

```shell
lvcreate -l 卷大小 -n lv1 vg1
mkfs.ext3 -j /dev/vg1/lv1
tune2fs -c 0 -i 0 /dev/vg1/lv1
```     
* 挂载     

```shell
mount /dev/mapper/vg1-lv1 /挂载位置
echo "/dev/mapper/vg1-lv1 /挂载位置 ext3 defaults 0 0" >>/etc/fstab
more /etc/fstab 
df -h
```
