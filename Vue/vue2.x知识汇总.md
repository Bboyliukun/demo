## vue中计算属性computer和普通属性method的区别是什么？
> computed属性是vue计算属性，是数据层到视图层的数据转化映射； 计算属性是基于他们的依赖进行缓存的，只有在相关依赖发生改变时，他们才会重新求值，也就是说，只要他的依赖没有发生变化，那么每次访问的时候计算属性都会立即返回之前的计算结果，不再执行函数；

#### computed是响应式的，methods并非响应式。
调用方式不一样，computed的定义成员像属性一样访问，methods定义的成员必须以函数形式调用
computed是带缓存的，只有依赖数据发生改变，才会重新进行计算，而methods里的函数在每次调用时都要执行。
#### computed中的成员可以只定义一个函数作为只读属性，也可以定义get/set变成可读写属性，这点是methods中的成员做不到的
computed不支持异步，当computed内有异步操作时无效，无法监听数据的变化
如果声明的计算属性计算量非常大的时候，而且访问量次数非常多，改变的时机却很小，那就需要用到computed；缓存会让我们减少很多计算量。

## vue的双向绑定的原理是什么？

> vue.js是采用数据劫持结合发布者-订阅者模式的方式，通过ES5提供的Object.defineProperty()方法来劫持(监视)各个属性的setter,getter，在数据变动的时发布消息给订阅者，触发相应的监听回调。并且，由于是在不同的数据上触发同步，可以精确的将变更发送发送给绑定的视图，而不是对所有的数据都执行一次检测。

具体的步骤：
1. 需要observer的数据对象进行递归遍历，包括子属性对象的属性，都加上getter和setter这样的话，给这个对象的某个值赋值，就会触发setter，那么就能监听到了数据变化
2. compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图
3. Watcher订阅者是Observer和compile之间通信桥梁，主要做的事情是：
   - 在自身实例化时往属性订阅器(dep)里面添加自己
   - 自身必须有一个update()的方法
   - 待属性变动dep.notice()通知的时候，能调用自身的update()方法，并触发compile中绑定的回调，则功成身退
4. MVVM作为数据绑定的入口，整合observer、compile和watcher三者，通过observer来监听自己的model数据变化，通过compile来编译模板指令，最终利用watcher搭起的observer和compile之间的通信桥梁，达到数据变化---试图更新;视图交互变化(input)-->数据model变更的双向绑定效果

> 版本比较： vue是基于依赖收集的双向绑定； 3.0版本之前使用Object.definePropetry,3.0新版使用Proxy

1. 基于数据劫持/依赖收集 的双向绑定的优点不需要显示的调用，Vue利用数据劫持+发布订阅，可以直接通知变化并且驱动视图直接得到精确的变化数据，劫持了属性setter，当属性值改变，我们可以精确的获取变化的内容newValue，不需要额外的diff操作
2. Object.defineProperty的缺点。
    - 不能监听数组：因为数组没有getter和setter。
    - 因为数组度不确定，如果太长性能负担太大只能监听属性，而不是整个对象，需要遍历循环属性只能监听属性变化，不能监听属性的删减
3. proxy的好处
   - 可以监听数组
   - 监听整个对象不是属性
   - 13种来截方法，强大很多
   - 返回新对象而不是直接修改原对象，更符合immutable；
4. proxy的缺点
   - 兼容性不好，而且无法用polyfill磨平；
   
## 说一下vue-router的原理是什么?
> 实现原理：vue-router的原理就是更新视图而不重新请求页面

#### vue-router可以通过mode参数设置为三种模式：
     hash模式
     history模式
     abstract模式。

1. hash模式 默认是hash模式，基于浏览器history api，使用window.addEventListener('hashchange',callback,false)对浏览器地址进行监听。当调用push时，把新路由添加到浏览器访问历史的栈顶。使用replace时，把浏览器访问历史的栈顶路由替换成新路由 hash的值等于url中#及其以后的内容。浏览器是根据hash值的变化，将页面加载到相应的DOM位置。锚点变化只是浏览器的行为，每次锚点变化后依然会在浏览器中留下一条历史记录，可以通过浏览器的后退按钮回到上一个位置

2. History history模式，基于浏览器history api ，使用window.onpopstate对浏览器地址进行监听。对浏览器history api中的pushState()、replaceState()进行封装，当方法调用，会对浏览器的历史栈进行修改。从而实现URL的跳转而无需加载页面 但是他的问题在于当刷新页面的时候会走后端路由，所以需要服务端的辅助来兜底，避免URL无法匹配到资源时能返回页面

