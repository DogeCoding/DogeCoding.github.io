# -bash:: command not found 问题解决
> 近来安装各种软件，诸如Macports，OpenCV，RVM……按照网上一些教程里面不自觉的更改了Terminal环境路径等等的配置，导致各种命令用不了。(手动懵逼)  
![command not found](http://upload-images.jianshu.io/upload_images/2590408-c887c4d9de67ba23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一打开Terminal简直各种被羞辱(这是一次修复后的，修复前ls，cd等等都不能用)。不多说下面给麻瓜法装软件擦屁股。
## 查看PATH
`$PATH`
看看是否包含 `/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin` 等关键路径。一般来说出错了这些路径是没有的。
>  - ./bin:    
> bin为binary的简写主要放置一些系统的必备执行档例如: `cat` 、 `cp`、 `chmod df`、 `dmesg`、 `gzip`、 `kill`、 `ls`、 `mkdir`、 `more`、 `mount`、 `rm`、 `su`、 `tar` 等。  
> -  _usr_bin:   
> 主要放置一些应用软体工具的必备执行档例如 `c++`、 `g++`、 `gcc`、 `chdrv`、 `diff`、 `dig`、 `du`、 `eject`、 `elm`、 `free`、 `gnome*`、  `gzip`、 `htpasswd`、 `kfm`、 `ktop`、 `last`、 `less`、 `locale`、 `m4`、 `make`、 `man`、 `mcopy`、 `ncftp`、  `newaliases`、 `nslookup passwd`、 `quota`、 `smb*`、 `wget` 等。  
> -  /sbin:   
> 主要放置一些系统管理的必备程式例如: `cfdisk`、 `dhcpcd`、 `dump`、 `e2fsck`、 `fdisk`、 `halt`、 `ifconfig`、 `ifup`、  `ifdown`、 `init`、 `insmod`、 `lilo`、 `lsmod`、 `mke2fs`、 `modprobe`、 `quotacheck`、 `reboot`、 `rmmod`、  `runlevel`、 `shutdown` 等。   
> -  _usr_sbin:   
> 放置一些网路管理的必备程式例如: `dhcpd`、 `httpd`、 `imap`、 `in.*d`、 `inetd`、 `lpd`、 `named`、 `netconfig`、 `nmbd`、 `samba`、 `sendmail`、 `squid`、 `swap`、 `tcpd`、 `tcpdump` 等。   
## 使用临时环境变量(重启后失效)
`export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`
这样就能暂时使用大部分命令了
## 进入当前用户home目录
`cd ~`
## 创建 `.bash_profile` 并编辑
`touch .bash_profile`
`open -e .bash_profile`
> `.bash_profile` 介绍  
>  Mac  和  Linux  终端一般用bash来进行解析。当bash在读完了整体环境变量的 `/etc/profile` 并借此调用其他配置文件后，接下来则是会读取用户自定义的个人配置文件。  
> bash读取的文件总共有三种：  
> - `~/.bash_profile `  
> - `~/.bash_login `  
> - `~/.profile`  
> 其实bash再启动是只读上面文件的一个，而读取的顺序则是依照上面的顺序。也就是说读到 `bash_profile` 就不读后面的了，如果 `bash_profile` 不存在，后面的才能有机会。  
![我的 `.bash_profile` 文件，简直一团乱！](http://upload-images.jianshu.io/upload_images/2590408-007d2a97f7ad7e01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后一句是我添加上去的，勉强能用了。
## 修改PATH
两种方法，都试试吧，我貌似是第一种方法起效的：
*  `echo "export PATH=xxxxxx:$PATH" >> ~/.bash_profile`
把"export PATH=xxxxxx:$PATH"输出打印到 `~/.bash_profile` 中去。
* 直接在前面打开的 `.bash_profile` 里面修改。
## 更新
* `cat ~/.bash_profile` 然后重启Terminal
* 或者 `source .bash_profile`

看看能不能用了吧，安装软件的时候还是要小心啊…
















