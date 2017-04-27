动态网站的一个基本权衡就是他们是动态的，每次一个用户请求一个页面，web服务器进行各种各样的计算-从数据库查询到模板渲染到业务逻辑-从而生成站点访问者看到的
页面。从处理开销的角度来看，相比标准的从文件系统读取文件的服务器调度，这是昂贵了不少。尽管对于大多数网站来说，这种开销不是什么大问题，因为大多数web应用不
过是想学学院的首页那样，都是小到中型的站点，流量也很少。但对于中到大型的站点来说，必须尽可能的减少开销。这就是缓存的由来。
缓存的意义在于把昂贵的计算结果保存起来一遍下次的访问，有缓存的站点的流程大概是这样子的：
给定一个url，检查页面是否在缓存中如果在，返回缓存的页面否则，生成该页面，把生成的页面保存在缓存中，返回生成的页面 django自带一个强大的缓存系统，提
供不同层次的缓存粒度：你可以缓存某个视图函数的输出，或者只是某个特别难生成的部分或者是整个站点。同时django也有类似“上流”缓存的东西，类似于Squid
和基于浏览器的缓存，这类缓存是你无法直接控制但是你可以通过一些提示（比如http头部）指定你网站的那些部分需要和如何缓存。 设置缓存
缓存系统需要少量的配置才能使用，你必须告诉系统你的缓存数据存放在哪里-数据库还是文件系统亦或是直接存在缓存中-
这是一个重要的决定，直接影响到你的缓存性能；当然，一些缓存类型本来就是比其他的快，这我们无法回避。
在项目的配置文件里面可以通过CACHES配置缓存，下面我们一一讲解django中可用的缓存系统 memcached Memcached
是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载从而显著提供网站性能，也是django中到目前为止最有效率的可用缓存。 Memcac
hed作为一个后台进程运行，并分配一个指定的内存量，它所做的全是提供一个添加，检索和删除缓存中任意数据的快速接口，所有的数据都是直接存储在内存中，所以就没有
了数据库或者文件系统使用的额外开销了。 在安装Memcached之后，你还需要安装一个绑定Memcached的东西，这里当做是插件之类的东西
，最常用的两个是python-mencached和pylibmc。 在django中使用Memcached，你需要在配置文件里的CACHES做如下设置： 根
据你选择的插件，选择对应的后端(BACKEND)，django.core.cache.backens.memcached.MemcachedCache或者d
jango.core.cache.backends.memcached.PyLibMCCache给LOCATION设置ip:port值，其中ip是Memca
ched进程的IP地址，port是Memcached运行的端口，或者unix:path值，其中path是Memcached Unix
socket文件所在的路径 下面的这个例子使用的是python-memcached缓存，服务器有三个（只需要在配置文件配置一次就行） CACHES = {
'default': { 'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
'LOCATION': [ '172.19.26.240:11211', '172.19.26.242:11211',
'172.19.26.244:11213', ] } } 这个是使用Unix socket的例子 CACHES = { 'default': {
'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache', 'LOCATION':
'unix:/tmp/memcached.sock', } 最后的一点是关于Memcached缓存的不足：缓存完全在内存中，一旦服务器崩溃，所有的数据将不复
存在，因此，不要把基于内存的缓存作为你唯一的数据存储方式，当然，所有的缓存都不是作为永久存储的，缓存只是缓存而不是存储，但基于内存的缓存是特别的仅仅的真的只
是“缓存”，稍“纵”即逝（纵这里是出现错误）。下面我们将见到一些不是那么稍纵即逝的缓存。 数据库缓存 在django中使用数据库缓存，首先你需要在数据库中建
立一个用于缓存的数据库表，可以参考下面的命令，注意cache_table_name不要和数据库中已经存在的表名冲突： python manage.py
createcachetable [cache_table_name] 一旦设置好了数据库表，在配置文件中班的BACKEND设置为django.core.c
ache.backends.db.DatabaseCache，LOCATION设置为你的数据库表名，下面是个例子： CACHES = {
'default': { 'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
'LOCATION': 'my_cache_table', }
注意的是数据库缓存使用的是你配置文件中的数据库，所以你不可以使用其他的数据库；如果你使用的是一个快速良好的索引数据库服务器的话，你的数据库缓存可以工作的更好
数据库缓存和多数据库 如果你真的要使用多个数据库用作数据库缓存的话，你需要设置路由说明给你的数据库缓存表。处于路由的目的，数据库缓存表会以一个在一个名为dj
ango_cache的应用下名为CacheEntry的模型出现，这个模型不会出现在模型缓存中，但这个模型的细节可以在路由中使用。
举个例子，下面的路由会把所有的缓存读操作导向cache_slave，所有的写操作导向cache_master，缓存表只会同步到cache_master
class CacheRouter(object): """A router to control all database cache
operations""" def db_for_read(self, model, **hints): "All cache read
operations go to the slave" if model._meta.app_label in ('django_cache',):
return 'cache_slave' return None def db_for_write(self, model, **hints): "All
cache write operations go to master" if model._meta.app_label in
('django_cache',): return 'cache_master' return None def allow_syncdb(self,
db, model): "Only synchronize the cache model on master" if
model._meta.app_label in ('django_cache',): return db == 'cache_master' return
None 如果你不提供路由导向，那么django将使用默认的数据库；当然，如果你不使用数据库缓存，你完全不需要担心路由导向的问题。哈哈 文件系统缓存 请使用
django.core.cache.backends.filebased.FileBasedCache赋值给后端变量BACKEND，LOCATION设置为缓
存所在的位置，注意是绝对位置（从根目录开始），下面是一个例子： CACHES = { 'default': { 'BACKEND':
'django.core.cache.backends.filebased.FileBasedCache', 'LOCATION':
'/var/tmp/django_cache', #'LOCATION': 'c:/foo/bar',#windows下的示例 } }
必须保证服务器对你列出的路径具有读写权限 本地内存缓存 如果你想具有内存缓存的优点但有没有能力运行Memcached的时候，你可以考虑本地内存缓存，这个缓存
是多进程和线程安全的，后端设置为django.core.cache.backends.lovMemCache CACHES = { 'default': {
'BACKEND': 'django.core.cache.backends.locmem.LocMemCache', 'LOCATION':
'unique-snowflake' } }
缓存LOCATION用来区分每个内存存储，如果你只有一个本地内存缓存，你可以忽略这个设置；但如果你有多个的时候，你需要至少给他们中一个赋予名字以区分他们。
注意每个进程都有它们自己的私有缓存实例，所以跨进陈缓存是不可能的，因此，本地内存缓存不是特别有效率的，建议你只是在内部开发测时使用，不建议在生产环境中使用
虚拟缓存 django自带一个虚拟缓存-不是真实的缓存，只是实现缓存的接口而已。 这在实际应用中时非常有用的，假如你有一个产品用发布的时候非常需要用到缓存，
但在开发和测试的时候你并不想使用缓存，同时你也不想等到产品发布的时候特别的去修改缓存相关的代码，那么你可以是用虚拟缓存，要启用虚拟缓存，你只需按下面的例子去
配置你的后端 CACHES = { 'default': { 'BACKEND':
'django.core.cache.backends.dummy.DummyCache', } } 使用自定义的缓存后端
要想使用自定义的cache后端，你可以用python的import格式导入你的后端模块，例如： CACHES = { 'default': {
'BACKEND': 'path.to.backend', } }
一方面你可以在自己的后端文件中导入或者使用django自带的缓存后端文件；另一方面，并不建议初学者使用自己的后端配置，出于这样或那样的原因。 缓存参数 除了
选择后端之外，我们还可以通过配置缓存参数来控制我们缓存的性能（到现在，我们至少知道了django所谓的参数，直接去看他们的源码就知道可以接受那些参数了，哈哈
） class BaseCache(object): def __init__(self, params): timeout =
params.get('timeout', params.get('TIMEOUT', 300)) try: timeout = int(timeout)
except (ValueError, TypeError): timeout = 300 self.default_timeout = timeout
options = params.get('OPTIONS', {}) max_entries = params.get('max_entries',
options.get('MAX_ENTRIES', 300)) try: self._max_entries = int(max_entries)
except (ValueError, TypeError): self._max_entries = 300 cull_frequency =
params.get('cull_frequency', options.get('CULL_FREQUENCY', 3)) try:
self._cull_frequency = int(cull_frequency) except (ValueError, TypeError):
self._cull_frequency = 3 self.key_prefix = smart_str(params.get('KEY_PREFIX',
'')) self.version = params.get('VERSION', 1) self.key_func =
get_key_func(params.get('KEY_FUNCTION', None)) 这里有一个配置CACHES的样例，使用文件系统缓存，timeo
ut是60秒，最大容量是1000个子项，在Unix系统下的/var/temp/django_cache缓存 CACHES = { 'default': {
'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache', 'LOCATION':
'/var/tmp/django_cache', 'TIMEOUT': 60, 'OPTIONS': { 'MAX_ENTRIES': 1000 } } }
使用缓存 每个站点的缓存 缓存设置好了之后，最简单的应用便是缓存你的整个站点。 你需要添加两个中间件到MIDDLEWRAE_CLASSES：django.m
iddleware.UpdateCacheMiddleware和django.middleware.cache.FetchFromCacheMiddlewa
re，注意的是，UpdateCache中间件一定要放在第一位，Fetch中间件必须放最后（因为中间件的顺序决定着运行的顺序）见下面示例：
MIDDLEWARE_CLASSES = ( 'django.middleware.cache.UpdateCacheMiddleware',
'django.middleware.common.CommonMiddleware',
'django.middleware.cache.FetchFromCacheMiddleware', )
然后你需要在项目配置文件中加入下面几个必须的设置： CACHE_MIDDLEWARE_ALIAS：用来存储的缓存别名CACHE_MIDDLEWARE_SEC
ONDS：每个页面应该被缓存的秒数CACHE_MIDDLEWARE_KEY_PREFIX：关键的前缀，当多个站点使用同一个配置的时候，这个可以设置可以避免发
生冲突；如果你不在乎的话， 你可以是用一个空字符串，建议你别这样做
如果请求或者响应的头部允许的时候，缓存中间件会缓存那些200的get或者head的相应页面。同一个url但是不同查询参数会被认为是不同的相应从而被分别缓存。
如果你设置了CACHE_MIDDLEWARE_ANONYMOUS_ONLY为真，那么只有匿名的请求会被缓存，这是一个禁用缓存非匿名用户页面的最简单的做法，注
意确保已经启用了认证中间件。 缓存中间件希望一个head请求可以被 一个有相同响应头部的get请求的响应
响应，因为这样的话，缓存中间件可以直接用一个get响应返回给一个head请求。 另外，缓存中间件会自动的设置少量的头部信息给每一个HttpResponse：
当一个新鲜的页面被请求的时候，会用当前时间打上一个Last_Modified的头部会用当前时间加上CACHE_MIDDLEWARE_SECONDS的值设置给
Expires头部用CACHE_MIDDLEWARE_SECONDS的值给Cache-
Control头部去设置一个页面的最大年龄（前提是视图函数没有设置该项） 每个视图函数的缓存 一个更加精细的使用缓存框架的方法是缓存单独一个视图函数的输出。
django.views.decorators.cache定义了一个cache_page的装饰器，使用了这个装饰器的视图的输出会被自动缓存。
cache_page(timeout, [cache=cache name], [key_prefix=key prefix])
cache_page只接受一个参数和两个关键字参数， timeout是缓存时间，以秒为单位cache：指定使用你的CACHES设置中的哪一个缓存后端key_
prefix：指定缓存前缀，可以覆盖在配置文件中CACHE_MIDDLEWARE_KEY_PREFIX的值 @cache_page(60 * 15,
key_prefix="site1") def my_view(request): 在url配置文件中使用缓存装饰器 和上面的类似，装饰器的位置发生了变化
from django.views.decorators.cache import cache_page urlpatterns = ('',
(r'^foo/(\d{1,2})/$', cache_page(60 * 15)(my_view)), ) 模板片段缓存
如果你想更精细的使用缓存，你可以是用cache这个模板标签。为了使用这个标签，记得使用{% load cache
%}标签，本人觉得现在没必要用到这么细的缓存控制，以后用到的时候再说吧 {% load cache %} {% cache 500 sidebar %}
.. sidebar .. {% endcache %} 底层的缓存API 有时候你不想缓存一个页面，甚至不想某个页面的一部分，只是想缓存某个数据库检索的结
果，django提供了底层次的API，你可以是用这些API来缓存任何粒度的数据，
如果你想了解所有的API，强烈建议你去看django\core\cache\backends目录下的cache.py文件，这里仅仅列举一些简单的用法 >>>
from django.core.cache import cache >>> cache.set('my_key', 'hello, world!',
30) >>> cache.get('my_key') # Wait 30 seconds for 'my_key' to expire... >>>
cache.get('my_key') None >>> cache.get('my_key', 'has expired') 'has expired'
>>> cache.set('add_key', 'Initial value') >>> cache.add('add_key', 'New
value') >>> cache.get('add_key') 'Initial value' >>> cache.set('a', 1) >>>
cache.set('b', 2) >>> cache.set('c', 3) >>> cache.get_many(['a', 'b', 'c'])
{'a': 1, 'b': 2, 'c': 3} 缓存 关键字前缀
前面有提到这个设置，一旦设置了这个，一个缓存关键字被保存或者检索的时候，django会自动加上这个前缀值，这使得缓存数据共享的时候很方便。 缓存版本
缓存被保存的时候是带上版本号的，比如 # Set version 2 of a cache key >>> cache.set('my_key',
'hello world!', version=2) # Get the default version (assuming version=1) >>>
cache.get('my_key') None # Get version 2 of the same key >>>
cache.get('my_key', version=2) 'hello world!' 利用版本这个特性，我们可以通过设置CACHE的VERSION控制
缓存的版本，当我们需要保留一些数据摒除另外一些的时候，我们可以通过修改需要保留数据的版本，和设置新的VERSION来达到目的 # Increment the
version of 'my_key' >>> cache.incr_version('my_key') # The default version
still isn't available >>> cache.get('my_key') None # Version 2 isn't
available, either >>> cache.get('my_key', version=2) None # But version 3 *is*
available >>> cache.get('my_key', version=3) 'hello world!' 缓存关键字转换
由上面的关键字前缀和关键字版本，我们可以知道，其实关键字是有由关键字，前缀和版本号组成的，默认他们之间是有冒号来连接的 def make_key(key,
key_prefix, version): return ':'.join([key_prefix, str(version),
smart_str(key)]) 如果你想用其他的字符来连接，请提供KEY_FUNCTION缓存设置 缓存关键字警告 Memcached对于关键字的要求是不
长于250字符，且不能包含空格或者控制字符，否则会抛出一个异常，因此，为了使代码更健壮和避免不愉快的意外，其他的内建缓存后端会发出一个警告django.co
re.cache.backends.base.CacheKeyWarning，如果关键字的使用会在Memcached引发一个错误
如果你在使用一个接受更大范围的关键字，然后想使用，你可以在你的INSTALLED_APPS中的一个的management模块中使用下面的代码
slience CacheKeyWarning import warnings from django.core.cache import
CacheKeyWarning warnings.simplefilter("ignore", CacheKeyWarning)
如果你想自定义关键字验证逻辑的话，你可以继承那个类，然后重写validate_key方法，下面以重写本地缓存后端为例子，请在模块中加入下面代码 from
django.core.cache.backends.locmem import LocMemCache class
CustomLocMemCache(LocMemCache): def validate_key(self, key): """Custom
validation, raising exceptions or warnings as needed.""" # ... “上流”缓存
到目前为止我们谈到的基本都是服务器端的缓存，但其他类型的缓存也是和网站开发密切相关的，比如“上流”缓存
缓存的页面，这是一些为用户缓存页面的系统，使得请求甚至没有到达的你的网站后端。 这是一些上流缓存的例子：
你的ISP（因特网提供商）可能会缓存一些页面，比如你访问   http://example.com/ 的时候，你的ISP可能没有访问到
example.com就把页面返回给你了 你的浏览器可能也会缓存一些网页
上游缓存是一个很好的效率提高，但不适合缓存所有的网页，特别是一些跟认证有关的页面的缓存可能会给网站带来一些隐患，那该如何决定是否在上游缓存呢？
很幸运的，我们可以指定Http头部信息来要求上游缓存根据我们的提示是否缓存我们的页面，接着往下看 使用Vary头部信息
当建立一个缓存关键字的时候，缓存机制会根据Vary头部信息决定是否那些 请求的头部信息
是需要考虑的。比如一个网页的内容决定于用户的语言偏好，那么可以说这个页面是“语言因人而异的”。 django默认使用请求路径和查询作为缓存关键字，比如/st
ories/2005/?order_by=author，这意味着所有请求该url的请求都是用同一个缓存页面，而不管user-agent是否一致，比如语言偏好
等等。然而，如果这个页面是根据不同的请求头部信息而生成不一样的页面的时候，你需要使用Vary头部去告诉缓存机制要根据这些不同的头部信息输出不同的内容。
为了做到这一点，django提供了vary_on_header视图装饰器 from django.views.decorators.vary import
vary_on_headers @vary_on_headers('User-Agent') def my_view(request): # ...
上面的这个例子，django的缓存机制会为每一个user-agent缓存一个单独的页面。 vary_on_header装饰器接受多个参数
，比如Cookie和User-Agent，多个参数之间是组合关系；另一方面，由于因为cookie的不同是比较常用的，django直接提供了一个vary_on
_cookie的装饰器，下面的这两种写法是等价的 @vary_on_cookie def my_view(request): # ...
@vary_on_headers('Cookie') def my_view(request): # ...
django还提供了一个有用的函数django.utils.cache.patch_vary_headers，这个函数设置或者添加内容到Vary 头部里面
from django.utils.cache import patch_vary_headers def my_view(request): # ...
response = render_to_response('template_name', context)
patch_vary_headers(response, ['Cookie']) return response 控制缓存：使用其他的头部信息 其他的一些关
于缓存的问题是：数据的私有性和数据应该被存储在级联的缓存的问题。一个用户面对两种类型的缓存：自己的浏览器缓存（私有的）和提供者的缓存（共有的）；一个共有的缓
存被多个用户使用同时被某个人控制，这将涉及到一些敏感的问题，比如你的银行账单。因此，我们的web应用需要一个方法告诉缓存那些数据是私有的，那些是可以共享的。
django提供了django.views.decorators.cache.cache_control视图装饰器解决这个问题 from
django.views.decorators.cache import cache_control
@cache_control(private=True) def my_view(request): # ...
注意的是，共有和私有这两控制设置是独立的，一个被设置另一个就会被移除（对于一个响应），对于同一个视图函数里面的不同响应，可以使用不同的设置，例如： from
django.views.decorators.cache import patch_cache_control from
django.views.decorators.vary import vary_on_cookie @vary_on_cookie def
list_blog_entries_view(request): if request.user.is_anonymous(): response =
render_only_public_entries() patch_cache_control(response, public=True) else:
response = render_private_and_public_entries(request.user)
patch_cache_control(response, private=True) return response
最后列举cache_control接收到额参数： public=Trueprivate=Trueno_cache=Trueno_transform=True
must_revalidate=Trueproxy_revalidate=Truemax_age=num_secondss_maxage=num_secon
ds

