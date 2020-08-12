---
title: react-hook
date: 2020-08-11 14:36:07
tags: react
categories: React JavaScript
cover: https://images.pexels.com/photos/1309095/pexels-photo-1309095.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
---

### react
> 用于构建用户界面的JavaScript库

- 声明式编程
- 组件化 JSX
- 一次学习、随处编写。
    - react 编写web端页面
    - 配合react native 编写手机App
    - 配合react 360开发vr界面
    
#### setState

##### setState是同步更新还是异步更新？

- 正常情况下，也就是没有使用Concurrent组件的情况下，是同步更新
    - 但是不会立即获取到最新的setState，因为setState只是单纯的将你传进来的新的state放在updateQuene这条链表上，等函数(合成事件比如onClick)结束时，会触发一个回调函数，这个回调函数才会真正的更新state以及重新渲染
- 使用Concurrent组件才是真正的异步渲染模式
    - 同样没有办法直接获取最新状态，并且在执行react更新、渲染的过程中使用真正的异步方法-postMessage

##### 如何setState立即拿到新的state

```javascript
// way-1
flushSync(()=> {
    this.setState({
        num: 1
    })
})
console.log(this.state.num)

// way-2
div.addEventListner('click',()=> {
     this.setState({
        num: 1
    })
    console.log(this.state.num)
})

// way3
setTimeout(()=>{
    this.setState({
        num: 1
    })
    console.log(this.state.num)
},0)


```

#### react hook

##### useState
> useState 是允许你在 React 函数组件中添加 state 的 Hook。参数是初始化的`state`，返回值是一个数组：第一个是`state`第二个值是更新`state`的方法。每次函数组件执行，状态都是`独立`的

下边这个例子，第一次点击Delay setCount时，count为0；
第二次点击setCount，count为0，并+1执行一个render；
第三次点击setCount，count为1，并+1执行一个render；
3s后执行计时器，此时记录的count值依旧是当时触发handleClick函数时传count值0

*如何拿到最新的`state`呢？*

- 想setCount 传入函数
- 可以使用 **ref**

```javascript
// 当点击Delay按钮之后，3s内点击setCount2次，页面发生了什么？
// count 由0变成1、2，3s后变为1
// 每次执行handleClick都是保存当前的状态count，执行delay时，3s后 count的值还是最开始的count 0


function Example2() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
      setTimeout(() => {
          setCount(count + 1);
      }, 3000);
  };

  return (
      <div>
          <p>{count}</p>
          <button onClick={() => setCount(count + 1)}>
              setCount
          </button>
          <button onClick={handleClick}>
              Delay setCount
          </button>
      </div>
  );
}

function App() {

  const [count, setCount] = useState(0)

  useEffect(()=> {
    setTimeout(()=> {
      setCount(count+1)
    },1000)
  },[count])

  return (
    <div className="App">
      <Example2 />
    </div>
  );
}
```


```javascript

// 当点击Delay按钮之后，3s内点击setCount2次，页面发生了什么？
// count 由0变成1、2，3s后变为3

function Example3() {
  const [count, setCount] = useState(0);

  const currentCount = useRef(count);

  currentCount.current = count

  const handleClick = () => {
      setTimeout(() => {
        setCount(preCount => preCount + 1 );
          // 执行计时器 currentCount.current 为2
          // 或者使用
          // setCount(currentCount.current  + 1);
      }, 3000);
  };

  console.log('currentCount',currentCount)

  return (
      <div>
          <p>{count}</p>
          <button onClick={() => setCount(count + 1)}>
              setCount
          </button>
          <button onClick={handleClick}>
              Delay setCount
          </button>
      </div>
  );
}

function App() {

  const [count, setCount] = useState(0)

  useEffect(()=> {
    setTimeout(()=> {
      setCount(count+1)
    },1000)
  },[count])

  return (
    <div className="App">
      <Example3 />
    </div>
  );
}
```

##### useEffect
> `useEffect`可以让你在函数组件中执行副作用操作

> `useEffect`的第一个参数为function，用来指明DOM更新之后做的操作；第二个参数（可选）；返回值为函数(可选)，effect清除机制，组件卸载用来清除计时器、消息订阅等



- DOM更新后执行你传入的Effect函数
- useEffect 放在函数组件内，确保可以直接访问到内部的`state`变量
- 默认每次渲染都会执行
    - 第二个传入[]时， effect 不依赖于 props 或 state 中的任何值，仅在组件挂载和卸载时执行
    - 当数组中有值，只有值发生变化才会执行effect，避免effect不必要的重复调用
    
##### useMemo
> 通过一些变量计算得到新值，通过把这些变量加入依赖deps，当deps未发生变化时，跳过计算
- 缓存耗时的计算
- 存储引用类型的数据，可以传入对象字面量，匿名函数等，甚至是 React Elements，避免父组件更新时，子组件不必要更新

```javascript
function Example(props) {
    const [count, setCount] = useState(0);
    const [foo] = useState("foo");

    // 当count变化时，main不会重新render
    const main = useMemo(() => (
        <div>
            <Item key={1} x={1} foo={foo} />
            <Item key={2} x={2} foo={foo} />
            <Item key={3} x={3} foo={foo} />
            <Item key={4} x={4} foo={foo} />
            <Item key={5} x={5} foo={foo} />
        </div>
    ), [foo]);

    return (
        <div>
            <p>{count}</p>
            <button onClick={() => setCount(count + 1)}>setCount</button>
            {main}
        </div>
    );
}

// 优化子组件


const data = useMemo(() => ({ id }), [id]);
return <Child data={data}>;

//--------------

function Parent() {
  const [count, setCount] = useState(1);
  const [val, setValue] = useState('');

 // 使用useCallback缓存函数    
  const getNum = useCallback(() => {
      return Array.from({length: count * 100}, (v, i) => i).reduce((a, b) => a+b)
  }, [count])

  return <div>
      <Child getNum={getNum} />
      <div>
          <button onClick={() => setCount(count + 1)}>+1</button>
          <input value={val} onChange={event => setValue(event.target.value)}/>
      </div>
  </div>;
}

// 优化前
const Child = React.memo(function ({ getNum }) {
  console.log('child-render')
  return <h4>总和：{getNum()}</h4>
})

// 优化后
const Child = function ({ getNum }) {
  console.log('child2-render')
  return <h4>总和：{getNum()}</h4>
}


```
