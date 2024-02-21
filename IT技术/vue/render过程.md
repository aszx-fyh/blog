```ts
import { h, render } from "vue";
const container = document.getElementById("app")!;
const app = h(
    {
        setup(props, ctx) {
            console.log("setup", props, ctx);
            return () => h("span", 2222);
        },
    },
);
render(app, container);
```

`render`由[[createRenderer]]创建
[[patchCompnent.png |组件挂载流程]]
### processComponent

mountComponent or updateComponent
### mountComponent
	packages\runtime-core\src\renderer.ts#1182
```ts
const compatMountInstance = 
	  __COMPAT__ && initialVNode.isCompatRoot && initialVNode.component
const instance: ComponentInternalInstance = compatMountInstance ||
      (initialVNode.component = createComponentInstance(
        initialVNode,
        parentComponent,
        parentSuspense
      ))
     // inject renderer internals for keepAlive
    if (isKeepAlive(initialVNode)) {
      ;(instance.ctx as KeepAliveContext).renderer = internals
    }
  setupComponent(instance)
   if (__FEATURE_SUSPENSE__ && instance.asyncDep) {
   //异步组件
   }
setupRenderEffect(
      instance,
      initialVNode,
      container,
      anchor,
      parentSuspense,
      isSVG,
      optimized
    )
```
[[createComponentInstance]]
[[setupComponent]]
[[asyncComponent]]
[[setupRenderEffect]]
### updateComponent
packages\runtime-core\src\renderer.ts#1258