目前大型互联网公司有腾讯、京东、美团、新浪、大众点评等都在使用。比如说腾讯的盖亚，基于docker部署管理，据说已经有万台规模，用于大数据处理。美团主要是用
于持续集成，自动构建方面，另外新浪也做了实践。容器化是一个大趋势，以后这方面的公司会越来越多。 Docker提供功能广泛，这里有几个的例子：
Images（镜像）：Docker可以通过Pull和Push命令构建对象到服务中心
Containers（容器）：Docker可以通过Start/Stop命令管理容器的生命周期
Logging（日志）：Docker可以通过stdout，stderro捕获输出所有的容器内部信息
Volumes（存储）：Docker可以创建和管理容器的相关文件存储
Networking（网络）：Docker可以创建管理虚拟的接口和内部所有容器之间的网络桥接
RPC：Docker服务器提供允许外部程序去控制所有容器的行为的API

