## 一、[cdn引入基本使用](./vue3_cdn引入.html)

1. template的写法
2. data属性必须是函数
3. 计算属性的使用
4. 其他属性（props、watch、emits、setup、生命周期等）

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vue3_CND引入</title>
</head>

<body>
  <div id="app">呵呵哈哈哈</div>

  <!-- 
    方式1：
    <script type="x-template" id="template">
      <h2>{{num}}</h2>
      <h2>{{computeNum}}</h2>
      <button @click="add">+</button>
      <button @click="sub">-</button>
    </script>
 -->

  <!-- 方式2： （https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/template）-->
  <template id="template">
    <h2>{{num}}</h2>
    <h2>{{computeNum}}</h2>
    <button @click="add">+</button>
    <button @click="sub">-</button>
  </template>

  <script src="https://unpkg.com/vue@next"></script>

  <script>
    const data = {
      template: '#template',
      data() {
        return {
          num: 0
        }
      },
      computed: {
        computeNum() { return this.num + 1 }
      },
      methods: {
        add() {
          this.num++
        },
        sub() {
          this.num--
        }
      }
    }
    const app = Vue.createApp(data)
    app.mount('#app')

  </script>
</body>

</html>
```

## 二、vue3源码阅读方法

1. 首先找到[vue3源码仓库](https://github.com/vuejs/vue-next)，拉取代码到本地
2. package.json文件夹中的脚本添加sourcemap："dev": "node scripts/dev.js --sourcemap"，执行yarn dev（打包dist）
3. 在packages/vue/examples下随便写一个HTML文件，并在其中使用vue，就可以观察具体vue内部运行情况。

```html
<!DOCTYPE html>
<html lang="en">
<body>
  <div id="app"></div>
  <script src="../dist/vue.global.js"></script>
  <script>
    debugger
    Vue.createApp({ template: `<div></div>` }).mount('#app')
  </script>
</body>
</html>
```

## 三、methods属性中的方法this绑定源码

vue-next源码`packages/runtime-core/src/componentOptions.ts`中的600行左右

```js
  if (methods) {
    for (const key in methods) {
      const methodHandler = (methods as MethodOptions)[key]
      if (isFunction(methodHandler)) {
        // In dev mode, we use the `createRenderContext` function to define
        // methods to the proxy target, and those are read-only but
        // reconfigurable, so it needs to be redefined here
        if (__DEV__) {
          Object.defineProperty(ctx, key, {
            value: methodHandler.bind(publicThis),
            configurable: true,
            enumerable: true,
            writable: true
          })
        } else {
          ctx[key] = methodHandler.bind(publicThis)
        }
        if (__DEV__) {
          checkDuplicateProperties!(OptionTypes.METHODS, key)
        }
      } else if (__DEV__) {
        warn(
          `Method "${key}" has type "${typeof methodHandler}" in the component definition. ` +
            `Did you reference the function correctly?`
        )
      }
    }
  }
```

最重要的一步就是`ctx[key] = methodHandler.bind(publicThis)`，将每一个methods属性中的方法的this都绑定publicThis并存到ctx对象中（在模板引擎中需要从ctx里面取相应的方法来调用）。我们可以在源码中找到`const publicThis = instance.proxy! as any`，所以是绑定的this是组件实例上的data代理对象。

## 四、vscode快捷添加代码片段

首先找到vscode的user Snippets选项：

![请添加图片描述](https://img-blog.csdnimg.cn/b3fbe08438a64e658ee12cf14b1e7fb3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6auY5qGl6Z2T5LuU,size_20,color_FFFFFF,t_70,g_se,x_16)

可以在[snippet-generator](https://snippet-generator.app)中自动生成设置代码：

![snippet-generator](https://img-blog.csdnimg.cn/02c45473c16344f6af200d4d4fe61b49.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6auY5qGl6Z2T5LuU,size_20,color_FFFFFF,t_70,g_se,x_16)

设置完之后就可以，全局使用快捷指令：

![快捷添加代码片段](https://img-blog.csdnimg.cn/d1343fe27b6e45dc88bbf87926dc7be1.gif)

## 五、模板语法

- React的开发模式：
  - React使用的jsx，类似于js的一种语法，将html标签在js中书写
  - 通过**babel**将jsx语法**编译**成**React.createElement**函数调用
  - 而React纯JS写法太过灵活，使他在编译时优化方面先天不足。所以，React的优化主要在运行时
- Vue也支持jsx开发模式：
  - 但是大多数情况下，使用基于HTML的模板语法（@click、v-bind、v-once等）；
  - 在模板中，允许以**声明式**的方式将**DOM**和**底层组件实例的数据（data）**绑定在一起；
  - 在底层实现中，Vue将模板编译成虚拟DOM渲染函数。
  - Vue使用模版语法，可以在编译时对确定的模版作出优化。

### 1】v-once

用于指定元素或者组件只渲染一次：

当数据发生变化时，**元素或者组件以及其所有的子元素（子组件）**将**视为静态内容**并且跳过（可以于性能优化）

### 2】v-html

默认情况下，如果我们展示的**字符串内容**本身是html格式的，那么vue并不会对其进行特殊的解析。如果我们希望这个字符串内容的html被vue解析出来就可以使用v-html

```vue
<template id='app'>
	<div v-html="info"></div>
</template>
<script>
const App = {
  template:'#app',
  data(){
    return {
      info:`<div style='color:red'>呵呵哈哈哈</div>`
    }
  }
}
</script>
```

### 3】v-pre

跳过元素和它的子元素的编译过程（跳过不需要滨海的节点，加快编译的速度）

不希望解析插值表达式`{{}}`语法，想要直接以字符串的形式渲染在页面上

### 4】v-cloak

这个指令保持在元素上，**直到关联组件实例结束编译**

比如：和css规则如`[v-cloak]{display:none}`一起使用时，这个指令可以隐藏未编译的v-cloak标记的组件，直到该组件实例编译结束准备完毕