3. abstract 不涉及和浏览器地址的相关记录。流程跟hash模式一样，通过数组维护模拟浏览器的历史记录栈 服务端下使用。使用一个不依赖于浏览器的浏览器历史虚拟管理后台

总结 hash模式和history模式都是通过window.addEvevtListenter()方法监听 hashchange和popState进行相应路由的操作。可以通过back、foward、go等方法访问浏览器的历史记录栈，进行各种跳转。而abstract模式是自己维护一个模拟的浏览器历史记录栈的数组。
## vue中计算属性computer和普通属性method的区别是什么？
> computed属性是vue计算属性，是数据层到视图层的数据转化映射； 计算属性是基于他们的依赖进行缓存的，只有在相关依赖发生改变时，他们才会重新求值，也就是说，只要他的依赖没有发生变化，那么每次访问的时候计算属性都会立即返回之前的计算结果，不再执行函数；

- computed是响应式的，methods并非响应式。
- 调用方式不一样，computed的定义成员像属性一样访问，methods定义的成员必须以函数形式调用computed是带缓存的，只有依赖数据发生改变，才会重新进行计算，而methods里的函数在每次调用时都要执行。
- computed中的成员可以只定义一个函数作为只读属性，也可以定义get/set变成可读写属性，这点是methods中的成员做不到的
- computed不支持异步，当computed内有异步操作时无效，无法监听数据的变化
如果声明的计算属性计算量非常大的时候，而且访问量次数非常多，改变的时机却很小，那就需要用到computed；缓存会让我们减少很多计算量。
## 说一下Vue的keep-alive是如何实现的，具体缓存的是什么？

keep-alive
props
include字符串或正则表达式，只有名称匹配的组件会被匹配
exclude字符串或正则表达式。任何名称匹配的组件都不会被缓存
max数字。最多可以缓存多少组件实例

> keep-alive 包裹动态组件时，会缓存不活动的组件实例

主要流程
 1. 判断组件name，不在include或者在exclude中，直接返回vnode，说明该组件不被缓存
 2. 获取组件实例key，如果由获取实例的key，否则重新生成。
 3. key生成规则，cid+"::"+tag，仅靠cid是不够的，因为相同的构造函数可以注册为不同的本地组件
 4. 如果缓存对象内存在，则直接从缓存对象中获取组件实例给vnode，不存在则添加到缓存对象中
 5. 最大缓存数量，当缓存数量超过max值时，清楚keys数组内的第一个组件
keep-alive的实现
``` const patternTypes:Array<Function>=[String,RegExp,Array]//接收：字符串、正则、数组
export default{
    name:'keep-alive',
    abstract:true,//一个抽象组件，自身不会渲染一个DOM元素，也不会出现在父组件中
    props：{
        include:patternTypes,//匹配的组件，缓存
        exclude:patternTypes,//不去匹配的组件，你缓存
        max:[String,Number],//缓存组件的最大实例数量，由于缓存的是组件实例(vnode)，数量过多的时候，会占用过多的内存，可以用max指定上限
    },
    create(){
        //用于初始化缓存虚拟DOM数组和vnode的key
        this.cache=Object.create(null)
        this.keys=[]
    },
    destroyed(){
        //销毁缓存cache的组件实例
        for(const key in this.cache){
            pruneCacheEntry(this.cache,key,this.keys)
        }
    },
    mounted(){
        //监控include和exclude的改变，根据最新的include和exclude的内容，来实时削减缓存的组建的内容
        this.$watch('include',(val)=>{
            pruneCache(this,(name=>matches(val,name)))
        })
        this.$watch('exclude',(val)=>{
            pruneCache(this,(name)=>!matches(val,name))
        })
    },
}
```

render函数
 1. 会在keep-alive组件内部去写自己的内容，所以可以去获取默认slot的内容，然后根据这个去获取组件
 2. keep-alive只对第一个组件有效，所以获取第一个子组件
 3. 和keep-alive搭配使用的一般由动态组件和route-view
