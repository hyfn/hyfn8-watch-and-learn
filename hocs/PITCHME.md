# Higher Order Components (HOCs)

---

### Definition:
A higher-order component (HOC) is an advanced technique in React for reusing component logic (alternative for Mixins pattern).

Concretely, a higher-order component is just a React component that wraps another component. It is usually implemented in the form of a function that takes a component and returns a new enhanced component.

A HOC composes the original component (conventionally called the wrapped/composed component) by wrapping it in a container component.

Whereas a component transforms props into UI, a higher-order component transforms a component into another component.

---

We should consider to start using HOCs in our React Apps to abstract common functionalities and reduce code redundancy.
---

### Usage:

* Code reuse, logic abstraction (example: make generic component toggleable)
* State abstraction and manipulation
* Establish Connection to a Store/API Service (perfect for adding logic inside componentDidMount/componentWillMount)
* Control over Inputs passed to the composed component. For example adding, restricting, and transforming the props that are being passed to the wrapped component.
* Intercept Rendering/Component Lifecycle Methods of wrapped component
* Add CSS styling to wrapped component

---
One of the articles: [HOCs in depth](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e), divides HOCs to 2 geneal types:

1. Props Proxy: The HOC manipulates the props being passed to the WrappedComponent.
2. Inheritance Inversion: The HOC extends the WrappedComponent.

In this presentation, we will mostly focus on Props Proxy HOCs, and only introduce the Inheritance Inversion HOCs briefly.
---

### Props Proxy HOCS Examples

---

### Example 1: Establish Connection to a Store/API Service

This stateless functional component does not want to concern itself with any data fetching.

```javascript
const Greeting = ({ name, description }) => {
  if (!name) { return <div>Connecting...</div> }

  return <div>Hi {name}! You are {description}</div>
}
```

---

Our HOC will handle that, and then will send the desired data to the wrapped component via props.

```javascript
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


Then to use the HOC, pass it the `Greeting` component:
```javascript
const ConnectedGreeting = connectMyComponent(Greeting)
<ConnectedGreeting description='awesome' />
```

---

Note that you could also pass in as many arguments as you want to the HOC. For example in order to control how the data gets fetched, and thus make your HOC more reusable.

```javascript
// This function takes a component...
const withSubscription = (WrappedComponent, grabDataFunc) => {
  // ...and returns another component...
  return class extends React.Component {
    constructor() {
      super()
      this.state = { data: null }
    }

    componentDidMount() {
      this.setState({ data: grabDataFunc(this.props) })
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent {...this.props} data={this.state.data} />
    }
  }
}
```

---

To to use:

```javascript
const BookWithSubscription = withSubscription(< Book />, fetchBookData )
const MagazineWithSubscription = withSubscription(< Magazine />, fetchMagazineData )
const NewspaperWithSubscription = withSubscription(< Newspaper />, fetchNewspaperData )
```

---

### NOTE: REDUX CONNECT

The redux `connect` method is actually a HOF.
```javascript
const connectComponent = connect(mapStateToProps, mapDispatchToProps)(CreateCampaignForm)
```

connect is a function that returns another function. The returned function is a HOC.


The HOC takes in a component and returns a new component that is connected to the Redux store.

```javascript
const hoc = connect(mapStateToProps, mapDispatchToProps)
const connectComponent = hoc(CreateCampaignForm)
```

In other words, connect is a higher-order function that returns a higher-order component!

---

### Example 2: State Abstraction
We can abstract state by providing **props** and **callbacks** to the wrapped component, very similar to how smart components will deal with dumb components

In the following State Abstraction example we abstract the value and onChange handler of the name input field.

---

```javascript
const enhanceComponent = (WrappedComponent) => {
  return class EnhancedComponent extends React.Component {
    constructor(props) {
      super(props)
      this.state = { name: '' }
    }
    onNameChange = (event) =>
      this.setState({ name: event.target.value })

    render() {
      const newProps = {
        name: {
          value: this.state.name,
          onChange: this.onNameChange
        }
      }
      return <WrappedComponent {...this.props} {...newProps}/>
    }
  }
}
```

You would use it like this

```javascript
class Example extends React.Component {
  render() {
    return <input name="user_name" {...this.props.name}/>
  }
}

