---
id: react_hooks
title: react Hook
---

> 注意：所有hook都不可以在if,for中使用，确保hook的顺序调用
### 1. useState
- 让函数组件拥有状态，曾经的函数组件是不能保持状态的，因为函数每一次调用后都会销毁，useState相当于把值储存在一个数组中，在通过顺序，在通过顺序在当前的数组中取值;
- 如果异步又更改了state的值，state会被异步的渲染覆盖，所以需要传入函数，通过函数的参数拿到最新的state，重新计算。
- `useState()`初始值如果用函数的话，可以进行惰性初始化，也就是说在state中传入一个函数，当用到的时候在进行调用计算

```jsx
import React, {useState} from 'react';
import ReactDOM from 'react-dom';

function App() {
  const [number, setNumber] = useState(0);
  const add = () => {
    setTimeout(() => {
    // 这里如果是函数number拿到的就是最新的，如果是直接设置值，会从初始值0开始计数
      setNumber((number) => number + 1)
    }, 3000);
  }
  return <div>
    <p>{number}</p>
    <button onClick={() => setNumber(number + 1)}>+</button>
    <button onClick={add}>+</button>
  </div>
}

ReactDOM.render(<App/>,document.getElementById('root'));
```
- 实现
```js
let hookState = [];  // state的数组
let hookIndex = 0;  // 当前的hook
let scheduleUpdate;  // 重新更新渲染vdom的方法
function useState (initalState) {
    // 如果有初始值就设置初始值，初始值有可能是函数
    hookState[hookIndex] = hookState[hookIndex] || (typeof initialState ==='function'?initialState():initialState);
    let currentIndex = hookIndex;  // 通过这个值来保持当前的index值，当前闭包内的currentIndex，就不会变化了。
    // 设置state的方法
    function setState(newState) {
        // 如果传入的是一个函数，多用于解决异步，拿不到最新的值的问题
        if(typeof newState === 'function') {
            // 如果是函数，拿最新的state计算
            newState = newState(hookState[currentIndex])
        }
        hookState[currentIndex] = newState;
        scheduleUpdate() // 重新更新渲染
    }
    // 这里的index++，就是按顺序取出每一个值
    return [hookState[hookIndex++], setState]  // 返回一个数组
}
// 在容器重新挂载的时候，调用渲染
export function render(vdom,container){
    mount(vdom,container);
    scheduleUpdate && scheduleUpdate = ()=>{
        hookIndex=0;//在状态修改后调试更新的时候，索引重置为0
        //然后再进行虚拟DOM的比较更新 vdom=<App/>={type:App}
        compareTwoVdom(container,vdom,vdom);
    }
}
```

### 2. useMemo useCallback
- react性能优化，最重要的一点就是减少渲染次数，不然函数每一个都会渲染；child没有props改变的时候，依旧会渲染，用了memo就不会了
- 这两个api就是用于节约react渲染的次数的
```jsx
// 使用
import React, {useState, useMemo, useCallback} from 'react';
import ReactDOM from 'react-dom';

function Child ({
  number,
  setNumber
}) {
  console.log('render Child');
  return <button onClick={() => setNumber(number + 1)}>{number}</button>
}
let MomeChild = React.memo(Child) // 想当于类组件的pureComponent

function App() {
  console.log('render App');
  const [name, setName] = useState('')
  const [number, setNumber] = useState(0)
  
  const numberMemo = useMemo(()=>number, [number])
  const setNumberMemo = useCallback(()=>{setNumber(number + 1)}, [number])

  return <div>
    <input type="text" value={name} onChange={(e) => setName(e.target.value)} />
    <MomeChild number={numberMemo} setNumber={setNumberMemo} />
  </div>
}

ReactDOM.render(<App/>,document.getElementById('root'));
```

```jsx
export function useMemo(factory,deps){
    if(hookStates[hookIndex]){  // 如果这一项有值，说明不是第一次进来的
        let [lastMemo,lastDeps]=hookStates[hookIndex];  // 就要和上一次的deps值作对比
        let same = deps.every((item,index)=>item===lastDeps[index]);  //如果是一样的
        if(same){
            hookIndex++; // 就到下一个hook那里
            return lastMemo;  // 返回上一个数据，不重新计算
        }else{  // 否则就重新计算
            let newMemo = factory();
            hookStates[hookIndex++]=[newMemo,deps];
            return newMemo;
        }
    }else{  // 是第一次进来要先执行一下
        let newMemo = factory();
        hookStates[hookIndex++]=[newMemo,deps];  // 赋值给当前数组的这一项
        return newMemo;
    }
}
export function useCallback(callback,deps){
    if(hookStates[hookIndex]){ // 组件更新，发现之前缓存里面这一项有值，说明不是第一次进来的
        let [lastCallback,lastDeps]=hookStates[hookIndex]; // 就要和上一次的deps值作对比
        let same = deps.every((item,index)=>item===lastDeps[index]);//如果是一样的
        if(same){//每一个hook都要占用一个索引
            hookIndex++;
            return lastCallback; // 直接返回之前的函数
        }else{
            hookStates[hookIndex++]=[callback,deps];
            return callback;
        }
    }else{ // 是第一次进来要先执行一下
        hookStates[hookIndex++]=[callback,deps];// 赋值给当前数组的这一项
        return callback;
    }
}
```

