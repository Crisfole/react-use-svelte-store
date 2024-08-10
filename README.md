# react-use-svelte-store

I like [Svelte](https://svelte.dev). You should too. Sometimes, though, I have to use [React](https://reactjs.com) for legacy projects or because it's what my clients want.

When I do have to use React, I really miss the state management built in with [Svelte](https://svelte.dev). The stores are beautifully simple and easy to comprehend. Updating them requires no redux action, no complex reducers, no higher order components that feed the component with the content of the stores.

React hooks enable behavior that's _really_ similar to Svelte's reactive assignments. Sharing global state in react _should_ be as easy as importing the state-containing store, and should _not_ require Context providers and consumers, redux reducers, higher order components, or any other such hacks. This package merges the two.

> [!WARNING]  
> This repository is old. React 18 and higher have the `useSyncExternalStore` hook which would be a more efficient tool for this. Eventually I will update this repository. Until then, PR's welcome.

## How do I get started?

I'm assuming you have react installed. If not, well, figuring that out is on you.

 1. `npm install react-use-svelte-store`
 2. Create a file `stores.ts`.
 3. Create a svelte store: `export const foos = writable<Foo[]>([]);`
 4. Consume the store in a component: `const $foos = useReadable(foos);`
 5. Update the store in a component: `foos.update(f => f.concat(new Foo()))`

I recommend keeping the svelte convention of 'dereferencing' the store value into a variable prefixed with `$`. It reminds you to pause and think.

## Docs

The package exports two hooks: `useReadable` and `useWritable`. The hooks are designed to mimic the `useState` hooks, returning the value and a setter. It also re-exports the core svelte stores so you don't have to include svelte as a dependency. Full documentation for the svelte stores can be found [here](https://svelte.dev/docs#svelte_store).

### useReadable

`useReadable` accepts a svelte store and returns the content of that store. When the content of the store changes, the state is updated, and the component is re-rendered.

#### Signature

```tsx
function useReadable<T>(store: Readable<T>): T;
```

#### Example

```tsx
import React from 'react';
import { useReadable } from 'react-use-svelte-store';
import { foos } from '../state';

export const MyList = () => {
    const $foos = useReadable(foos);
    return (
        <ul>
            {$foos.map(foo => (
                <li key={foo.id}>{foo.name}</li>
            ))}
        </ul>
    );
};
```

### useWritable

`useWritable` accepts a svelte store and returns an array (like `useState`) of the content of that store, a setter for the content of the store, and an updater (as in svelte). When the content of the store changes, the state is updated, and the component is re-rendered. The setter and updater returned will be constant as long as the store passed into `useWritable` is constant.

There is no real benefit to `useWritable` over `useReadable` and directly updating the store with `set` and `update`, but it's left here as a mnemonic for React developers.

#### Signature

```tsx
function useWritable<T>(store: Writable<T>): [T, (t: T) => void, (fn: (t: T) => T) => void ];
```

#### Example

```tsx
import React from 'react';
import { useWritable } from 'react-use-svelte-store';
import { foos } from '../state';

export const MyList = () => {
    const [$foos, setFoos, updateFoos] = useWritable(foos);
    return (
        <div>
            <ul>
                {$foos.map((foo, ix) => (
                    <li key={foo.id}>
                        <FooEditor
                            foo={foo}
                            onChange={(foo) => {
                                updateFoos(oldFoos => oldFoos.splice(ix, 1, foo))
                            }}
                        />
                    </li>
                ))}
            </ul>

            <button onClick={() => setFoos([])}>Clear</button>
        </div>
    );
};
```

### writable

| :information_source: Re-export from Svelte, for full documentation follow link. |
|:--------------------------------|

[`writable`](https://svelte.dev/docs#writable) creates a store that can be set, updated, and subscribed to externally.

### readable

| :information_source: Re-export from Svelte, for full documentation follow link. |
|:--------------------------------|

[`readable`](https://svelte.dev/docs#readable) creates a store that cannot be updated externally, the function it receives as its parameter is the only place in code that can alter the contents of the store.

### derived

| :information_source: Re-export from Svelte, for full documentation follow link. |
|:--------------------------------|

[`derived`](https://svelte.dev/docs#derived) creates a store based on the content of one or more other stores. It cannot be updated externally except by updating its 'upstream' stores.

### get

| :information_source: Re-export from Svelte, for full documentation follow link. |
|:--------------------------------|

[`get`](https://svelte.dev/docs#get) is an inefficient way to get the value out of a store in a one off fashion. Using it in the body of your component is probably a mistake. Using it inside event handlers is perfectly fine.