``` render(){
    function getFirstComponentChild(children:?Array<VNode>):?VNode{
        if(Array.isArray(children)){
            for(var i=0;i< children.length;i++){
                const c=children[i]
                if(isDef(c)&&isDef(c.componentOptions)||isAsyncPlaceholder(c)){
                    return c
                }
            }
        }
        const slot=this.$slots.default//获取默认插槽
        const vnode:VNode=getFirstComponentChild(slot)//获取第一个子组件
        const componentOptions:?VNodeComponentOptions=vnode && vnode.componentOptions//组件参数
        if(componentOptions){//是否有组件参数
            const name:?string=getComponentName(componentOptions)//获得组件名字
            const {include,exclude}=this
            if(
                //not include
                (include && (!name || !matches(include,name))) ||
                //excluded
                (exclude && name && matches(exclude,name))
            ){
                //如果不匹配当前组件的名字和include以及exclude
                //那么直接返回组件的实例
                return vnode
            }
            const {cache,keys}=this
            //获取这个组件的key
            const key:?string=vnode.key == null?componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}`:''):vnode.key
            if(cache[key]){
                //LRU缓存策略执行
                vnode.componentInstance=cache[key].componentInstance//组件初次渲染的时候componentInstance为undefined
                remove(keys,key)
                keys.push(key)
                //根据LRU缓存策略进行，将key从原来的位置移除，然后将这个key值放到最后
            }else{
                //在缓存列表里没有的话，则加入，同时判断当前加入之后，是否超过了max所设定的范围，是的话那么就移除
                //使用时间间隔最长的一个
                cache[key]=vnode
                keys.push(key)
                if(this.max && keys.length >parseInt(this.max)){
                    pruneCacheEntry(cache,key[0],keys,this._vnode)
                }
            }
            //将组件的keepAlive属性设置为true
            vnode.data.keepAlive=true//判断是否执行组件的created、mounted生命周期函数
        }
    }
    return vnode||(slot && slot[0])
}
```
keep-alive 具体是通过cache数组缓存所有组件的vnode实例。当cache内原有组件被使用时会将该组件key从keys数组中删除，然后push到keys数组最后面，方便清除最不常用组件

步骤总结
1. 获取keep-alive下第一个子组件的实例对象，通过它去获取这个组件的名字
2. 通过当前组件名去匹配原来include和exclude，判断当前组件是否需要缓存，不需要缓存直接返回当前组件的实例vnode
3. 需要缓存，判断它当前是否在缓存数组里面，存在的话就将它原来的位置上的key给移除，同时将这个组件的key放到数组最后面
4. 不存在的话，将组件key放入数组，然后判断当前key数组是否超过max所设置的范围，超过的话那就削减没使用时间最长的一个组件的key值
5. 最后将这个组件的keepAlive设置为true
keep-alive本身的创建过程和patch过程
缓存渲染的时候，会根据vnode.componentInstance(首次渲染 vnode.componentInstance为undefined)和keepAlive属性判断不会执行组件的created、mounted等钩子函数，而是对缓存的组件执行patch过程：直接把缓存的DOM对象直接插入到目标元素中，完成了数据更新情况下的渲染过程

首次渲染 组件的首次渲染：判断组件的abstract属性，才往父组件里面挂载DOM
```function initLifecycle(vm:Component){
    const options=vm.$options
    let parent=options.parent
    if(parent && !options.abstract){//判断组件的abstract属性，才往父组件里面挂载DOM
        while(parent.$options.abstract && parent.$parent){
            parent=parent.$parent
        }
        parent.$children.push(vm)
    }
    vm.$parent=parent
    vm.$root=parent?parent:$root:vm
    vm.$children=[]
    vm.$refs={}
    vm._watcher=null
    vm._inactive=null
    vm._directInactive=false
    vm._isMounted=false
    vm._isDestroyed=false
    vm._isBeingDestoryed=false
}
```
判断当前keepAlive和componentInstance是否存在来判断是否要执行组件perpatch还是执行创建componentInstance

```init(vnode:VNodeWithData,hydrating:boolean):?boolean{
    if(vnode.componentInstance && !vnode.componentInstance.__isDestroyed && vnode.data.keepAlive){//首次渲染 vnode.componentInstance为undefined
        const mounteNode:any=vnode
        componentVNodeHooks.prepatch(mountedNode,mountedNode)//prepatch函数执行的是组件更新的过程
    }else{
        const child=vnode.componentInstance=createComponentInstanceForVnode(vnode,activeInstance)
    }
    child.$mount(hydrating?vode.elm:undefined,hydrating)
}
```
prepatch操作就不会在执行组件的mounted和created声明周期函数，而是直接将DOM插入

LRU(least recently used)缓存策略
LRU缓存策略:从内存找出最久未使用的数据并置换新的数据 LRU(least recently used)算法根据数据的历史访问记录来进行淘汰数据，其核心思想是：如果数据最近被访问过，那么将来被访问的几率也更高。最常见的实现是使用一个链表保存缓存数据，详细算法实现如下

1. 新数据插入到链表头部
2. 每当缓存数据被访问，则将数据移到链表头部
3. 链表满的时候，将链表尾部的数据丢弃
## 关于对 Vue 项目进行优化，你有哪些方法?
1. 代码层面的优化
   1. v-if 和 v-show 区分使用场景
   2. computed 和 watch 区分使用场景
   3. v-for 遍历必须为 item 添加 key，且避免同时使用 v-if
   4. 长列表性能优化
   5. 事件的销毁
   6. 图片资源懒加载
   7. 路由懒加载
   8. 第三方插件的按需引入
   9. 优化无限列表性能
   10. 服务端渲染 SSR or 预渲染
2. Webpack 层面的优化
   - Webpack 对图片进行压缩
   - 减少 ES6 转为 ES5 的冗余代码
   - 提取公共代码
   - 模板预编译
   - 提取组件的 CSS
   - 优化 SourceMap
   - 构建结果输出分析
   - Vue 项目的编译优化
3. 基础的 Web 技术的优化
   - 开启 gzip 压缩
   - 浏览器缓存
   - CDN 的使用
   - 使用 Chrome Performance 查找性能瓶颈
## 为什么组件中的 data 必须是一个函数，然后 return 一个对象，而 new Vue 实例里，data 可以直接是一个对象？
> 因为组件是用来复用的，并且 JS 里对象是引用关系，如果组件中 data 是一个对象，那么这样作用域没有隔离性，子组件中的 data 属性值会相互影响，如果组件中 data 选项是一个函数，那么每个实例可以维护一份被返回对象的独立的拷贝，组件实例之间的 data 属性值不会互相影响；而 new Vue 的实例，是不会被复用的，因此不存在引用对象的问题。

## 直接给一个数组项赋值，Vue 能检测到变化吗？
由于 JavaScript 的限制，Vue 不能检测到以下数组的变动：

当你利用索引直接设置一个数组项时，例如：```vm.items[indexOfItem] = newValue```
当你修改数组的长度时，例如：```vm.items.length = newLength```
为了解决第一个问题，Vue 提供了以下操作方法：
```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// vm.$set，Vue.set的一个别名
vm.$set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```
为了解决第二个问题，Vue 提供了以下操作方法：
```
// Array.prototype.splice
vm.items.splice(newLength)
```
## Vue 组件间通信有哪几种方式？
Vue 组件间通信只要指以下 3 类通信：
- 父子组件通信
- 隔代组件通信
- 兄弟组件通信
下面我们分别介绍每种通信方式:

1. props / $emit 适用 父子组件通信 这种方法是 Vue 组件的基础，相信大部分同学耳闻能详，所以此处就不举例展开介绍。
2. ref 与 $parent / $children 适用 父子组件通信
> ref：如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例
$parent/$children：访问父 / 子实例

3. EventBus($emit/$on)适用于 父子、隔代、兄弟组件通信 这种方法通过一个空的 Vue 实例作为中央事件总线（事件中心），用它来触发事件和监听事件，从而实现任何组件间的通信，包括父子、隔代、兄弟组件。
4. $attrs/$listeners 适用于 隔代组件通信
  > $attrs:包含了父作用域中不被 prop 所识别 (且获取) 的特性绑定 ( class 和 style 除外 )。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 ( class 和 style 除外 )，并且可以通过v-bind="$attrs"传入内部组件。通常配合inheritAttrs选项一起使用。$listeners:包含了父作用域中的 (不含 .native 修饰器的) v-on事件监听器。它可以通过v-on="$listeners"传入内部组件

5. provide / inject 适用于 隔代组件通信 祖先组件中通过 provider 来提供变量，然后在子孙组件中通过 inject 来注入变量。 provide / inject API 主要解决了跨级组件间的通信问题，不过它的使用场景，主要是子组件获取上级组件的状态，跨级组件间建立了一种主动提供与依赖注入的关系。
6. Vuex 适用于 父子、隔代、兄弟组件通信。Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。每一个 Vuex 应用的核心就是 store（仓库）。“store” 基本上就是一个容器，它包含着你的应用中大部分的状态 ( state )。
> Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化。
## 说一下Vue 的父组件和子组件生命周期钩子函数执行顺序？
Vue 的父组件和子组件生命周期钩子函数执行顺序可以归类为以下 4 部分：

- 加载渲染过程
> 父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted

- 子组件更新过程
> 父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated

- 父组件更新过程
> 父 beforeUpdate -> 父 updated

- 销毁过程
> 父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed


## 使用过 Vue SSR 吗？说说 SSR？

> Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。 即：SSR大致的意思就是vue在客户端将标签渲染成的整个 html 片段的工作在服务端完成，服务端形成的html 片段直接返回给客户端这个过程就叫做服务端渲染。

#### 服务端渲染 SSR 的优缺点如下：

1. 服务端渲染的优点：
- 更好的 SEO： 因为 SPA 页面的内容是通过 Ajax 获取，而搜索引擎爬取工具并不会等待 Ajax 异步完成后再抓取页面内容，所以在 SPA 中是抓取不到页面通过 Ajax 获取到的内容；而 SSR 是直接由服务端返回已经渲染好的页面（数据已经包含在页面中），所以搜索引擎爬取工具可以抓取渲染好的页面
- 更快的内容到达时间（首屏加载更快）： SPA 会等待所有 Vue 编译后的 js 文件都下载完成后，才开始进行页面的渲染，文件下载等需要一定的时间等，所以首屏渲染需要一定的时间；SSR 直接由服务端渲染好页面直接返回显示，无需等待下载 js 文件及再去渲染等，所以 SSR 有更快的内容到达时间
2. 服务端渲染的缺点：
- 更多的开发条件限制： 例如服务端渲染只支持 beforCreate 和 created 两个钩子函数，这会导致一些外部扩展库需要特殊处理，才能在服务端渲染应用程序中运行；并且与可以部署在任何静态文件服务器上的完全静态单页面应用程序 SPA 不同，服务端渲染应用程序，需要处于 Node.js server 运行环境；
- 更多的服务器负载：在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用CPU 资源 (CPU-intensive - CPU 密集)，因此如果你预料在高流量环境 ( high traffic ) 下使用，请准备相应的服务器负载，并明智地采用缓存策略。

## 说一下Vue的$nextTick原理？
> js运行机制(Event Loop)

```js执行是单线程的，它是基于事件循环的```

所有同步任务都在主线程上执行的，形成一个执行栈。主线程外，会存在一个任务队列，只要异步任务有结果，就在任务队列中放置一个事件。当执行栈中的所有同步任务执行完后，就会读取去任务队列。那么对应的异步任务，会结束等待状态，进入执行栈。
```主线程不断重复第三步```
这里之线程的执行过程是一个tick，而所有的异步结果都是通过任务队列来调度。Event Loop分为宏任务和微任务，无论是执行宏任务还是微任务，完成后都会进入到下一个tick，并且在两个tick之间进程UI渲染 由于Vue DOM 更新是异步执行的，即修改数据时，视图不会立即更新，而是监听数据变化，并缓存在同一事件循环中，等同一数据循环中的所有数据变化完成之后，在同一进行视图更新。为了确保得到更新后的DOM，所以设置了Vue.nextTick()方法

```什么是$nextTick```
Vue的核心方法之一，官方文档解释如下：在下次DOM更新循环之后执行的延迟回调。在修改数据之后立即使用这个方法，获取到更新后的DOM

1. MutationObserver
MutationObserver是HTML5中的API，是一个用于监听DOM变动的接口，它可以监听一个DOM对象上发生的子节点删除、属性修改、文本内容修改等。调用过程是首先给它绑定回调，得到MO实例对象，这个回调在MO实例对象上监听到变动时触发。这里的MO回调是放在microtask中执行的
```
//创建MO实例
const Observer=new MutationObserver(callback);
const textNode="你好，世界！"
Observer.observe(textNode,{
    characterData:true//监听文本内容的修改
})
```

2. 源码分析
nextTick的实现单独有一个JS文件来维护它。nextTick源码主要分为两块：能力检测和根据能力检测以不同方式执行回调队列 能力检测 由于宏任务耗费的时间是大于微任务的，所以在浏览器支持的情况下，优先使用微任务。如果浏览器不支持微任务，再使用宏任务
```
//空函数，可用作函数占位符
import {noop} from './util';

