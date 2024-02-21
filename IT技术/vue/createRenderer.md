源码在 `packages\runtime-core\src\renderer.ts#296`
调用了`重载函数baseCreateRenderer`，返回`render`,`hydrate`,`createApp`。
```ts
function baseCreateRenderer(
  options: RendererOptions,
  createHydrationFns?: typeof createHydrationFunctions
){
const render=()=>{}
const hydrate=()=>{}
return {
    render,
    hydrate,
    createApp: createAppAPI(render, hydrate)
	}
}
```
```ts
export function createAppAPI<HostElement>(
  render: RootRenderFunction<HostElement>,
  hydrate?: RootHydrateFunction
):( rootComponent: Component,  rootProps?: Data | null)=>App {
	 return function createApp(rootComponent, rootProps = null) {
	  // App packages\runtime-core\src\apiCreateApp.ts#31
		 const app:App={
				_uid: vue内部编号
				_component:根组件
				_props:根props                                                                  
				_container:创建app的容器，一般是dom对象
				_context:createAppContext(),app上下文，保存了app的全局指令，组件，配置等信息
				_instance: ComponentInternalInstance | null，内部使用的instance，对第三方库或者工具有用
				get config() {// 获取全局配置
			        return context.config  //packages\runtime-core\src\apiCreateApp.ts#76
			    },
			    use(plugin: Plugin, ...options: any[]){
				    //注册插件
				    return app
			    },
			    mixin(mixin: ComponentOptions) {
				     //全局mixin，
				     return app
			    },
			    component(name: string, component?: Component): any {
				    // 全局组件，可以看到是注册到context上
				    context.components[name] = component
				    return app
			    }
			    directive(name: string, directive?: Directive) {
				    // 全局指令，同component
				    return app
			    },
			    mount( rootContainer: HostElement,isHydrate?: boolean,isSVG?: boolean){
				    const vnode = createVNode( rootComponent as ConcreteComponent, rootProps)
				    vnode.appContext = context //根vnode有appContext
				     if (__DEV__) {
					       context.reload = () => {//  HMR
				              render(cloneVNode(vnode), rootContainer, isSVG)
			           }
				     }
				     if (isHydrate && hydrate) {
			            hydrate(vnode as VNode<Node, Element>, rootContainer as any)
			          } else {
			            render(vnode, rootContainer, isSVG)
			          }
			           app._container = rootContainer
					   return getExposeProxy(vnode.component!) || vnode.component!.proxy
			    },
			    unmount(){
					    //卸载 app
			    },
			    provide(){
				    // provide/inject模式全局数据，可以在组件中使用inject获取
				    return app
			    }
		 }
		 return app
	 }
}
```