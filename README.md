## 一、cdn引入基本使用

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

