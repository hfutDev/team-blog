---
title: React系列学习笔记之表单详解
tags: react
categories: Tech-study
---

---
> **1. 可控组件和不可控组件**
> **2. 不同表单元素的使用**
> **3. 如何复用事件处理函数**

 **1.1 不可控组件**
```jsx
<input type="text" defaultValue="Hello World!">
```
前面这个简单的input组件**[react中有两大类组件，一类是自定义的组件，另一类是原生的HTML标签]**就是一个不可控组件。

因为组件中的数据和state中的数据不再对应，而是写死在了组件中。组件中的数据对整个组件来说是不可控的。

不可控组件组件下，如果我们要获取到`defaultValue`的值，就必须得去操作真实的DOM.
```jsx
<input type="text" ref="hello" defaultValue="Hello World!">
```
```jsx
var inputValue = React.findDOMNode(this.refs.hello).value;
```
给组件添加`ref`属性，然后通过`React.findDOMNode()`找到对应的节点。

 **1.2 可控组件**
```jsx
getInitialState: function() {
    return {
        hello: "Hello World!"
    };
}
```
```jsx
<input type="text" ref="hello" value={this.state.hello}>
```
上面这个就是可控组件，数据存储在state中，便于使用，读取的是JS对象而不是DOM节点。符合reac单向数据流的特点，数据从state-->render。
使用可控组件可以方便的对数据进行处理，比如在handleChange里面可以对数据进行方便的操作。

 **2. 不同表单元素的使用**
```jsx
<label htmlFor="name" className="name">name<label>
```
```jsx
<input type="checkbox" /* "radio" "text"*/
       value="A"
       checked={this.state.checked}
       onchange={this.handleChange} />
```
```jsx
<textarea value="A" rows="10" cols="30"
          onChange={this.handleChange} />
```
```jsx
<select value={this.state.hello}
        onChange={this.handleChange}>
    <option value="one">一</option>
    <option value="two">二</option>
</select>
```
说明一下上面select的value是每次被选中option的value

**3. 如何复用事件处理函数**
事件函数的可以有`bind`和`name`复用两种
```jsx
handleChange(name, event){
    //...
}

...

<input type="text" onChange={this.handleChange.bind(this, `input1`)}>
<input type="text" onChange={this.handleChange.bind(this, `input2`)}>
```
`bind`每次将第二参数传给handleChange的name,这样每次调用handleChange，就可以分辨出是哪个input发生了变化。
```jsx
handleChange(name,event){
    var name = event.target.name;
    ...
}

...

<input type="text" name="input1" onChange={this.handleChange}>
<input type="text" name="input2" onChange={this.handleChange}>
```
给input添加name属性，每次在handleChange中获取目标事件的name，确定要处理的input.达到复用事件处理函数。

两种方法都可以复用事件处理函数。不过通过name实现的复用性能比bind实现的低。前者每次都会在事件处理函数中获取一次name的值，后者直接在参数中直接传入。推荐使用bind.