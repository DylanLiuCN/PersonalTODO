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

### 4. 合入MR步骤    

* `git checkout develop`; `git pull origin develop:develop -f`     
  确保在合入代码前，你在我们的主干分支develop上，且与主库代码同步。其中origin为flink主库在你本地的库别名（当然你自己本地也有可能叫别的对应名称，通过`git remote -v`可以查看    
* `git fetch huawei srcBranch:localBranch`         
  获取MR分支代码，srcBranch即为MR的来源分支，localBranch为fetch下来的分支名称，名字你可以随意取    
* `git merge localBranch --squash`   
  将localBranch上的改动通过squash的方式合入到当前分支(develop)上
* `git commit -a --author="作者"`        
  作者即这个MR的author，写他/她在CodeClub对应的昵称即可     
* 上一步过后，这个地方会弹出一个vi交互窗口，你需要用insert模式插入MR的相关信息，即MR的标题、MR的message，第一行会被当做commit的标题，后面的所有内容会被当做commit message
* `git push huawei develop:develop`     
  将本地的develop分支（经过了步骤2、3、4，已经包含了MR的更改）push到我们主库的develop分支上
  
### 5. 代码回退    
* `git log`查看提交记录    
* `git reset --HARD *****`
* `git push origin *** -f`
