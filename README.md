
# boxoid

**boxoid** is a tool for prefilling React component props, as well as a styling utility. It's tiny and has no external dependencies. It has a single export. When combined with a className generation function, it's actually more flexible than classical **styled** functions.


```js
import box from 'boxoid'
import { css } from '@emotion/css'

export const Button = box('button', {
  className: css({ background: 'red' }),
})

// use it inside React as:
return <Button />

// Adding a further className is possible. Both classes will be combined.
return <Button className="extra-class"/>
```

It has a first-class TypeScript support. Prefilling turns required props into optional props.

```js
import box from 'boxoid'

export const OutlinedButton = box(Button, {
  variant: 'outlined',
})
```

The second argument can also be a function that takes props, and returns the props to be prefilled.

```js
import box from 'boxoid'

export const OutlinedButton = box('button', (props: { isDisabled?: boolean }) => ({
  disabled: props.isDisabled,
}))
```

There's no need for using some `shouldForwardProp` mechanism to avoid passing `isDisabled` down. Destructed props are automatically marked as non-passdown props thanks to ES6 Proxy. 

Mutliple style variants is a great example that demonstrates the advantages of **boxoid** over classical **styled** functions.

```js
import box from 'boxoid'
import cx from 'classnames'
import { css } from '@emotion/css'

const classes = {
  root: css({ ... }),
  ...
}

export const Button = box(
  'button',
  (props: { variant?: 'default' | 'danger'; disabled?: boolean }) => ({
    classNames: cx([
      classes.root,
      classes.variant[props.variant || 'default'],
      { [classes.disabled]: props.disabled },
    ]),
    disabled: props.disabled,
  })
)

```

In classical **styled**, multiple variants like above are:
  - **Less developer-friendly** as they force the developer to a "ternary operator hell"
  - **Less performant** as they conduct their own className generation internally, and this requires deep-comparisons of CSS objects in every render.


## A criticism against Material-UI styled API

In MUI v4, different flavors of **styled** is exported, as its defaut form is not enough. There are MUI components that receive multiple classes using the `classes` prop. For one-off styling, the `classes` prop can be used, however for reusing styles, there's an additional **styled** flavor called `withStyles`.

With **boxoid**, there's no need for any additional API to learn. `box` + `css` combination is sufficient.

```js
const StyledComponent = box(MuiComponent, {
  classes: {
    root: classes.root,
    paper: classes.paper,
  }
})
```

You can think of **boxoid** as a meta-**styled** function.

```js
// A reductionist interpretation
const styled = (as, styles) => box(as, { className: css(styles) })
```

You can also use hooks inside the `box` function!

```js
import { useAtom } from '@xoid/react'
import type { Atom } from 'xoid'

const Input = box('input', (props: { atom: Atom<string> }) => {
  value: useAtom(props.atom), 
  onChange: (e) => props.atom.set(e.target.value)
})
```
> [**xoid**](https://github.com/onurkerimov/xoid) is a framework-agnostic state management library by me. It aims to unify the API for local state, global state, and finite state machines. [See its website!](https://xoid.dev)

## Extension

Perhaps you want to add a `cx` prop that uses your favorite classnames library internally:

```js
import _box, { Box } from 'boxoid'
import cx, { ClassValue } from 'clsx'

const cxPlugin = (
  left: { cx?: ClassValue; className?: string },
  right: { cx?: ClassValue; className?: string }
) => {
  const { cx: cx1, className: cn1 } = left;
  const { cx: cx2, className: cn2 } = right;
  delete left.cx;
  delete right.cx;
  if (cx1 || cn1 || cx2 || cn2) right.className = cx(cx1, cn1, cx2, cn2);
};

export const box: Box<{ cx?: ClassValue }> = _box.bind(cxPlugin)
```

Plugins receive shallow copies of the following objects. It's safe to manipulate these objects in a mutable way.

```js
const Div = box('div', () => %left%)
const Div = box('div', %left%)
React.createElement(Div, %right%)
```