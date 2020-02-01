![](https://badge.fury.io/js/%40elm-react%2Fcomponent.svg) ![](https://david-dm.org/parasrah/elm-react-component.svg) ![](https://github.com/parasrah/elm-react-component/workflows/tests/badge.svg)

# [@elm-react/component](#description)

TODO: description
TODO: note to read about Elm & React, include links to both

## [Install](#install)

Using npm:

`npm i -S @elm-react/component`

Using yarn:

`yarn add @elm-react/component`

### Setting up Elm & React

This readme only covers the most common setup for Elm + React web projects. If your needs deviate from this, you should find the information you need on [the Elm website](https://elm-lang.org/).

> Regardless of which method you choose, please visit the [Assets](https://github.com/Parasrah/elm-react-component#assets) section of the readme to ensure you are only bundling a single Elm runtime in your application.

Adding Elm to your existing webpack configuration is fairly easy through the use of [elm-webpack-loader](https://github.com/elm-community/elm-webpack-loader), which has instructions to set it up [here](https://github.com/elm-community/elm-webpack-loader#elm-loader---). After you've set this up, you should be able to import Elm modules as seen in the [description](https://github.com/Parasrah/elm-react-component#description) above.

If instead you are using Create React App (which significantly limits configurability), you can visit [the CRA example](https://github.com/Parasrah/elm-react-component/tree/improvements/example#cra-example) in this repo which should guide you through that process.

## [Usage](#usage)

The goal of this library is to adopt familiar patterns from both React & Elm. With React we just use props:

```javascript
// Counter.jsx

import wrap from '@elm-react/component'

import Counter from './Counter.elm'

export default wrap(Counter)
```

```javascript
// App.jsx

import React, { useState } from 'react';

import Counter from './Counter'

function App() {
  const [count, setCount] = useState(0)

  return (
    <div className="App">
      <Counter
        className="counter"
        value={count}
        onChange={(change) => setCount(count + change)}
      />
    </div>
  );
}
```

And with Elm you just create an element and use ports:

```elm
-- Counter.elm

port className : (String -> msg) -> Sub msg


port value : (Int -> msg) -> Sub msg


port onChange : Int -> Cmd msg

.
.
.

main : Program Flags Model Msg
main =
    Browser.element
        { view = view
        , update = update
        , init = init
        , subscriptions = subscriptions
        }
```

The library will automatically convert React props to Elm ports:

* function props are converted to outgoing ports in Elm
* all other props are converted to incoming ports in Elm (subscriptions)

these functions are capable of receiving a single argument from the Elm element

### [Opts](#opts)

### [Complex Props](#complex-props)

There are two things you can pass in as props to an Elm component (provided you have defined their respective ports). As you saw above, you can pass in functions which will automatically be translated to outgoing ports, and you can pass in objects & primitives which get translated to incoming ports. Where this breaks down is objects that contain functions as properties.

For example:

```javascript
const MyElmComponent = wrap(MyElmModule)

export default () => (
  function onChange () {
    console.log('I changed!')
  }
  <MyElmComponent info={{ onChange, value: 1 }} />
)
```

This snippet is passing a complex object that contains a function as one of its properties. While this is completely valid, this will be translated to an incoming port, giving you no way to access the function because you can't decode it! In most cases, it's better to flatten your state like this:

```javascript
const MyElmComponent = wrap(MyElmModule)

export default () => (
  function onChange () {
    console.log('I changed!')
  }
  <MyElmComponent onChange={onChange} value={1} />
)
```

Now you can access all values, with the added benefit of having an easier time writing the decoder for the port.

### [Common Pitfalls](#pitfalls)

There is a drawback to having props automatically injected into an Elm element in that you have to be more aware of what is rendering your Elm components. For example, react-router injects props into every component it renders, which is probably not what you want. Currently the advice is to explicitly pass props to your Elm component. If this affects you and you have ideas on how to improve it, please feel free to open an issue with your ideas. 

## [Performance](#perf)

## [Asset Size](#assets)

Generally, Elm produces very small assets when compared to other frameworks (like React and Angular). Similar to those other frameworks though, it requires a runtime to operate. This is dealt with by the Elm compiler and isn't typically something you have to worry about, but in the case of using multiple `Browser.element`'s, it's easy to *accidentally bundle multiple instances of the Elm runtime into your app*. Luckily, it's also easy to avoid!

### With elm-webpack-loader

TODO: update w/ code from rescripts-elm to automatically pull in elm files, or if unable to do so instruct users on how to do themselves

### With CRA

There is currently [an issue](https://github.com/Parasrah/rescripts-elm/issues/3) open on the `@elm-react/rescripts-elm` package to deal with this problem.

## [Create React App](#cra)
