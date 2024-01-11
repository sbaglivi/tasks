You can split your code into components. Component names should start with an uppercase letter.
You can inject variables into your HTML by wrapping them in `{}`
If you need a string containing HTML to be dynamically parsed by your page, you can insert it into `{@html yourVar}` **this does not sanitize your input, be careful**

declaring a variable that changes whenever a button is pushed is as easy as:
```
<script>
    let count = 0;

    function increment() {
        count += 1;
    }
</script>
<button onclick:increment>
    Click me
</button>
<p>The value of count is: {count}</p>
```

You can also declare variables or actions that need to be updated whenever a "state" variable changes:
```
    $: double = count * 2;
    // or
    $: {
        double = count * 2; // actually this does not work, I guess if you need to declare a new variable you need to use syntax from above
        console.log(`The value of count has been updated to: ${count}`)
    }
```

In order for svelte to detect a change, it needs an assignments.
Certain array methods, like `push` won't trigger an update.
The idiomatic solution is to do `numbers = [...numbers, newValue]`
Rule of thumb is that variable that needs to be updated has to appear on left side of = operator.

To pass down data to a children component
In parent:
```
<script>
import Children from './Children.svelte';
</script>
<Children value={x} />
```
In children:
```
<script>
export let value;
</script>
<p>value is {value}</p>
```
Children props can have default values, replace line above with
`export let value = 42;`

If you have an objet containing props you'd like to pass down to a child component, rather than passing them one by one, you can spread them
```
props = {
    name: 'name',
    surname: 'surname',
    age: 0
}
<Children name={props.name} surname={props.surname} age={props.age} />
<Children {...props} />
```

You can conditionally render elements by wrapping them in an if block
```
{#if var > 5}
    <p>Var is greater than 5</p>
{:else if count < 5}
    <p>Var is less than 5</p>
{:else}
    <p>Var is 5</p>
{/if}
```

For variables that have a length (like arrays) that we want to iterate over, we can use the each block. The optional second value is the index
```
let colors = ['red', 'green', 'blue'];
{#each colors as color, i}
    <button style="background: {color}">
        {i+1}
    </button>
{/each}
```

When iterating over objects it's important to specify a key to identity each item. You can use the object themselves as keys but if possible
it's preferrable to use an id or name, because when data changes, svelte is able to recognize that an object has not changed despite not having
referential equality (despite the reference to where the value is stored not being the same).
```
{#each colors as color (color.id)} // what's between () is used as id
```

Often we have to deal with async data when building websites. Svelte has an HTML construct to make this easier:
```
{#await promiseName}
    <p>Waiting for result</p>
{:then number}
    <p>Result is: {number}</p>
{:catch error}
    <p>There was an error: {error}</p>
{/await}
```
You can omit the catch block if you're sure that the promise won't reject.
In case you don't want to show anything while you're waiting for the promise result, you can also omit the first block:
`{#await promiseName then number}...{/await}`

In general, you can listen to events on the dom with the `on:` keyword
`<div on:pointermove={handleMove}>`
Event handlers can also be declared inline `on:pointermove={(e) => {...}}`

You can add modifiers to event conditions example `on:click|once` will execute the handler only the first time the element is clicked.
Modifiers can be chained `on:click|once|self`, some of the most useful ones are:
- once: only the first time
- self: only if event target is the element itself
- preventDefault: runs e.preventDefault() before running the handler

You can dispatch custom events by using `createEventDispatcher`:
```
const dispatch = createEventDispatcher();
...
dispatch('yourEventName', {text: "Hello!"})
```
The first value you pass to dispatch is the name of the event, the second value is available under `event.detail`, "Hello!" will be under `event.detail.text`.
Another component can listen to this event with `on:yourEventName={handlerName}`

In svelte events do not bubble up indefinetely like in the DOM. They stop at the level in which the component that dispatches the event is located.
If you want an element to be forwarded you can declare in the 'intermediate component' an empty event handler
`<Inner on:yourEventName />`

Apparently when working with svelte we need to forward DOM events as well. For example if we have a button in a component but we want to handle it in the parent
component we do:
```
// in parent
<BigButton on:click={handleClick}>
// in child
<button on:click >Click</button>
```

When we need state to flow bottom up, we can use bindings:
```
let name = 'world'
<input bind:value={name} />
<p>Hello {name}!</p>
```

If you're dealing with a numeric input fields, svelte detects this and automatically converts the bound value to a number instead of a string.

