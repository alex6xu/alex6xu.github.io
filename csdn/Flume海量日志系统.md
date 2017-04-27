$wget http://cloud.github.com/downloads/cloudera/flume/flume-
distribution-0.9.4-bin.tar.gz $tar -xzvf flume-distribution-0.9.4-bin.tar.gz
$cp -rf flume-distribution-0.9.4-bin /usr/local/flume $vi /etc/profile #添加环境配置
export FLUME_HOME=/usr/local/flume export PATH=.:$PATH::$FLUME_HOME/bin
$source /etc/profile $flume #验证安装

