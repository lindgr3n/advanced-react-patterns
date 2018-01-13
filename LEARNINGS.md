# Advanced React Component Patterns by @kentcdodds

## 03. Write Compound Components
* Give the user full control of what is rendered by using children
* React.Children special array for react children
* React namespacing to render child components Toggle.On <--
* See 02.html

## 04. Make Compound React Components Flexible
* Give the user ability to nest children.
```
<Toggle onToggle={on => console.log('toggle', on)} >
  <Toggle.On>The button is on</Toggle.On>
  <Toggle.Off>The button is off</Toggle.Off>
  <div>
    <Toggle.Button />
  </div>
</Toggle>
```

* Render.Children only includes direct decendents to our Toggle button
* React.context
  - contextTypes
  - contextChilds

## 05. Make Enhanced React Components with Higher Order Components
Takes one component and returns a new component with some enhanced behavior
Used in internal component for wrapping e.g. toggleOn,Off,Button so just context referece is in one place.

## 06. Handle prop namespace clashes with Higher Order Components
Provided prop is the same as one existing in the wrapper. Because the order on how the properties are read.

<Component {...toggleContext} {..props}>
If an properties exist in toggleContext and in props props overwrite the one in toggleContext. 
Use props first and toggleContext will overwrite props.

Nested destructuring!
({toggle: {on, toggle}}) =>

## 07 Improve debuggability of Higher Order Components
Search for name of an HOC no hit in debugger?
<Unknown > when using inline arrowfunction
Need to set up a variable with the function and pass it to the HOC.
const MyToggle = ({toggle: {on, toggle}}) => (
  <button onClick={toggle}>
    {on ? 'on' : 'off'}
  </button>
)
const MyToggleWrapper = withToggle(MyToggle)

Inside the HOC we can use 
Wrapper.displayName = `withToggle(${Component.displayName || Component.name})` for making the debugger show a "better" instead of just Wrapper!

Use the HOC wrapper for the static assignment
static On = withToggle(ToggleOn) will make the display name alot better!

App.displayName can be used for change name in debugger!

TODO: Provide screens on this from the debugger!

## 08 Handle ref props with Higher Order Components
Can assign refs on HOC (that is stateless/No class) directly because it a stateless component!

Need to name props e.g. innerRef= and inside the HOC ref={innerRef} (After destructuring)

## 09 Improve Unit Testability of Higher Order Components
Expose the wrapped component from the HOC

Wrapper.WrappedComponent = Component

<MyToggleWrapper.WrappedComponent ...

## 10 Handle static properties properly with Higher Order Components
Want all static properties to be on the Wrapped component. 
Library hoist
return hoistNonReactStatics(Wrapper, Component)
Will take all non react specific properties

## 11 Use Render Props with React
Start from the "base" togglebuton!

Just pass a render property to our component
<Toggle
  onToggle={on => console.log('toggle', on)}
  render={({on, toggle}) => (
    <div>
      {on
        ? 'The button is on'
        : 'The button is off'}
      <Switch on={on} onClick={toggle} />
      <hr />
      <MyToggle on={on} toggle={toggle} />
    </div>
  )}
/>

