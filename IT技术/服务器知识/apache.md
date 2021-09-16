# apache

##  Listen.conf

 Listen指令指定服务器在那个端口或地址和端口的组合上监听接入请求。如果只指定一个端口，服务器将在所有地址上监听该端口。如果指定了地址和端口的组合，服务器将在指定地址的指定端口上监听。

使用多个Listen指令可以指定多个不同的监听端口和/或地址端口组合。服务器将会对列出的所有端口和地址端口组合上的请求作出应答。

例如，想要服务器接受80和8000端口上的请求，可以这样设置：

Listen 80
Listen 9007

为了让服务器在两个确定的地址端口组合上接受请求，可以这样设置：

Listen 192.170.2.1:9007
Listen 192.170.2.5:8000 

><VirtualHost *:9007>
>
>DocumentRoot "D:/phpstudy_pro/WWW/fecmall/appfront/web"
>
>  ServerName shop
>
>  
>
>  FcgidInitialEnv PHPRC "D:/phpstudy_pro/Extensions/php/php7.3.4nts"
>
>  AddHandler fcgid-script .php
>
>  FcgidWrapper "D:/phpstudy_pro/Extensions/php/php7.3.4nts/php-cgi.exe" .php
>
> <Directory "D:/phpstudy_pro/WWW/fecmall/appfront/web">
>
>   Options +Includes +FollowSymLinks +MultiViews
>
>   AllowOverride All
>
>  Require local
>
>  DirectoryIndex index.php index.html
>
> </Directory>
>
></VirtualHost>

 **conf\vhosts配置了虚拟主机后还要在系统hosts中建立ip和serverName的映射关系**

conf\vhosts里面ServerName和ServerAlias的关系？