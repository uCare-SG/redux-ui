# redux-ui: ui state without profanity

Think of redux-ui as **block-level scoping** for UI state. In this example each block-scope represents a component, and each variable represents a UI state key:

```js
{
  // Everything inside this scope has access to filter and tags. This is our root UI component.
  let filter = '';
  let tags = [];

  // Imagine the following scopes are a list of to-do tasks:
  {
    // Everything inside this scope has access to isSelected - plus all parent variables.
    let isSelected = true
  }
  {
    // This also has isSelected inside its scope and access to parent variables, but
    // isSelected is in a separate scope and can be manipulated independently from other
    // siblings.
    let isSelected = false
  }
}
```

Wrap your root component with the redux-ui `@ui()` decorator.  It's given a new scope for temporary UI variables which:

- are automatically bound to `this.props.ui`
- are auomatically passed any child component wrapped with the `@ui()` decorator
- will be automatically reset on componentWillUnmount (preventable via options)
- can be reset manually via a prop
- are updatable by any child component within the `@ui()` decorator

This is **powerful**. **Each component is reusable** and can still affect UI state for parent components.

### Setup

1. Add the redux-ui reducer to your reducers **under the `ui` key**:
  `combineReducers({ ...yourReducers, ui: uiReducer })`
2. In each 'scene' or parent component add the UI decorator with the key in
   which to save all state: `@ui('some-component')`
3. In each child component use the basic `@ui()` decorator; it will
   automatically read and write UI state to the parent component's UI key

### Usage

The `@ui` decorator injects four props into your components:

1. `uiKey`: The key passed to the decorator from the decorator (eg.
   'some-decorator' with `@ui('some-decorator')`
2. `ui`: The UI state for the component's `uiKey`
3. `updateUI`: A function accepting either a name/value pair or object which
   updates state within `uiKey`
4. `resetUI`: A function which resets the state within `uiKey` to its default

The decorator will set any default state specified (see below).
On `componentWillUnmount` the entire state in `uiKey` will be set to undefined.
You can also blow away state by calling	`resetUI` (for example, on router
changes).

### Decorator API

The decorator takes an object of options:

```
`@ui({
  key: 'some-name',
  persist: true,
  state: {
    uiVar1: ''
  }
})`
```

##### `key`: string, defaults to random characters

The name of the key used in the UI reducer under which we store all state.  If this is not specified it will be autogenerated based on the component name suffixed with a random hex code.  Components using the same key will share the same UI context, so don't supply a name to a list of components (generated in a loop) if they need their own UI state.

This is primarily used for setting `persist` below.

##### `persist`: bool, defaults to `false`

Set to true if the UI state for this component should persist after `componentWillUnmount`.  You must also explicitly define a `key` for this component, otherwise the component will randomize the key and load new UI state on instantiation.

**Note**: All parent UI components also need to set this to true for this to take effect. Think of block-level scoping again — if a parent scope quits all child scopes are also out of context.

##### `state`: object

All UI variables need to be explicitly defined in the state object.  This allows us to determine which scope a variable belongs to, as scope is inherited in the component tree.  Think of this as using `let` inside your block scopes.

### Examples

```js
import ui from 'redux-ui';

// Componnet A gets its own context with the default UI state below.
// `this.props.ui` will contain this state map.
@ui({
  state: {
    filter: '',
    isFormVisible: true,
    isBackgroundRed: false
  }
})
class A extends Component {
  render() {
    return (
      <div>
        // This will render '{ "filter": '', isFormVisible: true, isBackgroundRed: false }'
        <pre><code>{ this.props.ui }</code></pre>

        // Render child B
        <B />
      </div>
    );
  }
}

// B inherits context from A and adds its own context.
// This means that this.props.ui still contains A's state map.
@ui()
class B extends Component {
  render() {
    return <C />;
  }
}

// C inherits context from its parent B. This works recursively,
// therefore C's `this.props.ui` has the state map from `A` **plus**
// `someChildProp`.
//
// Setting variables within C updates within the context of A; all UI
// components connected to this UI key will receive the new props.
@ui({
  state: {
    someChildProp: 'foo'
  }
})
class C extends Component {
  render() {
    return (
      <div>
        <p>I have my own UI state C and inherit UI state from B and A</p>
        <p>If I define variables which collide with B or A mine will
        be used, as it is the most specific context.</p>
    );
  }
}
```

### Aims

UI state:

1. Should be global
2. Should be easily managed from each component via an action
3. Should be easy to reset (manually and when unmounting)

All of these goals should be easy to achieve.

---

MIT license.

Written by [Franklin Ta](https://github.com/fta2012) and [Tony Holdstock-Brown](https://github.com/tonyhb).