### 3. useReducer
- 一般用于处理state很复杂的情况

```jsx
import React, {useReducer} from 'react';
import ReactDOM from 'react-dom';

const ADD = 'ADD';
const MINUS = 'MINUS';

function reducer(state, action) {
  switch (action.type) {
    case ADD:
      return {number: state.number + 1}
    case MINUS:
      return {number: state.number - 1}
    default:
      break;
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, {number: 0})
  return <div>
    <p>{state.number}</p>
    <button onClick={() => dispatch({type: ADD})}>+</button>
    <button onClick={() => dispatch({type: MINUS})}>-</button>
  </div>
}

ReactDOM.render(<App/>,document.getElementById('root'));
```

```js
export function useState(initialState){
    return useReducer(null,initialState);
}

export function useReducer(reducer,initialState){
    //把老的值取出来，如果没有，就使用默认值
    hookStates[hookIndex]=hookStates[hookIndex]||(typeof initialState ==='function'?initialState():initialState);
    //新定义一个变量currentIndex 
    let currentIndex = hookIndex;//对于第一个useState的那个函数而言这个索不变的

    function dispatch(action){
        let lastState = hookStates[currentIndex];//获取老状态
        let nextState;
        if(typeof action=== 'function'){
            nextState=action(lastState);
        }
        if(reducer){
            nextState = reducer(nextState,action);
        }else{
            // usestate
            nextState=action;
        }
        //如果老状态和新状态不相等，用的是浅比较
        if(lastState !== nextState){
            hookStates[currentIndex]=nextState;
            scheduleUpdate();//当状态改变后要重新更新应用
        }//如果是一样的，什么都不做
    }
    return [hookStates[hookIndex++],dispatch];
}
```

### 4. useContext
- 用于父亲的值共享
```jsx
import React, { createContext, useContext, useReducer } from "react";
import ReactDOM from "react-dom";

const ADD = "ADD";
const MINUS = "MINUS";

function reducer(state, action) {
  switch (action.type) {
    case ADD:
      return { number: state.number + 1 };
    case MINUS:
      return { number: state.number - 1 };
    default:
      break;
  }
}

let Context = createContext({});

function Count() {
  let { state, dispatch } = useContext(Context);  // 取出
  return (
    <div>
      <p>{state.number}</p>
      <button onClick={() => dispatch({ type: ADD })}>+</button>
      <button onClick={() => dispatch({ type: MINUS })}>-</button>
    </div>
  );
}

function App() {
  const [state, dispatch] = useReducer(reducer, { number: 0 });
  return (
    // 使用context
    <Context.Provider value={{ state, dispatch }}>
      <Count />
    </Context.Provider>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

```js
function createContext(initialValue={}){
  let context = {Provider,Consumer};
  function Provider(props){  // 函数组件
     context._currentValue = context._currentValue||initialValue;
     Object.assign(context._currentValue,props.value);
     return props.children;
  }
  function Consumer(props){
    return props.children(context._currentValue);
  }
  return context;
}

function useContext(context){
// 拿到之前存的直接返回就可以了
 return context._currentValue;
}
```

### 5. useEffect
- 用于解决react函数的副作用问题（修改全局变量，开启定时器，调用数据库，调用接口等）
> 什么是副作用（概念相对于纯函数）？
>   1. 相同的输入会产生相同的输出
>   2. 不能修改本函数作用域之外的变量
```js
// 副作用举例
let obj = {age: 10};
function sum (a, b) {
  obj.age = 100;  // 在函数调用完成之后，改变了函数外部变量的值，这个函数就不是纯函数，而产生了副作用；
  return a + b + Math.random();
}
let r1 = sum(1,2);
let r2 = sum(1,2);
console.log(r1, r2); // 两个函数不一定有同样的输出（副作用的原因，没有副作用应该会有一样的输出）
```
- 两个参数，第一个参数是一个函数（第一个函数的返回值，是一个销毁函数），第二个参数是一个依赖数组。
```js
// 定时器（不添加依赖数组，和函数返回值）
function App() {
  const [count, setCount] = useState(0)
  // 这样会造成开启多个定时器，一直在累加定时器，数量急剧增加的问题
  useEffect(() => {
    console.log('开启定时器');
    const timer = setInterval(() => {
      setCount(count => count + 1)
    }, 1000)
  });
  return <div>{count}</div>
}
```
- 添加返回值
```js
function App() {
  const [count, setCount] = useState(0)
  // 这样每次执行，会开始一个定时器，然后结束一个定时器。很多个定时器一开一关
  useEffect(() => {
    console.log('开启定时器');
    const timer = setInterval(() => {
      setCount(count => count + 1)
    }, 1000)
    return () => {
      console.log('关闭定时器');
      clearInterval(timer)
    }
  });
  return <div>{count}</div>
}
// 调用的时候先开启一次，然后就先关闭，在开启，关闭先执行，然后在执行开启
// 开启定时器
// 关闭定时器
// 开启定时器
// 关闭定时器
// 开启定时器
```
- 添加第二个参数，依赖数组
```js
function App() {
  const [count, setCount] = useState(0)
  useEffect(() => {
    console.log('开启定时器');
    const timer = setInterval(() => {
      setCount(count => count + 1)  // 这里注意，定时器计时，要使用函数，那count最新的值
    }, 1000)
    return () => {
      console.log('关闭定时器');  // 添加依赖项为[]，就不会执行return了，因为没有第二次更新。第二次更新前会走return的函数
      clearInterval(timer)
    }
  }, [])  // 添加依赖项，如果是空数组，那么就只运行一次
  return <div>{count}</div>
}
// 这里只走一次开启定时器，为啥呢，因为没有第二次调用，第二次调用的时候，才会先运行return里面的函数，在调用启动定时器
// 开启定时器
```

**实现**
```js
/**
 * 为了保证此回调函数不是同步执行，而是在页面渲染后执行
 * @param {*} callback 回调函数 页面渲染完成后
 * @param {*} dependencies 依赖数组
 */
