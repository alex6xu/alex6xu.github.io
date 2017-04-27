在Linux安装了多版本Python时（例如python2.6和2.7），pip安装的包不一定是用户想要的位置，此时可以用 -t 选项来指定位置.
例如目标位置是/usr/local/lib/python2.7/site-packages/ ，要安装xlrd 这个包，则： $ pip install
-t /usr/local/lib/python2.7/site-packages/ xlrd 权限不够则在命令前加sudo。

