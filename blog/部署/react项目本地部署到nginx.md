## 环境准备 ##

- windows 7 8 10
- nginx-1.10.2
- npm 


## 生成项目静态文件 ##
执行如下命令：

	npm run build

将项目build文件夹下的文件拷贝到如下目录：

	C:\static\tomato

## 配置nginx ##
这里主要涉及到一下几点：

1、创建vhost文件夹，将各个站点的配置分割到单个conf文件，避免nginx.conf的臃肿，便于维护查错。创建vhost文件夹下的conf的首要工作就是在nginx.conf的http对象中加上`include vhost/*.conf;`，确保nginx启动的时候，启动vhost下的所有conf配置。

2、关于反向代理的处理，在监听到80对api接口的所有请求，通过配置proxy_pass来实现。

3、图片或者css以及less等静态资源请求路径的配置。

**nginx相关的配置有待进一步细致的了解。**


nginx安装目录的文件夹conf下的vhost文件夹下创建配置文件 **tomato.com.conf**，配置如下：

	server {
	        listen       80;
	        server_name  react.tomato.com;
	
			error_log c:/access.log debug;
	
			location /api/ {
			    rewrite  ^/apis\//(.*)$ /$1 break; 
		     	    proxy_pass http://127.0.0.1:3111;
		            proxy_set_header Host $host;
		            proxy_set_header X-Real-IP $remote_addr;
		            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
	        }
	
			location ~ .*?\.(js|css|jpg|png|jpeg|less|sass){                
		    	root C:\static\tomato; 
	        }	

	        location / {
	            root   C:\static\tomato;
	            index  index.html index.htm;
		    	try_files $uri $uri/ @rewrites;
	        }
	
			location @rewrites {
	            rewrite ^(.+)$ /index.html last;
	        }
	    }

上述配置nginx会监听80端口，也就是http的默认端口，然后将location的为/api的请求代理转发到127.0.0.1:3111端口的后端服务。
需要注意server_name为react.tomato.com域名，在本地DNS映射为127.0.0.1

----------
### react-router下的nginx配置 ###

有用到的前端路由为react-router中的browserhistory，因此在路由跳转的时候需要使用try_files在路由切换的时候始终返回index.html，由于使用history时的路由是 www.xxx.com/a/b/c，url 是指向真实 url 的资源路径。因此回去服务端查询该资源。往往资源是不存在的于是就会报404。

- 上述配置中，try_filrs会到硬盘资源根目录里查找资源。如果存在名为$uri的文件就返回，如果找不到在找名为$uri/的目录，再找不到就会执行@rewrites。（$uri指找文件， $uri/指找目录）

- rewrite是nginx中的重定向指令。^(.+)$ 是重定向规则。/index.html则是root目录下的重定向路径文件。

## 更改hosts ##
在hosts文件下增加如下DNS映射:

	127.0.0.1	react.tomato.com

当在浏览器下访问react.tomato.com的时候，通过DNS解析为127.0.0.1:80,然后由nginx处理请求，将静态资源返回，将api请求进行代理转发。

## 启动nginx ##
通过cmd在nginx安装目录下通过以下命令操作nginx :

	start nginx               //开启
	nginx.exe -s stop         //关闭
	nginx.exe -s reload       //重启

## 访问react.tomato.com ##
在浏览器下访问react.tomato.com即可。