When dealing with checkboxes, we bind to the checked property instead `bind:checked={yes}`

We can use bindings with selects as well:
```
options = [
    {
        id: 1,
        text: 'option 1'
    },
        {
        id: 2,
        text: 'option 2'
    }
]
let selected;
<select bind:value={selected}>
{#each options as option (option.id)}
    <option value={option}>
        {option.text}
    </option>
{/each}
</select>
```
In this example we didn't initialize selected to a value, until the binding is initialized we need to check if selected exist before trying to access one of its properties e.g. id

You can use `bind:group={varName}` to link a list of inputs (radio or checkboxes) to a single variable.
Radio groups are mutually exclusive, so the variable linked will take the value of the active radio input.
Checkbox groups are not exclusive, so the variable can be an array that takes on the list of active values.
```
possible = ['red', 'green', 'blue']
sizes = ['s', 'm', 'l']
let chosenSize = 1;
let chosenColors = [];
{#each sizes as size (size)}
    <label>
        <input value={size} type="radio" name="size" bind:group={chosenSize} />
    </label>
{/each}
{#each possible as color (color)}
    <label>
        <input value={color} type="checkbox" name="color" bind:group={chosenColors} />
    </label>
{/each}
```

The functionality of accumulating active values into an array, is also achievable with a select multiple.
```
<select bind:value={chosenColors}>
{#each colors as color (color)}
    <option>{color}</option>
{/each}
</select>
```
We were also able to omit the `value={v}` on the option object since the value is identical to the element's contents.

We can bind textareas just like we do text inputs: `<textarea bind:value={value}></textarea>`

When the name of a property matches the name of the variable to which it needs to be connected we can shorten it `bind:value={value}` -> `bind:value`. `src={src}` -> `{src}`

Lifecycle hooks:
to run a function when a component is rendered we use the `onMount` hook
```
import {onMount} from 'svelte';
onMount(() => {
    console.log("component mounted!");
    return () => {
        console.log("component unmounted!")
    }
})
```
If we need a clean-up function we need to return it from the onMount hook.

Other lifecycle hooks are `beforeUpdate` and `afterUpdate`. The first runs even before the component is mounted, so if you need to reference the component to which
the hook is connected you need to check for its existence first.

Another useful lifecycle hook is `tick`. 
When updates are made to the DOM in svelte, they're sent in batches. Using the tick function you can wait until the next DOM update before doing an action  
```
await tick();
this.selectionStart = selectionStart;
this.selectionEnd = selectionEnd;
```

Stores exist for state that does not belong inside the application's component hierarchy.
```
// declare writeable store
import {writable} from 'svelte';
export const count = writable(0);
// in another module, import the state var and update it 
import {count} from './stores.js';

function increment() {
    // update takes a function which gets the current value as first param
    count.update(n => n+1)
    // set just takes a value
    count.set(0);
}

// you can subscribe to a store, and be notified when it changes, by using subscribe
let localVar;
// the return value of subscribe is an unsubscribe function
const unsubscribeFunc = count.subscribe(v => localVar = v)
// bind it to a lifecycle hook to correctly remove the subscription and avoid memory leaks that would occur if 
// component was created and destroyed multiple time without cleaning subscription.
import {onDestroy} from 'svelte';
onDestroy(unsubscribe);

```

Rather than subscribe and unsubscribe and using a local var, there is a much shorter way of doing things which is just using the store
value through `<p>{$count}</p>`. This can only be done if the store variable is imported or declared at the top level scope of a component.

You can also create read-only stores. These stores can only be written to from the module where they're instantiated, other components can only read their content.
```
// takes initial value for store and a function responsible to update it, start function gets called when store gets its first subscriber
export const time = readable(new Date(), function start(set) {
    let interval = setInterval(() => {
        set(new Date())
    }, 1000)
    // return value is a clean up function that gets executed when last subscriber unsubscribes
    return function stop() {
        clearInterval(interval);
    }
})
```

You can create create derived stores 
`export const greeting = derived(name, ($name) => `Hello ${$name}!`);`

You can create custom store that expose only some of the original store methods
```
const {subscribe, set, update} = writeable(0);
return {
    subscribe,
    increment = () => update(n => n+1),
    decrement = () => update(n => n-1),
    reset: () => set(0)
}
```

If a store is writeable you can directly bind to the value of the store
`<input bind:value={$name} />`
This is also a valid way to update a writable store `on:click={() => $name += '!'}`