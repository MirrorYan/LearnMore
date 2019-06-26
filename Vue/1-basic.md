

# 基础

## 1. 模板语法

Vue.js 使用基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层 Vue 实例的数据。所有 Vue.js 的模板都是合法的 HTML，所以能被遵循规范的浏览器和 HTML 解析器解析。

底层实现：Vue 将模板编译成虚拟 DOM 渲染函数，结合响应系统，Vue 计算出最少要重新渲染多少组件，并把 DOM 操作次数减到最少。

### 1.1 插值

Vue 的模板插值有：文本插值和原始 HTML 插值两种方式。

#### 1.1.1 文本插值

Vue 的文本插值使用 Mustache 语法`{{ vlaue }}`。

```html
<span>Message: {{ msg }}</span>
```

Mustache 标签将会被替代为对应数据对象上`msg`属性的值。任何时候，只要绑定的数据对象上`msg`属性发生了改变，插值处的内容就会更新。

`v-once`指令：执行一次性插值，当数据改变时，插值处的内容不会更新。

```html
<!-- 这会影响到这个节点上的所有数据绑定 -->
<span v-once>{{ msg }}</span>
```

#### 1.1.2 原始 HTML 插值

双大括号会将数据解释为普通文本，当插值要输出 HTML 代码时，需要使用`v-html`指令。

