上传图片（大小2M），出现 nginx: 413 Request Entity Too Large 错误。
原来nginx默认上传文件的大小是1M，可nginx的设置中修改。 解决方法如下： 1.打开nginx配置文件 nginx.conf,
路径一般是：/etc/nginx/nginx.conf。 2.在http{}段中加入 client_max_body_size 10m;
20m为允许最大上传的大小。 3.保存后重启nginx，问题解决。

