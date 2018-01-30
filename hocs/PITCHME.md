# Higher Order Components (HOCs)

---

### Definition:
Concretely, a higher-order component is a function that takes a component and returns a new component.

```javascript


```

---

### Pros:
* Reuse a component logic (alternative for Mixins pattern).
* Encapsulates logic that can be shared easily.

---

### Usage:
* Providing fetching and providing data to any number of stateless function components
* Modify CSS of wrapped component
* Manipulate props before passing them to wrapped component

---

### Examples

---

#### Redux Connect

The redux connect method is actually a HOF.
It's output is a HOC that connects the component into the store.

```javascript
connect(mapStateToProps, mapDispatchToProps)(CreateCampaignFormExport) connect.
```

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
