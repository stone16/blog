---
title: React初探
date: 2020-01-27 21:58:30
categories: FrontEnd
tags:
    - React 
	- FrontEnd
top:
---
初探React,很喜欢Component这种方式，很大程度提高了复用性，如果抛除C/S的区别，感觉有点像mason，毕竟刚刚弃掉mason的坑，很有意思的React。


# 1. Hello World

React.Component   A component takes in parameters, called props and returns a hierarchy of views to display via the render method. 

To collect data from multiple children, or to have two child components communicate with each other, you need to declare the shared state in their parent component instead. **The parent component can pass the state back down to the children by using props; this keeps the child components in sync with each other and with the parent component.** 

    ReactDOM.render(
        <h1>Hello, world!</h1>,
        document.getElementById('root')
    );
    
# 2. JSX

jsx, 一种JavaScript的语法扩展。用来声明React当中的元素。可以任意在**大括号{}**里面使用**JS表达式**.

## 2.1 JS表达式

> Any valid unit of code that resolves to a value. 

### 2.1.1 分类

+ Arithmetic

+ String 

+ Logical

+ Primary Expressions

Basic keywords and general expressions in JS.

1. this: refer to the current object.
2. grouping operator() : controls the precedence of evaluation in expressions. 
3. new: to create an instance of a user-defined object type 
4. super: call functions on an object's parent.
5. spread operator: allow an expression to be expanded in places where multiple arguments or multiple elements are expected. 

         function f(x, y, z) { }
        var args = [0, 1, 2];
        f(...args);


+ Left hand side expressions


## 2.2 JSX 属性

编译之后，会被转化为普通的JS对象。这意味着可以在if 或者for语句里使用JSX，将其赋值给变量，当做参数传入或者作为返回值都可以。

    // 使用引号定义以字符串为值得属性
    const element = <div tabIndex="0"></div>;
    // 使用大括号来定义以js表达式为值得属性
    const element = <img src={user.avatarUrl}></img>;
    
JSX代表Objects, Babel转译器会把JSX转换成一个名为React.createEliment()的方法来调用

## 2.3 嵌套与防注入攻击

    const element = (
      <div>
        <h1>Hello!</h1>
        <h2>Good to see you here.</h2>
      </div>
    );
    
    React DOM在渲染之前会过滤所有传入的值，可以确保应用不会被注入攻击，因为所有内容渲染之前都已经被转化为了字符串，有效防止XSS。 
    
# 3. 元素渲染
React中的元素实际上是普通的对象，React DOM可以确保浏览器DOM的数据内容与React元素保持一致。
寻找React 根节点，渲染在根节点上

    const element = <h1>Hello, world</h1>;
    ReactDOM.render(element, document.getElementById('root'));

## 3.1 更新元素渲染

React 元素都是immutable的，更新界面的方式就是创建一个新的元素，然后将其传入`ReactDOM.render()` 

React DOM 会比较元素的内容的先后的不同，而在渲染过程中只会更新改变了的部分。

# 4. 组件 & props
组件将UI切分成一些独立的，可复用的部件，这样就可以专注于构建每一个单独的部件。概念上像**函数**一样，可以接受任意的输入值(props)，并返回一个在页面上展示的React元素。

## 4.1 函数定义组件

    function Welcome(props) {
        return <h1>Hello, {props.name}</h1>;
    }
    
## 4.2 ES6 class 定义组件

    class Welcome extends React.Component {
        render() {
            return <h1>Hello, {this.props.name}</h1>
        }
    }
    
## 4.3 组件渲染

    // React 元素可以使用户自定义的组件
    const element = <Welcome name="Sara" />;
    
**当React遇到的元素是用户自定义的组件，它会将JSX属性作为单个对象传递给该组件，这个对象被称为"props"**

> 组件名称必须大写

## 4.4 组合组件

组件可以在它的输出中引用其他组件，这样我们就可以用同一组件来抽象出任意层次的细节。

> 一个新的React应用程序的顶部是一个App组件。但是，如果要将React集成到现有应用程序中，则可以从下而上使用像Button这样的小组件作为开始，并逐渐运用到视图层的顶部。

> 组件的返回值只能有一个根元素。这也是我们要用一个<div>来包裹所有<Welcome />元素的原因。

## 4.5 提取组件
分割组件，

## 4.6 Props的只读性

所有的React组件必须像纯函数那样使用它们的props

# 5. State & 生命周期

更新UI的方法： `ReactDOM.render()`

还可以通过更新状态来更新UI，**状态是私有的，完全受控于当前组件**

## 5.1 将函数转换为类
定义为类的组件有状态这个特性，还有生命周期钩子。

