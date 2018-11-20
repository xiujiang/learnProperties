# VUE

#### 实例生命周期钩子

`created` 当实例被创建时执行代码

```javascript
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
```

也有一些其它的钩子，在实例生命周期的不同阶段被调用，如 [`mounted`](https://cn.vuejs.org/v2/api/#mounted)、[`updated`](https://cn.vuejs.org/v2/api/#updated)和 [`destroyed`](https://cn.vuejs.org/v2/api/#destroyed)。生命周期钩子的 `this` 上下文指向调用它的 Vue 实例。

#### vue 模板语法

1.最常见的方式：双大括号绑定文本。{{x}}

​	<v-once> 一次性插值，数据之后不会变化

​	<v-html> 输出html代码。

指令 (Directives) 是带有 `v-` 前缀的特殊特性。指令特性的值预期是**单个 JavaScript 表达式** (`v-for` 是例外情况，稍后我们再讨论)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

​	<v-bind> 缩写：

​	<v-on> 缩写 @

#### 计算属性

​	computed：

#### 方法

​	Method:

**计算属性是基于它们的依赖进行缓存的**。只在相关依赖发生改变时它们才会重新求值。这就意味着只要内容还没有发生改变，多次访问 计算属性会立即返回之前的计算结果，而不必再次执行函数。

这也同样意味着下面的计算属性将不再更新，因为 `Date.now()` 不是响应式依赖：

```javascript
computed: {
  now: function () {
    return Date.now()
  }
}
```

相比之下，每当触发重新渲染时，调用方法将**总会**再次执行函数。

#### 侦听属性

​	watch:

































