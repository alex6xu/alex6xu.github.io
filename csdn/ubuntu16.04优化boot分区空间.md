打开终端，在终端里依次输入一下命令，以解决/boot分区满的问题： 1.df -h （查看Ubuntu的文件系统 使用情况） 2\. uname -a
(查看当前使用的内核版本) 3.sudo apt-get autoremove linux-image- **-（按两次tab键）
4，若因为空间已用完，之前的image安装出错会提示： 这时使用apt-get -f install:修复之前broken的app.
(可能需要删除部分不用的文件） 再查看下内核和磁盘容量，发现释放了很多空间。   done

