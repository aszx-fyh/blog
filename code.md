### blob url => File

```js
function getFile(url) {
	return new Promise((resolve) => {
		const xhr = new XMLHttpRequest();
		xhr.open("GET", url);
		xhr.responseType = "blob";
		xhr.onload = function () {
			resolve(xhr.response);
		};
		xhr.send();
	});
}
getFile(this.imageList[0]).then((blob) => {
	console.log(new File([blob], "file"));
});
```
