```ts
import {createApp} from "vue"
import App from "./App.js";

const rootContainer = document.querySelector("#root");
// createApp源码在packages\runtime-dom\src\index.ts#65
createApp(App).mount(rootContainer);
```
`@vue/runtime-dom`是针对dom平台的包，其中`createApp`调用了`@vue/core`中的[[createRenderer]]方法创建`renderer`(渲染器)(`packages\runtime-core\src\renderer.ts#76`)，并对dom平台进行特殊处理。