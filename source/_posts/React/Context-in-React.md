---
title: React中的Context
date: 2021-05-25 18:52:51
tags:
  - React
  - Context
---

## Context定义和目的

Context 提供了一种在组件之间共享数据的方式，而不必显式地通过组件树的逐层传递 props。



## 应用场景

### 哪些数据会需要共享？

>  Context 设计目的是为了共享那些对于一个组件树而言是**“全局”的数据**，例如当前认证的用户、主题或首选语言。



## 使用步骤

### 1. 创建并初始化Context

```javascript
const MyContext = createContex(defaultValue);
```

创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

### 2. 订阅Context

```javascript
<MyContext.Provider value={/* 某个值 */}>
```

Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

这里有两个相关的概念

- Provider - Context提供者，或Context的订阅者。可以理解为通过Provider为其内部组件订阅了Context值的变动，一旦Context值有变化，就会触发内部组件重新渲染。
- Comsumer - Context消费者（消费组件），或者叫Context使用者。即在Provider内部使用```useContext()```来使用或消费Context的组件。这些组件通过useContext()获取、使用Context的最新值。



### 3. 使用Conext

#### 3.1 React组件中使用

```javascript
const value = useContext(MyContext);
```

在消费组件中引用Context。value会从组件树中离自身最近的那个匹配的Provider中读取到当前的Context值。



#### 3.2 纯函数式组件中使用

在纯函数式的组件中，可以使用```Consumer```来引用context的值。如果没有上层对应的Provider，value等同于传递给```createContext()```的```defaultValue```. 

```javascript
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```



### 4. Context的更新

#### 4.1 自上而下更新Context

自上而下更新指的是更新Provider的value值。当 Provider 的 `value` 值发生变化时，它内部的所有消费组件内通过```useContext```获取到的值会自动更新，并触发重新渲染。

```javascript
//App.js

// ....

export default function App() {
    //...
    
    // 
    const {contextValue, setContextValue} = React.useState(initialValue);

    // function to update the context value
    function updateContext(newValue) {
        // ...

        // 更新contextValue, ConsumerComponent1, ConsumerComponent2, ConsumerComponent3, ConsumerComponent11都会触发重新渲染。
        setContextValue(newValue)
    }

    ...
    return (
        <App>
            <MyContext.Provider value={contextValue}>
                <ConsumerComponent1>
                    <ConsumerComponent11>
    					// ....
                    </ComsumerComponent11>
                </ConsumerComponent1>

                <ConsumerComponent2 />
                <ConsumerComponent3 />
            </MyContext.Provider>
        </App>
    );
    
}
```



#### 4.2 自下而上（从消费组件）更新Context

在某些情况下，需要在某个消费组件内更新```context```，并且适配到整个程序。比如通过应用程序的```setting```组件修改UI风格。 这时就需要通过回调将更新一层层传递到对应的Provider，更新Provide对应的```value```，从而触发所有相关消费组件的更新。



```javascript
// app.js

export default function App() {
    ...
    const {contextValue, setContextValue} = React.useState(initialValue);

    // function to update the context value
    function updateContext(newValue) {
        // ...

        // 更新contextValue, ConsumerComponent1, ConsumerComponent2, ConsumerComponent3, ConsumerComponent11都会触发重新渲染。
        setContextValue(newValue)
    }

    ...
    return (
        <App>
            <MyContext.Provider value={contextValue}>
                <ConsumerComponent1>
                    <ConsumerComponent11 updateValue={updateContext}> // 通过回调形式的props, 在ConsumerComponent11中更新contextValue, 因为contextValue属于最顶层的Provider的值，所以也会触发ConsumerComponent1, ConsumerComponent2, ConsumerComponent3重新渲染。
                    </ComsumerComponent11>
                </ConsumerComponent1>

                <ConsumerComponent2 />
                <ConsumerComponent3 />
            </MyContext.Provider>
        </App>
    );
}
```



 #### 4.3 Provider嵌套

在一些情况下，可能会出现同一个Context的provider嵌套的情况，这时候可以理解为两个Context。不同的是，

```javascript
...
const {contextValue, setContextValue} = React.useState(initialValue);

// function to update the context value
function updateContext(newValue) {
    // ...
    
    // 更新contextValue, ConsumerComponent1, ConsumerComponent2, ConsumerComponent3, ConsumerComponent11都会触发重新渲染。
    setContextValue(newValue)
}

...
return (
	<App>
        <MyContext.Provider value={contextValue}>
            <ConsumerComponent1>
                <ConsumerComponent11 />
            </ConsumerComponent1>

            <ConsumerComponent2>
                ...
                // 如果只希望更新ComsumerComponent21, ComsumerComponent22中的值
                
                const localContextValue = useContext(MyContext); // 从上一层Provider中获取当前值

				const {tempContextValue, setTempContextValue} = React.useState(localContextValue);

				function updateTempContext(newValue) {
                    // 这里更新以后只会触发ConsumerComponent21和ConsumerComponent22的重新渲染
                    setTempContextValue(newValue); 
                }

				// 这里新建Provider，在ConsumerComponent21和ConsumerComponent22之间共享数据。
                <MyContext.Provider value={tempValue}>
                    <ConsumerComponent21>
                    	// 在ConsumerComponent21中通过useContext(MyContext)订阅
                    	// 获取到的值为离自身最近的那个匹配的Provider中读取到的Context值,即tempValue
                    </ConsumerComponent21>
                    <ConsumerComponent22>
                    </ConsumerComponent22>
				</MyContext.Provider value={contextValue}>
            </ConsumerComponent2>
            <ConsumerComponent3 />
        </MyContext.Provider>
    </App>
);

```





## 官方文档

官方文档请参考下边的基础和高级教程。

- [Hook API 索引 – React (reactjs.org)](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext) 
- [Context – React (reactjs.org)](https://zh-hans.reactjs.org/docs/context.html) 



