# css img标签缩放
总所周知<code>img</code>元素是替换元素,width和height不设置时(默认为auto),会按照图片本身的尺寸渲染.如果设置了其中一个而另一个不设置,
那么另一个会根据图片本身的宽高比自动计算生成.
```css
img{
    width:50%;/*只设置宽度,高度会根据宽高比自动计算.*/
}
```
## css中元素等比例缩放
关键是在要等比缩放的元素外面套一层div,并利用```padding-left```和```padding-right```百分比相对父元素的width计算.并且自身设置宽高充满父元素

```css
.container
{
    position: relative;
    width:100%;
    padding-top:56.25%;/* 9/16*100% 根据宽高比设置*/
    border:1px solid goldenrod;
    height: 0;
}
.cimg
{
    position:absolute;
    top:0;
    left:0;
    width: 100%;
    height: 100%;
}
```
```html
<div class="container">
   <img class="cimg" src='../img/desktop.png' alt="desktop"/>   
</div>
```