//错误处理函数
import {handleError} from './error';

//是否是IE、ios、内置函数
import {isIE,isIOS,isNative} from './env';

//用来存储所有需要执行的回调函数
const callbacks=[];

//标志下是否正在执行回调函数
const flag=false;

//对callback进行遍历，然后执行相应的回调函数
function shift(){
    flag=false
    //这里拷贝的原因是：有的回调函数执行过程中又往callbacks中加入内容
    //比如$nextTick的回调函数里还有$nextTick，后者应该放到下一轮的nextTick中执行，所以拷贝一份当前的，遍历执行完当前的即可，避免无休止的执行下去
    const cb=callbacks.slice(0)
    callbacks.length=0
    for(var i=0;i<cb.length;i++){
        cb[i]()
    }
}

const timeFunc;//异步执行函数，用于异步延迟调用flushCallbacks函数

if(typeof Promise !== "undefined" && isNative(Promise)){
    const p=Promise.resolve()
    timeFunc=()=>{
        p.then(flushCallbacks);
        //IOS的UIView、Peomise.then 回调被推入microTask队列，但是队列可能不会如期执行
        //因此，添加一个空计时器强制microTask
        if (isIOS) setTimeout(noop)
    }
    isUsingMicroTask=true;
}else if(!isIE && typeof MutationObserver !=='undefined' && (isNative(MutationObserver) || MutationObserver.toString === "[Object MutationObserverConstructor]")){
    //当原生Promise不可用时，使用原生MutationObserver
    let counter=1
    //创建MO实例，监听到DOM变动后会执行回调flushCallbacks
    const obs=new MutationObserver(flushCallbacks);
    const textNode=document.createTextNode(String(conter))
    obs.observe(textNode,{
        characterData:true,//监听目标的变化
    })
    //每次执行timeFunc都hi让文本节点的内容在0和1之间切换
    //切换之后将新值赋值到MO观察的文本节点上，节点内容变化会触发回调
    timeFunc=()=>{
        counter=(counter+1)%2
        textNode.data=String(counter)//触发回调函数
    }
    isUsingMicroTask=true；
}else if(typeof setImmediate !== 'undefined' && isNative(setImmediate)){
    timeFunc=()=>{
        setImmediate(flushCallbacks)
    }
}else{
    timeFunc=()=>{
        setTimeout(flushCallbacks,0)
    }
}
```
延迟调用优先级如下：Promise > MutationObserver > setImmediate > setTimeout
```
export function nextTick(cd? Function,ctx:Object){
    let _resolve
    //cb回调函数会统一处理压入callbacks数组
    callbacks.push(()=>{
        if(cb){
            try{
                cb.call(ctx)
            }catch(e){
                handleError(e,ctx,'nextTick')
            }
        }else if(_resolve){
            _resolve(ctx)
        }
    })
    //flag为false  说明本轮事件循环中没有执行过timeFunc函数
    if(!flag){
        flag=true;
        timeFunc()
    }
    //当不传入cb参数时，提供一个promise化的调用
    //比如nextTick().then(()=>{}),当_resolve执行时，就会跳转到then逻辑中
    if(!cd && typeof Promise !== 'undefined'){
        return new Promise(resolve=>{
            _resolve=resolve
        })
    }
}
```
nextTick.js对外暴露了nextTick这一个参数，所以每次调用Vue.nextTick时会执行:

- 把传入的回调函数cb压入callbacks数组
- 执行timeFunc函数，延迟调用flushCallbacks函数
- 遍历执行callbacks数组中的都有函数
- 这里的callbacks没有直接在nextTick中执行回调函数的原因是：保证在同一个tick内执行多次nextTick，不会开启多个异步任务，而是把这些异步任务都压成一个同步任务，在下一个tick执行完毕。

```使用方式
语法:Vue.nextTick([callback,context])
参数:
{Function}[callback]:回调函数，不传参数时提供promise调用
{Object}[context]:回调函数执行的上下文环境，不传默认是自动绑定到调用它的实例上
```
```
//改变数据
vm.message="changed";
//想要立即使用更新后的DOM，这样不行，因为设置message后DOM还没有更新
console.log(vm.$el.textContent)//并不会得到"changed"
//这样可以，nextTick里面的代码在DOM更新后执行
Vue.nextTick(function(){
    //DOM更新，可以得到"changed"
    console.log(vm.$el.textContent)
})
//作为一个promise使用，不传参数时回调
Vue.nextTick().then(function(){
    //DOM更新时的执行代码
})
```
Vue实例方法vm.$nextTick做了进一步封装，把context参数设置成当前的Vue实例 使用Vue.nextTick()是为了可以获取到更新后的DOM。触发时机:在同一事件循环中的数据变化后，DOM完成更新，立即执行Vue.nextTick()的回调

同一事件循环中的代码执行完毕 -- DOM更新 -- nextTick callback触发

#### 应用场景
- 在Vue生命周期的created()钩子函数进行的DOM操作一定要放在Vue.nextTick()的回调函数中，原因:是created()钩子函数执行时DOM其实并未进行渲染。

- 在数据变化后要执行的某个操作，而这个操作需要使用随数据改变而改变的DOM结构的时候，这个操作应该放在Vue.nextTick()的回调函数中，原因：Vue异步执行DOM更新，只要观察到数据变化，Vue将开启一个队列，并缓冲在同一个事件循环中发生的所有数据改变，如果同一个watcher被多次触发，只会被推入到队列中一次

### 总结
- Vue的nextTick其本质上是对JS执行原理EventLoop的一种应用
nextTick的核心应用：Promise、MutationObserver、setImmediate、setTimeout的原生JS的方法来模拟对应的微/宏任务的实现，本质时是为了利用JS的这些异步回调任务队列来实现Vue框架中自己的异步回调队列
- nextTick不仅是Vue内部的异步队列的调用方法，同时也允许开发者在实际项目中使用这个方法来满足实际应用中对DOM更新数据时的后续逻辑处理
nextTick是典型的将底层JS执行原理应用到具体案例中的示例
> 引入异步更新队列机制的原因

如果是同步更新，则多次对一个或多个属性赋值，会频繁触发UI/DOM的渲染，可以减少一些无用渲染

同时由于VirtualDOM的引入，每一次状态发生改变后，状态变化的信号会发送给组件，组件内部使用VirtualDOM进行计算得出需要更新的具体的DOM节点，然后对DOM进行更新操作，每次更新状态后的渲染过程需要更多的计算，而这种无用功也将浪费更多的性能，所以异步渲染变得更加至关重要。

## v-model是如何实现的，语法糖实际是什么？
> 语法糖
> 指计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。通常来说使用语法糖能够增加程序的可读性，从而减少程序代码出错的机会。糖在不改变其所在位置的语法结构的前提下，实现了运行时的等价。可以简单理解为：加糖前后的代码编译后一样，但是代码更简洁流畅、代码更语义自然

实现原理
1. 作用在普通表单元素上
动态绑定了input的value指向了message变量，并且在触发input事件的时候去动态把message设置为目标值
```
<input v-model="something"/>
//等同于
<input v-bind:value="message" v-on:input="message=$event.target.value"/>
//$event:当前触发的事件对象
//$event.target:当前触发的事件对象的dom
//$event.target.value:当前dom的value值
//在@input方法中，value-->something
//在:value中，something-->value
```
2. 作用在组件上
在自定义组件中，v-model默认会利用名为value的prop和名为input的事件

本质是一个父子组件通信的语法糖，通过prop和$.emit来实现 因此父组件v-model语法糖本质上可以修改为<child :value="text" @input="function(e){text=e}"></child> 在组件的实现中，我们是可以通过v-model属性来配置子组件接收的prop属性以及派发的事件名称
```
//父组件
<parent-input v-model="parent"></parent-input>
//等价于
<parent-input v-bind:value="parent" v-on:input="parent=$event.target.value"></parent-input>

