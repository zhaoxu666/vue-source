数据驱动

# Vue 与模板

使用步骤
1. 编写 页面 模板
    - 直接在 HTML 标签中写 标签
    - 使用template
    - 使用单文件 <template />
2. 创建 Vue 的实例
    - 在 Vue 的构造函数中提供：data, method,computed,watch,props,...
3. 将 Vue 挂载到页面中 （ mount ）

# 数据驱动模型

Vue 执行流程
 +  获得模板：模板中有 '坑'
 +  利用 Vue 构造函数中所以工的数据来 '填坑'，得到可以在页面中显示的 '标签了'
 +  将标签替换页面中原来有坑的标签

Vue 利用 我们提供的数据 和页面中 模板 生成了一个新的 HMTL 标签 （node元素），
替换到了 页面中 放置模板的位置

怎么实现？？？

# 简单的模板渲染




# 虚拟DOM

目标：
1. 怎么将真正的 DOM 转换为 虚拟DOM
2. 怎么将 虚拟DOM 转换为 真正的 DOM

思路与深拷贝类似

# 函数柯里化

参考资料：

- [函数式编程](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)
- [维基百科](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96)

概念：

1. 柯里化：一个函数原本有多个参数，只传入**一个**参数，生成一个新函数，由新函数来接收剩下的参数来运行得到结果
2. 偏函数：一个函数原本有多个参数，只传入**一部分**参数，生成一个新函数，由新函数来接收剩下的参数来运行得到结果
3. 高阶函数：一个函数**参数是一个函数**，该函数对参数这个函数进行架构，得到一个函数，这个加工用的函数就是高阶函数

为什么要使用柯里化？  为了提升性能，使用柯里化可以缓存一部分能力

使用两个案例来说明：

1. 判断元素
2. 虚拟DOM 的 render方法


1. 判断元素：

Vue 本质上是使用 HTML 的字符串作为模板，将字符串的模板 转换为 AST，在转换为 VNode
- 模板 -> AST（抽象语法树）
- AST -> VNode
- VNode -> DOM

哪一个阶段最消耗性能？
最消耗性能是字符串解析 （ 模板 -> AST ）

例子：let s = '1 + 2 * (3 + 4)'
写一个程序 ，解析这个表达式，得到结果( 一般化 )
我们一般会将这个表达式 转换为 "波兰式" 表达式，然后使用栈结构来运算

在 Vue 中 每一个标签可以是真正的 HTML 标签， 也可以是自定义组件，问怎么区分？？？

在 Vue 源码中其实将所有可以用的 HTML 标签已经存起来了

假设治理只考虑几个标签：

```js
    let tag = 'div,p,a,img,ul,li'.split(',')
```
需要一个函数 ，判断一个标签名是否为内置的标签

```js
    function isHTMLTag(tagName) {
        tagName = tagName.toLowerCase()
        tag.forEach(f => {
            if (f === tagName) {
                return true
            }
        })
        return false
    }
```

模板是任意编写的，可以写的很简单，也可以写的很复杂， indexOf 内部也是要循环的

如果有 6 种内置标签，而 模板中有 10 个标签需要判断，那么就需要执行60次 循环

2. 虚拟 DOM 的 render 方法

思考： vue 项目  *模板 转换为 抽象语法树（AST）* 需要执行几次？？？

+ 页面一开始加载需要渲染
+ 每一个属性（响应式）数据在发生变化的时候 需要渲染
+ watch,computed  等等

我们昨天写的代码 ，每次需要渲染的时候，模板就会被解析一次（注意，这里我们简化了解析方法）

render 的作用是将 虚拟DOM 转换为 真正的DOM 加到页面中

- 虚拟DOM 可以降级理解为 抽象语法树
- 一个项目运行的时候  模板是不会变得，就表示 AST不会变得

我们可以将代码进行优化 将虚拟DOM缓存起来， 生成一个函数 ，函数只需要传入数据，就可以得到真正的DOM


# 讨论

- 这样的闭包回内存泄漏吗？
    - 性能一定是会有问题
    - 尽可能的提高新能
- 原生的好多东西都忘记，不知道从哪学起？


# 问题

问题：
 - 没明白柯里化怎么就只循环一次
    - **缓存一部分行为**
- mountComponent 这个函数里面的内容 没太立即
- call


# 响应式原理

- 我们在使用Vue的时候，赋值属性、获得属性都是直接使用Vue实例

- 我们在设置属性值得时候，页面的数据要更新


```js
    Object.definProperty( 对象, '设置什么属性名',{
        writable  // 可以写
        configurable // 可配置
        enumerable // 控制属性是否可枚举，是否可以for in 循环
        get () {}  // 取值触发
        set () {}  // 赋值触发
    })
```


```js
  // 简化后的版本
    function defineReactive(target, key, value, enumerable) {
        // 函数内部就是一个局部作用域，这个value 就只在函数内使用的变量（闭包）
        Object.defineProperty(target, key, {
            configurable:true,
            enumerable: !!enumerable,

            get () {
                console.log(`读取o的${key}属性`)
                return value
            },
            set (newValue) {
                console.log(`设置o 的 ${key} 属性为：${newValue}`)
                value = newValue
            }
        })
    }
```

实际开发中，对象一般是有多级的

```js
    let o = {
        list: [{},{}],
        ads: [{},{}],
        user: {}
    }
```

怎么处理呢？？？

对于对象可以使用 递归来响应式化，但是数组我们也需要处理

- push
- pop
- shift
- unshift
- reverse
- sort
- splice

要做什么事情呢？
1. 在改变数组的数据的时候，要发出通知
    - Vue 2 中的缺陷，数组发生变化， 设置length 没法通知 （Vue3 中 使用 Proxy 语法 ES6 的语法 解决了这个问题）
2. 加入的元素应该变成响应式的

技巧： 如果一个函数已经定义了， 我们需要扩展其功能， 我们一般的处理办法：
1. 使用一个临时的函数名来存储函数
2. 重新定义原来的函数
3. 定义扩展的功能
4. 调用临时的那个函数

指引性，规范性， 规划性

