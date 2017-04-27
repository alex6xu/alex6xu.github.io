Django 和其他 Web 框架的 HTTP 处理的流程大致相同，Django 处理一个 Request 的过程是首先通过中间件，然后再通过默认的 URL
方式进行的。我们可以在 Middleware 这个地方把所有 Request 拦截住，用我们自己的方式完成处理以后直接返回 Response。 1.
加载配置 Django 的配置都在 “Project/settings.py” 中定义，可以是 Django 的配置，也可以是自定义的配置，并且都通过
django.conf.settings 访问，非常方便。 2. 启动 最核心动作的是通过
django.core.management.commands.runfcgi 的 Command 来启动，它运行
django.core.servers.fastcgi 中的 runfastcgi ， runfastcgi 使用了 flup 的 WSGIServer
来启动 fastcgi 。而 WSGIServer 中携带了 django.core.handlers.wsgi 的 WSGIHandler
类的一个实例，通过 WSGIHandler 来处理由Web服务器（比如Apache，Lighttpd等）传过来的请求，此时才是真正进入 Django
的世界。 3. 处理 Request 当有 HTTP 请求来时， WSGIHandler 就开始工作了，它从 BaseHandler 继承而来。
WSGIHandler 为每个请求创建一个 WSGIRequest 实例，而 WSGIRequest 是从 http.HttpRequest
继承而来。接下来就开始创建 Response 了。 4. 创建Response BaseHandler 的 get_response 方法就是根据
request 创建 response ， 而 具体生成 response 的动作就是执行 urls.py 中对应的view函数了，这也是
Django可以处理“友好URL”的关键步骤，每个这样的函数都要返回一个 Response 实例。此时一般的做法是通过 loader 加载 template
并生成页面内 容，其中重要的就是通过 ORM 技术从数据库中取出数据，并渲染到 Template 中，从而生成具体的页面了。 5. 处理Response
Django 返回 Response 给 flup ， flup 就取出 Response 的内容返回给 Web 服务器，由后者返回给浏览器。 总之，
Django 在 fastcgi 中主要做了两件事：处理 Request 和创建 Response ，
而它们对应的核心就是“urls分析”、“模板技术”和“ORM技术”。 如图所示，一个 HTTP 请求，首先被转化成一个 HttpRequest
对象，然后该对象被传递给 Request 中间件处理，如果该中间件返回了Response，则直接传递给 Response 中间件做收尾处理。否则的话
Request 中间件将访问 URL 配置，确定哪个 view 来处理，在确定了哪个 view 要执行，但是还没有执行该 view 的时候，系统会把
request 传递给 View 中间件处理器进行处理，如果该中间件返回了Response，那么该 Response 直接被传递给 Response
中间件进行后续处理，否则将执行确定的 View 函数处理并返回 Response，在这个过程中如果引发了异常并抛出，会被 Exception
中间件处理器进行处理。

