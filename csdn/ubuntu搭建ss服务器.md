需要准备： 一个墙外的VPS 1、安装shadowsocks服务器 更新软件源 sudo apt-get update1 安装PIP环境 sudo apt-
get install python-pip1 使用pip命令安装shadowsocks sudo pip install shadowsocks1
2、运行shadowsocks服务器 方式一：命令启动 sudo ssserver -p 8388 -k mypassword -m -rc4-md5 -d
start1 如果要停止运行，将上面的命令中的start换成stop。 方式二：配置文件（推荐，方便查看修改） vim
/etc/shadowsocks.json1 添加如下内容： { "server":"my_server_ip", "server_port":8388,
"local_address":"127.0.0.1", "local_port":1080, "password":"mypassword",
"timeout":300, "method":"rc4-md5" }123456789 //多个用户的配置 {
"server":"my_server_ip", "port_password":{ "9001":"pwd001", "9002":"pwd002",
"9003":"pwd003" }, "local_address":"127.0.0.1", "local_port":1080,
"timeout":300, "method":"rc4-md5" }12345678910111213 各字段的含义： 字段 含义 server
服务器的IP，VPS的公网IP，注意这也将是服务端监听的IP地址 server_port 服务器端口 local_port 本地端口 password
用来加密的口令 timeout 超时时间（单位/秒） method 加密方式。可选择 “bf-cfb”, “aes-256-cfb”, “des-cfb”,
“rc4″, 等等。默认是一种不安全的加密，推荐用 “aes-256-cfb”。 Tips：加密方式推荐使用rc4-md5，因为 RC4 比 AES
速度快好几倍，如果用在路由器上会带来显著性能提升。旧的 RC4 加密之所以不安全是因为 Shadowsocks 在每个连接上重复使用 key，没有使用
IV。现在已经重新正确实现，可以放心使用。 创建完毕后，赋予shadowsocks.json文件权限： sudo chmod 755
/etc/shadowsocks.json1 为了支持这些加密方式，还需要安装 sudo apt–get install python–m2crypto1
然后使用配置文件在后台运行： sudo ssserver -c /etc/shadowsocks.json -d start1 3、配置开机自启动 编辑
/etc/rc.local 文件 sudo vim /etc/rc.local1 在exit 0 这一行的上边加入如下
/usr/local/bin/ssserver –c /etc/shadowsocks.json1 或者使用加入命令启动如下：
/usr/local/bin/ssserver -p 8388 -k password -m rc4-md5 -d start1
重启服务器后，将会自动启动shadowsocks服务。 Tips:在win10使用shadowsocks 3.0版本会遇到
系统检测到在一个调用中尝试使用指针参数时的无效指针地址 ，是版本问题，用回旧版本就没问题了（2.5之前的），参考github

