# MAC(OS X)/Linux 下的刻录树莓派镜像的方法


其它镜像同理。


```
    
    df -h 
    #查看已挂载卷，通常是/dev/disk1s1 ,具体根据容量名字判断

    diskutil unmount /dev/disk1s1
    #防止被占用，先卸载，刻录完成后再次使用这个命令卸载
    #linux 下类似,注意是umount：
    sudo umount /dev/sdb1

    diskutil list
    #“/dev/disk1s1是分区，/dev/disk1是块设备，/dev/rdisk1是原始字符设备”
    #linux 下就是/dev/sdb

    sudo dd bs=4m if=pi.img of=/dev/rdisk1
    #dd命令写入镜像，if=[.ISO文件位置] of=[硬碟位置]需要是整个盘也就是disk或rdisk #bs=「读写速率单位M」， 其他参数 ‘dd --help’ 获得
    #linux 下bs后面不能用M，直接去掉或者自己查参数，磁盘位置of=/dev/sdb

    完了等待几分钟就好了。

```
参考资料

http://www.arefly.com/linux-mac-dd-burn-iso-to-usb/

