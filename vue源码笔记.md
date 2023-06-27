# Vue源码笔记

阅读的版本是2.7.14

## src/core

### src/core/index.ts
在这个文件中主要执行了以下几步：

1. 从./instance/index中导入Vue实例，然后使用initGlobalAPI(Vue),来初始化全局的API
2. 给Vue的原型对象上的 **`$isServer`**  添加get方法，**`$ssrContext`** 添加get方法，Vue构造函数的属性 **FunctionalRenderContext** 添加值，添加版本信息
3. 导出vue

所以我们接下来详细看一下为了完成以上的操作所引入的方法及组件等

### Vue
#### instance/index.ts
  声明一个名字为Vue的构造函数，传入参数options，这里的Vue也就是我们在Vue中使用new Vue()时候所实例化的实例
    
  **__DEV__**是一个全局变量用于在开发环境和生产环境之间进行区分
    在开发环境中为true，在这个阶段，通常需要包含一些额外的调试信息、开发工具和错误提示，以帮助开发者进行调试和开发
    生产环境中为false，在这个阶段，不再需要包含开发时的额外信息和调试工具，主要关注性能和代码体积的优化

  **this instanceof Vue**该语句主要是为了判断类型，这里的this指向实例化后的Vue实例，判断this对象是否为一个Vue实例
    为true时表示this是Vue实例，返回false时则不是

  在Vue函数内部首先进行判断，如果在开发环境中并且当前this对象不为Vue实例是则通过warn方法报错
  我们看到的报错消息是这样的
  ```js
    console.error(`[Vue warn]: ${msg}${trace}`)
  ```
  msg就是'Vue is a constructor and should be called with the `new` keyword'，这句话的意思是使用 Vue 构造函数创建实例时，应该使用 new 关键字，trace是一个空字符串

	然后调用实例方法_init传入参数options，完成实例的初始化

	**_init**，_init方法声明在src/core/instance/init.ts中，通过initMixin()的调用，会在Vue的原型对象上添加一个_init的方法

		```js
		Vue.prototype._init = function (options?: Record<string, any>) {
			//方法接收options参数，然后去声明一个vm的值，将this赋值给vm(该函数中的this对象指向的是Vue实例)
    const vm: Component = this
		//因为在当前文件的全局定义了一个uid为0，后面_init方法每一次执行都会有不同的uid生成，这样vm内部的_uid就可以是唯一的
    vm._uid = uid++
    let startTag, endTag
		//config.performance用来开启或关闭 Vue 的性能警告。性能警告只在开发模式下生效，生产模式下会被忽略。因此，在生产环境中将 config.performance 设置为 true 不会产生性能警告。 
		//mark会通过inBrowser判断是否目前在浏览器环境中，且浏览器是否支持window.performance
		//所以如果是在开发环境中开启了性能优化并且浏览器支持window.performance的情况下执行if中的语句
    if (__DEV__ && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }
    vm._isVue = true
    vm.__v_skip = true
    vm._scope = new EffectScope(true /* detached */)
    vm._scope._vm = true
    // options中_isComponent用于判断当前的options是否是一个组件的
    // 是的话执行initInternalComponent方法，否则执行mergeOptions方法
    if (options && options._isComponent) {
      // **initInternalComponent**接收两个参数，第一个是当前的vue实例，第二个是初始化传入的options。
      // 首先通过 Object.create((vm.constructor as any).options) 创建了一个新对象 opts，并将其赋值给 vm.$options。这样，vm.$options 就继承了组件构造函数的选项，包括组件的生命周期钩子、指令、计算属性等。
      // 接下来给opts的parent、_parentVnode、propsData、_parentListeners、_renderChildren、_componentTag、render、staticRenderFns赋值
      // initInternalComponent的作用是初始化组件的内部选项，为组件实例的 $options 属性赋值，并从父虚拟节点的 componentOptions 中提取一些信息，以便后续的组件渲染和交互
      initInternalComponent(vm, options as any)
    } else {
      // **mergeOptions**要接收三个参数parent，children，vm，在_init调用的时候分别对应resolveConstructorOptions(vm.constructor as any)，options || {}，vm，vm我们前面已经介绍过了，第二个children参数也好理解，所以先看一下第一个参数主要就是resolveConstructorOptions()这个方法
      // *resolveConstructorOptions*
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor as any),
        options || {},
        vm
      )
    }
    if (__DEV__) {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate', undefined, false)
    initInjections(vm)
    initState(vm)
    initProvide(vm)
    callHook(vm, 'created')
    if (__DEV__ && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
		```




