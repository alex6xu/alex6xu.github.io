修改的文件 文件 作用（详情看代码） 备注 proj/__init__.py
导入celery.py中的app，来保证只要django启动就可以用这个app执行shared_task   proj/celery.py
创建一个celery app，以项目名命名   proj/settings.py
主要配置三项东西INSTALLED_APP,BROKER_URL和序列器配置 或者改你指定的配置文件 demoapp/tasks.py
本文有一些shared_task在里面   CODE 我是参考这里 - > 官方最佳实践 proj/__init__.py from __future__
import absolute_import from .celery import app as celery_app1212
proj/celery.py # coding:utf8 from __future__ import absolute_import import os
from celery import Celery from django.conf import settings # set the default
Django settings module for the 'celery' program. #
这里我使用生产的配置，这里有点瑕疵，不会指定上面导入的settings，导致开发环境和生产环境没办法区分，想到办法再更新
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'proj.prod_settings') app =
Celery('proj') # Using a string here means the worker will not have to #
pickle the object when using Windows.
app.config_from_object('django.conf:settings') app.autodiscover_tasks(lambda:
settings.INSTALLED_APPS) @app.task(bind=True) def debug_task(self):
print('Request: {0!r}'.format(self.request)) 123456789101112131415161718192021
2223242512345678910111213141516171819202122232425 proj/prod_settings.py # 修改点1
INSTALLED_APP += [ 'kombu.transport.django', 'djcelery' ] # 修改点2 #
生产环境我使用rabbitmq, 也可以用redis等作为broker BROKER_URL =
'redis://172.23.18.116:6379/0' # 开发环境可以直接用django作为broker BROKER_URL =
'django://' # 修改点3 #: Only add pickle to this list if your broker is secured
#: from unwanted access (see userguide/security.html) CELERY_ACCEPT_CONTENT =
['json'] CELERY_TASK_SERIALIZER = 'json' CELERY_RESULT_SERIALIZER = 'json' # 这
里我注意到的不多，我只知道返回（包括错误）是json格式的，输入貌似也得是12345678910111213141516171819123456789101
11213141516171819 demoapp/tasks.py from __future__ import absolute_import from
celery import shared_task @shared_task def add(x, y): return x + y
@shared_task def mul(x, y): return x * y @shared_task def xsum(numbers):
return sum(numbers) 123456789101112131415161718123456789101112131415161718 注意
如果INSTALLED_APP里面注册app是demoapp，那么celery任务在import的时候要用（也就是.tasks前面保持一致） from
demoapp.tasks import add, mul, xsum11 的方式，不然会出现奇怪的错误 使用 首先启动worker #
这里我指定了配置，默认的话忽略info后面 python manage.py celery worker --loglevel=info
--settings=proj.prod_settings1212 然后试着执行shared_task add() # 同上忽略指定的配置文件 python
manage.py shell --settings=proj.prod_settings1212 也就是在django环境中执行： from
demoapp.tasks import add add.delay(4, 4)1212 上面的worker log中出现add()的return也就是16
说明celery配置完成 django-celery的定时任务功能 说白了就是celery beat定时或者定间隔给celery发送task
只需配置一项： # proj/settings.py CELERYBEAT_SCHEDULER =
'djcelery.schedulers.DatabaseScheduler'1212 然后 python manage.py migrate11
然后后台配置任务执行间隔  再然后： python manage.py celery beat11 可以看到celery
beat开始按之前admin后台设置的时间间隔或者crontab开始发送task给celery了，此时自然也需要celery
worker在running状态 celery beat发送的任务return的结果在celery log中可以看到  说明django-celery
定时任务配置成功

