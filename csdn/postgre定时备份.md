需要切换用户postgres, pg_dump命令 \----- 脚本： #!/bin/bash # /backup.sh cur-time=`date
+'%Y%m%d'` pg_dump  > /backup.dump echo 'postgre backup finished' \----
使用crontab定时 crontab -e编辑配置文件 0 0 * * * /bin/bash /backup.sh

