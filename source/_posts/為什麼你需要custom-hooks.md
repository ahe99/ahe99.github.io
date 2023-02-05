---
title: 為什麼你需要custom hooks（React）？
comments: false
cover: /img/posts/為什麼你需要custom-hooks/cover.png
date: 2022-02-06 00:47:04
updated:
tags:
  - 前端
  - React
  - Custom hooks
categories:
  - 技術
keywords:
description:
top_img:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---

---

## 前言

這篇所要提及的內容，是很菜很菜的時候認識到的，

深入學習了之後彷彿世界明亮了起來～

導入到了老舊的專案上面，很大程度減低了 React 組件檔案的複雜度

也讓我從一天一顆葉黃素變到了兩天一顆 🥲

## 事前須知

- ES6 destructuring assignment/ async + await
- axios/ request
- Basic React hooks and how they work
  - useState
  - useEffect

## 模擬場景

假設今天要來做一個 Todo-list ~~aka 前端框架的 Hello word~~

大致上結構上會長這個樣子:

一個頁面，一個 List，然後渲染出對應數量個 Item

![todo-page](/img/posts/為什麼你需要custom-hooks/todo-page.png)

於是照著這個思路

我們把`TodoPage`完成了 🥳

```js
//TodoPage.js
import { useEffect, useState } from 'react'
import axios from 'axios'

import { SomeLoader, TodoList } from './components'

const TodoPage = () => {
  const [todos, setTodos] = useState({
    data: [],
    isLoading: false,
    isError: false,
  })

  useEffect(() => {
    fetchTodos()
  }, [])

  const fetchTodos = async () => {
    setTodos((prev) => ({
      ...prev,
      isLoading: true,
    }))
    try {
      const { data } = await axios.get('https://api.fake.com/todos')
      const formattedData = formatData(data)

      setTodos((prev) => ({
        ...prev,
        data: formattedData,
        isLoading: false,
      }))
    } catch (error) {
      //error handling
      setTodos((prev) => ({
        ...prev,
        isLoading: false,
        isError: true,
      }))
    }
  }

  const formatData = (data) => {
    //data formatting
  }

  return (
    <div className="todolist">
      {todos.isLoading ? <SomeLoader /> : <TodoList todos={todos.data} />}
    </div>
  )
}
```

Ok 那這時候寫完了，會發現我們在**第 45 行**之後才終於碰到了 UI 的部分

而**8~46 行**都在做資料方面的事情

那那那...如果今天還要加上刪除、新增、修改

總共會有幾行呢？😨 如果還要加上篩選器呢 😨

...

於是

...

...

最後你會發現它變成了\_\_\_

在維護的時候不停上下跳動 💩

![jumping](/img/posts/為什麼你需要custom-hooks/jumping.png)

## Custom hooooook

~~為了鍵盤的安全著想~~，趕緊來認識一下 React custom hook

- 命名規則: useXXX.js
- 可複用，hook 之間可互相結合
- 不同組件/hook 會擁有獨立的 state

篇幅關係，有機會再跟大家介紹 hooks 的限制及規則

所以我們可以創建一個`useTodos`

並把相關的東西全丟進去

```js
//useTodos.js
import { useState, useEffect } from 'react'

export const useTodos = () => {
  const [todos, setTodos] = useState({
    data: [],
    isLoading: false,
    isError: false,
  })

  useEffect(() => {
    fetchTodos()
  }, [])

  const fetchTodos = async () => {
    setTodos((prev) => ({
      ...prev,
      isLoading: true,
    }))
    try {
      const { data } = axios.get('https://jsonplaceholder.typicode.com/posts')
      setTodos((prev) => ({
        ...prev,
        data: data,
        isLoading: false,
      }))
    } catch (error) {
      //error handling
      setTodos((prev) => ({
        ...prev,
        isLoading: false,
        isError: true,
      }))
    }
  }

  return {
    data: todos.data,
    isLoading: todos.isLoading,
    isError: todos.isError,
  }
}
```

然後回到我們一開始的`TodoPage`

優雅的引入並使用剛剛寫好的`useTodos`

於是乎世界又恢復了和平 😁

```js
//TodoPage.js (after)
import { useEffect, useState } from 'react'

import { SomeLoader, TodoList } from './components'
import { useTodos } from './useTodos'

const TodoPage = () => {
  const todos = useTodos()

  return (
    <div className="todolist">
      {todos.isLoading ? <SomeLoader /> : <TodoList todos={todos.data} />}
    </div>
  )
}
```

### 後話

雖然使用 custom hooks 很方便

但也由於它的便利性，也可能會造成災難

因此不論是新舊專案，事先做點規劃還是很重要的
