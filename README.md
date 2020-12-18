# react-use-svelte-store

I like [Svelte](svelte.dev). You should too. Sometimes, though, I have to use [React](reactjs.com) for legacy projects or becuase it's what my clients want.

When I do have to use React, I really miss the state management built in with [Svelte](svelte.dev). The stores are beautifully simple and easy to comprehend. Updating them requires no redux action, no complex reducers, no higher order components that feed the component with the content of the stores.

React hooks enable behavior that's _really_ similar to Svelte's reactive assignments. Sharing global state in react _should_ be as easy as importing the state-containing store, and should _not_ require Context providers and consumers, redux reducers, higher order components, or any other such hacks. This package merges the two.

## Bundle Size?

Don't worry. Only a small portion of svelte is actually runtime. `svelte/store` is part of that runtime. If you're using [Rollup](https://rollupjs.org) or [Webpack](https://webpack.js.org) properly it will treeshake the code and your bundle size will only increase by the size of the imported functions from 'svelte/store'.

## How do I get started?

I'm assuming you have react installed. If not, well, figuring that out is on you.

 1. `npm install svelte react-use-svelte-store`
 2. Create a file `stores.ts`.
 3. Create a svelte store: `export const foos = writable<Foo[]>([]);`
 4. Use the store in a component: `const $foos = useReadable(foos);`

I recommend keeping the svelte convention of 'dereferencing' the store value into a variable prefixed with `$`. It reminds you to pause and think.

## Docs

The package exports two hooks: `useReadable` and `useWritable`. These hooks are designed to at least nominally mimic the `useState` hooks.

### useReadable

`useReadable` accepts a svelte store and returns the content of that store. When the content of the store changes, the state is updated, and the component is re-rendered.

#### Signature

```
function useReadable<T>(store: Readable<T>): T;
```

#### Example

```
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

```
function useWritable<T>(store: Writable<T>): [T, (t: T) => void, (fn: (t: T) => T) => void ];
```

#### Example

```
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
