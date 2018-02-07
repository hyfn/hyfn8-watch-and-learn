# Higher Order Components (HOCs)

---

### Definition:
A higher-order component (HOC) is an advanced technique in React for reusing component logic (alternative for Mixins pattern).

Concretely, a higher-order component is a function that takes a component and returns a new component (with steroids).

Whereas a component transforms props into UI, a higher-order component transforms a component into another component.

A HOC composes the original component (conventionally called the wrapped/composed component) by wrapping it in a container component. A HOC is a pure function with zero side-effects.

---

### Usage:
* Provide data fetching to any number of stateless function components (perfect for adding logic inside componentDidMount/componentWillMount.)
* Add functioality to a component (make toggleable).
* Modify CSS of wrapped component
* Manipulate props before passing them to wrapped component

---

### Examples

---

#### Example 1: Make a simple component toggleable.

```
Class ToggleableMenu extends React.Component {
    render() {
        return (
            <div onclick={this.props.onclick}>
                <h1>{this.props.title}</h1>
            </div>
        )
    }
}
```

---

#### Example 2: Hydrate component with data.

```
const Greeting = ({ name }) => {
  if (!name) { return <div>Connecting...</div> }

  return <div>Hi {name}!</div>
}

const connectMyComponent = ComposedComponent =>
  class extends React.Component {
    constructor() {
      super()
      this.state = { name: "" }
    }

    componentDidMount() {
      // this would fetch or connect to a store
      this.setState({ name: "Michael" })
    }

    render() {
      return (
        <ComposedComponent
          {...this.props}
          name={this.state.name}
        />
      )
    }
  }
```

Then to use:
```
const ConnectedComponent = connectMyComponent(Greeting)
```

---

Note that you could also pass in as many arguments as you want to HOC to perhaps controller how the data gets fetched, and thus make your HOC more reusable.

```
// This function takes a component...
withSubscription = (WrappedComponent, grabDataFunc) => {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);

      this.state = {
        data: grabDataFunc(props)
      };
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />
    }
  }
}
```

---

#### Example 3: Redux Connect

The redux `connect` method is actually a HOF.
```javascript
connect(mapStateToProps, mapDispatchToProps)(CreateCampaignForm)
```

It is a function that returns another function.
The returned function is a HOC, which returns a component that is connected to the Redux store.
```javascript
hoc = connect(mapStateToProps, mapDispatchToProps)
connectComponent = hoc(CreateCampaignForm)
```

In other words, connect is a higher-order function that returns a higher-order component!

---

### Convention

---

* Convention: resist the temptation to modify a component’s prototype (or otherwise mutate it) inside a HOC. Instead of mutation, HOCs should use composition, by wrapping the input component in a container component.

---

Convention: Pass Unrelated Props Through to the Wrapped Component

```
render() {
  // Filter out extra props that are specific to this HOC and shouldn't be
  // passed through
  const { extraProp, ...passThroughProps } = this.props;

  // Inject props into the wrapped component. These are usually state values or
  // instance methods.
  const injectedProp = someStateOrInstanceMethod;

  // Pass props to wrapped component
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}

```

---

#### Note!

There are three patterns in React that are meant to be used in case that you want to decouple generic logic.

Those patterns are **Render Props, Function as Child and HOCs**. Each of them has its own uses cases plus pros and cons.

See the [Advanced React Patterns](https://medium.com/@jonatan_salas/advanced-react-patterns-lets-talk-about-render-props-function-as-child-and-hocs-c0cc4b5d6797) link in the resources section for additional information.

---


### [Caveats](https://reactjs.org/docs/higher-order-components.html#caveats)

---

* Don’t Use HOCs Inside the render Method

```
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}

```

---

The problem here isn’t just about performance — remounting a component causes the state of that component and all of its children to be lost.

Instead, apply HOCs outside the component definition so that the resulting component is created only once. Then, its identity will be consistent across renders. This is usually what you want, anyway.

---

* Static Methods Must Be Copied Over

When you apply a HOC to a component, though, the original component is wrapped with a container component. That means the new component does not have any of the static methods of the original component.

* Refs Aren’t Passed Through

While the convention for higher-order components is to pass through all props to the wrapped component, it’s not possible to pass through refs. That’s because ref is not really a prop — like key, it’s handled specially by React. If you add a ref to an element whose component is the result of a HOC, the ref refers to an instance of the outermost container component, not the wrapped component.

Basically just remember that the HOC is DIFFERENT than the wrapped component and so props, refs, and static methods **DO NOT transfer automatically!**

---

You may have noticed similarities between HOCs and a pattern called container components.
Container components are part of a strategy of separating responsibility between high-level and low-level concerns.
Containers manage things like subscriptions and state, and pass props to components that handle things like rendering UI.
HOCs use containers as part of their implementation. You can think of HOCs as parameterized (take in arguments) container components.
Basically HOCs are just generic containers, wrapped up in a function (that takes the wrapped component as an argument)

---

### [Cons](https://medium.com/@jonatan_salas/advanced-react-patterns-lets-talk-about-render-props-function-as-child-and-hocs-c0cc4b5d6797)

It adds to you a new component to your virtual DOM tree without knowing. Believe me, a lot of people didn’t notice this.

If you encapsulate logic and save results using internal component state, the component you wrap using a HOC will re-render fully. You may not want this, and to avoid this issue you will have to implement SCU.

---

### Resources:
* React Docs
    - [Higher Order Components](https://reactjs.org/docs/higher-order-components.html)
* Basic:
    - [react-component-patterns](https://levelup.gitconnected.com/react-component-patterns-ab1f09be2c82)
    - [React Patterns](https://github.com/chantastic/reactpatterns.com#higher-order-component)
* Advanced:
    - [Advanced React Patterns](https://medium.com/@jonatan_salas/advanced-react-patterns-lets-talk-about-render-props-function-as-child-and-hocs-c0cc4b5d6797)
    - [HOCs in depth](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e)

---

### Thanks.


