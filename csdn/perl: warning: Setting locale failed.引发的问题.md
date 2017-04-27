缘由 我用的是linode的vps，系统为ubuntu14.04lts 当apt-get安装软件时，都会报一个相同的错误，如下 perl: warning:
Setting locale failed. perl: warning: Please check that your locale settings:
LANGUAGE = (unset), LC_ALL = (unset), LC_TIME = "zh_CN.UTF-8", LC_MONETARY =
"zh_CN.UTF-8", LC_ADDRESS = "zh_CN.UTF-8", LC_TELEPHONE = "zh_CN.UTF-8",
LC_NAME = "zh_CN.UTF-8", LC_MEASUREMENT = "zh_CN.UTF-8", LC_IDENTIFICATION =
"zh_CN.UTF-8", LC_NUMERIC = "zh_CN.UTF-8", LC_PAPER = "zh_CN.UTF-8", LANG =
"en_US.UTF-8" are supported and installed on your system. perl: warning:
Falling back to the standard locale ("C"). 那是因为安装软件时，都会去执行一个update-
locale的命令，用来更新locale 这个命令是一个脚本，用perl写的，可以用whereis update-locale查到，位置在/usr/sbin
/update-locale 上述报错并不是因为update-locale命令而引起，update-locale这段脚本没有问题，而是因为perl
可以使用以下命令测试： perl -e exit 其实，真正的原因是perl为系统使用zh_CN.UTF-8，但系统不知道zh_CN.UTF-8是什么东西
解决方法 解决方法也很简单 apt-get install language-pack-zh-hans 安装一个中文语言，系统就知道zh_CN.UTF-
8了，这个时候用perl就不会报错了 深入了解 这种情况一般是vps比较常见，因为一般都是用ssh的方式连接到vps上的
sshd有这个机制，会把客户机上的语言环境带到远程的机器上 客户机一般都会设置zh_CN.UTF-8语言，用来显示中文，而远端的vps一般就只有en_US
.UTF-8zh_CN.UTF-8一旦带过去就会报找不到的错误，文章开头已经说的很清楚了 不靠谱的解决方法
网上还有些解决方法，并不是很靠谱，虽然从表面来看像解决问题了，但其实是把问题影藏了
比如在远程主机上的/etc/ssh/sshd_config文件里，将AcceptEnv LANG
LC_*这行注释掉然后重启远程的sshd，然后退出远程后，重新ssh上来。 这时，远程主机不会把客户机的语言环境（zh_CN.UTF-8）带过来
当然就不会再有报错，可惜的是，远程主机是无法正确显示中文的，问题还在，只是被影藏了。

