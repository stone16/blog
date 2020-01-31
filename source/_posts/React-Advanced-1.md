---
title: React Advanced(1)
date: 2020-01-30 23:12:13
categories: FrontEnd
tags:
    - FrontEnd
    - React
top:
---

For this part, aim to know all the necessary react knowledge to better do the development work. Start from the very beginning, and try to grab all the basic and advanced knowledge follow the authoritive doc. 

# 1. Main Concepts

## 1.1 JSX 

    const element = <h1>Hello World!</h1>

JSX, syntax extension to JS. JSX produces **React Elements**. And then we try to render them to the DOM. 

JSX out logic and markup together, react separates concerns with loosely coupled units called components that contain both. 

### 1.1.1 Embedding expressions in JSX 

    const name = 'Josh Perez';
    const element = <h1>Hello, {name}</h1>;
    
    ReactDOM.render(
      element,
      document.getElementById('root')
    );
    
Here, by wrapping name in curly braces, we call a variable. 

> Inside the curly brace, we can put any valid ***JS expression***.  

### 1.1.2 JSX is an Expression 

After compilation, JSX expressions become regular **JS function calls** and evaluate to **JS objects**. React DOM uses camelCase property naming convention 

JSX represents Object, Babel will help JSX to compile to React.createElement() call. 

## 1.2 Rendering Elements

### 1.2.1 Element 

An element describes what you want to see on the screen. 

    const element = <h1>Hello world!</h1>

Components are made of elements. 

> Applications built with React usually have a ***single root DOM node***. If you are integrating React into an existing app, you may have as many isolated root DOM nodes as you like. 

### 1.2.2 Updating the rendered element 

> ***React elements are immutable***

Once you create an element, you cannot change its children or attributes

### 1.2.3 Only updates what's necessary 

React DOM compares the element and its children to the previous one, and only applies the DOM updates necessary to bring the DOM to the desired state. 

> In this way, we can spend more time thinking how the UI should look like rather than how to change it over time eliminates a whole class of bugs. 

## 1.3 Components and Props 

### 1.3.1 Components

Components let you split the UI into independent, reusable pieces, and think about each piece in isolation. They accept arbitrary inputs(named props) and return React elements describing what should appear on the screen. 

When React sees an element representing a user-defined component, it **passes JSX attributes to this component** as a single object. We call this object “props”.

    function Welcome(props) {
      return <h1>Hello, {props.name}</h1>;
    }
    
    const element = <Welcome name="Sara" />;
    ReactDOM.render(
      element,
      document.getElementById('root')
    );
    
### 1.3.2 Extract Components

To make them better reusable. 

### 1.3.3 Props are ***Read only***

All React Components must act like pure functions with respect to their props. 

## 1.4 State and Lifecycle 

### 1.4.1 State 

State is private, and fully controlled by the component. 

### 1.4.2 Adding lifecycle methods to a Class 

In applications with many components, it's very important to free up resources taken by the components when they are destroyed. 

1. componentDidMount()
It runs after the component output has been rendered to the DOM

2. componentWillUnmount() 
3. Runs when the DOM need to be removed 

### 1.4.3 State Using tips

1. Do not modify state directly, instead, use setState
2. state updates may be asynchronous , props and state may be updated asynchronously, should not rely on their values for calculating the next state. 


    // Correct
    this.setState((state, props) => ({
      counter: state.counter + props.increment
    }));

In this way, That function will receive the previous state as the first argument, and the props at the time the update is applied as the second argument

3. State updates are merged 

### 1.4.4 The data flow down

This is commonly called a “top-down” or “unidirectional” data flow. Any state is always owned by some specific component, and any data or UI derived from that state can only affect components “below” them in the tree.

> If you imagine a component tree as a waterfall of props, each component’s state is like an additional water source that joins it at an arbitrary point but also flows down.

## 1.5 Handling Events 
Quite similar to handling methods in DOM, some differences: 
1. Syntax: 

    <button onClick={activateLasers}>
      Activate Lasers
    </button>

Use {}, `activateLasers` here means a function name