函数转换为类的步骤： 
1. 创建一个名称扩展为`React.Component`的类
2. 创建一个`render()`空方法
3. 将函数体移动到render()方法中
4. 在render()方法中，使用this.props替换props
5. 删除剩余的空函数声明

## 5.2 为类添加局部状态

    Class Clock extends React.Component {
        constructor(props) {
            super(props);
            this.state = {date: new Date()};
        }
        
        render() {
            return (
                <div>
                    <h1>Hello, world!</h1>
                    <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
                </div>
            );
        }
    }
    
    ReactDOM.render(
        <Clock/>
        document.getElementById('root')
    );
    
## 5.3 添加生命周期方法到类中

当组件第一次加载到DOM中时，生成定时器，挂载

    componentDidMount() {
        
    }

当Clock生成的这个DOM被移除时，清除定时器，卸载
    
    componentWillUnmount() {
        
    }
    
一个完整的Clock的例子： 
    
    
    class Clock extends React.Component {
        constructor(props) {
            super(props);
            this.state = {date: new Date()};
        }

        // 3. Called when Clock's output is injected into DOM 
        componentDidMount() {
            this.timerID = setInterval(
                () => this.tick(),
                    1000
            );
        }

        componentWillUnmount() {
            clearInterval(this.timerID);
        }
        
        // 4. when setState() is being called, render() is called 
        tick() {
            this.setState({
                date: new Date()
            });
        }

        // 2. Call render(), react know what need to be shown on screen. Update DOM 
        render() {
            return (
              <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
              </div>
            );
        }
    }

    ReactDOM.render(
    // 1. call Clock's constructor
      <Clock />,
      document.getElementById('root')
    );
    
## 5.4 如何使用状态

1. 不要直接更新状态

    use: this.setState({comment: 'hi'});
    
> 构造函数是唯一能够初始化this.state的地方

2. 状态更新可能是异步的

React可以将多个`setState()`调用合并成一个来提高性能

    // Wrong
    this.setState({
      counter: this.state.counter + this.props.increment,
    });
    
    // Correct
    this.setState((prevState, props) => ({
      counter: prevState.counter + props.increment
    }));
    
    // Correct
    this.setState(function(prevState, props) {
      return {
        counter: prevState.counter + props.increment
      };
    });


3. 当调用`setState()`的时候，React会将你提供的对象合并到当前状态。可以只提供state的一部分。


## 5.5 数据流动方向： 自顶向下

父组件或子组件都不知道某个组件是否有状态，组件可以选择将其状态作为属性传递给其子组件。

# 6. 事件处理

React事件绑定属性的命名采用驼峰式写法

采用jsx的语法你需要传入一个函数作为事件处理函数，而不是一个字符串。

    <button onClick={activateLasers}>
        Activate Lasers
    </button>

## 6.1 Toggle 

    class Toggle extends React.Component {
        constructor(props) {
        super(props);
        this.state = {isToggleOn: true};

        // This binding is necessary to make `this` work in the callback
        this.handleClick = this.handleClick.bind(this);
        }

        handleClick() {
            this.setState(prevState => ({
                isToggleOn: !prevState.isToggleOn
            }));
        }

        render() {
            return (
            // <button onClick={(e) => this.handleClick(e)}> 
            // 问题L每次渲染的时候都会创建一个不同的回调函数
                <button onClick={this.handleClick}>
                    {this.state.isToggleOn ? 'ON' : 'OFF'}
                </button>
            );
        }
    }

    ReactDOM.render(
        <Toggle />,
        document.getElementById('root')
    );
    
必须谨慎对待JSX回调函数中的this，类的方法默认不会绑定this的。如果你忘记绑定 `this.handleClick` 并把它传入 `onClick`, 当你调用这个函数的时候 `this` 的值会是 `undefined`。
## 6.2 Todolist

    class TodoApp extends React.Component {
        constructor(props) {
            super(props);
            this.state = { items: [], text: '' };
            this.handleChange = this.handleChange.bind(this);
            this.handleSubmit = this.handleSubmit.bind(this);
        }

        render() {
            return (
                <div>
                    <h3>TODO</h3>
                    <TodoList items={this.state.items} />
                    <form onSubmit={this.handleSubmit}>
                      <input
                        onChange={this.handleChange}
                        value={this.state.text}
                      />
                      <button>
                        Add #{this.state.items.length + 1}
                      </button>
                    </form>
                </div>
            );
        }

        handleChange(e) {
            this.setState({ text: e.target.value });
        }
            
        handleSubmit(e) {
            e.preventDefault();
            if (!this.state.text.length) {
                return;
            }
            const newItem = {
                text: this.state.text,
                id: Date.now()
            };
            this.setState(prevState => ({
                items: prevState.items.concat(newItem),
                text: ''
            }));
        }
    }
        
    class TodoList extends React.Component {
        render() {
            return (
                <ul>
                    {this.props.items.map(item => (
                      <li key={item.id}>{item.text}</li>
                    ))}
                </ul>
            );
        }
    }
        
    ReactDOM.render(<TodoApp />, mountNode);
    
