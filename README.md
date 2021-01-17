# 简答题

## 1、请简述 Vue 首次渲染的过程。

---

`core/instance/index.js`中定义 vue 构造函数，调用`_init()` 方法开始，同时注册了 vm 的实例成员，`core/index.js` 中 `initGlobalAPI` 给 `Vue` 初始化了静态成员；`web/runtime/index.js`注册平台相关组件指令, `__path__`, `$mount`方法；`entry-runtime-with-compiler.js` 替换了 `$mount`;
`_init()` 进入后, 判断是否是 Vue，用于后续不处理响应式, 然后合并选项，设置 渲染代理对象， 调用一些 init，给 vm 挂载实例成员，接着进入 `$mount`(with-compiler) , 在 `$mount` 中， 如果选项没有 `render` 就获取 `template` , `compileToFunctions()` 将 `template` 编译成`模板函数`, 然后 `mount`(runtime/index) 方法中调用 `mountComponent` , `callHook()` 调用 `beforeMount` 生命周期函数, 接着定义 `updateComponent` 函数, 为 `vm._update(vm._render(),hydrating)`, 将虚拟 dom 转换成真实 dom, 接着 `new Watcher()` 中调用了 `updateComponent`, 最后调用 `mounted` 钩子；`new Watcher()` 中定义了 watcher 实例的属性，将 `updateComponent` 赋值给 `this.getter`, 接着调用 watcher 成员 `get()`, `get()` 首先将 自身 `pushTarget`， 先渲染内部的组件，然后调用 `this.getter` 更新视图

## 2、请简述 Vue 响应式原理。

---

`vue` 中通过为每个 `object` value 创建 `observer` 实例来管理响应式; `observe()` 函数 首先判断是否是对象, 符合响应式的对象, 就创建一个 `observer` 对象, 并返回 `ob`; `Obsever` 类中 定义了对数组 和 对象 的响应式处理过程, 它回为对象 挂在 `__ob__` 属性, 如果是数组, 则重新定义数组方法, 为数组方法添加逻辑, 如对新插入的元素进行响应式处理, 通过 数组 的  `ob` 对象发送通知, 处理完数组方法后, 对数组中每一元素进行`observe()`; 如果是对象, 通过 `walk` 将对象每个 `key` 转换成响应式数据, 由 `defineReactive` 来实现, 函数中, 为每个属性添加 `Dep` 实例, 判断自对象, 并 `observe` 自对象, 接着 为这个 `key` 设置 `get`, `set`, `get` 中, 存在 `Dep.target` 则将此 `watcher` 实例添加到 `dep` 实例, 其中如果 `childOb` 存在则, 建立子对象的依赖, 如果子对象是数组, 则特殊处理 数组 的依赖, 最后 `get` 中返回值; `set` 中, 如果是新值, 观察子对象并设置 `childOb`, 通过 `dep.notify()` 发布通知;


## 3、请简述虚拟 DOM 中 Key 的作用和好处。

---

添加 `key` 的好处是能通过比较 `key` 来判断节点是否能复用, 减少 dom 更新, 相同的 `key` 的内容可能相同, 比较后不需要更新 dom, 不设置时, 节点比较结果为不同, 即使内容相同也需要更新 dom

## 4、请简述 Vue 中模板编译的过程。

---

`compileToFunctions(template,...)` 入口, 先检查缓存, 加载编译好的 render 函数, 如果缓存没由命中, 调用 `compile(template, options)` 开始编译, 它内部首先合并 `options`, 然后调用 `baseCompile(template.trim(), finalOptions)` 编译模板, `baseCompile`中, `parse()` 把 `template` 转换成 `AST`, `optimize()` 标记 `AST` tree 中的静态 sub trees, 静态节点在重新渲染的时候不会重新生成, `pattch` 时将跳过 静态子树, 最后通过 `gernerate()` 将 `AST` tree 生成 js 的 `render` 函数字符串; 最后 `compileToFunctions()` 执行完, `createFunction` 将上一步返回的 字符串代码转换成 函数的形式, `render` 和 `staticRenderFns` 创建完毕, 挂载到 vue
实例的 options 对应的属性中;