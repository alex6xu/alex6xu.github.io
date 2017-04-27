Ubuntu安装zabbix3.2的一些坑： 1，首先Ubuntu安装有现成的deb，很方便，但是分清是安装的是哪个版本，server安装all,
proxy安装proxy，agent 安装agent就够了。 2，server安装apache和php时要配置apache.conf,
还可能需要额外的库，sudo apt-get install libapache2-mod-php,当然还需要修改sources_list
3,配置agent.conf,   Server,ActivateServer 是zabbix Server的IP，HostName是本机host
name,需要和web页面上配置的host name一致， 4，web页面添加agent; configure -> host -> add
host:参数都是agent的参数，然后直接添加template就可以监控了 5，agent.conf中有个StartAgent,是本机的进程数，如果设为0
，本机就不会开启tcp监听，这样telnet就会refuse,网上好多坑。。。。
6，还有就是网络连接，如果非同一个网段，有nat,或者有防火墙，需要提前设置好，端口10050，10051要联通 7, 更新之后好像少了php的库
，需要安装php-mcmath php-mbstring php-xml......

