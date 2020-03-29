#### 高阶组件1

传入一个组件，返回另外一个组件，另外这个组件包裹着传入的组件

高阶函数两种形式：属性代理、反向继承1

@符装饰器

`class WrapperHello extends ...`

```javascript
@WrapperHello
class Hello extends ...
```

把Hello组件作为参数传入WrapperHello组件中，返回一个新的组件

#### redux中的compose，（中间件的机制）

```javascript
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }
  if (funcs.length === 1) {
    return funcs[0]
  }
  return funcs.reduce((a,b) => (...args) => a(b(...args)))
}

//例子
const f = (arg) => `f(${arg})`
const g = (arg) => `g(${arg})`
const h = (...args) => `h(${args.join('_')})`
console.log(compose(f,g,h)('a','b','c'))   // f(g(h(a_b_c)))  string
```

|            | a的值                      | b的值 | 结果                          |
| ---------- | -------------------------- | ----- | ----------------------------- |
| 第一次循环 | `f`                        | `g`   | `(...args)=>f(g(...args))`    |
| 第二次循环 | `(...args)=>f(g(...args))` | `h`   | `(...args)=>f(g(h(...args)))` |

```js
a的值变为 (...args) => f(g(...args)) 时，对应这个地方：(...args) => a(b(...args))
b(...args)相当于a函数的参数
```



所有compose多个中间件的机制就是一层一层的传递下去，像洋葱一样一层一层的剥开

#### 中间件的实现机制

```javascript
//createStore(reducer),调用后返回store,而使用中间件的作用就是增强store中的dispatch

const store = createStore(count,applyMiddleware(thunk))

//实现机制，执行applyMiddleware(thunk)方法后返回一个函数，将此函数作为参数传进createStore函数里面进行调用
export function createStore(reducer,enhancer){  //enhancer <=> applyMiddleware(thunk)
    if(enhancer){
        enhancer(createStore)(reducer)    //对createStore进行了一次拓展
    }
}

export function applyMiddleware(middleware){
    return createStore => (...args) => {    //此处的...args对应 reducer
        const store = createStore(...args)
        let dispatch = store.dispatch
        
        const midApi = {
            getState: store.getState,
            dispatch: (...args) => dispatch(...args)
        }
        //把原来的getState和dispatch传到middleware内部进行一次拓展，返回新的dispatch
        dispatch = middleware(midApi)(store.dispatch)   //比如thunk在此处进行调用，在applyMiddleware方法里调用
        //多个中间件
        //const middlewareChain = middlewares.map(v => v(midApi))
        //dispatch = compose(...middlewareChain)(store.dispatch)   相当于一层一层的dispatch出去，f(g(h(sotre.dispatch)))
        return {
            ...store,
            dispatch     //其实就是通过中间件包装一下dispatch，增强了一下dispatch
        }
    }
}

//chunk中间件  
const thunk = ({dispatch,getState}) => next => action => {
    if(typeof action === 'function'){   
//比如 dispatch(addGunSync())，addGunSync() 返回一个函数，然后就将返回的函数执行一下，就会调用dispatch(addGun())
       return action(dispatch,getState)
    }
    //如果不满足条件，直接调用下一个中间件，使用next；调用的时候，next参数接收的是原来未经处理的dispatch
    //如果满足条件，需要重新dispatch,调用dispatch即可
    return next(action)
}


```

#### actionCreator需要用到异步的情况

1、ajax请求数据

2、需要用到state里的数据

```jsx
export function toggleHighSpeed(){
    return (dispatch,getState) => {
        const { highSpeed } = getState()
        dispatch({type: SET_HIGH_SPEED, payload: !highSpeed})
    }
}
```



3、一个actionCreator里需要做两个action

```jsx
export function showCitySelect(current){
    return dispatch => {
        dispatch({type: XXX, payload: true})
        dispatch({type: yyy, payload: current})
    }
}
```



#### 错误捕获

```jsx
class App extends React.Component{
    state = {
        hasError: false
    }
	/*componentDidCatch(){
        this.setState({
            hasError: true
        })
	}*/     //两种写法都可以
    static getDerivedStateFromError(){
        return {hasError: true}
    }
}

```

#### react-redux中connect的实现原理

```jsx
//两层箭头函数，就是调用函数时还会返回一个函数
export const connect (mapStateToProps = state => state, mapDispatchToProps = {}) => (WrapComponent) => {
    return Class connectComponent extends React.Component {
        //然后会执行mapStateToProps函数
        //将mapDispatchToProps中的actionCreator使用bindActionCreators方法，将每个actionCreator包装成形如：（...args） => dispatch(creator(...args))
        render () {
            return <WrapComponent {...this.state.props}></WrapComponent>
        }
    }
} 

//connect(mapStateToPros, mapDispatchToProps)(App) 后，返回一个组件，如上connectComponent组件，当这个组件被调用的时候，就会执行render函数，因为render函数返回App组件的调用，然后就会调用App组件，所以在App组件中用this.props就可以拿到对应的值
```

#### Provider的原理

将store放到context中，让子组件可以访问到store

调用createStore(reducer)后返回的对象中有dispatch方法，subscribe方法、getState方法

#### redux的实现原理

```js
function createStore(reducer){
    let currentState = {}
    let currentListeners = []
    function getState () {
        return currentState
    }
    function subscribe (listener) {
        currentListeners.push(listener)
    }
    function dispatch (action) {
        currentState = reducer(currentState, action)
        currentListeners.forEach(v => v())
        return action
    }
    return { getState, subscribe, dispatch }
}
```

