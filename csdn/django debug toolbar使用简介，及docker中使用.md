Installation Getting the code The recommended way to install the Debug Toolbar
is via pip: $ pip install django-debug-toolbar If you aren’t familiar with
pip, you may also obtain a copy of thedebug_toolbar directory and add it to
your Python path. To test an upcoming release, you can install the in-
development versioninstead with the following command: $ pip install -e
git+https://github.com/jazzband/django-debug-toolbar.git#egg=django-debug-
toolbar Prerequisites Make sure that 'django.contrib.staticfiles' is set up
properly and add'debug_toolbar' to your INSTALLED_APPS setting: INSTALLED_APPS
= [ # ... 'django.contrib.staticfiles', # ... 'debug_toolbar', ] STATIC_URL =
'/static/' If you’re upgrading from a previous version, you should review
thechange log and look for specific upgrade instructions. URLconf Add the
Debug Toolbar’s URLs to your project’s URLconf as follows: from django.conf
import settings from django.conf.urls import include, url if settings.DEBUG:
import debug_toolbar urlpatterns += [ url(r'^__debug__/',
include(debug_toolbar.urls)), ] This example uses the __debug__ prefix, but
you can use any prefix thatdoesn’t clash with your application’s URLs. Note
the lack of quotes arounddebug_toolbar.urls. Middleware The Debug Toolbar is
mostly implemented in a middleware. Enable it in yoursettings module as
follows: MIDDLEWARE = [ # ...
'debug_toolbar.middleware.DebugToolbarMiddleware', # ... ] Old-style
middleware: MIDDLEWARE_CLASSES = [ # ...
'debug_toolbar.middleware.DebugToolbarMiddleware', # ... ] Warning The order
of MIDDLEWARE and MIDDLEWARE_CLASSES is important. Youshould include the Debug
Toolbar middleware as early as possible in thelist. However, it must come
after any other middleware that encodes theresponse’s content, such
asGZipMiddleware. Internal IPs The Debug Toolbar is shown only if your IP is
listed in the INTERNAL_IPSsetting. (You can change this logic with the
SHOW_TOOLBAR_CALLBACKoption.) For local development, you should add
'127.0.0.1' toINTERNAL_IPS. USE IN DOCKER : IP address that allowed me to
display django toolbar was the IP of the getway associated with my docker
container. To obtain IP of the gatway I run this comand docker inspect
my_container_name | grep -e '"Gateway"' # "Gateway": "172.18.0.1", In total my
settings look like this INSTALLED_APPS = ( 'debug_toolbar', ) INTERNAL_IPS =
['172.18.0.1']

