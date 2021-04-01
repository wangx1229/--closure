# 闭包

    函数能够记住并访问其所在的词法作用域中的变量，称为闭包

## 词法作用域 

编译器在编译的代码的第一个阶段会将js代码词法化，词法作用域就是在`词法化阶段`形成的作用域。通俗一点的说，就是函数在`声明的时候`形成的一个作用域

![area](https://github.com/wangx1229/Closure/blob/main/imgs/area.png)

从上图可以看到 全局作用域包含标识符count和main，main的作用域包含内部标识符count和bar，bar作用域又包含count和foo，最后foo标识符包含count。

foo 函数作用域—>bar 函数作用域—>main 函数作用域—> 全局作用域形成一个作用域链。


## 闭包的几个列子  
    var name = 'Allen'

    function bar() {
      console.log(name)
    }

    function foo() {
      var name = 'Bill'
      
      bar()
    }

    foo()

执行foo之后，输出内容是Allen还是Bill，他又是否形成一个闭包。


    var fn     // 声明一个变量用于存放addNumber函数
    
    function foo(callback) {    // 使用回调函数
      var name = 'foo'
      callback()
    } 

    function closure() {
      var number = 0
      var name = 'counter'

      function addNumber() {
        number += 1
        return number
      }

      function getNumber() {
        return number
      }

      function getName() {
        return name
      }

      fn = getNumber       // 场景一

      foo(getName)       // 场景二

      return addNumber     // 场景三
    }

    const add = closure()
    add()
    fn()

问题一 addNumber函数赋值给外部的fn fn函数是否会发生闭包  
问题二 getName做为参数传递给foo函数，name返回是什么 是否会产生闭包

    function wait(msg) {
      setTimeout(() => console.log(msg), 1000)
    }

    for(var i=1;i<=5;i++) {
      setTimeout(() => console.log(i), i*1000)
    }

    for(let i=1;i<=5;i++) {
      setTimeout(() => console.log(i), i*1000)
    }

    function useState(init) {
      let state = init

      function setState(newValue) {
        state = newValue
        // 这里会重新执行一边函数组件
      }

      return [state, setState]
    }

我们知道`useState`函数会返回当前的`state`以及修改`state`的方法`setState`。`setState`修改`state` 函数组件重新渲染，`useState`执行，我们的`state`每次只能拿到`init`值，这明显不是我们所期望的。

    states = []
    index = 0

    function useState(init) {
      const currentIndex = index
      states[currentIndex] = states[currentIndex] || init

      function setState(newValue) {
        states[currentIndex] = newValue
        // 执行渲染函数
      }

      index++
      return [states[currentIndex], setState]
    }
    
这里`states`是记录在函数组件内部的，并通过某种方法可以获取到更新后的数据。而在每一个`useState`内部 通过`index`当前读取当前对应的`state`的值，并通过修改`index`的来读取下一个`state`的值。而`setState`则使用了闭包来记录每一次的`index`，其实就是依据数组下标对数组进行修改。

    function foo() {
      let name = 'Allen'
      let feature = {
        gender: man,
        age: 16,
      }

      function setName(newName) {
        name = newName
      }

      function getName() {
        return name
      }

      function setFeature(obj) {
        feature = {
          ...feature,
          ...obj,
        }
      }

      function getFeature() {
        return feature
      }

      return {
        name,
        feature,
        setName,
        getName,
      }
    }

## 闭包的应用   

1. 变量私有化，隐藏变量防止污染。开发时候经常使用都会避免在全局声明变量，因为会导致一些意想不到的bug。解决方案可以采取闭包，封装私有变量在函数内部，或者采取命名空间的方法，其本质就是使用一个对象，包含多个属性，达到减少变量的目的。

2. 柯里化，预先设置一些参数
    const create = (x) => (y) => x + y

    const add = create(1)
    result = add(2)

    通过预先设置参数，从而得到你想要的函数。可以达到参数复用的目的。实际应用涉及到一些规则匹配，减少参数

3. 演变为模块化
  
> 1.闭包  

  我们平时使用一个函数，然后内部添加代码。例如在函数内部创建变量以及函数等等，这其实就是做了一层封装，因为在函数外部是无法访问到函数内部的作用域的，从而达到隐藏内部信息的目的。它遵循的是一种最小暴露原则，这种原则就是指在设计中，尽量少暴露出必要的内容，而把其他内容隐藏起来，例如设计模块。这就涉及到如何选择你的函数包含那些变量和内容。假如所有的变量函数，都放在最外层，就打破了最小暴露原则，因为可能有些函数或者变量仅仅是某些函数所私有的。

> 2.commonjs

Node 应用由模块组成，采用 CommonJS 模块规范。每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。在服务器端，模块的加载是运行时同步加载的；在浏览器端，模块需要提前编译打包处理。

> 3.AMD  

CommonJS规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数。由于Node.js主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以CommonJS规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范


    define(['module1', 'module2'], function(m1, m2){
      return 模块
    })

> 4.CMD  

CMD规范专门用于浏览器端，模块的加载是异步的，模块使用时才会加载执行。CMD规范整合了CommonJS和AMD规范的特点。在 Sea.js 中，所有 JavaScript 模块都遵循 CMD模块定义规范。

    define(function (require) {
      require('./module1')
    })

> 5.ESmodule

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。


## 闭包的缺点

闭包会阻止垃圾回收，造成内存泄漏

## 内存泄漏
