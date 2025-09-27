# react

#### 1.JSX语法

- **JSX是js和html结合的模版语法，方便列表渲染和条件渲染**

```plain
function App(){
  const list=[
    {id:1，name:'小吴'}，
    {id:2，name:'小李'},
    {id:3，name:'小花'},
  ]
  const listContent=list.map(item =>(
    <li key={item.id}>{item.name}</li>
  ))
  
  return (<ul>{listContent}</ul>)
}
```

-  **条件渲染**

```plain
//&&短路
{showHint &&<p><i>提示：你最喜欢的城市？</i></p>}

//三目运算符
<button onClick={() => {
    setShowHint(!showHint)}}>
    {showHint === false ? '隐藏提示' : '显示提示'}
</button>

//if-else
function Banner({ vip }) {
  let content;
  if (vip === 1) content = <GoldBanner />;
  else content = <NormalBanner />;
  return <section>{content}</section>;
}
```

+ **jsx文件使用大括号{}在标签内容和标签属性中进行插值**
+ **jsx文件只支持返回一个根元素，需求：不会产生额外的父级标签**

  - `->不需要写 key 或其他属性时=>用<> … </>`

  - `->需要写 key 或其他属性时=>用<React.Fragment> … </React.Fragment>`

+ **jsx文件允许通过键值对的形式来设置style <=style是键值的结构**

```plain
//将图片通过Import方法导入使用而不是在线url
import image from'./logo.svg
function App(){
  const imgstyleObj={
    width: 200,
    height: 200,
    //带有-的样式属性要写成驼峰式
    backgroundColor: 'grey'
  }
  
  return(
  //jsx文件要求单标签必须用/进行闭合
  <img	src={image}
        alt=''
        className='small'
        style={imgstyleObj}
    />)
}
```

+ **jsx的展开语法≠es6的展开运算符**

  - `->jsx的展开语法=>语法标记=>在编译的时候做一些额外操作`

  - `->es6中的展开运算符=>会以‘键:值’的形式直接放置+不能在没有容器的地方单独使用`

```plain
const imgData ={
/*JSX最终会由Babel等工具编译成普通的JS对象(React.createElement调用)
  使用className避免与JS关键字class(定义类的语法)互相冲突*/
  className:'small',
  style: {
    width: 200
    height:200
    backgroundColor: 'grey'
  }	
}

//return后面无论单行还是多行都带上(),不容易出错
return (
  //img标签中的src和alt属性不可提出会警告
  <img	src={image}
        alt=''
        {...imgData}
    />)
```

#### 2.key的作用

+ **列表渲染时在遍历生成的每个根元素都要有唯一标识key+key 不是全局唯一的，只指定父组件内部的顺序。**

```plain
const listContent =list.map(item =>(
  <Fragment key={item.id}>
    <li>{item.name}</li>
    <li>-----------</li>
  </Fragment>
))
```

+ **重置组件的状态**

```plain
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      //通过向组件传递不同的 key来重置组件状态
      <Form key={version} />
    </>
  );
}
```

`补充：将组件渲染在不同的位置也会重置组件的状态`

```plain
  //每当 Counter 组件从 DOM 中移除时，它的 state 会被销毁
  {isPlayerA &&<Counter person="Taylor" />}
  {!isPlayerA &&<Counter person="Sarah" />}
```

+ **需要严格匹配的场景使用key**

```plain
let image =images[index];
return(
  <>
    <button onclick={handleClick}>
      下一张
    </button>
    <h3>{images.length}张图片中的第{index+1}张</h3>
    
    //🚩 错误：复用<img> DOM->新图绘制(下载 → 解码 → 绘制)->屏幕上是上一帧旧图或空白
    <img src={image.src}/>
    
    //✅ 正确：key 更改->重新创建<img>DOM节点=>制造闪白=>图片与文本始终匹配
    <img key={image.src} src={image.src}/>
    
    <p>{image.place}</p>
  </>
)
```

#### 3.StrictMode严格模式

