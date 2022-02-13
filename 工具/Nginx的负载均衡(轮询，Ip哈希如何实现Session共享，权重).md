# Nginx的负载均衡(轮询，Ip哈希如何实现Session共享，权重)

**Nginx介绍**

Nginx (engine x) 是一个轻量级，高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。

**特点：**
- 1.开发语言：C语言；
- 2.占用内存小： 2M；
- 3.并发能力强  5万/秒  实际3万/秒
- 4.Nginx可以当作负载均衡器使用(软件级)

**Nginx命令**

要求:执行nginx命令需要在nginx.exe根目录中执行.
```
1.启动命令     start nginx
2.重启命令     nginx -s reload
3.关闭命令     nginx -s stop
4.DOS批量关闭进程： taskkill /f /fi "imagename eq nginx.exe"
```

**反向代理入门**

Nginx中的配置文件config

说明:用户访问localhost:80 如何跳转到页面中.
```
server {
    	#nginx默认监听的端口号80
        listen       80;
        #拦截用户的请求路径  
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

		#反向代理配置  / 代表全部请求
        location / {
        	#root是关键字  代表跳转文件目录
            root   html;
            #index是关键字 默认欢迎页面
            index  index.html index.htm;
        }
	  }
```
**反向代理图片服务器**
```
#配置图片服务器
	server {
		listen 80;
		server_name image.jt.com;
            
            #执行转发的过程
		#个别计算机,不能识别\,换位/
            #反向代理配置  / 代表全部请求都转发
		location / {
			root D:/1-JT/image;
		}
	}
```
**修改hosts文件**

路径: C:\Windows\System32\drivers\etc\hosts

hosts作用是实现域名和ip的映射

配制:
```
#京淘项目环境配置
127.0.0.1   image.jt.com
127.0.0.1   manage.jt.com
127.0.0.1   www.jt.com
127.0.0.1   sso.jt.com
127.0.0.1   cart.jt.com
127.0.0.1   order.jt.com
```
**Nginx实现负载均衡**

![mskt_47](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_47.png)

**1 轮循策略**

策略说明:根据配置文件的顺序,依次访问tomcat服务器.
```
server{
	listen 80;
	server_name manage.jt.com;
	location / {
		proxy_pass http://jtWindows;
	}
}
	#定义windows集群
	upstream jtWindows {
		server localhost:8081;
		server localhost:8082;
		server localhost:8083;
	}
```

**权重方式(旧公司)**

需求:公司中服务器可能会更新迭代.导致物理设备处理能力不同.如果这时采用轮询的机制,可能会影响用户使用.</br>
说明:为了让性能更好的服务器更多的承担访问压力.所以采用权重策略.
```
#定义windows集群  1.默认 轮询 2.权重
	upstream jtWindows {
		server localhost:8081 weight=6;
		server localhost:8082 weight=3;
		server localhost:8083 weight=1;
	}
```

**IP_HASH(了解)过时**

需求:如何实现一次登录,可以访问其他相互信任的系统</br>
问题:如果将用户信息使用Session保存.则可能会导致用户频繁的登录.用户体验差.</br>
业务需求:要求实现session共享!!!,用户要求只登陆一次,即可访问全部的tomcat集群.

解决思路:
- 1.nginx实现Session共享. 用户的信息紧紧的绑定到了nginx中.耦合性高.</br>
  缺点:</br>
  如果nginx服务器宕机,则直接影响用户.
  nginx服务器中保存了大量的用户Session信息.内存开销大.
- 2.url重写技术.sessionId=UUID,动态拼接SessionId</br>
  ![mskt_48](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_48.png)
- 3:采用IPHASH策略：将用户信息(IP地址)动态的与服务器进行绑定.将来用户访问的服务器只有一台.</br>
  ![mskt_49](https://alexleon.oss-cn-shanghai.aliyuncs.com/markdown-pic/%E9%9D%A2%E8%AF%95%E8%80%83%E9%A2%98/mskt_49.png)</br>
  优点:利用用户的IP地址进行hash运算.之后与服务器进行动态的绑定.变相的实现Session共享.</br>
  缺点:
- 会导致严重的负载不均.
- 如果其中某台服务器宕机. 会转到新服务器，但是直接影响绑定用户，即新服务器需要再次登陆。

**Nginx高级属性配置**

- down属性

属性作用:标识故障机. nginx以后不再访问该机器.

- backup属性

说明:一般配置集群时会添加一些备用机.防止主机宕机之后没有足够的服务器响应用户.导致服务器异常.

触发策略: 当主机遇忙时/主机全部宕机时.backup主机生效.
```
#定义windows集群  1.默认 轮询 2.权重 3.IPhash
	upstream jtWindows {
		#ip_hash;
		server localhost:8081 down;
		server localhost:8082 down;
		server localhost:8083 backup;
	}
```

**Nginx高可用配置**

max_fails=1  	:定义最大的失败次数,该服务器连续失败几次,不再访问该服务器.

fail_timeout=60s   定义访问的周期 60秒
```
#定义windows集群  1.默认 轮询 2.权重 3.IPhash
	upstream jtWindows {
		#ip_hash;
		server localhost:8081 max_fails=1 fail_timeout=60s;
		server localhost:8082 max_fails=1 fail_timeout=60s;
		server localhost:8083 max_fails=1 fail_timeout=60s;
	}
```
定义超时时间
```
proxy_connect_timeout       1;  
proxy_read_timeout          1;  
proxy_send_timeout          1;  
```
