```xml
server {
        listen    443;
        server_name www.test123.com;
 
       #开启ssl证书认证
        ssl on;
        access_log  /data/nginx/www.test123.com.access.log main;
        keepalive_timeout 60;
 
       #证书路径，根据实际情况改写
        ssl_certificate  /data/nginx/cert/www.test123.com.com.pem;
        ssl_certificate_key /data/nginx/cert/www.test123.com.com.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        #禁止在header中出现服务器版本，防止黑客利用版本漏洞攻击
        server_tokens off;
 
}
server {
    listen 80;
    server_name www.test123.com;
 
   #核心代码
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}
```

