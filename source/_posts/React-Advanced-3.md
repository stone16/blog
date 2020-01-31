---
title: React Advanced(3)
date: 2020-01-30 23:14:44
categories: FrontEnd
tags:
    - FrontEnd
    - React
top:
---
# 1. Error Boundaries 

## 1.1 Why need error boundaries 

In the past, JavaScript errors inside components used to corrupt React’s internal state and cause it to emit cryptic errors on next renders. These errors were always caused by an earlier error in the application code,** but React did not provide a way to handle them gracefully in components, and could not recover from them**.

## 1.2 Intro 

A js error in a part of the UI shouldn't break the whole app. 

> Error boundaries are React components that catch JS errors anywhere in their child component tree,log these errors, and display a fallback UI instead of the component tree, log those errors, and display a fallback UI instead of the component tree that crashed. 

Notice: Error boundaries do not catch errors for: 

1. Event handler
2. Asynchrounous code 
3. server side rendering 
4. errors thrown in the error boundary itself 

A class component becomes an error boundary if it defines either (or both) of the lifecycle methods `static getDerivedStateFromError()` or `componentDidCatch()`. Use `static getDerivedStateFromError()` to render a fallback UI after an error has been thrown. Use `componentDidCatch()` to log error information.

    class ErrorBoundary extends React.Component {
      constructor(props) {
        super(props);
        this.state = { hasError: false };
      }
    
      static getDerivedStateFromError(error) {
        // Update state so the next render will show the fallback UI.
        return { hasError: true };
      }
    
      componentDidCatch(error, info) {
        // You can also log the error to an error reporting service
        logErrorToMyService(error, info);
      }
    
      render() {
        if (this.state.hasError) {
          // You can render any custom fallback UI
          return <h1>Something went wrong.</h1>;
        }
    
        return this.props.children; 
      }
    }
    
    
We can use it as a regular component: 

    <ErrorBoundary>
      <MyWidget />
    </ErrorBoundary>
    
## 1.3 Notifications

1. Error boundaries work like a JS `catch {}` block, but for components. 
2. Only calss components can be error boundaries.
3. Error boundaries only catch errors in the components below them in the tree. 
4. It cannot catch an error within itself.

# 2. Forwarding Refs 

Ref forwarding is a tech for ***automatically passing a ref through a component to one of its children***

## 2.1 Forwarding Refs to DOM components 


    function FancyButton(props) {
      return (
        <button className="FancyButton">
          {props.children}
        </button>
      );
    }

In this example, FancyButton wrap a button, and we can reuse in our dev work. But the problem is FancyButton is expected to be used in a similar manner as a regular DOM button. We might need to access their DOM nodes for ***managing focus, selection, or animations***.

> Ref forwarding is an opt-in feature that lets some components take a ref they receive, and pass it further down to a child. 



    const FancyButton = React.forwardRef((props, ref) => (
      <button ref={ref} className="FancyButton">
        {props.children}
      </button>
    ));
    
    // You can now get a ref directly to the DOM button:
    const ref = React.createRef();
    <FancyButton ref={ref}>Click me!</FancyButton>;

Here, use `React.forwardRef()` to obtain the ref passed to it, and then forward it to the DOM button that it renders. 

Here is a step-by-step explanation of what happens in the above example:

1. We create a React ref by calling `React.createRef` and assign it to a ref variable.
2. We pass our ref down to `<FancyButton ref={ref}>` by specifying it as a JSX attribute.
3. React passes the ref to the `(props, ref) => ...` function inside forwardRef as a second argument.
4. We forward this ref argument down to `<button ref={ref}>` by specifying it as a JSX attribute.
5. When the ref is attached, `ref.current` will point to the <button> DOM node.

# 3. Higher-Order Components (KEY FACTOR)

> A higher order component(HOC) is an advanced technique in React for reusing component logic. It's a pattern that emerges from React's compositional nature. 

> Concretely, ***a higher order component is a function that takes a component and returns a new component***

Whereas a component transforms props into UI, a higher-order component transforms a component into another component.

## 3.1 Use HOCs For cross cutting concerns 

