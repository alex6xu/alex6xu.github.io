在网站开发阶段，对于静态资源文件比如JS，CSS等文件都是未经过压缩合并处理的，这对于访问量巨大的网站来说不仅浪费带宽，而且也会影响网站的访问速度
。django-compressor的作用就是将静态文件压缩合并成一个文件，不仅减少了网站的请求次数，还能节省网络带宽。 本文分为两部分，第一部分介绍set
tings文件相关配置对静态文件的影响，然后再讨论Compressor的如何使用。如果你对setting文件非常了解不妨直接从第二部分开始阅读。
第一部分：Setting文件配置 早期的django处理静态资源要比较啰嗦，还要配置urlpatterns，不过自从django1.6开始加入了django
.contrib.staticfiles这个内置app后，开发环境下处理静态资源就方便很多。django.contrib.staticfiles是djang
o的内置(build-in)app，用于处理js、css、images等静态资源。首先确保这个app已经包含在INSTALLED_APPS中，django1
.6及以上版本是默认包含该app在其中的。 一：指定STATIC_URL STATIC_URL = '/static/'
STATIC_URL是客户端访问静态资源的根路径，比如：模版中定义的资源路径是： {% load staticfiles %}  最终渲染后的效果是：  默
认django会从每个app目录的static子目录下查找静态文件，因此通常情况下你都是将相关静态文件放在各自的app/static目录下。Django怎么
知道从app/static目录查找静态文件呢？原来django有个默认配置项STATICFILES_FINDERS，默认值：
("django.contrib.staticfiles.finders.FileSystemFinder",
"django.contrib.staticfiles.finders.AppDirectoriesFinder")
AppDirectoriesFinder模块就是负责在app/static目录下找静态文件的。至于FileSystemFinder我们稍后介绍。
二：指定STATICFILES_DIRS 像jquery，bootstrap等公用的静态资源文件在很多个app中都会共用到，如果是放在某个app中显得不符p
ython哲学，因此django提供了一个配置可以指定任意目录来存放这些公用的资源文件。配置参数是：STATICFILES_DIRS，比如：
STATICFILES_DIRS = ( os.path.join(BASE_DIR, "static"), '/var/www/static/', ) 这
意味着静态文件可以放在磁盘的任何一个位置（只要有权限访问）现在应该明白FileSystemFinder的作用了吧。就是用来查找定义在STATICFILES_
DIRS中的静态文件的。 三：指定STATIC_ROOT 上面的配置就是在开发环境下Django对静态资源的处理过程，那么在生产环境下是怎么处理的呢？如果还
是这样由django自己来处理，那么累死Django了，对于静态资源直接由Nginx这样的代理去处理就行，而且Nginx处理这些非逻辑的资源性能也非常高。于
是Django提供了一个非常方便的静态资源管理命令django.contrib.staticfiles将系统要用的资源文件从不同目录收集到统一的目录中去，然
后在Nginx的配置中指定这些静态资源的位置即可。 收集这些静态文件需要先指定存放这些静态资源的目录：STATIC_ROOT
STATIC_ROOT="/var/www/foofish.net/static/" 运行collectstatic管理命令 python
manage.py collectstatic 运行该命令，所有静态资源都将拷贝到STATIC_ROOT指定的目录中。 四：配置Nginx
在Nginx的配置文件中指定凡是来自/static/路径的请求直接访问STATIC_ROOT。 location /static { alias
/var/www/foofish.net/static/; # your Django project's static files }
第二部分：Compressor的使用 django-compressor 的安装配置非常简单 安装: pip install
django_compressor 配置: COMPRESS_ENABLED = True INSTALLED_APPS = ( # other apps
"compressor", ) STATICFILES_FINDERS = (
'django.contrib.staticfiles.finders.AppDirectoriesFinder',
'django.contrib.staticfiles.finders.FileSystemFinder',
'compressor.finders.CompressorFinder',) Django-
Compressor开启与否取决于DEBUG参数，默认是COMPRESS_ENABLED与DEBUG的值相反。因为Django-Compressor的功能本
身是用在生产环境下项目发布前对静态文件压缩处理的。因此想在开发阶段(DEBUG=True)的时候做测试使用，需要手动设置COMPRESS_ENABLED=T
rue 使用 在模板文件中添加模板标签 {% load compress %} {% load compress %} #处理css {% compress
css %}  {% endcompress %} #处理js {% compress js %}  {% endcompress %}
执行命令：python manage.py compress ,最终文件将合并成:
f18b10165eed.css、9d1f64ba50fc.js这两文件在STATIC_ROOT目录下面。
每次修改了js、css文件后，都需要重新加载最新的文件到STATIC_ROOT目录下去，因此需要重新运行命令： python manage.py
collectstatic python manage.py compress

