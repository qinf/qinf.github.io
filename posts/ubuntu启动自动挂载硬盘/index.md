# Ubuntu启动自动挂载硬盘


## 启动自动挂载硬盘
```bash
# 查看磁盘分区
sudo fdisk -l
# 挂载磁盘
sudo mount /dev/sdb /data/
# 查看磁盘挂载UUID
sudo blkid /dev/sdb
# 修改/etc/fstab文件
sudo vi /etc/fstab
# 添加磁盘信息，[UUID=************] [挂载磁盘分区]  [挂载磁盘格式]  0  2
# 第一数字0，0是开机不检查磁盘，1是开机检查磁盘
# 第二个数2，0表示交换分区，1表示启动分区，2表示普通分区 
UUID=${UUID} /media/qinf/D ntfs defaults  0  2
```
&lt;!-- more --&gt;

## Docker container 启动重启
```bash
docker run --restart=always -d --name ${container_name} -p ${port}:80 -v /localdir:/mount_dir ${image_name}
```

---

> Author:   
> URL: http://localhost:1313/posts/ubuntu%E5%90%AF%E5%8A%A8%E8%87%AA%E5%8A%A8%E6%8C%82%E8%BD%BD%E7%A1%AC%E7%9B%98/  

