# js属性描述符
js可以用```Object.defineProperty```给对象添加属性并设置属性描述符。如:
```javascript
let myobj={};
Object.defineProperty(myobj,'a',{
    value:5,
    writable:true,
    configurable:true,//配置描述符
    enumerable:true,//枚举描述符

    get(){},
    set(){},
})
```
```value```和```writable```被称为数据描述符,而```get```和```set```被称为访问描述符。这两组描述符是有基友关系的,value和writable一组,get,set一组,不能拆开,两组也不能同时存在。

```Object.propertyIsEnumerable```用来检测属性是否可以枚举。即enumerable描述符是否为true,若不可枚举则遍历对象属性的时候不会出现该属性。
```javascript
myobj.propertyIsEnumerable('name');//false
```
## 给想遍历的对象定义@@iterator
```javascript
let myobj=[1,2,3,4,4,5];
Object.defineProperty(myobj,Symbol.iterator,{
    enumerable:false,
    writable:false,
    configurable:true,
    value:function(){
        let o=this;//指向调用者
        let index=0;
        let keys=Object.keys(o);
        return {//一个具有next函数的对象,next函数返回一个具有value,和done的对象。
            next:function(){
                
                return {
                    value:o[keys[index++]]
                    done:keys.length<index
                }
            }
        }
    }
})
let it=myobj[Symbol.iterator]();
it.next();
it.next();
...
for(let v of myobj){//寻找内置或自定义的@@iterator对象并调用它的next方法来遍历数据。就是Symbol.iterator这个属性
    console.log(v);
}
```