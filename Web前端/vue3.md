

# 语法

## KeepAlive

被`keep-alive`包含的组件不会被再次初始化，也就意味着**不会重走生命周期函数**。

## 动态参数

使用方括号来指定动态参数名：`v-bind:[ObjName]='abc'`
## slot
父组件使用子组件时，可以向子组件中传入模板，而子组件将这传入的模板渲染到`<slot\>`处。

`<slot>`里面也能加东西，作为其默认值（当父组件不传输如何东西的时候显示）。

slot默认命名为default，也可以通过name属性命名（named slot）。之后父类传模板时使用`v-slot`来指定目标slot名。`slot:header`可以简写为`#header`。

## 依赖注入

使用`provide`和`inject`可以向该组件及其子组件放入和取出一个“全局变量“（可以是响应式数据）。可以在main.js里使用。

inject的第二个参数可以指定默认值。


# 使用


## ts封装axios

[在项目中用ts封装axios，一次封装整个团队受益😁 - 掘金](https://juejin.cn/post/7071518211392405541)