+ **组件内部的功能审查和排错：**
  + `在开发环境、组件首次挂载时，把同一个初始化函数同步调用两次，生成两次虚拟 DOM，但不会提交到真实 DOM（页面不会闪）`
  + `用户只会看到第一次的渲染结果，第二次的结果会被扔掉，用来查找“意外的不纯性”，而在生产环境只会跑一遍`


```plain
// 函数组件会在每次渲染运行两次。
function TodoList() {

  const [todos, setTodos] = useState(() => {
    // 初始化函数在初始化期间会运行两次。
    return createTodos();
  });

  function handleClick() {
    setTodos(prevTodos => {
      // 更新函数在每次点击中都会运行两次
      return [...prevTodos, createTodo()];
    });
  }
  // ...
```

+ **初次渲染时, React 会调用根组件，后续渲染会调用内部状态更新触发了渲染的函数组件。**
+ **纯函数->只根据输入参数或闭包变量计算返回值**
+ **函数不纯->做了“对外”的事或“同一输入不同输出”，如使用了外部可变状态（随机数、Date.now()，全局变量...）、发网络请求、写日志(cosole.log)、 …**

```plain
setTodos(prevTodos => {
  // 🚩 错误：修改传入的参数（prevTodos）
  prevTodos.push(createTodo());

   // ✅ 正确：使用新状态整体替换
  return [...prevTodos, createTodo()];
});

```

```plain
let guest = 0;
//🚩 错误：正在更改预先存在的变量
function Cup() {
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

//✅ 正确：输出只依赖 props 传入的只读参数
function Cup({ index }) {
  return <h2>Tea cup for guest #{index}</h2>;
}
```

#### 4.react组件的创建方式

- ##### ->函数组件（官方推荐）

- ##### ->类组件（比较复杂冗余，包含复杂的生命周期）

  ```
  // 函数组件（推荐）
  function Hello() {
    return <h1>Hello React</h1>;
  }
  
  // 类组件
  class Hello extends React.Component {
    render() {
      return <h1>Hello React</h1>;
    }
  }
  ```

#### 5.React 中数组状态的增删改查

```plain
//增  也可以用concat拼接
setList(prev => [...prev, newItem]);
//删  也可以用slice
setList(prev => prev.filter(t => t.id !== id));
//改
setList(prev => prev.map(item => (
  item.id === editId ? { ...item,name:newName } : item)));
//查
const lowerKeyword = useMemo(() => keyword.trim().toLowerCase(), [keyword]);
const showList = useMemo(() =>
  list.filter(item => item.name.toLowerCase().includes(lowerKeyword)),
  [list, lowerKeyword]
);

return (
  <div>
    {showList.map(item =>(<ListItem key={item.id} {...item}) />)}
  </div>
)
// 注意数组8个会改变自身的方法，使用像sort、reverse方法前先拷贝一份数据
const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list];
    nextList.reverse();  
    setList(nextList);
  }
```

#### 6.常用hook函数

 ——只能在组件或自定义 Hook的最顶层调用

React 靠调用顺序在内部维护一条隐式链表，每次渲染都按固定顺序把 Hook 状态依次存进去；

