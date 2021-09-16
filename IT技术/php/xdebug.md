# 开启xdebug调试

php.ini配置文件中加入

[XDebug]
zend_extension=php_xdebug.dll
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.remote_host = "127.0.0.1"
xdebug.remote_port = 9999
xdebug.remote_mode = "req"

php_xdebug.dll可以从[xdebug]( https://xdebug.org/ )下载，放到php的扩展目录(ext)里

## vscode使用xdebug断点调试php

vscode 添加调试配置文件launch.json

```json
 "configurations": [

  {

   "name": "Listen for XDebug",

   "type": "php",

   "request": "launch",

// php.ini里面设置的xdebug端口

   "port": 9999

  },

  {

   "name": "Launch currently open script",

   "type": "php",

   "request": "launch",

   "program": "${file}",

   "cwd": "${fileDirname}",

   "port": 9999

  }

 ]
```