const EnhancedComponent = enhanceComponent(Example)
<EnhancedComponent additionalProps='some stuff' />
```
---
Something to note.
The last example could have been made a little cleaner using the decorator syntax. The decorator syntax gives us an elegnant and simple way to add HOC to any component.

```
@enhanceComponent
class Example extends React.Component {
  render() {
    return <input name="user_name" {...this.props.name}/>
  }
}
```

Now
```javascript
<Example additionalProps='some stuff' />
```
is the same as:
```javascript
<EnhancedComponent additionalProps='some stuff' />
```
---

### NOTE: CONTAINER COMPONENTS

You may have noticed similarities between HOCs and the container components pattern.
Container components are part of a strategy of separating responsibility between high-level and low-level concerns.

Containers manage things like subscriptions and state, and pass props to components that handle things like rendering UI.
HOCs use containers as part of their implementation. You can think of HOCs as parameterized (take in arguments) container components.
Basically HOCs are just generic containers, wrapped up in a function (that takes the wrapped component as an argument)

---
### Example 3: Logic (and state) Abstraction: Make Toggleable
---
Let's say we want to toggle the `props.children` of a component whenever a user clicks on that component. Let's create a HOC that will handle the toggle functionality.

```javascript
const makeToggleable = (Clickable) => {
 return class extends React.Component {
    constructor() {
        super()
        this.state = { show: false }
    }

    toggle = () => {
        this.setState({ show: !this.state.show })
    }

    render() {
        return (
            <div>
                <Clickable
                    {...this.props}
                    onclick={this.toggle}
                />
                {this.state.show && this.props.children}
            </div>
        )
    }
 }
}
```
---

Our `Menu` component will not have to concern itself with any toggle functionality or state management since the HOC already handles those.

Using the decorator syntax we have:

```javascript
@makeToggleable
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

Same as:
```javascript
const ToggleableMenu = makeToggleable(RegularMenu)
```
---

Then to use it:

```javascript
Class MenuList extends React.Component {
    render() {
        return (
            <div>
                <ToggleableMenu title='First Menu'>
                    <ol>
                        <li>Toggleable list item 1</li>
                        <li>Toggleable list item 2</li>
                        <li>Toggleable list item 3</li>
                    </ol>
                </ToggleableMenu>
                <ToggleableMenu title='Second Menu'>
                    <div>Toggleable Div</div>
                </ToggleableMenu>
                <ToggleableMenu title='Third Menu'>
                    <img src="myImage.jpg" alt="Toggleable Image">
                </ToggleableMenu>
            </div>
        )
    }
}
```
---

### Example 4: Wrapping the Wrapped Component with other elements

You can wrap the WrappedComponent with other components and elements for styling, layout or other purposes.

Example: Wrapping for styling purposes
```
const cssHOC = (WrappedComponent) => {
  return class StyledComponent extends React.Component {
    render() {
      return (
        <div style={{display: 'block'}}>
          <WrappedComponent {...this.props}/>
        </div>
      )
    }
  }
}
```

(This HOC does not really fall into the category of a Props Proxy HOC, but I thought it is worth sharing this type of HOCs example as well)

---
### Inheritance Inversion HOCS

```javascript
const iiHOC = (WrappedComponent) => {
  return class Enhancer extends WrappedComponent {
    render() {
      return super.render()
    }
  }
}
```

Inheritance Inversion allows the HOC to have access to the WrappedComponent instance via this, which means it has access to the state, props, component lifecycle hooks and the render method.

Remember to always call super.[lifecycleHook] so you don't break the WrappedComponent.
---

What can you do with Inheritance Inversion?
- Render Highjacking
- Manipulating state

---
### Render Hijacking

It is called Render Highjacking because the HOC takes control of the render output of the WrappedComponent and can do all sorts of stuff with it.

render refers to the WrappedComponent.render method
---