And in our component on render we just call this.props.render(...
``` 
render() {
  return this.props.render({
    on: this.state.on,
    toggle: this.toggle,
  })
}
```

Inside our component we set the provided properties to the state properties!

Bam mindblown why use HOC!
Dont need to care about contextTypes, wrapping, displayname and namecollision!
Dont know from where the prop is comming from inside our wrapper
just render it!

## 12 Use Prop Collections with Render Props
Can create collection of props to be applied on the provided input

```
render() {
  return this.props.render({
    on: this.state.on,
    toggle: this.toggle,
    togglerProps: {
      'aria-expanded': this.state.on,
      onClick: this.toggle,
    },
  })
}
```

Then the user can just spread the collection
```
render={({on, toggle, togglerProps}) => (
  <div>
    <Switch on={on} {...togglerProps} />
    <hr />
    <button {...togglerProps}>
      {on ? 'on' : 'off'}
    </button>
  </div>
)}
``` 
## 13 Use Prop Getters with Render Props
Be aware of the overriding of the props!
Make our component compose things!

Instead of just calling an object (togglerProps) we make an getter that will return the object. getTogglerProps()

```
getTogglerProps = (props) => {
  return {
    'aria-expanded': this.state.on,
    onClick: this.toggle,
    ...props,
  }
}
```
But need to take care of when user is passing an onClick so it dont get override
So we could take out onClick and handle it sepratly by calling the users passing onClick and the our toggle. Also make sure to check if there exist an provided onClick prop!
```
getTogglerProps = ({onClick, props} = {}) => {
  return {
    'aria-expanded': this.state.on,
    onClick: (...args) => {
      onClick && onClick(..args)
      this.toggle(...args)
    },
    ...props,
  }
}
```
Can make it more generic by using an helper function compose
const compose = (...fns) => (...args) => fns.forEach(fn => fn && fn(...args))
Take all functions and all arguments and for each function (if it exist) call it with the args.

Can rewrite the onClick to use our compose! Butiful!
```
getTogglerProps = ({onClick, props} = {}) => {
  return {
    'aria-expanded': this.state.on,
    onClick: compose(onClick, this.toggle),
    ...props,
  }
}
```

## 14 Use Component State Initializers
static defaultProps
When setting up the state in the component use initializeState and set the state to it. Then if you need to use e.g an reset button you can just setState(this.initialState) ! No need to duplicate the 
```
static defaultProps = {
  defaultOn: false,
  onToggle: () => {},
  onReset: () => {},
}
initialState = {on: this.props.defaultOn}
state = this.initialState
```

Provide an reset for the user in the renderer
``` 
render() {
  return this.props.render({
    on: this.state.on,
    toggle: this.toggle,
    reset: this.reset,
    getTogglerProps: this.getTogglerProps,
  })
}
```
Then the user can use it by 
```
<button onClick={() => toggle.reset()}>
  Reset
</button>
```

Then we need a function on our component that reset the state.
reset = () => 
  this.setState(this.initialState, () =>
    this.props.onReset(this.state.on),
  )

  Here we have an callback that is provided to the user when the state is complete! (Remeber that tha setState is async call so need callback to know when it is done!)

  This callback provider onReset we provide in defaultProps so the user dont need to add it!

  ## 15 Make Controlled React Components with Control Props
  Want the ability to control the state from outside!

  We can in our components render provide the ability to give the own state. Where we check if it exist on the props otherwise we use our own state
  ```
  render() {
    return this.props.render({
      on: this.props.on !== undefined
        ? this.props.on
        : this.state.on,
      toggle: this.toggle,
      reset: this.reset,
      getTogglerProps: this.getTogglerProps,
    })
}
```

Need to keep track if we are controlling the state or the user is!
Can use a helperfunction
```
isOnControlled() {
  return this.props.on !== undefined
}
```

Then we check all places we update state and if controlled we just call the callback
```
reset = () => {
  if (this.isOnControlled()) {
    this.props.onReset(!this.props.on)
  } else {
    this.setState(this.initialState, () =>
      this.props.onReset(this.state.on),
    )
  }
}
```

## 16 Implement a React Context Provider
Need to recap this!
Provider Pattern
When nesting props instead of passing them down you could use context
ToggleProvider uses static class Renderer that is used to render the passed children. Then we connect a statless function that uses the context so we can react the toggle props (Bah not fully with this!) Amazing tho!

## 17 Implement a Higher ORder Component with Render Props
Can implement a HOC from an render prop not realy the other way around

"Just" add to provide ability to use render prop
```
function withToggle(Component) {
  function Wrapper(props, context) {
    const {innerRef, ...remainingProps} = props
    return (
      <ConnectedToggle
        render={toggle => (
          <Component
            {...remainingProps}
            toggle={toggle}
            ref={innerRef}
          />
        )}
      />
    )
  }
  Wrapper.displayName = `withToggle(${Component.displayName ||
    Component.name})`
  Wrapper.propTypes = {innerRef: PropTypes.func}
  Wrapper.WrappedComponent = Component
  return hoistNonReactStatics(Wrapper, Component)
}
```
Then this render prop call with ConnectedToggle
```
function Subtitle() {
  return (
    <ConnectedToggle
      render={toggle =>
        toggle.on
          ? '👩‍🏫 👉 🕶'
          : 'Teachers are awesome'}
    />
  )
}
```
becomes this HOC call with withToggle
```
const Subtitle = withToggle(
  ({toggle}) =>
    toggle.on
      ? '👩‍🏫 👉 🕶'
      : 'Teachers are awesome',
)
```

## 18 Rerender Descendants Through shouldComponentUpdate
Cases when an component have shouldComponentUpdate set to return false all children below that will not be updated!

Need to give the ability to let the children from that kind of blocker component to be updated.
Library "react-broadcast"
Do all context managment for us.

Done by setting a channel
static channel = '__toggle_channel__' 
And then in our ToggleProvider we use ReactBroadcast.Broadcast in the render to broadcast out to all our subscibers

So in our ConnectedToggle we subscribe to the BroadCast
```
function ConnectedToggle(props, context) {
  return (
    <ReactBroadcast.Subscriber
      channel={ToggleProvider.channel}
    >
      {toggle => props.render(toggle)}
    </ReactBroadcast.Subscriber>
  )
}
```

Cool!

## 19 Use Redux with Render Props
Rendux!
Local redux management inside our component!