```html
<!-- 输出普通文本 -->
<p>Using mustaches: {{ rawHtml }}</p>
<!-- 作为HTML代码渲染 -->
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

> <font color=red>Notice：</font>绝对不要对用户提供的内容使用 HTML 插值，因为站点上动态渲染的 HTML 很容易受到 XSS 攻击，所以最好只对可信内容使用 HTML 插值。

Mustache 语法不能作用在 HTML 特有属性上，这个时候可以使用`v-bind`指令：

```html
<!-- 将ID设为对应对象上的dynamicId属性值 -->
<div v-bind:id="dynamicId"></div>
```

对于布尔特性，`v-bind`指令绑定的特性值为`null`、`undefined`、`false`时，那么渲染的时候，不会渲染对应的 HTML 特性。

#### 1.1.3 JS 表达式

Vue.js 支持模板中的插值使用 JS 表达式（每个插值都只能包含单个表达式）。

```html
<!-- 它们在所属 Vue 实例的数据作用域下作为 JS 被解析 -->
{{ number + 1 }}
{{ ok ？ 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list' + id"></div>
```

> <font color=orange>Note：</font>模板表达式都被放在沙盒中，只能访问全局变量的一个白名单（如`Math`和`Date`），访问不到用户定义的全局变量。

### 1.2 指令

指令是带有`v-`前缀的特殊特性。指令特性的值预期是单个 JS 表达式（`v-for`例外）。

指令的职责：当表达式的值改变，将其产生的连带影响，响应式地作用于 DOM。

#### 1.2.1 参数

一些指令能接受一个参数，在指令名后以冒号(`:`)表示。

```html
<a v-bind:href="url">...</a>
```

#### 1.2.2 动态参数

v2.6.0 开始，可以用方括号括起来的 JS 表达式作为一个指令的参数：

```html
<a v-on:[eventName]="doSomething">...</a>
```

**对参数表达式的约束：**

1. 回避大写键名。

   ```html
   <!-- 在DOM中使用模板时这段代码会被转换为v-bind:[someattr] -->
   <a v-bind:[someAttr]="value"> ... </a>
   ```

2. 使用没有空格或引号的表达式。

   ```html
   <!-- 这会触发一个编译警告 -->
   <a v-bind:['foo' + bar]="value"> ... </a>
   ```

#### 1.2.3 修饰符

修饰符是以半角句号(`.`)指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。

```html
<!-- .prevent修饰符告诉v-on指令对于触发的事件调用event.preventDefault() -->
<form v-on:submit.prevent="onSubmit">...</form>
```

### 1.3 缩写

对于一些频繁用到的指令来说，使用`v-`前缀会比较繁琐。同时在构建 Vue 管理所有模板的 SPA 时，这个前缀没那么重要。因此 Vue 为`v-bind`和`v-on`这两个最常用的指令提供了简写。

#### 1.3.1 `v-bind`缩写

```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>
<!-- 缩写 -->
<a :href="url">...</a>
```

#### 1.3.2 `v-on`缩写

```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>
<!-- 缩写 -->
<a @click="doSomething">...</a>
```

---

## 2. 计算属性和侦听器

### 2.1 计算属性

虽然模板内嵌入 JS 表达式是被允许的，但模板中放入太多的复杂逻辑会让模板过重且难以维护。所以对于任何复杂逻辑，都应当使用计算属性。

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```javascript
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

#### 2.1.1 计算属性缓存 vs 方法

可以通过在表达式中调用方法，达到和上面一样的效果：

```html
<p>Reversed message: "{{ reversedMsg() }}"</p>
```

```javascript
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

虽然2种方式最终结果完全相同，但它们的缓存方式不同：

- 计算属性基于它们的响应式依赖进行缓存，只在相关响应式依赖发生改变时它们才会重新求值。如果相关响应式依赖未发生改变，但触发了重新渲染，不会再次执行计算属性的函数。
- 每当触发重新渲染，调用方法总会再次执行函数。

#### 2.1.2 计算属性 vs 侦听属性

Vue 提供了**侦听属性**来观察和响应 Vue 实例上的数据变动。

```html
<div id="demo">{{ fullName }}</div>
```

```javascript
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

这时候，代码是命令式且重复的。而使用计算属性，代码不会那么冗余：

```javascript
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

#### 2.1.3 计算属性的 setter

计算属性默认只有 getter，必要时也可以提供 setter：

```javascript
computed: {
  fullName: {
    get: function() {
      return `${this.firstName} ${this.lastName}`
    },
    set: function(newValue) {
      let names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// 在控制台运行 vm.fullName = 'John Doe' 时，setter 被调用
// vm.firstName 和 vm.lastName 会相应地更新
```

### 2.2 侦听器

当需要数据变化时执行异步或开销较大的操作时，使用侦听器是最有用的。

```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
<!-- AJAX 库和通用工具的生态已够丰富，Vue 核心代码没有重复提供这些功能 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: { // 使用侦听器限制访问频率
    question: function(newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing ...'
      this.debouncedGetAnswer()
    }
  },
  created: function() {
    // _.debounce 是一个通过Lodash限制操作频率的函数
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function() {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking ...'
      var vm = this
      anxios.get('https://yesno.wtf/api')
      	.then(function() {
          vm.answer = _.capitalize(response.data.answer)
      	}).catch(function(error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>
```

---

## 3. Class 与 Style 绑定

操作元素的 class 列表和内联样式是数据绑定的一个常见需求。因为它们都是属性，所以我们可以用 `v-bind` 处理它们：只需要通过表达式计算出字符串结果即可。不过，字符串拼接麻烦且易错。因此，在将 `v-bind` 用于 `class` 和 `style` 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。

### 3.1 绑定 HTML Class

#### 3.1.1 对象语法

通过传给`v-bind:class`一个对象来动态地切换 class：

```html
<div v-bind:class="{ active: isActive }"></div>
<!-- active这个class存在与否取决于数据属性isActive的truthiness -->
```

`v-bind:class`也可以与普通的 class 属性共存。

```html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
<!-- 当hasError=flase, isActice=true 这个div的class="static active" -->
<!-- 当hasError=true, isActice=false 这个div的class="static text-danger" -->
```

绑定的数据对象不必内联定义在模板里。

```html
<div v-bind:class="classObj"></div>
```

```javascript
data: {
  classObj: {
    active: true,
    'text-danger': false
  }
},
computed: {
  classObject: function() {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

#### 3.1.2 数组语法

通过把一个数组传给`v-bind:class`来应用一个 class 列表：

```html
<div v-bin:class="[activeClass, errorClass]"></div>
```

```javascript
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

通过使用三元表达式根据条件切换列表中的 class：

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

在数组语法中也可以使用对象：

```html
<div v-bind:class="[{active: isActive}, errorClass]"></div>
```

#### ❤3.1.3 用在组件上



### 3.2 绑定内联样式

#### 3.2.1 对象语法

`v-bind:style`的对象语法看着像 CSS，但其实是一个 JS 对象。CSS 属性名可以用**驼峰式**或**短横线分隔**来命名。

```html
<div v-bind:style="{ color: activeColor, font-size: fontSize + 'px' }"></div>
```

```javascript
data: {
  activeColor: 'red',
  fontSize: 30
}
```

将样式直接绑定到一个样式对象，会让模板更清晰：

```html
<div v-bind:style="styleObj"></div>
```

```javascript
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

对象语法常常结合返回对象的计算属性使用。

#### 3.2.2 数组语法

`v-bind:style`的数组语法可以将多个样式对象应用到同一个元素上：

```html
<div v-bind:stlye="[baseStyles, overridingStyles]"></div>
```

#### 3.2.3 自动添加前缀

当`v-bind:style`使用需要添加浏览器引擎前缀的 CSS 属性时，Vue 会自动侦测并添加响应的前缀。

#### 3.2.4 多重值

v2.3.0+支持为`style`绑定中的属性提供一个包含多个值的数组，常用于提供多个带前缀的值。

```html
<div :style="{ display:['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

这样只会渲染数组中最后一个被浏览器支持的值。

---

## 4. 条件渲染

### 4.1 v-if

`v-if`指令用于条件性地渲染一块内容，这块内容只会在指令的表达式返回 truthy 值的时候被渲染。

```html
<h1 v-if="awesome">Vue is awesome</h1>
```

#### 在 template 元素上使用 v-if 条件渲染分组

因为`v-if`是一个指令，所以必须将它添加到一个元素上。但如果想切换多个元素，可以把一个`<template>`元素当做不可见的包裹元素，并在杀昂面使用`v-if`。最终的渲染结果不会包含`<template>`元素。

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

### 4.2 v-else

可以用`v-else`指令来表示`v-if`的”else 块“：

```html
<div v-if="Math.random() > 0.5">Now you see me</div>
<div v-else>Now you don't<div>
```

`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。

### 4.3 v-else-if

> v 2.1.0 新增

`v-else-if`充当`v-if`的”else if“块，可以连用：

```html
<div v-if="type === 'A'"> A </div>
<div v-else-if="type === 'B'"> B </div>
<div v-else-if="type === 'C'"> C </div>
<div v-else> Not A/B/C </div>
```

它也必须紧跟在带`v-if`或`v-else-if`的元素之后。

#### 用 key 管理可复用的元素

用`key`管理可复用元素的好处是：

- Vue 会更高效地渲染元素，让 Vue 变得非常快。

- 条件切换中，只切换不同的地方。

  ```html
  <template v-if="loginType === 'username'">
    <label>Username</label>
    <input placeholder="Enter your username">
  </template>
  <template v-else>
    <label>Email</label>
    <input placeholder="Enter your email address">
  </template>
  ```

  在上面的代码中切换`loginType`不会清楚用户已经输入的内容，因为两个模板使用了相同的元素怒，`<input>`不会被替换掉，仅仅是替换的它的`placeholder`。

  `<label>`标签仍会被高效复用，因为它们没有添加`key`属性

### 4.4 v-show

另一个用于根据条件展示元素的选项是`v-show`指令。

```html
<h1 v-show="ok">Hello~</h1>
```

带有`v-show`的元素始终会被渲染并保留在 DOM 中，只是显示和隐藏的问题。`v-show`只是简单地切换元素 CSS 的`display`属性值。

> <font color=red>Notive：</font>`v-show`不支持`template`元素，也不支持`v-else`。

### 4.5 v-if vs v-show

- `v-if`：

  真正的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当的被撤销和重建；

  它是**惰性的**，如果在初始渲染时条件为假，则什么也不做，直至条件第一次为真时才会开始渲染条件块。

  正因为它会对 DOM 树进行插入和移除的操作，因此它的开销较大。

- `v-show`：

  不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

  由于元素总是会被渲染，因此它的初始渲染开销较大。

---

## 5. 列表渲染

### 5.1 用 v-for 把一个数组对应为一组元素

可以用`v-for`指令基于一个数组来渲染一个列表。`v-for`指令需要使用`item in items`形式的特殊语法，其中`items`是源数据数组，`item`是被遍历的数组元素的**别名**。

```html
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

```javascript
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

 在 `v-for` 块中，可以访问所有父作用域的属性。`v-for` 还支持一个可选的第二个参数，即当前项的索引。

```html
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```

```javascript
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

也可以用`of`替代`in`作为分隔符，因为它更接近 JS 迭代器的语法。

### 5.2 在 v-for 里使用对象

可以用`v-for`来遍历一个对象的属性。

```html
<ul id="v-for-object" class="demo">
  <li v-for="vlaue in object">
    {{ value }}
  </li>
</ul>
```

```javascript
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
```

还可以用第三个参数作为索引：

```html
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }} : {{ value }}
</div>
```

> <font color=orange>Notice：</font>在遍历对象时，会按`Object.keys()`的结果遍历，但是不能保证它的结果在不同的 JS 引擎下都一致。

### 5.3 维护状态

当 Vue 正在更新使用`v-for`渲染的元素列表时，它默认使用”就地更新“的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并确保他们在每个索引位置正确渲染。

这个默认的模式是高效的，但是==只适用于不依赖子组件状态或临时 DOM 状态== (例如：表单输入值) ==的列表渲染输出==。

为了给 Vue 一个提示，让它能跟踪每个节点的身份，从而重用和重新排序现有元素，需要为每项提供一个唯一 `key` 属性：

```html
<div v-for="item in items" v-bind:key="item.id">
  <!-- content -->
</div>
```

尽可能在使用`v-for`时，提供`key`属性，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为来获取性能上的提升。它是 Vue 识别节点的一个通用机制，`key`并不是只与`v-for`特别关联。

### 5.4 数组更新检测

#### 5.4.1 变异方法

Vue 将被侦听的数组的**变异方法**进行了包裹，所以它们也将会触发视图更新。被包裹过的方法包括：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

#### 5.4.2 替换数组

变异方法会改变调用了这些方法的原始数组。相比之下，也有非变异方法，它们不会改变原始数组，而==总是返回一个新数组==（如 `filter()`、`concat()` 和 `slice()` ）。当使用非变异方法时，可以用新数组替换旧数组：

```javascript
example.items = example1.items.filter(function(item) {
  return item.message.match(/Foo/)
})
```

> <font color=red>Announcements：</font>
>
> 由于 JS 的限制，Vue 不能检测以下数组的变动：
>
> 1. 利用索引直接设置一个数组项时。
>
>    解决这个问题有两种方式：
>
>    - 通过`Vue.set()`方法，或者`Array.$set()`实例方法，它是全局方法`Vue.set`的一个别名。
>    - 通过`Array.prototype.splice()`方法。
>
> 2. 修改数组的长度时。
>
>    解决这个方法可以使用`splice()`。

### 5.5 对象变更检测注意事项

由于 JS 的限制，Vue 不能检测对象属性的添加或删除，对于已经创建的实例，Vue 不允许动态添加根级别的响应式属性，但可以用`Vue.set(object, propertyName, value)`或`vm.$set`方法向嵌套对象添加响应式属性。

有时可能需要为已有对象赋值多个新属性，为了能响应式， 最好这样：

```javascript
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

### 5.6 显示过滤 / 排序后的结果

要显示一个数组经过过滤或排序后的版本，但不改变或重置原始数据。这种情况下可以创建一个计算属性来返回过滤或排序后的数组。当计算属性不适用时（如在`v-for`循环中）可以使用一个方法：

```html
<li v-for="n in even(numbers)">{{ n }}</li>
```

```javascript
data: {
  numbers: [1, 2, 3, 4, 5]
},
methods: {
  even: function(numbers) {
    reutrn this.numbers.filter(function(number) {
      return number % 2 === 0
    })
  }
}
```

### 5.7 在 v-for 里使用值范围

`v-for`也可以接受整数，这时候它会吧模板重复渲染对应次数。

```html
<div>
  <span v-for="n in 3">{{ n }}</span>
  <!-- 相当于 -->  
  <span>1</span>
  <span>2</span>
  <span>3</span>
</div>
```

### 5.8 在 template 上使用 v-for

通过在 `<template>`上使用`v-for`来渲染一段包含多个元素的内容。

```html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

### 5.9 v-for 与 v-if 一同使用

> <font color=orange>Notice：</font>不推荐在同一元素上使用`v-if`和`v-for`

当它们处于同一节点，`v-for`的优先级比`v-if`更高，`v-if`将跟别重复运行于每个`v-for`循环中。

这种优先级的机制，在只想为部分项渲染节点的情况下很有用：

```html
<li v-for="todo in todos" v-if="!todo.isComplete">{{ todo }}</li>
```

如果目的是有条件地跳过循环的执行，可以将`v-if`置于外层元素（或`<template>`上）。

```html
<ul v-if="todos.length">
  <li v-for="todo in todos">{{ todo }}</li>
</ul>
<p v-else>No todos left!</p>
```

### ❤5.10 在组件上使用 v-for



---

## 6.事件处理

### 6.1 事件监听

**监听事件**

用`v-on`指令监听 DOM 事件，并在触发时运行一些 JS 代码。

```html
<button v-on:click="counter++">Add 1</button>
<p>The button above has been clicked {{ counter }} times.</p>
```

```javascript
data: {
  counter: 0
}
```

**事件处理方法**

`v-on`还可以接收一个需要调用的方法名，来处理复杂的处理逻辑，在`methods`对象中定义方法。

```html
<button v-on:click="eventName"></button>
```

**内联处理器中的方法**

`v-on`还可以在内联 JS 语句中调用方法。

```html
<button v-on:click="say('hi')">Say hi</button>
```

可以用特殊变量`$event`把原始的 DOM 事件传入方法：

```html
<button v-on:click="warn('From cannot be submitted yet.', $event)">
  Submit
</button>
```

```javascript
methods: {
  warn: function(message, event) {
    if (event) event.preventDefault() // 可以访问原生事件对象
    alert(message)
  }
}
```

### 6.2 事件修饰符

事件处理程序中调用`event.preventDefault()`或`event.stopPropagation()`是很常见的需求，但最好是，方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。

为此，Vue 为`v-on`提供了事件修饰符。

- `.stop`：阻止事件继续传播（冒泡）。

  ```html
  <a v-on:click.stop="doThis"></a>
  ```

- `prevent`：防止执行预设的行为（如果事件可取消，则取消该事件，而不停止事件的进一步传播）。

  ```html
  <a v-on:click.prevent="onSubmit"></a>
  ```

- `.capture`：事件捕获由外到内（与事件冒泡方向相反）。

- `.self`：只触发自己范围内的事件（不包含子元素）。

- `.once`：只触发一次。

  其他修饰符只能对原生的 DOM 事件起作用的修饰符，`.once`修饰符还能被用到自定义的组件事件上。

- `.passive`：执行默认方法。

  > 因为每次产生事件，浏览器都会去查询一下是否有 `preventDefault`函数阻止此次事件的默认动作。所以浏览器本身是没有办法对这种场景进行优化的，这种场景下，用户的手势事件无法快速产生，会导致页面无法快速执行滑动逻辑，从而让用户感到页面卡顿。
  >
  > 加上`.passive`就是为了告诉浏览器不用去查询，直接执行默认动作。
  >
  > 这个修饰符尤其能够提升移动端的性能。
  >
  > <font color=red>Notice：</font>`.passive`和`.prevent`冲突，不能同时绑定在一个监听器上。

### 6.3 按键修饰符

Vue 允许为`v-on`在监听键盘事件时添加按键修饰符：

```html
<!-- 只有在key是Enter时调用 -->
<input v-on:keyup.enter="submit">
```

可以直接将`KeyboardEvent.key`暴露的任意有效按键名转换为其他名字来作为修饰符。

```html
<input v-on:keyup.page-down="onPageDown">
<!-- 处理函数只会在$event.key=PageDown时被调用 -->
```

#### 6.3.1 按键码

> `keyCode`的事件用法已被废弃。

使用`keyCode`特性而是允许的：

```html
<input v-on:keyup.13="submit">
```

为了在必要的情况下支持旧的浏览器，Vue 提供了常用的按键码的别名：

- `.enter`
- `.tab`
- `.delete`
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

> 有些按键（`.esc`以及所有的方向键）在 IE9 中`key`值不一样，这些内置的别名能够适配不同的浏览器。

还可以通过全局`config.keyCodes`对象自定义按键修饰符别名：

```javascript
Vue.config.keyCodes.f1 = 112
```

### 6.4 系统修饰键

#### 6.4.1 系统键盘按键修饰键

可以用以下修饰符来实现仅在按下响应按键时才触发鼠标或键盘事件的监听器。

- `.ctrl`

- `.alt`

- `.shift`

- `.meta`

  > <font color=orange>Notice：</font>在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。

 ```html
<!-- Ctrl + click 才能触发doSomething -->
<div @click.ctrl="doSomething">Do something</div>
 ```

修饰键与常规按键不同，在和`keyup`事件一起用时，事件触发时修饰键必须处于按下状态。

#### 6.4.2 .exact 修饰符

`.exact`修饰符允许控制由精确的系统修饰符组合触发的事件。

```html
<!-- 即使Alt或Shift被一起按下时也会触发 -->
<button @click.ctrl="onClick">A</button>
<!-- 当且仅当Ctrl被按下的时候才触发 -->
<button @click.ctrl.exact="onClick">A</button>
<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

#### 6.4.3 鼠标按钮修饰符

- `.left`
- `.right`
- `.middle`

这些修饰符会限制处理函数只响应特定的鼠标按钮。

### 6.4.4 为什么在 HTML 中监听事件

这种事件监听的方式违背了关注点分离。但因为所有的 Vue 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。使用`v-on`的好处：

1. 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法。
2. 因为无须在 JavaScript 里手动绑定事件，ViewModel 代码可以是纯粹的逻辑，和 DOM 完全解耦，更易于测试。
3. 当一个 ViewModel 被销毁时，所有事件处理器都会被自动删除，无须担心如何清理它们。

---

## 7. 表单输入绑定

### 7.1 基础用法

可以用`v-model`指令在表单`<input>`、`<textarea>`、`<select>`元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。

`v-model`本质是语法糖。它监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

> <font color=red>Notice：</font>`v-model`会忽略所有表单元素的`value`、`checked`、`selected`特性的初始值，总是将 Vue 实例的数据作为数据来源。所以应该通过 JS 在组件的`data`选项中声明初始值。

`v-model`在内部为不同的输入元素使用不同的属性并抛出不同的事件：

- text 和 textarea 使用`vlaue`属性和`input`事件；
- checkbox 和 radio 使用`checked`属性和`change`事件；
- select 字段将 `value`作为 prop 并将`change`作为事件。

> 对于需要使用输入法的语言，`v-model`不会在输入法组合文字过程中更新。如果想处理这个过程，可以使用`input`事件。

**文本**

```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

**多行文本**

```html
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placehoder="add multiple lines"></textarea>
```

**复选框**

单个复选框，绑定到布尔值：

```html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```

多个复选框绑定到同一个数组：

```html
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
```

```javascript
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

**单选按钮**

```html
<div id="example-4">
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>
```

```javascript
new Vue({
  el: '#example-4',
  data: {
    picked: ''
  }
})
```

**选择框**

- 单选时：

  ```html
  <div id="example">
    <select v-model="selected">
      <!-- 空值禁用选项 -->
      <option disabled value="">请选择</option>
      <option>A</option>
      <option>B</option>
      <option>C</option>
    </select>
    <span>Selected: {{ selected }}</span>
  </div>
  ```

  ```javascript
  data: {
    selected: ''
  }
  ```

  > 如果`v-model`表达式的初始值未能匹配任何选项，`selected`元素将被渲染为未选中状态。在 IOS 中，这会导致用户无法选择第一个选项，因为这样的情况下 IOS 不会触发 change 事件。因此更推荐提供一个空值的禁用选项。
  
- 多选时：

  ```html
  <select v-model="selected" multiple style="width: 50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
  ```

  ```javascript
  data: {
    selected: []
  }
  ```

- 用`v-for`渲染的动态选项：

  ```html
  <select v-model="selected">
    <option v-for="option in options" v-bind:value="option.value">
    	{{ option.text }}
    </option>
  </select>
  ```

### 7.2 值绑定

对于单选按钮、复选框、选择框的选项，`v-model`绑定的值通常是静态字符串（对于复选框也可以是布尔值）。

用`v-bind`可以实现将值绑定到 Vue 实例的一个动态属性上，并且这个属性的值可以不是字符串。

**复选框**

```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
<!--
  选中时 toggle="yes"
  未选中 toggle="no"
-->
```

> <font color=orange>Notice：</font>这里的`true-value`和`false-value`特性不会影响输入控件的`value`特性，因为浏览器在提交表单时不会包含未被选中的复选框。如果要确保表单中这两个值中的一个能被提交，最好换用单选按钮。

**单选按钮**

```html
<input type="radio" v-model="pick" v-bind:value="a">
<!-- 当选中时 pick === a -->
```

**选择框的选项**

```html
<selected v-model="selected">
  <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</selected>
<!-- 选中时：
  selected值的类型 => object
  selected.number => 123
-->
```

### 7.3 修饰符

`.lazy`

默认情况下，`v-model`在每次`input`事件触发后将输入框的值与数据进行同步（除使用输入法组合文字时）。添加`lazy`修饰符，使用`change`事件进行同步。

```html
<!-- 在change时而不是input更新时 -->
<input v-model.lazy="msg">
```

`.number`

自动将用户的输入值转为数值类型。如果这个值无法被`parseFloat()`解析，则会返回原始值。

`.trim`

自动过滤用户输入的首尾空白字符。

### ❤7.4 在组件上使用`v-model`



---

## 8. 组件基础

### 8.1 基本语法

组件是可复用的 Vue 实例，且带有一个名字。为了能在模板中使用，这些组件必须先注册，以便 Vue 能够识别。

```javascript
Vue.component('component-name', {
  // ...
  template: 'HTML Code ...'
})
```

可以在一个通过`new Vue`创建的 Vue 根实例中，把组件作为自定义元素来使用：

```html
<div id="components-demo">
  <component-name />
</div>
```

因为组件是可复用的 Vue 实例，所以它们与`new Vue`接收相同的选项（如：`data`、`computed`、`watch`、`methods`以及生命周期钩子等）。仅有的例外是像`el`这样的根实例特有的选项。

### 8.2 组件的复用

组件可以复用无数次，每用一次组件，就会有一个它的新实例被创建。

#### 8.2.1 data 必须是一个函数

一个组件的`data`选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立拷贝：

```javascript
data: function () {
  return {
    props: values
  }
}
```

如果 Vue 没有这条规则，被复用的相同组件之间的数据就会互相影响。

### 8.3 组件的组织

通常一个应用会以一颗嵌套的组件树的形式来组织：

![组件树](img/components-tree.png)

组件的注册有2中类型：

- **全局注册**：

  ```javascript
  Vue.component('component-name', {
    // options ...
  })
  ```

  全局注册的组件可以用在其被注册之后的任何（通过`new Vue`）新建的 Vue 根实例，包括其组件树中的所有子组件的模板中。

- **局部注册**

### 8.4 通过 prop 向子组件传递数据

Prop 是在组件上注册的一些自定义特性。当一个值传递给一个 prop 特性的时候，它就变成了那个组件实例的一个属性。

```javascript
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})
```

一个组件默认可以拥有任意数量的 prop，任何值都可以传递给 prop。

prop 被注册之后，就可以把数据作为一个自定义特性传递进来：

```html
<blog-post title="My journey with Vue" />
<blog-post title="Blogging with Vue" />
<blog-post title="Why Vue is so fun" />
```

可以使用`v-bind`来动态传递 prop，就像模板中 HTML 元素自身的特性一样。

### 8.5 单个根元素

每个组件只能有一个根元素，如果组件中有多个要渲染的元素，可以将模板的内容包裹在一个父元素内。

如果有太多的值需要传递，可以只设置一个 prop，将要传递的值作为一个对象传给 prop，子组件中通过这个 prop 的属性来获取值。这样就可以不用在模板调用组件时，写太多 prop 相关的代码。

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  v-bind:content="post.content"
  v-bind:publishedAt="post.publishedAt"
  v-bind:comments="post.comments"
/>
<!-- 所有的值放在一个object中传递 -->
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
/>
```

### 8.6 监听子组件事件

子组件可以通过调用内置的`$emit()`方法并传入事件名称来触发一个事件。

```html
<blog-post
  v-on:add-num="number += 1"
/>
<!-- 这里的事件名称就是add-num -->
```

```javascript
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button v-on:click="$emit('add-num')">Add Number</button>
    </div>
  `
})
```

**使用事件抛出一个值**

通过将值作为`$emit()`的第二个参数传递来为事件抛出一个特定的值。

```html
<button v-on:click="$emit('enlarge-text', 0.1)">Enlarge text</button>
```

然后当在父级组件监听这个事件时，通过 `$event`访问到被抛出的这个值：

```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
/>
```

如果这个事件处理函数是一个方法：

```html
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
/>
```

那么这个值将会作为第一个参数传入这个方法：

```javascript
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount 
  }
}
```

**在组件上使用 v-model**

> <font color=pink>Remember：</font>
>
> ```html
> <input v-model="searchText">
> <!-- 等价于 -->
> <input
>   v-bind:value="searchText"
>   v-on:input="searchText = $event.target.value"
> >
> ```

自定义事件也可以用于创建支持`v-model`的自定义输入组件。

```html
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
/>
```

为了让它正常工作，这个组件内的`input`必须：

- 将其`value`特性绑定到一个名叫`value`的 prop 上。
- 在其`input`事件被触发时，将新的值通过自定义的`input`事件抛出。

```javascript
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```

```html
<custom-input v-model="searchText" />
```

### 8.7 通过插槽分发内容

Vue 有自定义的`<slot>`元素来向插槽添加内容：

```javascript
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```

```html
<alert-box>Something bad happened.</alert-box>
```

这样，组件中的文本就会被很自然地插入到了`slot`中，渲染的结果是不带`<slot>`元素的。

### 8.8 动态组件

通过 Vue 的`<component>`元素加`is`特性，来实现不同组件之间的动态切换。

```html
<component v-bind:is="currentTabComponent"></component>
```

`currentTabComponent`可以是：

- 一个已注册组件的名字。
- 一个组件的选项对象。

### 8.9 解析 DOM 模板时的注意事项

有些 HTML 元素，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，只能出现在其他某些特定的元素内部。

这会导致使用有约束条件的元素时遇到一些问题。

```html
<table>
  <!-- 这个自定义组件会被作为无效的内容提升到外部，导致最终渲染结果出错 -->
  <blog-post-row></blog-post-row>
</table>
```

`is`特性可以提供一个变通的办法：

```html
<table>
  <tr is="blog-post-row"></tr>
</table>
```

> <font color="orange">Notice：</font>如果从一下来源使用模板，这条限制是不存在的：
>
> - 字符串（如：`template:'...'`）；
> - 单文件组件（`.vue`）
> - `<script type="text/x-template">`