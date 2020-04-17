##  配合vue history路由模式部署

```nginx
location /h5/web/Marketing/{
       try_files $uri /h5/web/Marketing/index.html ;
} 
```

## Apache、IIS、Nginx等绝大多数web服务器，都不允许静态文件响应POST请求，否则会返回“HTTP/1.1 405 Method not allowed”错误。
 
```nginx
server {
   listen       80;
   server_name  域名;
    
   location /{
      root /www/文件目录;
      index index.html index.htm index.php;
      error_page 405 =200 http://$host$request_uri;
   }
}
```