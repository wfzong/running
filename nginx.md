# nginx 配置
**Nginx** 是一个异步框架的 Web服务器，也可以用作反向代理，负载平衡器 和 HTTP 缓存。  
## Web服务器
由[域名](domain.md)一节了解到，当我们注册了一个域名，并将其解析到指定的服务器，服务器如何处理这个请求呢，这个时间就该nginx上场了。  
由于目前只有这一台主机，同时又想响应多个域名的请求，就需要用到vhost（虚拟主机），具体配置如下：
```
# vim /etc/nginx/nginx.conf
```
进入nginx的配置文件后，在倒数第2行插入
```
include /etc/nginx/vhost/*.conf;
```
保存退出编辑后，进入nginx目录并创建对应的文件夹和文件
```
# cd /etc/nginx/
# mkdir vhost
# cd vhost
```
例如已经将 test.mimei.net.cn 解析到服务器 39.106.197.173，现在创建对应的配置文件：mimei.net.cn-test.conf，如下
```
# vim mimei.net.cn-test.conf

server {
        listen  80;
        server_name  test.mimei.net.cn;
        root /var/www/mimei.net.cn/test/;

        access_log  /var/log/nginx/mimei.net.cn-test.log  main;

        location / {
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root /var/www/mimei.net.cn/test/;
        }

        location ~ /.ht {
            deny  all;
        }
}

```
从配置文件可以知道，日志文件放在 /var/log/nginx/mimei.net.cn-test.log，网站根目录在：/var/www/mimei.net.cn/test/
，现在重新载入nginx配置文件，并在网站根目录创建一个文件：
```
# nginx -s reload
# vim /var/www/mimei.net.cn/test/index.html

<DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>测试页面</title>
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
</head>
测试页面
<body>
</body>
</html>

```
这里在浏览器里输入域名：http://test.mimei.net.cn/，可以看到：  
![测试页面](./images/nginx/01.png)  
nginx配置虚拟主机就完成了。