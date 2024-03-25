# SolidJs: Reactive Functional Programming

_A crash course on SolidJS_<!--more-->

- [1. Understand reactivity](#1-understand-reactivity)
  - [Ground TRUTH: reactive function reruns whenever its outcome changes](#ground-truth-reactive-function-reruns-whenever-its-outcome-changes)
- [2. Build a Reactive System](#2-build-a-reactive-system)
  - [`createSignal` and `createEffect` from scratch](#createsignal-and-createeffect-from-scratch)
  - [Implement "`createMemo`" \& "`untrack`"](#implement-creatememo--untrack)
- [3. Rendering with Reactivity](#3-rendering-with-reactivity)
  - [Create component in SolidJS](#create-component-in-solidjs)
- [4. Control Flow](#4-control-flow)
  - [`<Show/>`: when signal=true then run callback](#show-when-signaltrue-then-run-callback)
  - [`<Switch/>`: deal with multiple conditions](#switch-deal-with-multiple-conditions)
  - [`<Dynamic/>`: for multiple avail options](#dynamic-for-multiple-avail-options)
  - [`<ErrorBoundary/>`: wraps and insulate the broken component](#errorboundary-wraps-and-insulate-the-broken-component)
  - [`<For/>`: javascript `forEach` dress like a component](#for-javascript-foreach-dress-like-a-component)
  - [`<Index/>`: `<For/>` but with keyed index](#index-for-but-with-keyed-index)
  - [`<Portal/>`: make modals in document.body](#portal-make-modals-in-documentbody)
- [5. Managing Props and Lifecycles](#5-managing-props-and-lifecycles)
  - [`mergeProps` for default values](#mergeprops-for-default-values)
  - [`splitProps`: split props without losing reactivity](#splitprops-split-props-without-losing-reactivity)
  - [`children`: wraps children elements with extra reactivity](#children-wraps-children-elements-with-extra-reactivity)
  - [`onMount`: do things once html loaded](#onmount-do-things-once-html-loaded)
  - [`onCleanup`: run things when current scope is triggered to re-evaluate](#oncleanup-run-things-when-current-scope-is-triggered-to-re-evaluate)
- [6. DOM Binding](#6-dom-binding)
- [7. State Management](#7-state-management)
- [8. Stores](#8-stores)
  - [Nested reactivity: updating list without Diffing](#nested-reactivity-updating-list-without-diffing)
  - [Create Store: nested activity without extra signals](#create-store-nested-activity-without-extra-signals)
- [9. Context](#9-context)
- [10. Simple TODO list example](#10-simple-todo-list-example)
- [11. Further Resources](#11-further-resources)


## 1. Understand reactivity

SolidJS uses:

- `createSignal`: Fine-grained signals that hold a value and represent that value overtime
<iframe
  src="https://carbon.now.sh/embed?bg=rgba%28173%2C208%2C248%2C0%29&t=cobalt&wt=none&l=javascript&width=680&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=const%2520%255Bcount%252C%2520setCount%255D%2520%253D%2520createSignal%280%29%253B%250Aconsole.log%28count%28%29%29%2520%252F%252F0%250A%250AsetCount%285%29%250Aconsole.log%28count%28%29%29%2520%252F%252F5"
  style="width: 504px; height: 279px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

- `createEffect`: All reactive variables(signals) are function calls, and might be wrapped in effect functions; the wrapping function reruns everytime reacitive variables change
<iframe
  src="https://carbon.now.sh/embed?bg=rgba%28173%2C208%2C248%2C0%29&t=cobalt&wt=none&l=javascript&width=680&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=const%2520%255Bcount%252C%2520setCount%255D%2520%253D%2520createSignal%280%29%253B%250A%250AcreateEffect%28%28%29%253D%253E%257B%250A%2520%2520console.log%28%2522the%2520count%2520is%2522%252C%2520count%28%29%29%253B%250A%257D%29%253B%250A%252F%252F%2520print%253A%2520the%2520count%2520is%25200%250A%250AsetCount%285%29%250A%252F%252F%2520print%2520again%253A%2520the%2520count%2520is%25205"
  style="width: 504px; height: 354px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

- `createMemo`: Derived variables(Memos) are composed of signals, and are re-calculated when dependent signals change
<iframe
  src="https://carbon.now.sh/embed?bg=rgba%28173%2C208%2C248%2C0%29&t=cobalt&wt=none&l=javascript&width=680&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=const%2520%255Bfirst%252C%2520setFirst%255D%2520%253D%2520createSignal%28%2522john%2522%29%253B%250Aconst%2520%255BLast%252C%2520setLast%255D%2520%253D%2520createSignal%28%2522smith%2522%29%253B%250Aconst%2520fullName%2520%253D%2520createMemo%28%28%29%253D%253E%2560%2524%257Bfirst%257D%2520%2524%257Blast%257D%2560%29%253B%250A%250AcreateEffect%28%28%29%253D%253E%257B%250A%2520%2520console.log%28%2522my%2520name%2520is%2522%252C%2520fullName%28%29%29%253B%250A%257D%29%253B%250A%252F%252F%2520print%253A%2520my%2520name%2520is%2520john%2520smith%250A%250AsetFirst%28%2522will%2522%29%253B%250A%252F%252F%2520print%2520again%253A%2520my%2520name%2520is%2520will%2520smith"
  style="width: 589px; height: 391px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

- `untrack(fn)/batch(fn)`: prevent reactive tracking within the scope of `fn` function/ waits to apply changes until `fn` completes

### Ground TRUTH: reactive function reruns whenever its outcome changes

The example shows how a conditionally reactive function changes its outcome when signals it depends vary.

```typescript
const [firstName, setFirstName] = createSignal("John");
const [lastName, setLastName] = createSignal("Smith");
const [showFullName, setShowFullName] = createSignal(true);

const displayName = createMemo(() => {
  if (!showFullName()) return firstName();
  return `${firstName()} ${lastName()}`;
});

createEffect(() => log("My name is", displayName()));
// print: My name is John Smith
setLastName("Legend");
// print: My name is John Legend
setShowFullName(false);
// print: My name is John
setLastName("Lange");
// NOTHING!!! showFullName is false, "createEffect" results the same thing!
setShowFullName(true);
// print: My name is John Lange
```

## 2. Build a Reactive System

### `createSignal` and `createEffect` from scratch

```javascript
// create a stack of observers subscribing the signal,
// last in first out
const context = [];

function subscribe(observer, subscriptions) {
  subscriptions.add(observer);
}

export function createSignal(value) {
  const subscriptions = new Set();

  const read = () => {
    // get the lastest observer
    const observer = context[context.length - 1];
    // if observer exists, then add it as this signal's
    // subscriber
    if (observer) subscribe(observer, subscriptions);

    return value;
  };
  const write = (newvalue) => {
    value = newvalue;
    // now that the signal value is updated,
    // all the subscribers of this signal
    // should be rerun
    // p.s. we clone subcriptions here to avoid
    // infinite loop(see next snippet)
    for (const observer of [...subscriptions]) {
      observer.execute();
    }
  };

  return [read, write];
}

export function createEffect(fn) {
  const effect = {
    // after the first execution,
    // notice this is always executed after "value=newvalue" in "write"
    execute() {
      // make this effect observer of the signal
      // so that "read" in "fn" can find this effect
      context.push(effect);
      // then run the function that wraps "read"
      fn();
      // pop this effect out of the stack
      context.pop();
    },
  };
  // the first execution when effect created.
  effect.execute();
}
```

The implementation of "`createSignal`" and "`createEffect`" above is not optimal, as it does not prevent repetivie executions that cause the same result. For example, if we have the following snippet:

```javascript
const [count, setCount] = createSignal(0);
const [count2, setCount2] = createSignal(2);
const [show, setShow] = createSignal(true);

createEffect(() => {
  if (show()) console.log("count1:", count());
  else console.log("count2", count2());
});
// print count1 0

setShow(false);
// print: count2 2
setCount(10);
// repetitive print: count2 2
// but we don't want this! In real-world application
// this repetitive behavior is the source of sluggish reactions!
```

"`count2 2`" is printed twice above. To fix this, we can create a "`cleanup`" function to make `effec` and `count` forget about each other when `setShow(false)`.

```javascript
// this function decouples signal and effect
function cleanup(observer) {
  for (const dep of observer.dependencies) {
    // make signals forget about the effect
    dep.delete(observer);
  }
  // make effect forget about the signals
  observer.dependencies.clear();
}

// we also rewrite "subscribe" here to make
// signal and effect are linked in BOTH ways
function subscribe(observer, subscriptions) {
  subscriptions.add(observer);
  observer.dependencies.add(subscriptions);
}
export function createEffect(fn) {
  const effect = {
    execute() {
      cleanup(effect);
      context.push(effect);
      fn();
      context.pop();
    },
    dependencies: new Set(),
  };

  effect.execute();
}
```

Now, "`count2 2`" is only logged once. What happened above can be briefly summarized as the following set of steps:

- "`createEffect`" first execution makes the effect and the signals, "`count`" and "`show`", linked in both ways, because of the calls, "`show()`" and "`count()`"
- "`setShow(false)`" cleans up the links above, but the calls, "`show()`" and "`count2()`", in the `fn` makes the effect now linked with "`count2`" and "`show`"
- when "`setCount(10)`" is executed, the effect is no longer linked to "`count`", and "[ ...subscriptions]" is empty, so nothing new is printed out

### Implement "`createMemo`" & "`untrack`"

"`createMemo`" is commonly used to create **derived signal** from signals created by "`createSignal`". Here we create a simplified version of "`createMemo`" of SolidJS. Notice how "`fn`", "`setSignal`", and "`createEffect`" are nested below:

```javascript
function createMemo(fn) {
  const [signal, setSignal] = createSignal();
  createEffect(() => setSignal(fn()));
  return signal;
}

setCount(0);
setCount2(2);
// derived signal "sum" is based on "count" and "count2"
const sum = createMemo(() => count() + count2());

// assuming you're using count and count2 from the previous lisings
createEffect(() => console.log(count(), count2(), sum()));
// print: 0,2,2

setCount(10);
//print two lines
// 10 2 12
// 10 2 12
// what happened under the hood:
// `subscriptions` in `setCount` has two members:
// the effect in `createMemo`, and the one at the end
// execute of the first `createEffect` calls `setSignal(fn())` with `fn()->12`
// because `signal` created by `createMemo` subscribes one effect(the one at the end)
// `observer.execute()` in the function `setSignal(fn())` results in the first
// "10 2 12" printed out
// coming back to the for loop in `setCount`, we are now iterating at the effect
// at the bottom. `observer.execute()` now results in the second "10 2 12" printed
```

We now implement the `untrack` function, which takes another function as its argument, and allow argument function to be run without triggering any dependent effect.

```javascript
function untrack(fn) {
  const preContext = context;
  // we first empty current context,
  // so the function execution does NOT
  // trigger any subscription
  context = [];
  // we then fun the function
  const res = fn();
  // reload the context
  context = preContext;
  // return the result
  return res;
}

// assuming you're using createSignal and createEffect from above
const [count, setCount] = createSignal(0);

createEffect(() => console.log(untrack(count)));
// print 0
setCount(10);
// print nothing, as "subscriptions" is empty in count signal.
```

## 3. Rendering with Reactivity

The example in the previous section is essentially how SolidJS deals with reactivity. Lukily, SolidJS abstracts away the details of state management and effect trigger for us. In this section, let us use examples to show functionalities of SolidJS, and demisify its performance.

### Create component in SolidJS

In the following snippet, we create a simple counter component that increments by 1 per second, and also allows increments by clicking a button. Then rendered result can be checked [here](https://playground.solidjs.com/anonymous/67f946dc-897c-4094-a986-7ba6ee17cbcd)

```ts
import { createSignal} from "solid-js";
import { render } from "solid-js/web"

function Counter(){
    const [count, setCount] = createSignal(0);
    console.log("Counter");
    // print "Counter" only once, no matter
    // how many times you click the button

    return (
        <>
        {/*createEffect under the hood*/}
        <h1>{`the count is ${count()}`}</h1>
        {/* onClick is a createEffect under the hood */}
        <button onClick={()=>setCount(()=>count()+1)}>click</button>
        </>
    )
}

function App(){
    return <>
      {/*two counters have their own state mgmt*/}
      <Counter/>
      <Counter/>
    </>
}

render(App, document.body);
```

We can make the two counters sharing the same state as the following:

```javascript
import { createSignal } from "solid-js";
import { render } from "solid-js/web";

function Counter(props) {
  console.log("Counter");
  return (
    <>
      <h1>{`the count is ${props.children}`}</h1>
      <button onClick={props.onClick}>click</button>
    </>
  );
}

function App() {
  const [count, setCount] = createSignal(0);
  return (
    <>
      <Counter onClick={() => setCount(() => count() + 1)}>{count()}</Counter>
      <Counter onClick={() => setCount(() => count() + 2)}>
        {count() * 2}
      </Counter>
    </>
  );
}

render(App, document.body);
```

The rendered result can be found [here](https://playground.solidjs.com/anonymous/a9b63481-0947-40b6-a1ad-cb0d805290f1). Thus, the performance does NOT scale with number of components, or size of code, it scales on interactivity.

All rendering has an effect under the hood, so it does not matter how many components you have created, but the logic you want to implement with the signals. Like the "`props`" above is a good example of how we flatten the component logic by preparing properties with the signal "`count`".

As we will see in later sections, SolidJS deviates from other framework like Vue orReact in that rendering is tied to respond to data instead of rerendering full components whenever there is a change in data.

## 4. Control Flow

SolidJS ships with components for flow control, the table below gives an overview of what they are:

| conditionals       | use                                                 |
| :----------------- | :-------------------------------------------------- |
| `<Show/>`          | display content when a condition is met             |
| `<Switch/>`        | multi-conditional logic (ie select/case, if/elseif) |
| `<Dynamic/>`       | swap rendered element or component                  |
| `<ErrorBoundary/>` | display when an error is uncaught                   |

| Loops      | use                                       |
| :--------- | :---------------------------------------- |
| `<For/>`   | loop automatically keyed by reference/key |
| `<Index/>` | loop automatically keyed by index         |

### `<Show/>`: when signal=true then run callback

You can find the official tutorial [here](https://www.solidjs.com/tutorial/flow_show), but the example there can be rewritten by making the logout button keyed:

```javascript
{
  /*The fallback prop acts as the else and will show when the condition passed to when is not truthy.*/
}
<Show when={loggedIn()} fallback={<button>Log in</button>}>
  {/* logout button is keyed by loggedIn signal*/}
  {(loggedIn) => <button>Log out</button>}
</Show>;
```

By making the logout button keyed, we can re-render it whenever the "`loggedIn`" signal changes. By default, "`<Show/>`" only renders once when the signal first changes its value.

### `<Switch/>`: deal with multiple conditions

It's quite like switch in other programming languages, except that the "`default`" in this case is the "`callback`" of the "`Switch`" tag. Official tutorial [here](https://www.solidjs.com/tutorial/flow_switch).

```javascript
<Switch fallback={<p>{x()} is between 5 and 10</p>}>
  <Match when={x() > 10}>
    <p>{x()} is greater than 10</p>
  </Match>
  <Match when={5 > x()}>
    <p>{x()} is less than 5</p>
  </Match>
</Switch>
```

### `<Dynamic/>`: for multiple avail options

You can still use `<Switch/>` to do the same thing, but "`<Dynamic/>`" is more concise in the following example from the official tutorial:

```javascript
const RedThing = () => <strong style="color: red">Red Thing</strong>;
const GreenThing = () => <strong style="color: green">Green Thing</strong>;
const BlueThing = () => <strong style="color: blue">Blue Thing</strong>;

const options = {
  red: RedThing,
  green: GreenThing,
  blue: BlueThing,
};

function App() {
  const [selected, setSelected] = createSignal("red");

  return (
    <>
      <select
        value={selected()}
        onInput={(e) => setSelected(e.currentTarget.value)}
      >
        <For each={Object.keys(options)}>
          {(color) => <option value={color}>{color}</option>}
        </For>
      </select>
      <Dynamic component={options[selected()]} />
    </>
  );
}
```

### `<ErrorBoundary/>`: wraps and insulate the broken component

The following component will crash without the errorBoundary:

```javascript
{
  /*the p element will shown instead*/
}
<ErrorBoundary fallback={<p>broken component here</p>}>
  <Broken />
</ErrorBoundary>;
```

The callback prop above can also be function.

### `<For/>`: javascript `forEach` dress like a component

```javascript
function App() {
  const [cats, setCats] = createSignal([
    { id: "J---aiyznGQ", name: "Keyboard Cat" },
    { id: "z_AbfPXTKms", name: "Maru" },
    { id: "OUtn3pvWmpg", name: "Henri The Existential Cat" },
  ]);

  return (
    <ul>
      <For each={cats()}>
        {(cat, i) => (
          <li>
            <a
              target="_blank"
              href={`https://www.youtube.com/watch?v=${cat.id}`}
            >
              {i() + 1}: {cat.name}
            </a>
          </li>
        )}
      </For>
    </ul>
  );
}
```

### `<Index/>`: `<For/>` but with keyed index

The example above can be rewritten as the following. Notice how `i` is just a value not a function.

```javascript
<Index each={cats()}>
  {(cat, i) => (
    <li>
      <a target="_blank" href={`https://www.youtube.com/watch?v=${cat().id}`}>
        {i + 1}: {cat().name}
      </a>
    </li>
  )}
</Index>
```

### `<Portal/>`: make modals in document.body

The popup div element in the example below is outside of "`app-container`".

```javascript
function App() {
  return (
    <div class="app-container">
      <p>Just some text inside a div that has a restricted size.</p>
      <Portal>
        <div class="popup">
          <h1>Popup</h1>
          <p>Some text you might need for something or other.</p>
        </div>
      </Portal>
    </div>
  );
}
```

## 5. Managing Props and Lifecycles

### `mergeProps` for default values

```javascript
export default function Greeting(props) {
  const merged = mergeProps({ greeting: "Hi", name: "John" }, props);
  return (
    <h3>
      {merged.greeting} {merged.name}
    </h3>
  );
}
```

### `splitProps`: split props without losing reactivity

Destructing props in the following component makes them lose reactivity:

```javascript
export default function Greeting(props) {
  const { greeting, name, ...others } = props;
  return (
    <h3 {...others}>
      {greeting} {name}
    </h3>
  );
}
```

Instead, we can maintain reactivity with splitProps:

```javascript
function Greeting(props) {
  const [local, others] = splitProps(props, ["greeting", "name"]);
  return (
    <h3 {...others}>
      {local.greeting} {local.name}
    </h3>
  );
}
```

### `children`: wraps children elements with extra reactivity

SolidJS propagates updates by wrapping potentially reactive expressions in object getters, meaning that accesses to props are deferred until where they are used. But, repeating access can lead to repeating child components. For that reason, `children` helper creates a memo around the `props.children` to interact with the children directly. The following example shows its use:

```javascript
import { createEffect, children } from "solid-js";

function ColoredList(props) {
  const c = children(() => props.children);
  // interact with children by changing their colors
  // If we interacted with props.children directly,
  // not only would we create the nodes multiple times,
  // but we'd find props.children to be a function
  createEffect(() => c().forEach((item) => (item.style.color = props.color)));
  return <>{c()}</>;
}

function App() {
  const [color, setColor] = createSignal("purple");

  return (
    <>
      <ColoredList color={color()}>
        <For each={["Most", "Interesting", "Thing"]}>
          {(item) => <div>{item}</div>}
        </For>
      </ColoredList>
      <button onClick={() => setColor("teal")}>Set Color</button>
    </>
  );
}
```

### `onMount`: do things once html loaded

This helper hook is useful in use cases where you need to load some data after indicative html is loaded, e.g., "loading" indicator. "`onMount`" is just a createEffect call that is non-tracking, which means it never re-runs. It is just an Effect call but you can use it with confidence that it will run only once for your component, once all initial rendering is done.

```javascript
function App() {
  const [photos, setPhotos] = createSignal([]);
  onMount(async () => {
    const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
    setPhotos(await res.json());
  });

  return <>
    <h1>Photo album</h1>

    <div class="photos">
      <For each={photos()} fallback={<p>Loading...</p>}>{ photo =>
        <figure>
          <img src={photo.thumbnailUrl} alt={photo.title} />
          <figcaption>{photo.title}</figcaption>
        </figure>
      }</For>
    </div>
  </>;
}
```

### `onCleanup`: run things when current scope is triggered to re-evaluate 

The example blow keeps printing `cleanup`, as `onCleanup` reruns repetitively:

```javascript 
import { render } from "solid-js/web";
import { createSignal, onCleanup, createEffect} from "solid-js";

function Counter() {
  const [count, setCount] = createSignal(0);
    
    createEffect(()=>{
        // subsribe to this effect
        count()
        // trigger re-evaluate of the component
        const t = setInterval(() => setCount(count() + 1), 1000);
        // onCleanup reruns on every re-evaluation
        onCleanup(() => {
        clearInterval(t);
        console.log("cleanup");
        });
    });
    
  return <div>Count: {count()}</div>;
}

render(() => <Counter />, document.getElementById('app'));
```

## 6. DOM Binding 

SolidJS provides interface to interact with DOM directly. The following links are common ones to use.

- DOM Events tutorial [here](https://www.solidjs.com/tutorial/bindings_events)
- Use ClassList prop for component [tutorial](https://www.solidjs.com/tutorial/bindings_classlist)
- Create ref to element in JS script by adding ["ref"](https://www.solidjs.com/tutorial/bindings_refs)  attribute to JSX element
- Use [forwarding ref](https://www.solidjs.com/tutorial/bindings_forward_refs) to use references from parent component in current component
- Pass an object with a variable number of properties to component [tutorial here](https://www.solidjs.com/tutorial/bindings_spreads)
- `clickOutside` effect for modal made possible with [custom directive](https://www.solidjs.com/tutorial/bindings_directives)

## 7. State Management

SolidJS provides extra mechanism to support more granular updates. 

## 8. Stores

### Nested reactivity: updating list without Diffing

In SolidJS, a change made in one object element in a list will not trigger DOM diffing, and only one single location gets updated in DOM. It accomplishes this by using nested reactivity. In an example of Todo list app， one can define a todo item by using the following object structure:

```typescript
{
    id: number,
    text: string,
    completed: boolean
}
```
from which we can define a function "`toggleTodo`" that toggles todo item once it's complete as 

```typescript 
const toggleTodo = (id) => {
  setTodos(
    todos().map((todo) => (todo.id !== id ? todo : { ...todo, completed: !todo.completed })),
  );
};
```

There is nothing wrong with the method above, but if todo list is rendered using `<ul>` or `<For>`, then `toggleTodo` function will cause re-rendering of one of the todo item. This is not efficient enough because the only thing changed is `todo.completed`, so it's better to just change the DOM element relevant only to `todo.completed`. To achieve this, we can turn `completed` into a signal as well, i.e. fine-grained reactivity. The following initializes a todo item with nested signals:

```typescript 
const addTodo = (text) => {
  const [completed, setCompleted] = createSignal(false);
  setTodos([...todos(), { id: ++todoId, text, completed, setCompleted }]);
};
```
Now we can update the completion state by calling setCompleted without any additional diffing. This is because we've moved the complexity to the data rather than the view.

```typescript 
const toggleTodo = (id) => {
  const todo = todos().find((t) => t.id === id);
  if (todo) todo.setCompleted(!todo.completed())
}
```

Of course, you now should replace `todo.completed` with `todo.completed()`.

### Create Store: nested activity without extra signals 

The nested activity shown above can be achieved by using `createStore`. The create function takes an initial state, wraps it in a store, and returns a readonly proxy object and a setter function.

```typescript 
const [state, setState] = createStore(initialValue);
// read value
state.someValue;
// set value
setState("path", "to", "value", newValue);
```

In the example of todo list, `addTodo` and `toggleTodo` can be re-written as the following: 

```javascript 
const [todos, setTodos] = createStore([]);
const addTodo = (text) => {
  setTodos([...todos, { id: ++todoId, text, completed: false }]);
};
const toggleTodo = (id) => {
  setTodos(
    (todo) => todo.id === id,
    // completed is top-level, and thus, one string here
    "completed",
    // flip the value of completed
    (completed) => !completed
  );
};
```

## 9. Context 

A good example of using context in SolidJS can be found [here](https://www.solidjs.com/tutorial/stores_context).

## 10. Simple TODO list example 

With SolidJS, control over a todo list becomes more granular, and developer can be more focused on logic among the signals. However, there are still some caveats when nesting `setSignal` function with other functions as indicated by the comments in the example [here](https://stackblitz.com/edit/github-rsbzcu?file=src%2FApp.tsx)

## 11. Further Resources 

SolidJS might be often used for client-side rendering, but the community is moving towards SSR since 2019. The following links are worth checking if you want to use SolidJS for SSR. At the time of writing, the author still prefers `htmx`+simple `React` for SSR.

- SSR with SolidJS [solid-start](https://start.solidjs.com/getting-started/project-setup)
- Primitives [solid-primitives](https://github.com/solidjs-community/solid-primitives)
- More packages for SolidJS [solid-ecosystem](https://www.solidjs.com/ecosystem)