### Render Hijacking Example: Conditional rendering

```javascript

const iiHOC = (WrappedComponent) => {
  return class Enhancer extends WrappedComponent {
    render() {
      if (this.props.loggedIn) {
        return super.render()
      } else {
        return null
      }
    }
  }
}

(You cannot edit or create props of the WrappedComponent instance, because a React Component cannot edit the props it receives, but you can change the props of the elements that are outputted from the render method.)

```

---

### Convention

---

* Convention: resist the temptation to modify a component’s prototype (or otherwise mutate it) inside a HOC. Instead of mutation, HOCs should use composition, by wrapping the input component in a container component.

```
const myHoc = (WrappedComponent) => {
    /// BAD
    WrappedComponent.prototype.componentWillReceiveProps = (nextProps) => {
        ...
    }

    // The fact that we're returning the original input is a hint
    // that it has been mutated.
    return WrappedComponent;
}
```

---

Convention: Pass Unrelated Props Through to the Wrapped Component

```
const myHoc = (WrappedComponent) =>
    class extends React.Component {
        render() {
            // Filter out extra props that are specific to this HOC and shouldn't be
            // passed through
            const { extraProp, ...passThroughProps } = this.props

            // Inject props into the wrapped component.
            // These are usually state values or instance methods.
             const injectedProp = someStateOrInstanceMethod

            // Pass props to wrapped component
             return (
              <WrappedComponent
                  injectedProp={injectedProp}
                 {...passThroughProps}
                />
            )
        }
    }
```

---

### Note!

There are three patterns in React that are meant to be used in case that you want to decouple generic logic.

Those patterns are **Render Props, Function as Child, and HOCs**. Each of them has its own uses cases plus pros and cons.

See the [Advanced React Patterns](https://medium.com/@jonatan_salas/advanced-react-patterns-lets-talk-about-render-props-function-as-child-and-hocs-c0cc4b5d6797) link in the resources section for additional information.

---


### [Caveats](https://reactjs.org/docs/higher-order-components.html#caveats)

---
* First and foremost, using HOCs adds a new component to your virtual DOM tree. Be aware of this.
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


The problem here isn’t just about performance — remounting a component causes the state of that component and all of its children to be lost.

Instead, apply HOCs outside the component definition so that the resulting component is created only once. Then, its identity will be consistent across renders. This is usually what you want, anyway.

---

* Static Methods Must Be Copied Over

When you apply a HOC to a component, the original component is wrapped with a container component. That means the new component does not have any of the static methods of the original component.

```
// Define a static method
WrappedComponent.staticMethod = function() {/*...*/}
// Now apply a HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

Basically just remember that the component returned from the HOC is DIFFERENT than the wrapped component and so props, refs, and static methods **DO NOT transfer automatically** between wrapped component and its wrapper.

---

### Potential Uses For Hyfn:

* CSS HOCs in order to maintain styling consistency across the platforms
* User Permissions HOCs that restrict rendering according to access permissions.
*

Any functinoality that could be reusable between the subapps!
---

### Resources:
* React Docs
    - [Higher Order Components](https://reactjs.org/docs/higher-order-components.html)
    - [React Reconciliation](https://reactjs.org/docs/reconciliation.html)
* Basic:
    - [react-component-patterns](https://levelup.gitconnected.com/react-component-patterns-ab1f09be2c82)
    - [React Patterns](https://github.com/chantastic/reactpatterns.com#higher-order-component)
* Decorators:
    - [HOC as decorators](https://medium.com/@mappmechanic/react-utility-higher-order-components-as-decorators-tc39-stage-2-9e9f3a17688a)
    - [ES7 Decorators in ReactJS](https://medium.com/@jihdeh/es7-decorators-in-reactjs-22f701a678cd)
* Advanced:
    - [Advanced React Patterns](https://medium.com/@jonatan_salas/advanced-react-patterns-lets-talk-about-render-props-function-as-child-and-hocs-c0cc4b5d6797)
    - [HOCs in depth](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e)

---

### Thanks.


