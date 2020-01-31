---
title: React Advanced(2)
date: 2020-01-30 23:13:49
categories: FrontEnd
tags:
    - FrontEnd
    - React
top:
---

In this part, dive deeper into React. In the previous post, found a lot of new things. Though I can write some jsx code, but I have to admit it's really ugly...wihout reuse, with some useless states for no reasons, duplicate, boring. That's why try to write some articles following authoritive docs. There are some of my thoughts inside, hope it can help you. :) 

# 1. Accessibility 
Also known as a11y, is the design and creation of websites that can be used by everyone. 

## 1.1 Semantic HTML 

Sometimes, we break HTML sementics when we add `<div>` elements to our JSX to make our React code work, especially working with lists and table. In these case, we should use ***React Fragments*** to group together multiple elements. 

    import React, { Fragment } from 'react';
    
    function ListItem({ item }) {
      return (
        <Fragment>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      );
    }
    
    function Glossary(props) {
      return (
        <dl>
          {props.items.map(item => (
            <ListItem item={item} key={item.id} />
          ))}
        </dl>
      );
    }
    
Map a collection of items to an array of fragments as you would any other type of elements as well: 

    function Glossary(props) {
      return (
        <dl>
          {props.items.map(item => (
            // Fragments should also have a `key` prop when mapping collections
            <Fragment key={item.id}>
              <dt>{item.term}</dt>
              <dd>{item.description}</dd>
            </Fragment>
          ))}
        </dl>
      );
    }
    
# 2 Refs and the DOM 

> Refs provide a way to access DOM nodes or React elements created in the render method. 

Refs offer another way to change a child outside of the typical dataflow - use props from parent to child. **The child to be modified could be an instance of a React Component, or it could be a DOM element.** 

## 2.1 When to use Refs 

1. Managing focus, text selection, media playback 
2. triggering imperative animations
3. integrating with third party DOM libraries 

## 2.2 How to use Refs 

Use by `React.createRef()`, to create a ref

    class MyComponent extends React.Component {
      constructor(props) {
        super(props);
        this.myRef = React.createRef();
      }
      render() {
        return <div ref={this.myRef} />;
      }
    }

When a ref is passed to an element in render, a reference to the node becomes accessible at the ***current*** attribute of the ref

    const node = this.myRef.current;
    
## 2.3 Value of the ref

1. When the ref attribute is used on an HTML element, the ref created in the constructor with React.createRef() receives the underlying DOM element as its current property.
2. When the ref attribute is used on a custom class component, the ref object receives the mounted instance of the component as its current.

    class CustomTextInput extends React.Component {
      constructor(props) {
        super(props);
        // create a ref to store the textInput DOM element
        this.textInput = React.createRef();
        this.focusTextInput = this.focusTextInput.bind(this);
      }
    
      focusTextInput() {
        // Explicitly focus the text input using the raw DOM API
        // Note: we're accessing "current" to get the DOM node
        this.textInput.current.focus();
      }
    
      render() {
        // tell React that we want to associate the <input> ref
        // with the `textInput` that we created in the constructor
        return (
          <div>
            <input
              type="text"
              ref={this.textInput} />
            <input
              type="button"
              value="Focus the text input"
              onClick={this.focusTextInput}
            />
          </div>
        );
      }
    }

Here, React will assign the current property with the DOM element when the component mounts, and assign it back to null when it unmounts. Ref updates happen before componentDidMount or componentDidUpdate lifecycle methods. 

***!!! We cannot use ref attribute on function componnets because they don't have instances. ***

# 3. Code Splitting

## 3.1 Bundling

Most React Apps will have their files bundles using tools like webpack or browserify. Bundling is the process of following imported files and merging them into a single file. This file can then be included on a webpage to load an entire app at once. 

## 3.2 Code splitting 

Bundling is great, but as your app grows, bundle will grow too. Especially if you are including large third party libraries. 

To void winding up with a large bundle, it's good to get ahead of the problkem and start splitting your bundle. 

Code-splitting your app can help you “lazy-load” just the things that are currently needed by the user, which can dramatically improve the performance of your app. While you haven’t reduced the overall amount of code in your app, you’ve avoided loading code that the user may never need, and reduced the amount of code needed during the initial load.

## 3.3 Dynamic `import()`

The beast way to introduce code-splitting into app is through the dynamic import() syntax. 

    import { add } from './math';
    
    console.log(add(16, 26));
    
Before, now with dynamic import: 

    import("./math").then(math => {
      console.log(math.add(16, 26));
    });
    
## 3.4 React.lazy 

lets you render a dynamic import as a regular component. 

    import OtherComponent from './OtherComponent';
    
    function MyComponent() {
      return (
        <div>
          <OtherComponent />
        </div>
      );
    }
    
Switch to: 

    const OtherComponent = React.lazy(() => import('./OtherComponent'));
    
    function MyComponent() {
      return (
        <div>
          <OtherComponent />
        </div>
      );
    }
    

## 3.5 Suspense 

If the modile containing other component is not yet loaded by the time current one renders, we must **show some fallback content** while we are waiting for it to load. 

    const OtherComponent = React.lazy(() => import('./OtherComponent'));
    
    function MyComponent() {
      return (
        <div>
          <Suspense fallback={<div>Loading...</div>}>
            <OtherComponent />
          </Suspense>
        </div>
      );
    }
    
The fallback prop accepts any React elements that you want to render while waiting for the component to load. 

# 4. Context 

Context provides a way to pass data through the component tree without having to pass props down manually at every level. 

some props, like locale preference, UI theme, that are required by many components within an application. **Context provides a way to share values like these between components without having to explicitly pass a prop throught every level of the tree.**

