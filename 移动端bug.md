* ios 聚焦input框导致页面放大
    > ios的input框font-size不得小于16px
    > [参考此文](https://stackoverflow.com/questions/2989263/disable-auto-zoom-in-input-text-tag-safari-on-iphone)

* video全屏再还原后层级变高,导致层级变高覆盖页面其他元素,解决方案如下
```
<video
			webkit-playsinline="true"
            playsinline="true"
			x5-video-player-type="h5"
			x5-video-orientation="portraint"
			class="video"
			object-fit="cover"
		></video>
```