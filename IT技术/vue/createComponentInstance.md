`packages\runtime-core\src\component.ts#463`
类型 `packages\runtime-core\src\component.ts#204`

component instance 链
可以在源码vue.js 5628行 mountComponent函数里断点查看instance值
```ts
export function createComponentInstance(
  vnode: VNode,
  parent: ComponentInternalInstance | null,
  suspense: SuspenseBoundary | null
) {
const type = vnode.type as ConcreteComponent
  // inherit parent app context - or - if root, adopt from root vnode
  const appContext =
    (parent ? parent.appContext : vnode.appContext) || emptyAppContext

  const instance: ComponentInternalInstance = {

    uid: uid++,

    vnode,// 创建instance的vnode

    type,// 对应的组件

    parent,// 父instance

    appContext,

    root: null!, // to be immediately set//instance链的根

    next: null,

    subTree: null!, // will be set synchronously right after creation

    effect: null!,

    update: null!, // will be set synchronously right after creation
     
    scope: new EffectScope(true /* detached */),

    render: null,

    proxy: null,

    exposed: null,

    exposeProxy: null,

    withProxy: null,
// 数据注入
    provides: parent ? parent.provides : Object.create(appContext.provides),

    accessCache: null!,

    renderCache: [],

  

    // local resolved assets

    components: null,

    directives: null,

  

    // resolved props and emits options
//组件注册的props和emits
    propsOptions: normalizePropsOptions(type, appContext),

    emitsOptions: normalizeEmitsOptions(type, appContext),

  

    // emit
//emit函数
    emit: null!, // to be set immediately

    emitted: null,

  

    // props default value
// props default为函数时，求值并保存
    propsDefaults: EMPTY_OBJ,

  

    // inheritAttrs

    inheritAttrs: type.inheritAttrs,

  

    // state，组件渲染时状态（实时数据）

    ctx: EMPTY_OBJ,

    data: EMPTY_OBJ,

    props: EMPTY_OBJ,

    attrs: EMPTY_OBJ,

    slots: EMPTY_OBJ,

    refs: EMPTY_OBJ,

    setupState: EMPTY_OBJ,

    setupContext: null,

  

    // suspense related

    suspense,

    suspenseId: suspense ? suspense.pendingId : 0,

    asyncDep: null,

    asyncResolved: false,

  

    // lifecycle hooks

    // not using enums here because it results in computed properties

    isMounted: false,

    isUnmounted: false,

    isDeactivated: false,

    bc: null,

    c: null,

    bm: null,

    m: null,

    bu: null,

    u: null,

    um: null,

    bum: null,

    da: null,

    a: null,

    rtg: null,

    rtc: null,

    ec: null,

    sp: null

  }

  if (__DEV__) {

    instance.ctx = createDevRenderContext(instance)

  } else {

    instance.ctx = { _: instance }

  }

  instance.root = parent ? parent.root : instance

  instance.emit = emit.bind(null, instance)

  

  // apply custom element special handling

  if (vnode.ce) {

    vnode.ce(instance)

  }

  

  return instance


}
```

[[EffectScope]]