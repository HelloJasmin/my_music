#### React 新特性

1. Context

   ```jsx
   //context相当于全局变量，可以直接将值传给所有后代，context不应该大范围使用
   import React,{createContext} from 'react'
   const BatteryContext = createContext()
   const OnlineContext = createContext()
   function Middle(){
     return (<Leaf/>)
   }
   function Leaf(){
     return (
       <BatteryContext.Consumer>
         {
           battery => (
             <OnlineContext.Consumer>
               {
                 online => <h1>Battery: {battery},Online:{String(online)}</h1>
               }
             </OnlineContext.Consumer>
           )
         }
       </BatteryContext.Consumer>
     )
   }
   
   export default function Example4(){
     return(
       <BatteryContext.Provider value={60}>
         <OnlineContext.Provider value={true}>
           <Middle/>
         </OnlineContext.Provider>
       </BatteryContext.Provider>
     )
   }
   
   ```

   

2. ContextType

   ```jsx
   //用了contextType不需要使用<Consumer>
   import React,{createContext} from 'react'
   const BatteryContext = createContext()
   function Middle(){
     return (<Leaf/>)
   }
   class Leaf extends React.Component {
     static contextType = BatteryContext
     render() {
       const battery = this.context
       return (
         <h1>{battery}</h1>
       )
     }
   }
   export default function Example4(){
     return(
       <BatteryContext.Provider value={60}>
           <Middle/>
       </BatteryContext.Provider>
     )
   }
   ```

   

3. lazy

   ```jsx
   //没有使用到就不会导入（加载）,要配和Suspense使用
   import React,{lazy,Suspense} from 'react'
   const About = lazy(() => import(/*webpackChunkName:"about"*/'./About'))
   class Example4 extends React.Component {
     constructor(props) {
       super(props);
       this.state = {  }
     }
     
     render() { 
       return ( 
         <Suspense fallback={<div>loading</div>}>
           <About></About>
         </Suspense>
       );
     }
   }
    
   export default Example4;
   ```

   

4. Suspense

5. memo

   ```jsx
   //相当于shouldComponentUpdate,用于无状态组件性能优化
   const Foo = memo(function Foo(props){
       return <div>{props.count}</div>
   })
   ```

#### 类组件的不足

1. 状态逻辑复用难；缺少复用逻辑，渲染属性和高阶组件导致层级冗余

2. 趋向复杂难以维护

   生命周期函数混杂不相干逻辑

   相干逻辑分散在不同生命周期中，比如组件挂载的时候增加定时器，卸载的时候要取消定时器

3. this指向困扰

   内联函数过度创建新句柄

   类成员函数不能保证this

#### hooks的优势

1. 函数组件无this问题
2. 自定义hook方便复用状态逻辑
3. 副作用的关注点分离。（可以一个功能用一个useEffect函数，互不影响，关注点分离）

#### useState

1. useState怎么知道返回的是当前组件的count而不是其他组件的count?       因为js是单线程

2. 如果一个组件有多个state,那么useState怎么知道哪一次调用返回哪一个state呢？   按照第一次运行的次序来顺序返回的

   ```jsx
   function App(props){
   	const [count,setCount] = useState(() => {
   		return props.defaultCount || 0      //只会在组件初始化的时候执行一次，优化性能
   	})
   }
   ```

#### useEffect

```jsx
useEffect(() => {
    document.title = count
})

useEffect(() => {
    window.addEventListener('resize',onResize)
	return () => {
		window.removeEventListener('resize',onResize)
	}
},[])    //useEffect传入第二个参数 []，当数组里的每一项都不变时，这个副作用就只在初始化的时候执行一次
```

#### useContext

 可以帮助我们跨越组件层级直接传递变量，实现共享 

 关于`Context`还有一个比较重要的点是：当Context Provider的value发生变化是，他的所有子级消费者都会rerender 

