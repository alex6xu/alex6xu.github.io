django里面有一个东西叫Context_processor from django.template import RequestContext
from django.shortcuts import render_to_response #do something or get extra
context def flush_nav(): #do some flush job pass def myprocessor(request):
flush_nav() #以下为视图方法 def index(request): #do something return
render_to_response("index.html",context_instance=RequestContext(request,
processors=[myprocessor])) def views_1(request): #do something return
render_to_response("views1.html",context_instance=RequestContext(request,
processors=[myprocessor]))
如下所述，你可以在每个views方法里面都调用它，也可以把这个模板处理器放到settings文件里面，让它作为一个全局处理器起作用，如：
TEMPLATE_CONTEXT_PROCESSORS = ( 'project_name.app_name.views.myprocessor' )
TEMPLATE_CONTEXT_PROCESSORS = ( "django.core.context_processors.auth",
"django.core.context_processors.debug", "django.core.context_processors.i18n",
"django.core.context_processors.media", "myapp.processor.foos", )