## 4.1 When to use Context 

    class App extends React.Component {
      render() {
        return <Toolbar theme="dark" />;
      }
    }
    
    function Toolbar(props) {
      // The Toolbar component must take an extra "theme" prop
      // and pass it to the ThemedButton. This can become painful
      // if every single button in the app needs to know the theme
      // because it would have to be passed through all components.
      return (
        <div>
          <ThemedButton theme={props.theme} />
        </div>
      );
    }
    
    class ThemedButton extends React.Component {
      render() {
        return <Button theme={this.props.theme} />;
      }
    }

With context, we can avoid passing props through intermidiate elements: 

    // Context lets us pass a value deep into the component tree
    // without explicitly threading it through every component.
    // Create a context for the current theme (with "light" as the default).
    const ThemeContext = React.createContext('light');
    
    class App extends React.Component {
      render() {
        // Use a Provider to pass the current theme to the tree below.
        // Any component can read it, no matter how deep it is.
        // In this example, we're passing "dark" as the current value.
        return (
          <ThemeContext.Provider value="dark">
            <Toolbar />
          </ThemeContext.Provider>
        );
      }
    }
    
    // A component in the middle doesn't have to
    // pass the theme down explicitly anymore.
    function Toolbar(props) {
      return (
        <div>
          <ThemedButton />
        </div>
      );
    }
    
    class ThemedButton extends React.Component {
      // Assign a contextType to read the current theme context.
      // React will find the closest theme Provider above and use its value.
      // In this example, the current theme is "dark".
      static contextType = ThemeContext;
      render() {
        return <Button theme={this.context} />;
      }
    }

Notice: 
1. `contextType`
2. `context.provider`

## 4.2 Before using context 

Context is primarily used when some data needs to be accessible by many components at different nesting levels. 

Multiple other choices can be used to resolve similar problems. 

1. Pass down the combined component itself 


    <Page user={user} avatarSize={avatarSize} />
    // ... which renders ...
    <PageLayout user={user} avatarSize={avatarSize} />
    // ... which renders ...
    <NavigationBar user={user} avatarSize={avatarSize} />
    // ... which renders ...
    <Link href={user.permalink}>
      <Avatar user={user} size={avatarSize} />
    </Link>
    

    function Page(props) {
      const user = props.user;
      const userLink = (
        <Link href={user.permalink}>
          <Avatar user={user} size={props.avatarSize} />
        </Link>
      );
      return <PageLayout userLink={userLink} />;
    }
    
    // Now, we have:
    <Page user={user} />
    // ... which renders ...
    <PageLayout userLink={...} />
    // ... which renders ...
    <NavigationBar userLink={...} />
    // ... which renders ...
    {props.userLink}
    
## 4.3 API

1. `React.createContext`

    const MyContext = React.createContext(defaultValue);
    
Create a context object. When React renders a component that subscribes to this Context object it will read the current context value from the closest matching Provider above it in the tree. **Read context value from provider!**

The defaultValue argument is only used when a component does not have a matching Provider above it in the tree. This can be helpful for testing components in isolation without wrapping them.

2. `Context.Provider`

    <MyContext.Provider value={/* some value */}>
    
Every Context object comes with a Provider React Component that allows consuming components to subscribe to context changes. One Provider can be connected to many consumers. Providers can be nested to override values deeper within the tree. All consumers that are descendants of a Provider will re-render whenever the Provider’s value prop changes. The propagation from Provider to its descendant consumers is not subject to the shouldComponentUpdate method, so the consumer is updated even when an ancestor component bails out of the update.

3. `Class.contextType`

contextType property on a class can be assigned a context object created by `React.createContext()`. This lets you consume the nearest current value of that Context type using this.context. You can reference this in any of the lifecycle methods including the render function.

4. `Context.consumer`


    <MyContext.Consumer>
      {value => /* render something based on the context value */}
    </MyContext.Consumer>
    
A react component that subscribes to context changes. This lets you subscribe to a context within a function component.

## 4.4 Example 

theme-context.js 

    export const themes = {
      light: {
        foreground: '#000000',
        background: '#eeeeee',
      },
      dark: {
        foreground: '#ffffff',
        background: '#222222',
      },
    };
    
    export const ThemeContext = React.createContext(
      themes.dark // default value
    );
    
theme-button.js


    import {ThemeContext} from './theme-context';
    
    class ThemedButton extends React.Component {
      render() {
        let props = this.props;
        let theme = this.context;
        return (
          <button
            {...props}
            style={{backgroundColor: theme.background}}
          />
        );
      }
    }
    ThemedButton.contextType = ThemeContext;
    
    export default ThemedButton;
    
app.js

    import {ThemeContext, themes} from './theme-context';
    import ThemedButton from './themed-button';
    
    // An intermediate component that uses the ThemedButton
    function Toolbar(props) {
      return (
        <ThemedButton onClick={props.changeTheme}>
          Change Theme
        </ThemedButton>
      );
    }
    
    class App extends React.Component {
      constructor(props) {
        super(props);
        this.state = {
          theme: themes.light,
        };
    
        this.toggleTheme = () => {
          this.setState(state => ({
            theme:
              state.theme === themes.dark
                ? themes.light
                : themes.dark,
          }));
        };
      }
    
      render() {
        // The ThemedButton button inside the ThemeProvider
        // uses the theme from state while the one outside uses
        // the default dark theme
        return (
          <Page>
            <ThemeContext.Provider value={this.state.theme}>
              <Toolbar changeTheme={this.toggleTheme} />
            </ThemeContext.Provider>
            <Section>
              <ThemedButton />
            </Section>
          </Page>
        );
      }
    }
    
    ReactDOM.render(<App />, document.root);

# 5. Reference 

1. [React Doc](https://reactjs.org/docs/context.html)