```jsx

// 第一步：创建需要共享的context
const ThemeContext = React.createContext('light');
 
class App extends React.Component {
  render() {
    // 第二步：使用 Provider 提供 ThemeContext 的值，Provider所包含的子树都可以直接访问ThemeContext的值
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}
// Toolbar 组件并不需要透传 ThemeContext
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}
 
function ThemedButton(props) {
  // 第三步：使用共享 Context
  const theme = useContext('ThemeContext');
  render() {
    return <Button theme={theme} />;
  }
```

#### useMemo和useCallback

useMemo实在渲染期间执行的

```jsx
const double = useMemo(()=>{return count * 2},[count])   //只要第二个参数数组里有一项发生了改变就会执行，如果是空数组就只执行一次, 且useMemo函数是有返回值的
```

```jsx
useMemo(() => fn) 等价于 useCallback(fn)
使用useCallback不能阻止创建新的函数，很可能被创建的函数被直接抛弃不用了，解决过度渲染的问题
```

```jsx
const onClick = useCallback(() => {
	console.log("click")
	setClickCount(clickCount + 1)
},[clickCount])   //这里不需要[clickCount,setClickCount]这样写，因为react能够保证每次渲染时返回的setState函数是同一个句柄
或者
const onClick = useCallback(() => {
    setClickCount(clickCount => clickCount + 1)
},[])
```

#### useRef

1. 获取子组件或者dom节点的句柄

2. 渲染周期之间共享数据的存储

   如果需要用到上一次渲染的数据，就将数据存到useRef中，下一次渲染就能用上一次的数据了

   跨越组件渲染周期并且不会引发重渲染
   
   用于用来存储任何数据
   
   ```jsx
   let it = useRef()
   useEffect(() => {
       it.current = setInterval(() => {
           setCount(count => count + 1)
       },1000)
   },[])
   useEffect(() => {
       clearInterval(it.current)
   })
   //可以设置默认值
   const prevCurrentState = userRef(currentState);
   ```
   

#### useReducer

```jsx

    // 定义初始化值
    const initState = {
        name: '',
        pwd: '',
        isLoading: false,
        error: '',
        isLoggedIn: false,
    }
    // 定义state[业务]处理逻辑 reducer函数
    function loginReducer(state, action) {
        switch(action.type) {
            case 'login':
                return {
                    ...state,
                    isLoading: true,
                    error: '',
                }
            case 'success':
                return {
                    ...state,
                    isLoggedIn: true,
                    isLoading: false,
                }
            case 'error':
                return {
                    ...state,
                    error: action.payload.error,
                    name: '',
                    pwd: '',
                    isLoading: false,
                }
            default: 
                return state;
        }
    }
    // 定义 context函数
    const LoginContext = React.createContext();
    function LoginPage() {
        const [state, dispatch] = useReducer(loginReducer, initState);
        const { name, pwd, isLoading, error, isLoggedIn } = state;
        const login = (event) => {
            event.preventDefault();
            dispatch({ type: 'login' });
            login({ name, pwd })
                .then(() => {
                    dispatch({ type: 'success' });
                })
                .catch((error) => {
                    dispatch({
                        type: 'error'
                        payload: { error: error.message }
                    });
                });
        }
        // 利用 context 共享dispatch，如果是子组件直接用props传值就行
        return ( 
            <LoginContext.Provider value={dispatch}>
                <...>
                <LoginButton />     
            </LoginContext.Provider>
        )
    }
    function LoginButton() {
        // 子组件中直接通过context拿到dispatch，触发reducer操作state
        const dispatch = useContext(LoginContext);
        const click = () => {
            if (error) {
                // 子组件可以直接 dispatch action
                dispatch({
                    type: 'error'
                    payload: { error: error.message }
                });
            }
        }

```

可以看到在useReducer结合useContext，通过context把dispatch函数提供给组件树中的所有组件使用 ，而不用通过props添加回调函数的方式一层层传递。

使用Context(或者说dispatch)相比回调函数的优势：