只要顺序变了，链表就错位--->不能在循环、条件语句或嵌套函数(如事件处理函数、回调函数）中调用它or提取一个新组件并将状态移入其中

- **条件语句调用hook->条件变化->每次渲染时的调用顺序不一致->匹配不准；**
- **循环语句调用hook->循环次数变化->每次渲染时的调用顺序不一致->匹配不准；**

+ **嵌套函数调用hook->不属于顶层渲染流程，React 无法跟踪其状态**

##### `useState`

会返回state和state更新函数组成的数组

- **`set`函数 仅更新下一次_渲染的状态变量。如果在调用`set` 函数后读取状态变量，则仍会得到在调用之前显示在屏幕上的旧值 。**
- **如果Object.is判断你提供的新值与当前 `state`相同，React 将 跳过重新渲染该组件及其子组件。**
- **React18->无论哪种触发源，只要是在同一事件循环里的多次 `setState`，都会被压入同一队列，事件处理完后统一合并成一次渲染，再一次性提交到真实 DOM。**
-  **让子组件共享一个状态->把状态提升到父组件函数**
- **删除不必要的state**

```plain
const [items, setItems] = useState(initialItems);
//🚩 错误：避免重复的state：selectedItem 的内容与 items 列表中的某个项是同一个对象
 const [selectedItem, setSelectedItem] = useState(items[0]);

const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
//🚩 错误：避免冗余的 state：可以从 firstName 和 lastName 中计算出 fullName
const [fullName, setFullName] = useState('');
```

+ **状态与渲染树中的位置相关**
  
    - 不同位置的组件及其状态是独立的
    - 相同位置的相同组件会使得 state 被保留下来
    - 相同位置的不同组件会使 state 重置 (销毁状态)
    - 组件嵌套定义
    
      - 🚩 `错误：组件嵌套定义->外组件每次渲染都会生成新的不同的内组件->重复重置内组件状态`
      
      
        - `✅ 正确：组件都定义在最顶层`
      
    
    
    - 状态和组件在 UI 树中的位置相关,而不是在 JSX 中的位置
    
      - =>每次渲染时,React会对比上一次渲染的虚拟 DOM和本次渲染的虚拟 DOM
    
      -  ->决定哪些组件可以复用（保留内部状态），哪些需要销毁重建（重置内部状态）。
    
    
      -  ->组件类型一致：标签名相同才有可能复用。
    
    
      -  ->身份标识一致：优先看key,key同保留,没有 key看组件在父容器中的位置索引(第几个子元素)

```plain
export default function App() {
  ...
  🚩 错误：if 内外有两个return 语句，它们带有不同的 <Counter /> 
          ->误以为勾选状态变化state 会被重置
  ✅ 正确：无论是否勾选，App 组件都会返回一个包裹着<Counter />作为第一个子组件的div。
          ->属于在相同位置状态不重置 
  if (reverse) {
    return (
      <>
        <Field key="lastName" label="姓氏" /> 
        <Field key="firstName" label="名字" />
        {checkbox}
      </>
    );
  }
  return (
    <>
      <Field key="firstName" label="名字" /> 
      <Field key="lastName" label="姓氏" />
      {checkbox}
    </>
    );    
}
```

+ 防止state深度嵌套进行扁平化

  - ->每个数据项用唯一 ID 存储，避免重复。
  - ->用 ID 数组（如`childIds`）替代直接嵌套对象，表达父子 / 关联关系。

  - ->所有数据项在顶层以`{ [id]: 数据项 }` 形式存储，便于直接访问。

```plain
import { useState } from 'react';

// 扁平化的状态结构（非嵌套）
const [state, setState] = useState({
  // 所有产品以 ID 为键存储在顶层
  products: {
    1: { id: 1, name: "手机", categoryId: 101 },
    2: { id: 2, name: "笔记本", categoryId: 102 },
    3: { id: 3, name: "华为手机", categoryId: 1011 }
  },
  // 所有分类以 ID 为键存储，用 childIds 数组关联子分类
  categories: {
    100: { id: 100, name: "电子设备", childIds: [101, 102] },
    101: { id: 101, name: "手机", childIds: [1011] },
    1011: { id: 1011, name: "国产手机", childIds: [] },
    102: { id: 102, name: "电脑", childIds: [] }
  }
});

// 子组件：通过 ID 递归渲染层级结构
function PlaceTree({ id, parentId, placesById, onComplete }) {
  const { title, childIds } = placesById[id]; // 直接通过 ID 访问数据
  
  return (
    <li>
      {title}
      <button onClick={() => onComplete(parentId, id)}>完成</button>
      
      {/* 递归渲染子项（通过 childIds 数组） */}
      {childIds.length > 0 && (
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      )}
    </li>
  );
}

// 状态更新->直接操作对应 ID 的数据，无需深层解构
function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  const handleComplete = (parentId, childId) => {
    setPlan(prev => ({
      ...prev,
      // 直接定位到父节点，更新 childIds 数组
      [parentId]: {
        ...prev[parentId],
        childIds: prev[parentId].childIds.filter(id => id !== childId)
      }
    }));
  };
}
```

 **陷阱**

+ `set` 函数 仅更新下一次 渲染的状态变量，不会立马更新状态

```plain
const [name,setName]=useState('Taylor')

function handleClick() {
  //如果需要立马使用最新值,在set函数之前先用变量保存值
  const nextName='Robin';
  setName(nextName);
  console.log(name);   // "Taylor"
  console.log(nextName)// "Robin"
}
```

+ <font style="background-color:#FFFFFF;"> </font><font style="background-color:#FFFFFF;">根据先前的 state 更新 state</font>

```plain
const [age, setAge] = useState(42);

//🚩 错误：在本次渲染的函数作用域里，age 永远绑定到 42，只保留最后一次 setAge(43)
function handleClick() {
  setAge(age + 1); // 43
  setAge(age + 1); // 43
  setAge(age + 1); // 43
}

//✅ 正确：使用更新函数拿最新值=>在下一次渲染期间按照相同的顺序调用它们
function handleClick() {
//命名惯例：setAge->a  setFirstName->fn   a=>a+1属于更新函数
  setAge(a => a + 1); // setAge(43)
  setAge(a => a + 1); // setAge(44)
  setAge(a => a + 1); // setAge(45)
}
```

+ <font style="background-color:#FFFFFF;"> </font><font style="background-color:#FFFFFF;">更新状态中的对象和数组 </font>

```plain
//🚩 错误：状态被认为是只读的，直接修改属性不会触发重新渲染
form.firstName = 'Taylor';
//✅ 正确：使用更新函数整体替换对象
setForm({
  ...form,
  firstName: 'Taylor'
});
```

+ <font style="background-color:#FFFFFF;"> </font><font style="background-color:#FFFFFF;">重复创建初始状态</font>

```plain
function TodoList() {
  //🚩 错误：不会触发重新渲染，但会重复执行初始化函数createInitialTodos(),浪费性能
  const [todos, setTodos] = useState(createInitialTodos());

  //✅ 正确：传函数本身而不是调用结果,只在首次渲染时才会调用它获取返回值
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
  }
```

+ <font style="background-color:#FFFFFF;">state存储一个函数本身</font>

```plain
//🚩 错误：react只会认为someFunction是个初始化函数，初始渲染的时候调用它
const [fn, setFn] = useState(someFunction);
//✅ 正确：存储函数本身
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  //🚩 错误：react只会认为someFunction是个更新函数，重新渲染的时候调用它
  setFn(someOtherFunction);
  //✅ 正确：更新函数状态
  setFn(() => someOtherFunction);
}
```

+ <font style="background-color:#FFFFFF;">错误地指定事件处理函数=>报错</font><font style="background-color:#FFFFFF;">“Too many re-renders” </font>

```plain
// 🚩 错误：在渲染过程中调用事件处理函数
return <button onClick={handleClick()}>Click me</button>

// ✅ 正确：将事件处理函数传递下去
return <button onClick={handleClick}>Click me</button>

// ✅ 正确：传递一个内联函数
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

##### `useReducer`

 **->使用更加规范更加统一的方式对状态的操作方式进行管理**

**->useReducer需要是纯净的**

**->调用 `dispatch` 函数 不会改变当前渲染的 state**

```plain
import { useReducer, useState } from "react"
//reducer 函数通过响应 actions 来决定 状态如何更新
function countReducer(state, action) {
  switch (action.type) {
    case "increment":
      return state + 1
    case "decrement":
      return state - 1
    default:
      throw new Error()
  }
}
export default function App() {
  //接收 reducer 函数和初始状态作为参数
  //返回当前状态 state 和用于触发状态更新的 dispatch 函数
  const [state, dispatch] = useReducer(countReducer, 0)
  //通过 dispatch 函数派发包含 type 属性的 action 对象，明确告知 reducer"发生了什么操作"
  const handleIncrement = () => dispatch({ type: "increment" })
  const handleDecrement = () => dispatch({ type: "decrement" })

  return (
    <div style={{ padding: 10 }}>
      <button onClick={handleIncrement}>+</button>
      <span> {state} </span>
      <button onClick={handleDecrement}>-</button>
    </div>
  )
}
```

##### `useRef`

- **本质是一个普通的 JS 对象（`{ current: ... }）,在组件多次渲染中保持引用不变**

- **Ref.current值是可以直接修改的，值变化后React 不会重新渲染组件**

```plain
//保留不用于渲染的值，组件重新渲染的时候ref将返回相同的对象
//存储一个 interval ID 并在以后检索它清理定时器
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}

function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

- **引用状态改变前的值**

```plain
const [count, setCount] = useState(0)
  const prevCount = useRef()

  function handleClick() {
    //用.current去接收值，修改current不会触发页面更新
    prevCount.current = count
    setCount(count + 1)
  }
```

- **引用Dom，使用Dom自带事件**

```plain
const inputRef = useRef(null)

  function handleClick() {
    inputRef.current.focus()
  }

  <input type="text" ref={inputRef} />
```

- **引用自定义组件+使用内部方法**

  - `可用于初始化第三方库实例（非 React 组件）->初始化一次实例，后续复用`

  - `->用相同值调用实例方法无副作用，且库自身会在组件卸载时清理资源`

```plain
import { useRef, forwardRef, useImperativeHandle } from 'react'

//引用子组件时，组件的定义需要通过const这种函数表达式来定义
//使用forwardRef将匿名组件函数整个包起来，子组件要用ref形参去接收父组件传进来的ref
const Child = forwardRef(function (props, ref) {
  useImperativeHandle(ref, () => ({
    // 暴露给父组件的方法和属性要通过useImperativeHandle
    myFn: () => {
      console.log('子组件myFn方法')
    }
  }))
  return ...
})

export default function App() {
  const childRef = useRef()

  function handleClick() {
    childRef.current.myFn()
  }

  return (
    <div>
      <Child ref={childRef} />
      <button onClick={handleClick}>按钮</button>
    </div>
  )
}
```

- **不要在渲染期间写入或者读取ref.current除了初始化ref**

```plain
function MyComponent() {

  // 🚩 不要在渲染期间写入 ref
  myRef.current = 123;
  // 🚩 不要在渲染期间读取 ref
  return <h1>{myOtherRef.current}</h1>;

   useEffect(() => {
    // ✅ 可以在 Effect 中读取和写入 ref
    myRef.current = 123;
  });
  function handleClick() {
    // ✅ 可以在事件处理程序中读取和写入 ref
    doSomething(myOtherRef.current);
  }
}
```

##### `useEffect`

**->希望在组件加载或更新（非用户操作）的时候执行一些副作用**

**启动同步仅执行setup->读取的值一变就先停止旧同步并清理→再启动新同步->组件卸载时清理函数执行，彻底停止同步**

**需要与网络(websocket)、某些浏览器 API(持续获取用户位置如地图导航)或第三方库(如 Firebase 实时数据库)保持连接等操作可以写在useEffect中**

```plain
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId);
    connection.connect();
  	return () => {
      connection.disconnect();
  	};
    //设置依赖项数组,和副作用函数相关联状态
    //Effect代码中使用的每个响应式的值都必须声明为依赖项<-Effect依赖列表是由周围代码决定的
   //任意一项改变 → 旧连接先断开 → 新连接再建立
  }, [serverUrl, roomId]);
  // ...
}
```

- **在组件渲染的时候执行一次以后不再变更**

  + 页面初始化加载不随用户交互变化的数据(如商品分类列表、地区选项、系统配置）

  + 监听不随组件状态变化的全局事件(如页面滚动、窗口大小变化、键盘快捷键)，只需在组件挂载时添加一次监听，卸载时清理

  + 初始化订阅第三方服务的消息(如 WebSocket 连接、推送通知)，后续保持连接


```plain
useEffect(() => {
  // setup：添加监听
  const handleScroll = () => console.log('滚动了');
  window.addEventListener('scroll', handleScroll);

  // cleanup：移除监听（此时不执行，卸载时才执行）
  return () => {
    window.removeEventListener('scroll', handleScroll);
  };
}, []); // 依赖为空，只执行一次
```

**陷阱**

- 没有依赖数组的 Effect 会在每次渲染后运行,下面的代码会陷入死循环


```plain
const [count, setCount] = useState(0);
useEffect(() => {
//🚩不应该使用 Effect 来转换渲染所需的数据
set函数->重新渲染->useEffect->set函数
  setCount(count + 1);
});
```

- 在 Effect 中根据先前 state 更新 state


```plain
function Counter() {
  const [count, setCount] = useState(0);
  
 //🚩count是依赖项,每次 count 更改时再次执行 cleanup 和 setup
  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); 
    }, 1000)
    return () => clearInterval(intervalId);
  }, [count]); 
  
  // ✅使用 c => c + 1 状态更新器传递值给 setCount
  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1); //
    }, 1000);
    return () => clearInterval(intervalId);
  }, []); // ✅现在 count 不是一个依赖项
