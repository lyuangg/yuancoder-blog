---
title: "Nextjs 全栈框架 CSR"
date: 2024-08-29T21:42:58+08:00
draft: false
tags:
- react
- nextjs
- javascript
- 全栈
- CSR
categories:
- 开发
---

<!--more-->


Next.js 是一个 基于 React 的全栈开发框架, 国外很多全栈开发都会使用这个框架。 


Next.js 可以开发服务端和前端， 支持3中渲染页面方式。

1. 服务器端渲染（Server-Side Rendering, SSR）, 类似于 PHP 模式。
2. 静态站点生成（Static Site Generation, SSG）, 相当于 PHP 生成静态页面。
3. 客户端渲染（Client-Side Rendering, CSR）, 这个是默认的 React 特性。


 服务端的特性主要是： API 路由 和 中间件(Middleware)


 SSR 和 SSG 的渲染模式，官方文档都讲的很详细，也很容易理解和实现。

 这篇文章主要讲一下 CSR 的实现方式：


#### 1. 创建服务端API 接口，通过 Next.js 的 API 路由实现。


 ```js
 // pages/api/data.js

export default function handler(req, res) {
  res.status(200).json({ message: 'Hello, World!' });
}
 ```

#### 2. 使用 useEffect 钩子进行客户端渲染 

```js
// pages/bsr-page.js
import { useEffect, useState } from 'react';

export default function BSRPage() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data')
      .then(response => response.json())
      .then(data => setData(data));
  }, []);

  return (
    <div>
      <h1>Data from BSR</h1>
      {data ? (
        <pre>{JSON.stringify(data, null, 2)}</pre>
      ) : (
        <p>Loading...</p>
      )}
    </div>
  );
}
```

#### 使用动态导入（Dynamic Import）的方式

Next.js 支持通过 next/dynamic 模块来进行动态导入，这可以让你将某些组件推迟到客户端渲染时才加载：

```js
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(() => import('../components/HeavyComponent'), {
  ssr: false,
});

export default function Page() {
  return (
    <div>
      <h1>Client-Side Rendering with Dynamic Import</h1>
      <DynamicComponent />
    </div>
  );
}

```