## 6.3 向事件处理程序传递参数

    <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
    <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
    
参数 e 作为 React 事件对象将会被作为第二个参数进行传递。通过箭头函数的方式，事件对象必须显式的进行传递，但是通过 bind 的方式，事件对象以及更多的参数将会被隐式的进行传递。


## 6.4 bind 向监听函数传参， 事件对象e需要放在最后

    class Popper extends React.Component{
        constructor(){
            super();
            this.state = {name:'Hello world!'};
        }
        
        preventPop(name, e){    //事件对象e要放在最后
            e.preventDefault();
            alert(name);
        }
        
        render(){
            return (
                <div>
                    <p>hello</p>
                    {/* Pass params via bind() method. */}
                    <a href="https://reactjs.org" onClick={this.preventPop.bind(this,this.state.name)}>Click</a>
                </div>
            );
        }
    }
        
# 7. 条件渲染
可以创建不同的组件来封装各种你需要的行为。然后根据应用的状态变化只渲染其中的一部分。(if)

    function Greeting(props) {
        const isLoggedIn = props.isLoggedIn;
        if (isLoggedIn) {
            return <UserGreeting />;
        }
        return <GuestGreeting />;
    }

    ReactDOM.render(
      // Try changing to isLoggedIn={true}:
      <Greeting isLoggedIn={false} />,
      document.getElementById('root')
    );
    
    
## 7.1 与运算符 && 
    
    function Mailbox(props) {
        const unreadMessages = props.unreadMessages;
        return (
            <div>
              <h1>Hello!</h1>
              {unreadMessages.length > 0 &&
                <h2>
                  You have {unreadMessages.length} unread messages.
                </h2>
              }
            </div>
        );
    }

    const messages = ['React', 'Re: React', 'Re:Re: React'];
    ReactDOM.render(
      <Mailbox unreadMessages={messages} />,
      document.getElementById('root')
    );
    
**在 JavaScript 中，true && expression 总是返回 expression，而 false && expression 总是返回 false。**

## 7.2 阻止组件渲染

    function WarningBanner(props) {
        if (!props.warn) {
            return null;
        }

        return (
            <div className="warning">
                Warning!
            </div>
        );
    }

    class Page extends React.Component {
        constructor(props) {
            super(props);
            this.state = {showWarning: true}
            this.handleToggleClick = this.handleToggleClick.bind(this);
        }
    
        handleToggleClick() {
            this.setState(prevState => ({
                showWarning: !prevState.showWarning
            }));
        }
    
        render() {
            return (
                <div>
                    <WarningBanner warn={this.state.showWarning} />
                    <button onClick={this.handleToggleClick}>
                    {this.state.showWarning ? 'Hide' : 'Show'}
                    </button>
                </div>
            );
        }
    }
    
    ReactDOM.render(
      <Page />,
      document.getElementById('root')
    );
    

# 8. 列表 & Keys

## 8.1 渲染多个组件

    const numbers = [1, 2, 3, 4, 5];
    const listItems = numbers.map(
        (number) => <li>{number}</li>
    );
    
    ReactDOM.render(
        <ul>{listItems}</ul>
        documnet.getElementById('root')
    );
    
## 8.2 基础列表组件

    function NumberList(props) {
        const numbers = props.numbers;
        const listItems = numbers.map((number) =>
            <li key={number.toString()}>
                {number}
            </li>
        );
        return (
            <ul>{listItems}</ul>
        );
    }

    const numbers = [1, 2, 3, 4, 5];
    ReactDOM.render(
      <NumberList numbers={numbers} />,
      document.getElementById('root')
    );

## 8.3 Keys

Keys可以在DOM中的某些元素被增加或删除的时候帮助React识别哪些元素发生了变化。最好是该元素在列表中拥有的独一无二的字符串。使用来自数据的id作为元素的key

元素的key只有在它和它的兄弟节点对比时才有意义。

# 9.表单

HTML 表单元素与React中其他DOM元素有所不同，因为表单元素本来就保留一些内部状态了。会构造一个处理提交表单并可访问用户输入表单数据的函数。标准方法是使用受控组件。

## 9.1 受控组件