2. use `e.preventDefault()` to block the default behavior
3. when using React, you should generally not need to call `addEventListener` to add listeners to a DOM element after it is created. Instead, just provide a listener when the element is initially rendered. 


    class Toggle extends React.Component {
      constructor(props) {
        super(props);
        this.state = {isToggleOn: true};
    
        // This binding is necessary to make `this` work in the callback
        this.handleClick = this.handleClick.bind(this);
      }
    
      handleClick() {
        this.setState(state => ({
          isToggleOn: !state.isToggleOn
        }));
      }
    
      render() {
        return (
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
    
>  Be careful of `this`, since class methods are not bound by default. Need to bind this.handleClick. 

4. You can try to not bind this if you declare the method in such way!


    class LoggingButton extends React.Component {
      // This syntax ensures `this` is bound within handleClick.
      // Warning: this is *experimental* syntax.
      handleClick = () => {
        console.log('this is:', this);
      }
    
      render() {
        return (
          <button onClick={this.handleClick}>
            Click me
          </button>
        );
      }
    }
    
Let's figure out what's ` () => {}` means here: 

***() contains some variables, used in an arrow function to return an object***

***{} contains some statement, actually, it's a special syntax in JSX. It contains a JS expression, can be a variable***

## 1.6  Conditional Rendering 

User can create distinct components that encapsulate behavior you need, ANd we can **render only some of them**. 

1. Using if to do conditional rendering
2. Inline if with Logical && Operator 


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

It works because in JS, `true && expression` always evaluateds to `expression`, and `false && expression` always evaluates to `false`. 

3. Inline if-else with conditional operator 

    `condition ? true : false`
    
4. Return null will prevent a component from render, but it will not affect the component's lifecycle methods. 

## 1.7 Lists and Keys 

### 1.7.1 Rendering Multiple Components 

    const numbers = [1, 2, 3, 4, 5];
    const listItems = numbers.map((number) =>
      <li>{number}</li>
    );
    
    // warning will be shown indicating you need provide key 
        function NumberList(props) {
      const numbers = props.numbers;
      const listItems = numbers.map((number) =>
        <li>{number}</li>
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

A “key” is a special string attribute you need to include when creating lists of elements. 

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
    
## 1.7.2 Keys

It helps React identify which items have changed, are added, or are removed. Keys should be given to the elements inside the array to give the elements a stable identity. 

    const todoItems = todos.map((todo, index) =>
      // Only do this if items have no stable IDs
      <li key={index}>
        {todo.text}
      </li>
    );

Using index as key might have some negative influence, thus we'd better find some other specific string as the key. 

!!! Keys only make sense in the context of the surrounding array. 

    function ListItem(props) {
      const value = props.value;
      return (
        // Wrong! There is no need to specify the key here:
        <li key={value.toString()}>
          {value}
        </li>
      );
    }
    
    function NumberList(props) {
      const numbers = props.numbers;
      const listItems = numbers.map((number) =>
        // Wrong! The key should have been specified here:
        <ListItem value={number} />
      );
      return (
        <ul>
          {listItems}
        </ul>
      );
    }
    
    const numbers = [1, 2, 3, 4, 5];
    ReactDOM.render(
      <NumberList numbers={numbers} />,
      document.getElementById('root')
    );
    
Correct Version: 

    function ListItem(props) {
      // Correct! There is no need to specify the key here:
      return <li>{props.value}</li>;
    }
    
    function NumberList(props) {
      const numbers = props.numbers;
      const listItems = numbers.map((number) =>
        // Correct! Key should be specified inside the array.
        <ListItem key={number.toString()}
                  value={number} />
    
      );
      return (
        <ul>
          {listItems}
        </ul>
      );
    }
    
    const numbers = [1, 2, 3, 4, 5];
    ReactDOM.render(
      <NumberList numbers={numbers} />,
      document.getElementById('root')
    );
    
## 1.7.3 Embedding map() in JSX 

    function NumberList(props) {
      const numbers = props.numbers;
      const listItems = numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
    
      );
      return (
        <ul>
          {listItems}
        </ul>
      );
    }
    
After map:

        function NumberList(props) {
      const numbers = props.numbers;
      return (
        <ul>
          {numbers.map((number) =>
            <ListItem key={number.toString()}
                      value={number} />
    
          )}
        </ul>
      );
    }
    
## 1.8 Forms

Form elements natuarlly keep some internal state.
    
    <form>
      <label>
        Name:
        <input type="text" name="name" />
      </label>
      <input type="submit" value="Submit" />
    </form>

React can use controlled components to achieve this. 

### 1.8.1 Controlled Components -- Form 

In HTML, input, textaream select typically **maintain their own state and update it based on user input**. In React, mutable state is typically kept in the state property of components, and only updated with `setState()`.

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

Controlled Component, in this way, use `state` in React to control the change of text. With a controlled component, ***every state mutaion will have an associated handler function***, This makes it straightforward to modify or validate user input. 

### 1.8.2 Controlled Components -- textarea 

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
              Essay:
              <textarea value={this.state.value} onChange={this.handleChange} />
            </label>
            <input type="submit" value="Submit" />
          </form>
        );
      }
    }
    
Have the value property, and associate it with state. change it by handler function, and deal with it with a chain of handlers. 

### 1.8.3 Controlled Components -- select 

    <select>
      <option value="grapefruit">Grapefruit</option>
      <option value="lime">Lime</option>
      <option selected value="coconut">Coconut</option>
      <option value="mango">Mango</option>
    </select>
    
coconut has been selected with the selected property 

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
              Pick your favorite flavor:
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

By setting the start state as cconut, we realize the same function as you see in the HTML part. But come to be more extenable since when we want to change its states, we only need to change this component's state. 

### 1.8.4 Handling multiple inputs 

when need to handle multiple controlled input elements, you can add a name attribute to each element and let the handler function choose what to do based on the value of `event.target.name`.

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
    
`handleInputChange` here controls two input, one is a checkbox, another is a scrolled banner. That's the reason we need to judge target type in function. However, I do think there is no need to combine those two sub component, and use one handler function, at least in this example. 

> Notice: in `setState`, we use `[name] : value`. It's because ***name is a computed property, [] means need to compute here***. 

## 1.9 Lifting state up

Happen when several components need to reflext the same changing data. Need to lifting the shared state up to their closest common ancestor. 

    const scaleNames = {
      c: 'Celsius',
      f: 'Fahrenheit'
    };
    
    function toCelsius(fahrenheit) {
      return (fahrenheit - 32) * 5 / 9;
    }
    
    function toFahrenheit(celsius) {
      return (celsius * 9 / 5) + 32;
    }
    
    function tryConvert(temperature, convert) {
      const input = parseFloat(temperature);
      if (Number.isNaN(input)) {
        return '';
      }
      const output = convert(input);
      const rounded = Math.round(output * 1000) / 1000;
      return rounded.toString();
    }
    
    function BoilingVerdict(props) {
      if (props.celsius >= 100) {
        return <p>The water would boil.</p>;
      }
      return <p>The water would not boil.</p>;
    }
    
    class TemperatureInput extends React.Component {
      constructor(props) {
        super(props);
        this.handleChange = this.handleChange.bind(this);
      }
    
      handleChange(e) {
        this.props.onTemperatureChange(e.target.value);
      }
    
      render() {
        const temperature = this.props.temperature;
        const scale = this.props.scale;
        return (
          <fieldset>
            <legend>Enter temperature in {scaleNames[scale]}:</legend>
            <input value={temperature}
                   onChange={this.handleChange} />
          </fieldset>
        );
      }
    }
    
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
    
    ReactDOM.render(
      <Calculator />,
      document.getElementById('root')
    );

1. Lifting state, React calls the function specified as onChange on the DOM <input>. In our case, this is the handleChange method in the TemperatureInput component.
2. The handleChange method in the TemperatureInput component calls **this.props.onTemperatureChange()** with the new desired value. Its props, including onTemperatureChange, were provided by its parent component, the Calculator.
3. When it previously rendered, the Calculator has specified that onTemperatureChange of the Celsius TemperatureInput is the Calculator’s handleCelsiusChange method, and onTemperatureChange of the Fahrenheit TemperatureInput is the Calculator’s handleFahrenheitChange method. So either of these two Calculator methods gets called depending on which input we edited.
4. Inside these methods, the Calculator component asks React to re-render itself by calling this.setState() with the new input value and the current scale of the input we just edited.
5. React calls the Calculator component’s render method to learn what the UI should look like. The values of both inputs are recomputed based on the current temperature and the active scale. The temperature conversion is performed here.
6. React calls the render methods of the individual TemperatureInput components with their new props specified by the Calculator. It learns what their UI should look like.
7. React calls the render method of the BoilingVerdict component, passing the temperature in Celsius as its props.
8. React DOM updates the DOM with the boiling verdict and to match the desired input values. The input we just edited receives its current value, and the other input is updated to the temperature after conversion.


Usually, the state is first added to the component that needs it for rendering, Then it other components also need it, you can lift it up to their closest common ancestor. 

## 1.10 Compisition vs Inheritance 

React has a powerful composition model, and we recommend **using composition** instead of inheritance to **reuse code** between components. 

### 1.10.1 Containment

Some component don't know their children ahead of time. This is especially common for components like SideBar or Dialog that represent generic boxes. 

We can use `children` props to pass children elements derectly into their output. 

    function FancyBorder(props) {
      return (
        <div className={'FancyBorder FancyBorder-' + props.color}>
          {props.children}
        </div>
      );
    }
    
    function WelcomeDialog() {
      return (
        <FancyBorder color="blue">
          <h1 className="Dialog-title">
            Welcome
          </h1>
          <p className="Dialog-message">
            Thank you for visiting our spacecraft!
          </p>
        </FancyBorder>
      );
    }
Anything inside the `<FancyBorder>` JSX tag gets passed into the FancyBorder component as a children prop. Since FancyBorder renders `{props.children}` inside a `<div>`, then passed elements appear in the final output. 

And also, we can use your own props and pass it inside. 

    function SplitPane(props) {
      return (
        <div className="SplitPane">
          <div className="SplitPane-left">
            {props.left}
          </div>
          <div className="SplitPane-right">
            {props.right}
          </div>
        </div>
      );
    }
    
    function App() {
      return (
        <SplitPane
          left={
            <Contacts />
          }
          right={
            <Chat />
          } />
      );
    }
    
### 1.10.2 Specialization 

A more specific component renders a more generic one and configures it with props: 

    function Dialog(props) {
      return (
        <FancyBorder color="blue">
          <h1 className="Dialog-title">
            {props.title}
          </h1>
          <p className="Dialog-message">
            {props.message}
          </p>
        </FancyBorder>
      );
    }
    
    function WelcomeDialog() {
      return (
        <Dialog
          title="Welcome"
          message="Thank you for visiting our spacecraft!" />
      );

### 1.10.3 Example 

    function Dialog(props) {
      return (
        <FancyBorder color="blue">
          <h1 className="Dialog-title">
            {props.title}
          </h1>
          <p className="Dialog-message">
            {props.message}
          </p>
          {props.children}
        </FancyBorder>
      );
    }
    
    class SignUpDialog extends React.Component {
      constructor(props) {
        super(props);
        this.handleChange = this.handleChange.bind(this);
        this.handleSignUp = this.handleSignUp.bind(this);
            this.state = {login: ''};
          }
        
          render() {
            return (
              <Dialog title="Mars Exploration Program"
                      message="How should we refer to you?">
                <input value={this.state.login}
                       onChange={this.handleChange} />
                <button onClick={this.handleSignUp}>
                  Sign Me Up!
                </button>
              </Dialog>
            );
          }
        
          handleChange(e) {
            this.setState({login: e.target.value});
          }
        
          handleSignUp() {
            alert(`Welcome aboard, ${this.state.login}!`);
          }
        }
        
# 2. Thinking - How to build an product from scratch

1. Start with a mock: UI + Json API
2. Break the UI into a Component Hierarchy 
3. Separate to different components, following ***single responsibility principle***. A component should ideally only do one thing. 
4. Build a static version in react 
    * decouple styling and interactivity 
    * static version always use props, since state is reserved only for interactivity 
    * Larger project, easier bottom up. 
    * Don't repeat yourself

5. Identify the minimal representation of UI state
    * Is it passed in from a parent via props? If so, it probably isn’t state.
    * Does it remain unchanged over time? If so, it probably isn’t state.
    * Can you compute it based on any other state or props in your component? If so, it isn’t state.
6. Idenfify where your state should live 

# Reference 
1. [REACT DOC](https://reactjs.org/docs/hello-world.html)