//子组件
<input v-bind:vaule="message" v-on:input="onmessage"/>
props:{value:message}
methods:{
    onmessage(e){
        $emit('input',e.target.value)
    }
}
```
默认情况下，一个组件上的v-model会把value用作prop并且把input用作event但是一些输入类型比如单选框和复选框按钮可能想使用value prop来达到不同的目的。使用model选项可以回避这个情况产生的冲突。
js监听输入框输入数据的变化，用oninput事件，数据改变以后就会立刻触发这个事件。通过input事件把数据$emit出去，在父组件接收。父组件设置v-model的值为input$emit获取过来的值。
## 子组件可以直接改变父组件的数据么？说说你的理由？
> 主要是为了维护父子组件的单向数据流

每次父组件发生更新时，子组件中所有的prop都将会刷新为最新的值

如果这样做的话，Vue会在浏览器控制台中发出警告

Vue提倡单向数据流，即父级props的更新会流向子组件，但是反过来则不行。这是为了防止意外的改变父组件的状态，使得应用的数据流变得难以理解，导致数据流混乱。如果破环了单向数据流，当应用复杂情况时，debug的成本会非常高

只有通过$emit派发一个自定义事件，父组件接收后，由父组件修改.
## 能说下 vue-router 中常用的 hash 和 history 路由模式实现原理吗？
1. hash模式的实现原理
早期的前端路由的实现就是基于 location.hash 来实现的。其实现原理很简单，location.hash 的值就是 URL 中 # 后面的内容。比如下面这个网站，它的 location.hash 的值为 '#search'：

https://www.word.com#search
hash 路由模式的实现主要是基于下面几个特性：

URL 中 hash 值只是客户端的一种状态，也就是说当向服务器端发出请求时，hash 部分不会被发送； hash 值的改变，都会在浏览器的访问历史中增加一个记录。因此我们能通过浏览器的回退、前进按钮控制hash 的切换。可以通过 a 标签，并设置 href 属性，当用户点击这个标签后，URL 的 hash 值会发生改变；或者使用 JavaScript 来对 loaction.hash 进行赋值，改变 URL 的 hash 值。我们可以使用 hashchange 事件来监听 hash 值的变化，从而对页面进行跳转（渲染）
2. history 模式的实现原理
HTML5 提供了 History API 来实现 URL 的变化。其中做最主要的 API 有以下两个：history.pushState() 和 history.replaceState()。这两个 API 可以在不进行刷新的情况下，操作浏览器的历史纪录。唯一不同的是，前者是新增一个历史记录，后者是直接替换当前的历史记录，如下所示：
```
window.history.pushState(null, null, path)
window.history.replaceState(null, null, path)
```
history 路由模式的实现主要基于存在下面几个特性：
```
pushState 和 repalceState 两个 API 来操作实现 URL 的变化
我们可以使用 popstate 事件来监听 url 的变化，从而对页面进行跳转(渲染)
history.pushState() 或 history.replaceState() 不会触发 popstate 事件，这时我们需要手动触发页面跳转（渲染）
```

## 说一下Vue的生命周期以及每个阶段做的事情
```beforeCreate(创建前)``` 在数据观测和初始化事件还没有开始

```created(创建后)``` 完成数据观测，属性和方法的运算，初始化事件，$el属性还没有显示出来

```beforeMounted(挂载前)```在挂载开始之前被调用，相关的render函数首次被调用。实例已完成下面的配置：编译模板，把data里面的数据和模板生成html。此时还没有挂载html到页面上

```mounted(挂载后)```在el被新创建的vm.$el替换，并挂载到实例上去之后调用。实例已经完成下面的配置：用上面编译好的html内容替换el属性指向的DOM对象。完成模板中的html渲染到html页面中，此过程中可进行ajax交互

```beforeUpdate(更新前)```在数据更新之前调用，发生在虚拟dom重新渲染和打补丁之前。可以在该钩子中进一步更改状态，不会触发重复渲染过程

```update(更新后)```在由于数据更改导致的虚拟DOM重新渲染和打补丁之后调用。调用时，组件dom已经更新，所以可以执行依赖于dom操作。但是在大多数情况下，在此期间避免更改状态，因为这可能会导致更新无法循环。此钩子在服务端渲染期间不被调用

```beforeDestroy(销毁前)``` 在实例销毁之前调用。实例仍然完全可以用。

```destroyed(销毁后) ```在实例销毁之后调用。调用后，所有的事件监听器会被移除，所有的子实例也会被销毁。此钩子在服务端渲染期间不被调用

## 说一下Vue template到render的过程？
过程解析
> vue的模板编译过程如下：template - ast - render函数

vue在模板编译中执行compileToFunctions将template转化成render函数

```compileToFunctions```的主要核心点:

1. 调用parse方法将template转化为ast树(抽象语法树)
parse的目的：是把template转化为ast树，它是一种用js对象的形式来描述整个模板。

解析过程：利用正则表达式顺序解析模板，当解析到开始标签、闭合标签、文本的时候都会分别执行对应的回调函数，来达到构成ast树

ast元素节点(type)总共三种类型：普通元素--1，表达式--2，纯文本--3

2. 对静态节点做优化 这个过程主要分析出哪些是静态节点，在其做一个标记，为后面的更新渲染可以直接跳过，静态节点做优化
深度遍历AST,查看每个子树的节点元素是否为静态节点或者静态节点根。如果为静态节点，他们生成的dom永远不会改变，这对运行时模板更新起到优化作用

3. 生成代码 generate将ast抽象语法树编译成render字符串并将静态部分放到staticRender中，最后通过new Function(render)生成render函数


## 介绍下 vue-router 中的导航钩子函数
1. 全局的钩子函数 beforeEach 和 afterEach
beforeEach 有三个参数，to 代表要进入的路由对象，from 代表离开的路由对象。next 是一个必须要执行的函数，如果不传参数，那就执行下一个钩子函数，如果传入 false，则终止跳转，如果传入一个路径，则导航到对应的路由，如果传入 error ，则导航终止，error 传入错误的监听函数

2. 单个路由独享的钩子函数 beforeEnter，它是在路由配置上直接进行定义的

3. 组件内的导航钩子主要有这三种：beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave。它们是直接在路由组件内部直接进行定义的