```

混淆 “副作用” 和 “用户主动触发的事件”

```plain
function ProductPage({ product, addToCart }) {
  // 🔴 避免：在 Effect 中处理属于事件特定的逻辑
  //在两个按钮的 click 事件处理函数中都调用showNotification()感觉有点重复
  //所以你可能想把这个逻辑放在一个 Effect 中
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`已添加 ${product.name} 进购物车！`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
 
  // ✅事件特定的逻辑在事件处理函数中处理
  function buyProduct() {
    addToCart(product);
    showNotification(`已添加 ${product.name} 进购物车！`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
}
```

当 props 变化时重置state

```plain
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');
  // 🚩避免：当 prop 变化时，在 Effect 中重置 state
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}

export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      //✅使用key告诉React它们是不同的个人资料组件
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ 当 key 变化时，该组件内的 comment 或其他 state 会自动被重置
  const [comment, setComment] = useState('');
  // ...
}
```

##### `useMemo `

- **->useMemo缓存计算的结果,类似computed**

```plain
function TodoList({ todos, tab, theme }) {
  //🚩每次渲染都重新计算：没有依赖数组
  const visibleTodos = useMemo(() => filterTodos(todos, tab));
  //✅确保你已将依赖项数组指定为第二个参数
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

- **useMomo防止过于频繁地触发 Effect** 

```plain
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // 🚩每次渲染options(响应式值)依赖都会发生改变->不断地重新连接到聊天室
 const options = {
    serverUrl: 'https://localhost:1234',
    roomId: roomId
  }

  const options = useMemo(() => 
  // 🚩() => { 是箭头函数体的开始标志,因此 { 大括号不是对象的一部分
    () => {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    }
    //✅显式编写 return 语句
    () => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ 只有当 roomId 改变时才会被改变

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ 只有当 options 改变时才会被改变
```

##### `useCallback`

------**useCallback只用于性能优化**

**useCallback缓存函数引用（配合`memo`避免子组件不必要渲染）<= 默认情况下，当一个组件重新渲染时，React 会递归地重新渲染它的所有子组件**

```plain
import { useState, memo, useCallback } from 'react';
//通过const这种函数表达式来定义，使用react本身的memo方法缓存组件函数->
//如果向子组件传入的prop没有发生变化的话，子组件就不会受到父组件更新的影响
const Button = memo(function ({ onClick }) {
  console.log('Button渲染了');
  return <button onClick={onClick}>子组件</button>;
});

function App() {
  const [count, setCount] = useState(0);
  
  //JS中，function () {} 或者 () => {} 总是会生成不同的函数
  //每次父组件重新渲染,handleClick函数会重新创建->函数对应的地址改变
  //使用useCallback钩子对函数进行缓存,确保多次重新渲染总是相同的函数,直到依赖发生改变
  const handleClick = useCallback(() => {
    console.log('点击按钮');
  }, []);

  // 记忆化回调：仅在依赖项变化时更新引用
  const handleUpdate = useCallback(() => {
    // 在记忆化回调中更新 state不会重新触发渲染
    setCount(prev => prev + 1); 
  }, []); // 空依赖：回调引用始终不变
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleUpdate}>点击</button>
      ...
    </div>
  );
}
```

- **useCallback优化自定义 Hook**

```plain
 const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);//如果你忘记使用依赖数组,useCallback每一次都将返回一个新的函数

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);
  
  //将自定义 Hook返回的任何函数都包裹在 useCallback 中
   return {
    navigate,
    goBack,
  };
```

##### `useContext`

**跨组件传递数据，避免 props 层层透传**

- **每当 `App`出现重新渲染(如路由更新), React 会重新渲染树中调用`useContext` 的所有组件**

  - **->如果基础数据如`user` 没有更改，则不需要重新渲染它们**

  - **->使用 `useCallback`包装 `login` 函数，并将对象创建包装到`useMemo`**

```plain
import { createContext, useContext, useState, useCallback, useMemo } from 'react';

//创建 Context容器(默认值设为 null,无 Provider 时返回此值)
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState({
    name: '张三',
    role: '普通用户',
    isLogin: true
  });

  // 记忆化登录方法，避免频繁创建新函数
  const login = useCallback((response) => {
    storeCredentials(response.credentials); 
    setUser(response.user); 
  }, []); // 空依赖：方法引用稳定

  // 记忆化Context值，仅在依赖变化时更新
  const contextValue = useMemo(() => ({
    user, 
    login, 
    updateUserRole: (newRole) => { 
      setUser(prev => ({ ...prev, role: newRole }));
    }
  }), [user]); //仅当user变化时,contextValue才更新

  return (
    //Provider 会将 value 注入到其包裹的所有子组件中,子组件可通过 useContext 直接读取
    <UserContext.Provider value={contextValue}>
        <UserInfo />
        <UserAction />
    </UserContext.Provider>
  );
}

function UserAction() {
  //返回的值始终是最新的:context变化->React 会自动重新渲染读取 context 的组件
  const { user, updateUserRole } = useContext(UserContext);

  return (
      <button onClick={() => updateUserRole('管理员')}>
        将 {user.name} 升级为管理员
      </button>
  );
}

```

- **覆盖组件树一部分的 context**

```plain
//useContext()总是在调用它的组件上面搜索(不包括调用组件本身)最近的 provider。
 <ThemeContext value="dark">
  ...
  <ThemeContext value="light">
    <Footer />
  </ThemeContext>
  ...
</ThemeContext>
```

#### 7.通信方式 

- **父传子->props** 

```plain
//可用{title='默认标题',content}对props进行解构同时设置默认参数值
function Article(props){
  return (
    <div>
      <h2>{props.title}</h2>
      <p>{props.content}</p>
    </div>
)}

export default function App(){
  return(
    //可以把内容提到对象里，使用展开语法{...对象名}传递
    <Article	title='标题1' content='内容1'/>
 )} 
```

- **将 JSX 作为 props 传递给子组件，类似于插槽**

```plain
function List({ children }) {
  return <ul>{children}</ul>;
}
...传递DOM
<List>
    <li>列表项1</li>
</List>

function Button({ children }) {
  return <button>{children}</ul>;
}
...传递text
<Button>我是列表1</Button>
```

- **子传父->先通过props传递父组件的函数，子组件在合适的时候调用函数并传递状态回父组件**

```plain
import { useState } from 'react';

function Detail({ onActive }) {
  const [status, setStatus] = useState(false);

  function handleClick() {
    const next = !status;   // 先算新值
    setStatus(next);        // 更新自身状态
    //传递的状态是更新之前的<-react的state更新是异步的
    onActive(next);         // 把新值回传给父组件
  }

  return (....);
}

export default function App() {
  function handleActive(status) {
    console.log(status);   // 控制台依次打印 false → true → false ...
  }

  return (
    <div>
      <Detail onActive={handleActive} />
    </div>
  );
}
```

-  **多层级传递->useContext**  

- **父组件直接操作子组件->useRef**
- **使用Redux / Zustand轻量外部 store** 
- **localStorage + 自定义 hook**

