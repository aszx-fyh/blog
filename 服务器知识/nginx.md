## 配合 vue history 路由模式部署

```nginx
location /h5/web/Marketing/{
       try_files $uri /h5/web/Marketing/index.html ;
}
```

## Apache、IIS、Nginx 等绝大多数 web 服务器，都不允许静态文件响应 POST 请求，否则会返回“HTTP/1.1 405 Method not allowed”错误。

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

## location

```nginx
location / {
     #root 和 alias的区别,alias会截掉location匹配到的路径
     set $root "xxxx"; #设置变量
     add_header Content-Type application/json;#改变response content-type
     return 200 $http_referer;#输出变量内容,用来调试
     try_files #列出需要匹配的文件

    root /activity , try_files $uri /20210622/index.html , /static/js/xx.js  的区别,静态资源以root为根目录,
    root /activity/20210622 ,  20210622/static/js/xx.js

}
```

```nginx
 location  ~ ^/(.*?)/ {
        # add_header Content-Type application/json;

        # if ($uri ~ ^/(.*?)/){
        #      set $activity_path $1;
        # }

        # try_files $uri /$1/index.html;
        # try_files $uri /$activity_path/index.html;
        # try_files $uri @activity;


        # return 200 $uri.$1;


        #   return 200 $scheme$server_addr$request_uri;
}
```

## rewrite

> rewrite ^/(.\*) http://www.baidu.com/ permanent; # 匹配成功后跳转到百度，执行永久 301 跳转

| 标记符号  | 说明                                                 |
| --------- | ---------------------------------------------------- |
| last      | 本条规则匹配完成后继续向下匹配新的 location URI 规则 |
| break     | 本条规则匹配完成后终止，不在匹配任何规则             |
| redirect  | 返回 302 临时重定向                                  |
| permanent | 返回 301 永久重定向                                  |

## 变量

```nginx
$args ：这个变量等于请求行中的参数，同$query_string
$content_length ： 请求头中的Content-length字段。
$content_type ： 请求头中的Content-Type字段。
$document_root ： 当前请求在root指令中指定的值。
$host ： 请求主机头字段，否则为服务器名称。
$http_user_agent ： 客户端agent信息
$http_cookie ： 客户端cookie信息
$limit_rate ： 这个变量可以限制连接速率。
$request_method ： 客户端请求的动作，通常为GET或POST。
$remote_addr ： 客户端的IP地址。
$remote_port ： 客户端的端口。
$remote_user ： 已经经过Auth Basic Module验证的用户名。
$request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
$scheme ： HTTP方法（如http，https）。
$server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
$server_name ： 服务器名称。
$server_port ： 请求到达服务器的端口号。
$request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri ： 与$uri相同。
```
