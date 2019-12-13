# X-HTTP-METHOD-OVERRIDE请求头

在php 的Laravel框架里面，用来模拟http 方法。比如有的客户端只支持get,post方法，不支持put,delete等方法。那就用下面的方法模拟

```html
<input name="_method" type="hidden" value="put" /> 
```

然后会发送X-HTTP-METHOD-OVERRIDE:PUT这样的请求头

Laravel就可以读取X-HTTP-METHOD-OVERRIDE头作为本次请求的method