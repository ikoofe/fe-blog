---
title: React整洁的代码-编写更好更整洁代码的有效方式
date: 2021-03-05 10:43:55
tags: React
---

> 本文源于翻译[React Clean Code - Simple ways to write better and cleaner code](https://dev.to/thawkin3/react-clean-code-simple-ways-to-write-better-and-cleaner-code-2loa	)

# React整洁的代码-编写更好更整洁代码的有效方式

整洁的代码不仅仅是能运行的代码，整洁的代码易于阅读，简单易懂，条理清晰。在本文中，我们将介绍八种编写 React 整洁代码的方法。

这篇文章仅提供了一些建议，如果不赞同也没有关系，这些实践对我个人编写 React 代码很有帮助。让我们开始吧！

## 1、只有一个条件的条件渲染

在条件渲染中，如果只有条件为 `true` 时渲染某些内容，而在条件为 `false` 时不渲染任何内容，请不要使用三目运算符，改用 `&&` 运算符。


Bad example:

```js
import React, { useState } from 'react'

export const ConditionalRenderingWhenTrueBad = () => {
  const [showConditionalText, setShowConditionalText] = useState(false)

  const handleClick = () =>
    setShowConditionalText(showConditionalText => !showConditionalText)

  return (
    <div>
      <button onClick={handleClick}>Toggle the text</button>
      {showConditionalText ? <p>The condition must be true!</p> : null}
    </div>
  )
}
```

Good example:

```js
import React, { useState } from 'react'

export const ConditionalRenderingWhenTrueGood = () => {
  const [showConditionalText, setShowConditionalText] = useState(false)

  const handleClick = () =>
    setShowConditionalText(showConditionalText => !showConditionalText)

  return (
    <div>
      <button onClick={handleClick}>Toggle the text</button>
      {showConditionalText && <p>The condition must be true!</p>}
    </div>
  )
}
```

## 2、两个条件的条件渲染

在条件渲染中，如果需要在条件为 `true` 时渲染一种内容，而在条件为 `false` 时渲染另一种内容，请使用三目运算符。

Bad example:

```js
import React, { useState } from 'react'

export const ConditionalRenderingBad = () => {
  const [showConditionOneText, setShowConditionOneText] = useState(false)

  const handleClick = () =>
    setShowConditionOneText(showConditionOneText => !showConditionOneText)

  return (
    <div>
      <button onClick={handleClick}>Toggle the text</button>
      {showConditionOneText && <p>The condition must be true!</p>}
      {!showConditionOneText && <p>The condition must be false!</p>}
    </div>
  )
}
```
Good example:

```js
import React, { useState } from 'react'

export const ConditionalRenderingGood = () => {
  const [showConditionOneText, setShowConditionOneText] = useState(false)

  const handleClick = () =>
    setShowConditionOneText(showConditionOneText => !showConditionOneText)

  return (
    <div>
      <button onClick={handleClick}>Toggle the text</button>
      {showConditionOneText ? (
        <p>The condition must be true!</p>
      ) : (
        <p>The condition must be false!</p>
      )}
    </div>
  )
}
```

## 3、布尔型 props

当组件的 prop 为 `true` 时，只需要提供属性名即可，例如：`myTruthyProp`，`myTruthyProp={true}` 这样的写法是没有必要的。

Bad example:

```js
import React from 'react'

const HungryMessage = ({ isHungry }) => (
  <span>{isHungry ? 'I am hungry' : 'I am full'}</span>
)

export const BooleanPropBad = () => (
  <div>
    <span>
      <b>This person is hungry: </b>
    </span>
    <HungryMessage isHungry={true} />
    <br />
    <span>
      <b>This person is full: </b>
    </span>
    <HungryMessage isHungry={false} />
  </div>
)
```

Good example:

```js
import React from 'react'

const HungryMessage = ({ isHungry }) => (
  <span>{isHungry ? 'I am hungry' : 'I am full'}</span>
)

export const BooleanPropGood = () => (
  <div>
    <span>
      <b>This person is hungry: </b>
    </span>
    <HungryMessage isHungry />
    <br />
    <span>
      <b>This person is full: </b>
    </span>
    <HungryMessage isHungry={false} />
  </div>
)
```

## 4、字符串 props

字符串属性值可以用双引号赋值，不需要使用大括号或反引号。

Bad example:

```js
import React from 'react'

const Greeting = ({ personName }) => <p>Hi, {personName}!</p>

export const StringPropValuesBad = () => (
  <div>
    <Greeting personName={"John"} />
    <Greeting personName={'Matt'} />
    <Greeting personName={`Paul`} />
  </div>
)
```

Good example:

```js
import React from 'react'

const Greeting = ({ personName }) => <p>Hi, {personName}!</p>

export const StringPropValuesGood = () => (
  <div>
    <Greeting personName="John" />
    <Greeting personName="Matt" />
    <Greeting personName="Paul" />
  </div>
)
```

## 5、事件处理函数

如果一个事件处理函数只接受 `Event` 对象这一个参数，可以像 `onChange={handleChange}` 这样直接将该函数作为事件处理对象，而不需要用匿名函数将该函数再包裹一层：`onChange={e => handleChange(e)}`。

Bad example:

```js
import React, { useState } from 'react'

export const UnnecessaryAnonymousFunctionsBad = () => {
  const [inputValue, setInputValue] = useState('')

  const handleChange = e => {
    setInputValue(e.target.value)
  }

  return (
    <>
      <label htmlFor="name">Name: </label>
      <input id="name" value={inputValue} onChange={e => handleChange(e)} />
    </>
  )
}
```
Good example:

```js
import React, { useState } from 'react'

export const UnnecessaryAnonymousFunctionsGood = () => {
  const [inputValue, setInputValue] = useState('')

  const handleChange = e => {
    setInputValue(e.target.value)
  }

  return (
    <>
      <label htmlFor="name">Name: </label>
      <input id="name" value={inputValue} onChange={handleChange} />
    </>
  )
}
```

## 6、传递组件作为 props

当组件作为 prop 被传递给另一个组件时，被传递的组件如果不需要其他的 props，则无需使用函数再包裹一层。

Bad example:

```js
import React from 'react'

const CircleIcon = () => (
  <svg height="100" width="100">
    <circle cx="50" cy="50" r="40" stroke="black" stroke-width="3" fill="red" />
  </svg>
)

const ComponentThatAcceptsAnIcon = ({ IconComponent }) => (
  <div>
    <p>Below is the icon component prop I was given:</p>
    <IconComponent />
  </div>
)

export const UnnecessaryAnonymousFunctionComponentsBad = () => (
  <ComponentThatAcceptsAnIcon IconComponent={() => <CircleIcon />} />
)
```

Good example:

```js
import React from 'react'

const CircleIcon = () => (
  <svg height="100" width="100">
    <circle cx="50" cy="50" r="40" stroke="black" stroke-width="3" fill="red" />
  </svg>
)

const ComponentThatAcceptsAnIcon = ({ IconComponent }) => (
  <div>
    <p>Below is the icon component prop I was given:</p>
    <IconComponent />
  </div>
)

export const UnnecessaryAnonymousFunctionComponentsGood = () => (
  <ComponentThatAcceptsAnIcon IconComponent={CircleIcon} />
)
```

## 7、未定义的 props

未定义的 props 是被自动拦截的，如果未定义的 prop 是允许的话，则不用担心未定义的回调。

Bad example:

```js
import React from 'react'

const ButtonOne = ({ handleClick }) => (
  <button onClick={handleClick || undefined}>Click me</button>
)

const ButtonTwo = ({ handleClick }) => {
  const noop = () => {}

  return <button onClick={handleClick || noop}>Click me</button>
}

export const UndefinedPropsBad = () => (
  <div>
    <ButtonOne />
    <ButtonOne handleClick={() => alert('Clicked!')} />
    <ButtonTwo />
    <ButtonTwo handleClick={() => alert('Clicked!')} />
  </div>
)
```

Good example:

```js
import React from 'react'

const ButtonOne = ({ handleClick }) => (
  <button onClick={handleClick}>Click me</button>
)

export const UndefinedPropsGood = () => (
  <div>                                                                                                               
    <ButtonOne />
    <ButtonOne handleClick={() => alert('Clicked!')} />
  </div>
)
```

## 8、state 赋值依赖于之前的 state

如果新的 state 依赖于之前的 state，则将之前的 state 作为参数，使用函数赋值的方式进行 state 赋值。React 的 state 更新是批量进行的，不这样写的话，更新时可能会导致意想不到的结果。

Bad example:

```js
import React, { useState } from 'react'

export const PreviousStateBad = () => {
  const [isDisabled, setIsDisabled] = useState(false)

  const toggleButton = () => setIsDisabled(!isDisabled)

  const toggleButton2Times = () => {
    for (let i = 0; i < 2; i++) {
      toggleButton()
    }
  }

  return (
    <div>
      <button disabled={isDisabled}>
        I'm {isDisabled ? 'disabled' : 'enabled'}
      </button>
      <button onClick={toggleButton}>Toggle button state</button>
      <button onClick={toggleButton2Times}>Toggle button state 2 times</button>
    </div>
  )
}
```

Good example:

```js
import React, { useState } from 'react'

export const PreviousStateGood = () => {
  const [isDisabled, setIsDisabled] = useState(false)

  const toggleButton = () => setIsDisabled(isDisabled => !isDisabled)

  const toggleButton2Times = () => {
    for (let i = 0; i < 2; i++) {
      toggleButton()
    }
  }

  return (
    <div>
      <button disabled={isDisabled}>
        I'm {isDisabled ? 'disabled' : 'enabled'}
      </button>
      <button onClick={toggleButton}>Toggle button state</button>
      <button onClick={toggleButton2Times}>Toggle button state 2 times</button>
    </div>
  )
}
```

## 其他实践

以下实践不是 React 特定的，而是用 JavaScript（以及任何编程语言）编写整洁代码的好实践。

- 将复杂逻辑提取到具有清晰名称的函数中
- 将魔法数字提取为常量
- 使用明确命名的变量

愉快的编码吧！

