---
title: 浅析Vue响应式原理
tags:
  - Vue
translate_title: analysis-of-vue-response-principle
date: 2019-05-15 16:36:52
---

Vue的双向绑定基本原理是Obeject.defineProperty()劫持对象属性来实现修改数据改变视图，修改视图改变数据。那么具体是如何在数据和视图建立起关联的呢？


通过查看Vue源码我们了解到Vue内部实现了`Observer` `Watcher` `Compile` `Dep` 四个类，我们理一下逻辑这四个类是如何关联工作的呢？
我们先简单介绍一下这四个类是干什么的。
1、`Observer`：Observer通过Obeject.defineProperty劫持data的每个属性，这样我们就能监听到对象属性的`setter`和`getter`
2、`Compile`：Compile是用来解析模板中的插值和指令
3、`Watcher`：Watcher是用来当数据发生变化时执行更新模板的方法
4、`Dep`：依赖收集器，主要是用来收集Watcher的，当数据变化时会通知收集到的Watcher进行更新

了解了上面四个类的简单作用之后，我们来梳理一下它们是如何工作的。
下面是一个简单的例子
``` html
    <div id="app">
        {{text}}
        <input v-model="text">
    </div>

    <script>
        const vm = new Vue({
            el:'#app',
            data:{
                text:1
            }
        });
        vm.text = 2;
    </script>
```
1、new Vue的时候内部会进行一些初始化，其中最重要的就是Observer通过Obeject.defineProperty劫持data对象，这个时候就能监听到对象属性的变化了
2、Compile解析模板也就是`#app`里面的内容，解析到插值{{text}}，此时将其替换为实际值，同时实例化一个`Watcher`来保存更新操作
3、上一步我们可以看到`Watcher`被实例化了，那么实例化的时候做了什么操作呢。
    1.存储`Compile`中实例化时传递过来的更新方法。并且声明一个`update`方法来调用更新。
    2.`Dep.target`设置为`this`也就是告诉依赖收集器当前要收集的Watcher对象。
    3.然后访问一次`data.text`触发`getter`，在`getter`中判断`Dep.target`如果存在的话则将其添加到Dep的subs中
    4.当`data.text`被修改的时候则会触发`setter`，Dep就会遍历subs调用`Watcher`的update方法进行视图更新

总结，我们可以理解为每个属性都有一个发布者`Dep`用来收集订阅者也就是`Watcher`，`Compile`在进行模板解析的时候如果匹配到相应的插值或者指令则会创建一个`Watcher`并且将视图更新方法通过回调函数的方式存入`Watcher`，而`Watcher`则可以通过`update`方法来调用这个回调函数。紧接着`Watcher`会将自己也就是`this`存入全局的`Dep.target`中并且访问一次数据对象的属性来触发`getter`，这个时候Dep就会把`Watcher`存起来，也就是对象属性拥有了一个订阅者`Watcher`。当发生赋值操作的时候会触发对象属性的`setter`来调用这个对象属性拥有的发布者也就是`Dep`，通过遍历`Dep`的`subs`也就是订阅者列表来调用每一个`Watcher`的`update`方法，达到修改数据更新视图的目的。
那么通过修改视图达到改变数据是如何实现的呢，其实简单来说也是`Compile`解析模板发现`v-model`，然后监听该节点的input事件，在事件触发时修改对象属性，这样就会触发对象属性的`setter`，如果模板上有在其他地方绑定这个属性那么就又回到我们上面的说的，解析模板给属性的`Dep`添加`Watcher`，然后当更新属性会触发`Watcher`的update这么一个流程中了。