1. 对比回调函数的自定义命名，Context（或者说传dispatch）的Api更加明确，我们可以更清晰的知道哪些组件使用了dispatch、应用中的数据流动和变化。这也是React一直以来单向数据流的优势。
2. 更好的性能：如果使用回调函数作为参数传递的话，因为每次render函数都会变化，也会导致子组件rerender。当然我们可以使用[useCallback解决这个问题](https://link.juejin.im/?target=https%3A%2F%2Freactjs.org%2Fdocs%2Fhooks-faq.html%23how-to-read-an-often-changing-value-from-usecallback)，但相比`useCallback`React官方更推荐使用useReducer，因为React会保证dispatch始终是不变的，不会引起consumer组件的rerender。

总结：如果需要通过子组件去改变父组件的state,尽量在父组件通过useReducer，传递dispatch给子组件，然后通过dispatch去改变父组件的dispatch，比如去哪儿项目中的火车票综合筛选浮层组件中，浮层组件先维护一份综合筛选条件的state，子组件去toggle一个筛选条件需要改变浮层组件的state，那么就可以通过useReducer传递一个dispatch给子组件，通过dispatch(action)去改变父组件的state

因为在这个综合筛选浮层中，有两个地方改变checkedTrainTypes，一个是重置按钮，一个是toggle按钮时的更新操作，所以不利于统一管理checkedTrainTypes，追踪数据的变化，要通过useReducer统一管理起来，捕获到对指定数据的各种更新行为

+ useReducer 需要异步初始化，传入第三个参数

  ```jsx
  const [localCheckedTicketTypes, dispatch] = useReducer(chekedReducer,checkedTicketTypes,(checkedTicketTypes) => {
  	return ...checkedTicketTypes
  })
  ```

  

  

#### hooks重构去哪儿网

+ 对于具有纯粹属性输入的组件，一般可以使用memo进行性能优化，没有输入任何的props一定可以使用memo优化

+ 初始化样式

  ```js
  npm i normalize.css --save
  import 'normalize.css/normalize.css'
  ```

+ ```
  npm i classnames --save
  <div className={classnames('city-li',{hidden: !show})}></div>
  ```

+ svg的一个用法,

  ```jsx
  import switchImg from './imgs/switch.svg'
  <img src={switchImg} width="70" height="40" alt="switch"/>
  ```

+ ```jsx
  //属性选择器；scrollIntoView,定位到显示的位置
  document.querySelector(`[data-cate='${alpha}']`).scrollIntoView()
  ```

+ 日期函数工具库 

  ```js
  npm i dayjs --save
  dayjs(date).format('YYYY-MM-DD')
  //用法二
  import 'dayjs/locale/zh-cn'
  date.locale('zh-cn').format('ddd')  //得到周几
  ```

  

+ useMemo和useCallback的区别，useMemo返回缓存的变量，useCallback，返回缓存的函数

+ ```jxs
  days = new Array(startDay.getDay() ?  startDay.getDay() - 1 : 6 ).fill(null).concat(days)
  //fill、concat
  ```

+ ```js
  ['a','b'].includes('a')
  ```

+ ```js
  URL解析工具：npm i urijs --save
  imort URI from 'urijs'
  const queries = URI.parseQuery(window.loaction.search)
  ```

+ 父组件传下来的数据，子组件不能直接改，要不就通过调用父组件传过来的方法修改

+ ```js
  npm i left-pad --save   //左侧填充
  import leftPad form 'left-pad'
  leftPad(a,2,'0')    //如果a是3，则补充后变为03
  ```

+ 如果组件内的数据不需要马上提交到redux中，则先存在组件内的state中

+ 数据联动，只看高铁动车，与综合筛选浮层中选中的条件进行联动，在reducer函数里面进行联动，将highSpped的值对应于checkedTranTypes，如下代码

+ reducer函数中，如果操作一个对象，要选拷贝一个对象，不要直接操作state

  ```jsx
  function checkedTrainTypes(state={},action){
      const {type,payload} = action
      switch(type){
          case ACTION_SET_CHECKED_TRAN_TYPES:
              return payload
          case ACTION_SET_HIGH_SPPED:
              const highSpeed = payload
              const newCheckedTrain = {...state}
              if(highSpeed){
                  newCheckedTrain[1] = true
                  newCheckedTrain[5] = true
              }else{
                  delete newCheckedTrain[1]
                  delete newCheckedTrain[5]
              }
              return newCheckedTrain
          default:
              return state
      }
  }
  ```

+ 设置浏览器tab页的title，document.title = xx

+ 使用{props.children}，可以实现使用通用组件，里面有细微区别的地方，就通过{props.children}传入
+ PropTypes.oneOfType[PropTypes.string, PropTypes.number].isRequired   //两种中的一种复核就行

+ for循环遍历数组  for(let key of keys){}

+ Object.assign(target,source1,source2)

+ 不需要eslint检测的文件，要写在 .eslintignore中 如  `src/serviceWorker.js`

+ 所谓的hooks就是在组件的不同解读，来调用的内置函数

+ actionCreators里面一定不要直接使用原始数据，用一个新的对象接受原始数据，再对新的对象进行操作，否则原始数据改变了，造成监听不到数据改变的问题

####  forwardRef

> 直接将父组件创建的ref挂载到子组件的某个dom元素上 

 将ref父类的ref作为参数传入函数式组件中，本身props只带有children这个参数，这样可以让子类转发父类的ref,当父类把ref挂在到子组件上时，子组件外部通过forwrardRef包裹，可以直接将父组件创建的ref挂在到子组件的某个dom元素上 

```js
function InputWithLabel(props) {
  // 这里的myRef为通过外部打入的父级ref节点
  const { label, myRef } = props;
  const [value, setValue] = useState("");
  const handleChange = e => {
    const value = e.target.value;
    setValue(value);
  };

  return (
    <div>
      <span>{label}:</span>
      <input type="text" ref={myRef} value={value} onChange={handleChange} />
    </div>
  );
}

// 这里用forwardRef来承接得到父级传入的ref节点，并将其以参数的形式传给字节点
const RefInput = React.forwardRef((props, ref) => (
  <InputWithLabel {...props} myRef={ref} />
));

// 调用该RefInput的过程
function App() {
  // 通过useRef hook 获得相应的ref节点
  const myRef = useRef(null);

  const handleFocus = () => {
    const node = myRef.current;
    console.log(node);
    node.focus();
  };

  return (
    <div className="App">
      <RefInput label={"姓名"} ref={myRef} />
      <button onClick={handleFocus}>focus</button>
    </div>
  );
}
```

####  useImperativeHandle 

> 在函数式组件中，用于定义暴露给父组件的ref方法 

 用forwardRef获得的是整个节点，但是有时候我们通过ref只需要暴露一部分参数就行了，为了解决这个问题，我们就需要用到useImperativeHandle来指定放出一部分我们需要的方法或者属性给父级 

```js
/*1、子组件内部自建一个_innerRef来获取ref元素
2、将通过forwarfRef传入的ref元素通过useImperativeHandle来进行绑定，指定该子组件对外暴露的方法或属性
通
3、过_innerRef调用响应的方法然后同时在useImperativeHandle中写代码即可，这样可以只暴露一部分方法属性，而不是整个底层的input原生DOM节点*/
function InputWithLabel(props) {
  const { label, myRef } = props;
  const [value, setValue] = useState("");
  const _innerRef = useRef(null);
  const handleChange = e => {
    const value = e.target.value;
    setValue(value);
  };

  const getValue = () => {
    return value;
  };

  useImperativeHandle(myRef, () => ({
    getValue,
    focus() {
      const node = _innerRef.current;
      node.focus();
    }
  }));

  return (
    <div>
      <span>{label}:</span>
      <input
        type="text"
        ref={_innerRef}
        value={value}
        onChange={handleChange}
      />
    </div>
  );
}

const RefInput = React.forwardRef((props, ref) => (
  <InputWithLabel {...props} myRef={ref} />
));

function App() {
  const myRef = useRef(null);

  const handleFocus = () => {
    const node = myRef.current;
    console.log(node);
    node.focus();
  };

  return (
    <div className="App">
      <RefInput label={"姓名"} ref={myRef} />
      <button onClick={handleFocus}>focus</button>
    </div>
  );
}
```





 

 

 

 

 

 

 

 

 

 