CommentList component that subscribes to an external data source to render a list of comments: 

    class CommentList extends React.Component {
      constructor(props) {
        super(props);
        this.handleChange = this.handleChange.bind(this);
        this.state = {
          // "DataSource" is some global data source
          comments: DataSource.getComments()
        };
      }
    
      componentDidMount() {
        // Subscribe to changes
        DataSource.addChangeListener(this.handleChange);
      }
    
      componentWillUnmount() {
        // Clean up listener
        DataSource.removeChangeListener(this.handleChange);
      }
    
      handleChange() {
        // Update component state whenever the data source changes
        this.setState({
          comments: DataSource.getComments()
        });
      }
    
      render() {
        return (
          <div>
            {this.state.comments.map((comment) => (
              <Comment comment={comment} key={comment.id} />
            ))}
          </div>
        );
      }
    }

A component for subscribing to a single blog post:

    class BlogPost extends React.Component {
      constructor(props) {
        super(props);
        this.handleChange = this.handleChange.bind(this);
        this.state = {
          blogPost: DataSource.getBlogPost(props.id)
        };
      }
    
      componentDidMount() {
        DataSource.addChangeListener(this.handleChange);
      }
    
      componentWillUnmount() {
        DataSource.removeChangeListener(this.handleChange);
      }
    
      handleChange() {
        this.setState({
          blogPost: DataSource.getBlogPost(this.props.id)
        });
      }
    
      render() {
        return <TextBlock text={this.state.blogPost} />;
      }
    }
    
Much of those two component are similar, need an abstraction that allows us to define the logic in a single place and share it across many components.

Write a function that creates components, like CommentList and BlogList, that subscribe to DataSourse. 

    const CommentListWithSubscription = withSubscription(
      CommentList,
      (DataSource) => DataSource.getComments()
    );
    
    const BlogPostWithSubscription = withSubscription(
      BlogPost,
      (DataSource, props) => DataSource.getBlogPost(props.id)
    );
    
When CommentListWithSubscription and BlogPostWithSubscription are rendered, **CommentList and BlogPost will be passed a data prop with the most current data retrieved from DataSource**:

    // This function takes a component...
    function withSubscription(WrappedComponent, selectData) {
      // ...and returns another component...
      return class extends React.Component {
        constructor(props) {
          super(props);
          this.handleChange = this.handleChange.bind(this);
          this.state = {
            data: selectData(DataSource, props)
          };
        }
    
        componentDidMount() {
          // ... that takes care of the subscription...
          DataSource.addChangeListener(this.handleChange);
        }
    
        componentWillUnmount() {
          DataSource.removeChangeListener(this.handleChange);
        }
    
        handleChange() {
          this.setState({
            data: selectData(DataSource, this.props)
          });
        }
    
        render() {
          // ... and renders the wrapped component with the fresh data!
          // Notice that we pass through any additional props
          return <WrappedComponent data={this.state.data} {...this.props} />;
        }
      };
    }
    
Note that a HOC doesn’t modify the input component, nor does it use inheritance to copy its behavior. Rather, a HOC composes the original component by wrapping it in a container component. A HOC is a pure function with zero side-effects.

The wrapped component receives all the props of the container, along with a new prop, data, which it uses to render its output. The HOC isn't concerned with how or why the data is used, and the wrapped component isn't concerned with where the data came from. 

# 4. Fragments 

Fragments let you group a list of children without adding extra nodes to the DOM. 
    
    render() {
      return (
        <React.Fragment>
          <ChildA />
          <ChildB />
          <ChildC />
        </React.Fragment>
      );
    }
    
## 4.1 Why introduce `Fragments`

    <table>
      <tr>
        <div>
          <td>Hello</td>
          <td>World</td>
        </div>
      </tr>
    </table>
In this example, if there are two components, and we want to seperate it, and make `<div>` doesn't work(group elements inside, but keep table format works), we can modify it to: 

    class Columns extends React.Component {
      render() {
        return (
          <React.Fragment>
            <td>Hello</td>
            <td>World</td>
          </React.Fragment>
        );
      }
    }