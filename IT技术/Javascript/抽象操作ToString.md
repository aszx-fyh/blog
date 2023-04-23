# es规范中的ToString抽象操作摘录
本文整理自《你不知道的JavaScript(中卷)》 
好像看到有些地方把ToString抽象操作写成[[ToString]]，下文就用这个代替吧。
[[ToString]]负责处理非字符串到字符串的强制类型转化，像Object.prototype.toString方法调用的时候,内部就调用[[ToString]]。
基本类型的字符串化规则为:null=>'null',undefined=>'undefined',true=>'true'。
Number类型转化遵循通用规则。
对普通对象来说:除非自行定义,否则toString()(Object.prototype.toString)返回内部属性[[class]]的值,如[object Object]。
数组的默认toString()方法经过了重新定义,将所有单元字符串化后再用','连接。
### JSON字符串化
```JSON.stringify()```也用到了[[ToString]]
安全的JSON值(JSON-safe)可以使用```JSON.stringify```字符串化。不安全的JSON值有undefined,function,symbol和包含循环引用(对象之间相互引用,形成一个无限循环)的对象。支持JSON的语言无法处理它们。
```JSON.stringify```在对象中遇到undefined,function,symbol时会自动忽略,在数组中则会返回null(以保证单元位置不变)。
对象中如果定义了```toJSON()```方法,JSON字符串化时会首先调用该方法,并将其返回值字符串化。```toJSON()```应该返回一个能够被字符串化的安全JSON值。

```js
JSON.stringify(obj[,replacer:[Array|Function],space])
let a={
    b:42,
    c:'42',
    d:[1,2,3]
}
JSON.stringify(a,['b','c']);//'{"b":42,"c":"42"}'
JSON.stringify(a,(k,v)=>{
    if(k!=='c')return v;
});//'{"b":42,"d":[1,2,3]}'
```
```JSON.stringify```的第二个参数replaceer可以是一个数组,也可以是一个函数,
是数组:那必须是一个字符串数组,其中包含字符串化要处理的对象的属性名,除此之外的其他属性则被过滤掉。
是函数:这个函数会对对象本身调用一次,然后对对象中的每个属性调用一次,每次传递两个参数,键和值。如果要忽略某个键就返回undefined,否则就返回指定的值。
第三个参数space用来指定缩进格式,暂略。
