我的系统同时安装有Python 2和python 3，我需要在python 3.4.0上面安装BeautifulSoup4，而直接采用下面命令： sudo
apt-get install python-bs411 则将BeautifulSoup4安装在了python
2.7.6上面。采用什么方法将其安装在与python2.7.6共存的python 3.4.0上面呢？ 方法 先使用命令sudo apt-get
install python3-pip安装上pip3;然后使用sudo pip3 install beautifulsoup4，即可安装成功。
采用以下命令检验是否安装成功： python3 > from bs4 import BeautifulSoup1212
如果没有什么错误的提示信息，即表示安装成功。