export function useEffect(callback,dependencies){
  if(hookStates[hookIndex]){
    /* 第二次执行的时候，才会去调用销毁函数和判断依赖数组中的内容 */
    let [destroyFunction,lastDependencies] = hookStates[hookIndex];
    let allTheSame = dependencies&&dependencies.every((item,index)=>item===lastDependencies[index]);
    if(allTheSame){
        hookIndex++;  // 如果一样的就下一个hook，不进行更新
    }else{
        // 1、先调用销毁函数
        destroyFunction&&destroyFunction();
        // 2. 再开启
        // 把回调放在了宏任务队列中
        setTimeout(()=>{
            let destroyFunction = callback();   // 函数的返回值，没有调用，而是存了起来
            hookStates[hookIndex++] = [destroyFunction,dependencies];
        });
    }
  }else{//说明是第一次渲染
    setTimeout(()=>{
        let destroyFunction = callback(); // 函数的返回值，没有调用，而是存了起来
        hookStates[hookIndex++] = [destroyFunction,dependencies]; 
    });
  }
}
```
### 6. useLayoutEffect
- 浏览器时间环（微任务 —> 渲染 —> 宏任务）
- `useLayoutEffect`会DOM更新后，浏览器绘制前执行(绘制的时候，浏览器已经更新了，动画有过渡效果了)，使用方法和`useEffect`类似，只是执行时间不同，`useEffect`在渲染后执行（这个时候如果用动画，是有效果的。因为浏览器这个时候渲染过了，在设置动画，才会有改变）
- `useEffect`不会阻塞浏览器的渲染,`useLayoutEffect`会阻塞浏览器的渲染
![image](../static/img/useLayoutEffect.jpg)
```jsx
import React, { useEffect, useRef, useLayoutEffect } from "react";
import ReactDOM from "react-dom";
function App() {
  const ref = useRef();
  useEffect(() => {  // 浏览器已经渲染完了，这个时候在重新触发重绘，会引发动画效果
    ref.current.style.WebkitTransform = 'translate(500px)';
    ref.current.style.transition = 'all 3000ms'
  })
  useLayoutEffect(() => {  // 浏览器还没有开始渲染，设置属性，在渲染的时候一步到位了，不会有动态的改变
    ref.current.style.WebkitTransform = 'translate(500px)';
    ref.current.style.transition = 'all 3000ms'
  })

  let style = {
    width: '100px',
    height: '100px',
    backgroundColor: 'red'
  }
  return <div ref={ref} style={style}></div>
}

ReactDOM.render(<App />, document.getElementById("root"));
```

- 实现
```js
// 和useEffect类似，放在了微任务中，在渲染前执行
export function useLayoutEffect(callback,dependencies){
  if(hookStates[hookIndex]){
      let [destroyFunction,lastDependencies] = hookStates[hookIndex];
      let allTheSame = dependencies&&dependencies.every((item,index)=>item===lastDependencies[index]);
      if(allTheSame){
          hookIndex++;
      }else{
          destroyFunction&&destroyFunction();
          //把函数放在了微任务队列中
          queueMicrotask(()=>{  // 只有这里放在了微任务中
              let destroyFunction = callback();
              hookStates[hookIndex++] = [destroyFunction,dependencies];
          });
      }
    }else{//说明是第一次渲染
      Promise.resolve().then(()=>{
          let destroyFunction = callback();
          hookStates[hookIndex++] = [destroyFunction,dependencies];
      });
    }
}
```

### 7. useRef
- 获取真实dom
```jsx
export function useRef(initialState){
    hookStates[hookIndex]= hookStates[hookIndex]||
    {current:initialState};
    return hookStates[hookIndex++];
}
// 在createDOM中就把dom指向给了ref.current
export function createDOM(vdom){
    if(ref)//通过虚拟DOM创建真实DOM之后，虚拟DOM的ref属性的current属性等于真实DOM
       ref.current = dom;
    return dom;
}
```