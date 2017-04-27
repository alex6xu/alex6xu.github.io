1.tornado的demo  首先写一个tornado的demo
在生产环境中，一般使用单个的进程启动，为了简单起见，这里我们使用multiprocessing模块启动多个进程，模拟生产环境
#!/usr/bin/python #-*-encodeing:utf-8-*- import tornado.web import
tornado.ioloop import tornado.options import multiprocessing from
tornado.options import define,options import os,sys define("port",
default=9000, help="run on the given port", type=int) class
BaseHandler(tornado.web.RequestHandler): def get_current_user(self): return
self.get_secure_cookie('user') def get_template_path(self): return
os.path.join(os.path.dirname(__file__),'templates') class
MainHandler(BaseHandler): @tornado.web.asynchronous @tornado.web.authenticated
def get(self): name=tornado.escape.xhtml_escape(self.current_user)
self.write('Hello'+self.current_user) self.finish() class
LoginHandler(BaseHandler): def lower(self,string): return string.lower() def
get(self): self.write('''  Username: Password: '''), def post(self): if not
self.request.headers.get('Cookie'): self.write('Please enable your Cookie
option of your broswer.') return
self.set_secure_cookie('user',self.get_argument('username'),expires_days=1)
self.redirect('/') settings={
'static_path':os.path.join(os.path.dirname(__file__),'static'),
'cookie_secret':'F/hsxF7kTIWGO1F6HrH78Rf4bMRe5EyFhjtReh6x+/E=',
'login_url':'/login', 'debug':True, } app=tornado.web.Application([
(r'/',MainHandler), (r'/login',LoginHandler), ],**settings) if __name__ ==
'__main__': tornado.options.parse_command_line() def run(mid,port): print
"Process %d start" % mid sys.stdout.flush() app.listen(port)
tornado.ioloop.IOLoop.instance().start() jobs=list() for mid,port in
enumerate(range(9010,9014)):
p=multiprocessing.Process(target=run,args=(mid,port)) jobs.append(p) p.start()
运行后,会启动4个进程  2.配置nginx的反相代理
参考http://my.oschina.net/chenlei123/blog/128299安装好nginx后  进入nginx的配置目录：  cd
/usr/local/nginx/conf  备份原配置文件  mv nginx.conf  nginx.conf.bak20130507
使用编辑器打开nginx.conf  #user nginx; worker_processes 1; error_log
/var/log/nginx/error.log; pid /var/run/nginx.pid; events { worker_connections
1024; use epoll; } http { # Enumerate all the Tornado servers here upstream
frontends { server 127.0.0.1:9010; server 127.0.0.1:9011; server
127.0.0.1:9012; server 127.0.0.1:9013; } include /etc/nginx/mime.types;
default_type application/octet-stream; access_log /var/log/nginx/access.log;
keepalive_timeout 65; proxy_read_timeout 200; sendfile on; tcp_nopush on;
tcp_nodelay on; gzip on; gzip_min_length 1000; gzip_proxied any; gzip_types
text/plain text/html text/css text/xml application/x-javascript
application/xml application/atom+xml text/javascript; # Only retry if there
was a communication error, not a timeout # on the Tornado server (to avoid
propagating "queries of death" # to all frontends) proxy_next_upstream error;
server { listen 80; # Allow file uploads client_max_body_size 50M; location ^~
/static/ { root /var/www; if ($query_string) { expires max; } } location =
/favicon.ico { rewrite (.*) /static/favicon.ico; } location = /robots.txt {
rewrite (.*) /static/robots.txt; } location / { proxy_pass_header Server;
proxy_set_header Host $http_host; proxy_redirect false; proxy_set_header X
-Real-IP $remote_addr; proxy_set_header X-Scheme $scheme; proxy_pass
http://frontends; } } } 重启nginx  /etc/init.d/nginx_init reload
在浏览器中打开服务器ip的80端口就可以访问tornado的demo应用了

