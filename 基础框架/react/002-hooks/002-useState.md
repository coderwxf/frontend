```shell
const [状态， 状态更新函数] = useState(初始值)
```

状态值 

+ 第一次使用初始值
+ 之后都使用之前的状态值

状态更新函数 

+ 每次组件更新获取的都是新的函数
+ 执行状态更新函数
  + 更新状态
  + 通知视图更新

```jsx
import { useState } from 'react'

export default function Child(props) {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>{ count }</div>
      <button onClick={() => setCount(count + 1)}>increment</button>
    </>
  )
}
```

```jsx
// 初始值也可以传入函数，该函数只会在组件第一次渲染时被执行
let [x, setX] = useState(() => props.x + props.y);
```



## 更新原理

![image.png](https://s2.loli.net/2024/09/30/vzXB6MHQPVqJo35.png) 

使用类组件本质是创建实例 => 类组件更新本质是重新执行render方法，生成新的VDOM后进行dom-diff

使用函数组件本质是执行函数 => 每次执行产生一个新的执行上下文 => 生成一个新的闭包

可以理解为状态被保存在了一个和组件相关的数据结构上，而函数只是普通的更新函数



```jsx
const Demo = function Demo() {
    let [num, setNum] = useState(0);

    const handle = () => {
        setNum(100);
        setTimeout(() => {
          // 虽然UI更新了，但是因为闭包原理还是之前的闭包，所以num的值是0，不是100
	        console.log(num); // 0 
        }, 2000);
    };

    return <div className="demo">
        <span className="num">{num}</span>
        <Button type="primary"
                size="small"
                onClick={handle}>
            新增
        </Button>
    </div>;
}

export default Demo;
```

![image.png](https://s2.loli.net/2024/09/30/njXqDvoWM5KtJEy.png) 



### ==更新方式==

推荐 => 一个状态对应一个useState => 因为useState并不支持部分状态更新 => 需要自己手动实现

```jsx
import { useState } from 'react'

export default function Child(props) {
  const [point, setPoint] = useState({
    x: 0,
    y: 0
  })

  function changePoint() {
    setPoint({
      ...point,
      x: 20
    })
  }

  return (
    <>
      <div>{ JSON.stringify(point) }</div>
      <button onClick={changePoint}>click</button>
    </>
  )
}
```

```jsx
const point1 = {x: 10}

const point2 = Object.assign(point, {
  x: 20
})

// Object.assign(obj1, obj2, obj3) => 实际修改的obj1对象本身，并不是生成一个全新对象
// `useState`内部实现了类似于默认SCU的优化，此时状态将不会得到有效更新
console.log(point1 === point2) // => true
```



### 批处理

通过`useState`更新状态，状态的更新也是异步的，会被批量更新

所以`useState`也可以像`setState`那样传入函数



假设存在状态`x`对应的值是0

则

```jsx
const handle = () => {
  setX(x + 1)
  setX(x + 1)
  setX(x + 1)
}
```

执行handle方法后，x的状态值是`1`



也可以传入函数，此时执行handle方法的返回结果是`3`

```jsx
const handle = () => {
  setX(prevX => prevX + 1)
  setX(prevX => prevX + 1)
  setX(prevX => prevX + 1)
}
```



## 伪代码

```jsx
let _state; // 实际状态被存储在与vdom相关的一个数据结构中
function useState(initialValue) {
  	// null会被视为有效初始值
    if (typeof _state === "undefined") {
			_state = typeof initialValue === 'function' ? initialValue() : initialValue;
    }
		
  	// 所以状态更新函数每次都是全新的，并不复用
    return [_state, function setState(value) {
       newState = typeof value === 'function' ? value(_state) : value        
      
        // 如果新状态值和旧状态值完全一致，则停止更新「 没有更新必要, 类似于SCU的功能 」
      	if (Object.is(newState, _state)) return
      
        _state = newState;
        // 通知视图更新
      	// 1. 重新执行函数组件，生成新的vdom
        // 2. 进行DOM-DFF 获取Patch包
      	// 3. UI更新
    }];
}

let [num, setNum] = useState(0);
```



