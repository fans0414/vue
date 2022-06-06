## Vue源码分析
### Vue首次渲染过程
- Vue构造函数 调用 _init(options)
- _init(options) 调用 vm.$mount(vm.$options.el) (此处为加工后的$mount方法)
- vm.$mount(vm.$options.el) 中将模板转换成render函数
- vm.$mount(vm.$options.el) 调用 mount.call(this, el, hydrating) (原始$mount()方法)
- 原始$mount()方法 调用 mountComponent(this, el, hydrating)
- mountComponent()中定义updateComponent()
  - updateComponent() 中调用 vm._update(vm._render(), hydrating)
- mountComponent还创建watcher实例
  - new Watcher()传入updateComponent()
- new Watcher 创建时会立即调用一次get()
- get()会调用updateComponent函数
- 依次调用vm._render(),vm._update()完成首次渲染

### 状态更新后流程
- 触发watcher的回调函数，即updateComponent()
- updateComponent() 调用 vm._update()
- vm.\_update()首先会调用 vm._render() 创建vnode对象***
- vm._update()会将新旧vnode进行对比
- vm._update()会调用vm.\_\_patch\_\_() 进行对比新旧节点差异，并渲染差异到DOM树上
- vm.__patch__()即patch函数
  - patch函数通过createPatchFunction函数创建
- patch 中对比新旧节点差异
  - 情况一 新节点不存在，老节点存在 调用invokeDestoryHook() 执行destory钩子函数
  - 情况二 旧节点不存在，新节点存在
    - 创建组件但不挂载的视图的情况
    - 调用createElm()将新vnode转换成真实DOM元素，但不挂载
  - 情况三 新老节点都存在，且旧vnode是真实DOM元素
    - 说明是首次渲染
    - 调用emptyNodeAt() 创建DOM元素对应的vnode节点
    - 调用createElm()创建对应的DOM元素
    - 调用removeVnodes()移除老节点
  - 情况四 老节点存在，新节点存在  且新旧vnode相同
    - 调用patchVnode()比较差异
    - 调用removeVnodes()移除老节点
- 核心patchVnode()
  - 情况一 新vnode没有text属性，且新旧vnode子元素不相同
    - 调用#updateChildren() 对比差异并渲染
  - 情况二 新vnode没有text属性，但有子元素，且旧vnode没有子节点
    - 调用setTextContent(elm,"")清空旧节点对应DOM的文本
    - 调用addVnodes()将新节点的子节点插入到旧节点的对应的DOM元素上
  - 情况三 新vnode没有text属性且没有子节点，但旧vnode有子节点
    - 调用removeVnodes()删除旧节点中的子节点	
    - 移除对应的事件
    - 触发destory钩子函数
  - 情况四 旧节点有文本，新节点没有文本
    - 调用setTextContent(elm,"") 清空旧节点文本内容
  - 情况五 新旧vnode都有text属性且不相同
    - 使用新vnode的text填充旧vnode对应DOM元素文本内容
- 核心updateChildren() (Diff算法)
  - 两种校验
  - 四大比较
  - 额外比较
  - 收尾工作