在HTML当中，像`<input>,<textarea>, 和 <select>`这类表单元素会维持自身状态，并根据用户输入进行更新。但在React中，可变的状态通常保存在组件的状态属性中，并且只能用 setState() 方法进行更新。

    class NameForm extends React.Component {
        constructor(props) {
            super(props);
            this.state = {value: ''};
    
            this.handleChange = this.handleChange.bind(this);
            this.handleSubmit = this.handleSubmit.bind(this);
        }

        handleChange(event) {
            this.setState({value: event.target.value});
        }

        handleSubmit(event) {
            alert('A name was submitted: ' + this.state.value);
            event.preventDefault();
        }

        render() {
            return (
                <form onSubmit={this.handleSubmit}>
                    <label>
                      Name:
                      <input type="text" value={this.state.value} onChange={this.handleChange} />
                    </label>
                    <input type="submit" value="Submit" />
                </form>
            );
        }
    }
    

## 9.2 textarea标签

    class EssayForm extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
              value: 'Please write an essay about your favorite DOM element.'
            };
    
            this.handleChange = this.handleChange.bind(this);
            this.handleSubmit = this.handleSubmit.bind(this);
        }
    
        handleChange(event) {
            this.setState({value: event.target.value});
        }
    
        handleSubmit(event) {
            alert('An essay was submitted: ' + this.state.value);
            event.preventDefault();
        }
    
        render() {
            return (
                <form onSubmit={this.handleSubmit}>
                    <label>
                    Name:
                    <textarea value={this.state.value} onChange={this.handleChange} />
                    </label>
                    <input type="submit" value="Submit" />
                </form>
            );
        }
    }
    
## 9.3 select标签

React中，不适用selected属性表明选中项，而是在根select标签上用value属性来表示选中项。这在受控组件中更方便，因为只需要在一个地方更新组件。

    class FlavorForm extends React.Component {
        constructor(props) {
        super(props);
        this.state = {value: 'coconut'};

        this.handleChange = this.handleChange.bind(this);
        this.handleSubmit = this.handleSubmit.bind(this);
    }

    handleChange(event) {
        this.setState({value: event.target.value});
    }

    handleSubmit(event) {
        alert('Your favorite flavor is: ' + this.state.value);
        event.preventDefault();
    }

        render() {
            return (
                <form onSubmit={this.handleSubmit}>
                    <label>
                        Pick your favorite La Croix flavor:
                        <select value={this.state.value} onChange={this.handleChange}>
                        <option value="grapefruit">Grapefruit</option>
                        <option value="lime">Lime</option>
                        <option value="coconut">Coconut</option>
                        <option value="mango">Mango</option>
                        </select>
                    </label>
                    <input type="submit" value="Submit" />
                </form>
            );
        }
    }
    
## 9.4 多个输入的解决方法

通过给每个元素添加一个name属性，来让处理函数根据event.target.name的值来选择做什么

    class Reservation extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
                isGoing: true,
                numberOfGuests: 2
            };
    
            this.handleInputChange = this.handleInputChange.bind(this);
        }
    
        handleInputChange(event) {
            const target = event.target;
            const value = target.type === 'checkbox' ? target.checked : target.value;
            const name = target.name;
    
            this.setState({
                [name]: value
            });
        }
    
        render() {
            return (
                <form>
                    <label>
                        Is going:
                        <input
                            name="isGoing"
                            type="checkbox"
                            checked={this.state.isGoing}
                            onChange={this.handleInputChange} />
                    </label>
                    <br />
                    <label>
                        Number of guests:
                        <input
                            name="numberOfGuests"
                            type="number"
                            value={this.state.numberOfGuests}
                            onChange={this.handleInputChange} />
                    </label>
                </form>
            );
        }
    }


# 10. 状态提升

## 10.1 摄氏度华氏度的例子

状态分享是通过将state数据提升至离需要这些数据的组件最近的父组件来完成的

    class Calculator extends React.Component {
        constructor(props) {
            super(props);
            this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
            this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
            this.state = {temperature: '', scale: 'c'};
        }

        handleCelsiusChange(temperature) {
            this.setState({scale: 'c', temperature});
        }

        handleFahrenheitChange(temperature) {
            this.setState({scale: 'f', temperature});
        }

        render() {
            const scale = this.state.scale;
            const temperature = this.state.temperature;
            const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
            const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

            return (
              <div>
                <TemperatureInput
                  scale="c"
                  temperature={celsius}
                  onTemperatureChange={this.handleCelsiusChange} />
        
                <TemperatureInput
                  scale="f"
                  temperature={fahrenheit}
                  onTemperatureChange={this.handleFahrenheitChange} />
        
                <BoilingVerdict
                  celsius={parseFloat(celsius)} />
        
              </div>
            );
        }
    }