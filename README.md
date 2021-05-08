## 准备环境

涉及工具：umi-library

### 初始化项目

```js
# 创建目录
$ mkdir umi-library-demo && cd umi-library-demo

# 初始化
$ yarn init -y

# 安装依赖
$ yarn add umi-library --save-dev
```

添加配置文件.umirc.library.js

```js
export default {
  entry: 'src/index.js',
  esm: 'rollup',
  cjs: 'rollup',
};
```

给 package.json 添加 script:

```js
+ "scripts": {
+    "doc:dev": "umi-lib doc dev"
+ },
```

这时, 你已经可以通过以下命令跑起来:

```js
$ yarn run doc:dev
```

浏览器访问http://127.0.0.1:8001/，即可看到我们的组件开发环境。

### 开发组件

规划目录结构, 入口为 src/index.js，Foo 为我们的第一个组件
.
├── .umirc.library.js # 配置
├── package.json
└── src
├── Foo # 组件
│ └── index.js
└── index.js # 入口

Foo 组件代码如下:

```js
// src/Foo/index.js
import * as React from 'react';

export default function (props) {
  return (
    <button
      style={{
        fontSize: props.size === 'large' ? 40 : 20,
      }}
    >
      {props.children}
    </button>
  );
}
```

接下来跑一下我们的组件，在 src/Foo 目录下创建 index.mdx，基于 mdx，你可以使用 markdown 加 jsx 语法来组织文档。

```js
---
name: Foo
route: /
---

import { Playground } from 'docz';
import Foo from './';

# Foo Component

## Normal Foo

<Foo>Hi</Foo>

## Large Foo with playground

<Playground>
    <Foo size="large">Hi</Foo>
</Playground>
```

### 组件打包

组件开发测试完成后，需要打包成不同的产物以适应不同的场景。默认使用 rollup 打包生成三个格式的包:

- cjs: CommonJs，能被 Node 和 打包工具如 webpack 使用。
- esm: ES Module，支持静态分析可以 tree shaking。
- umd: Universal Module Definition 通用包，既能像 cjs 一样被使用，也可以发布到 cdn，通过 script 的方式被浏览器使用。
  修改 package.json

```js
-  "main": "index.js",
+  "main": "dist/index.js",
+  "module": "dist/index.esm.js",
   "scripts": {
    "doc:dev": "umi-lib doc dev",
+   "dev": "umi-lib build --watch",
+   "build": "umi-lib build",
    "test": "umi-test"
  },
```

使用命令

```js
# 监控文件变化并打包
$ yarn run dev

# 打包
$ yarn run build
```

### 验证产物

为了验证我们的产物是否可用，我们可以基于 umi 创建一个小 demo 使用一下,，在项目下创建目录 example，目录结构:
example/
└── pages
└── demo-foo
└── index.js
我们创建了 demo-foo 这个页面, 并使用 Foo 组件，其 index.js 代码:

```js
import { Foo } from '../../../dist';

export default function () {
  return <Foo size="large">hello, world</Foo>;
}
```

我们跑一下

```js
$ cd example
$ umi dev

# 如果没有 umi 这个命令, 请安装
$ yarn global add umi
```

### 发布组件

组件开发好，发布到 npm registry 就可以被大家使用，也可以发布到私有 registry 内部使用。如果没有 npm 账号需要先注册，然后登陆 yarn login。
修改 package.json，添加发布 script，发布前执行测试用例，并且包里只含 dist 目录:

```js
+ "files": ["dist"],
  "scripts": {
+   "pub": "yarn run test && yarn publish",
    "test": "umi-test"
  },
```

执行命令

```js
$ yarn run pub
```

发布成功后，你就可以在 npm 看到 umi-library-demo
对于其他用户，就可以使用以下命令来安装使用这个包。

```js
# 使用 yarn
$ yarn add umi-library-demo --save

#使用 npm
$ npm install umi-library-demo --save
```

### 发布文档

在我们的组件开发完毕，文档相应写完后我们需要打包和部署文档，以便使用者查阅。
首先修改 package.json，添加 script:

```js
  "scripts": {
    "doc:dev": "umi-lib doc dev",
+   "doc:build": "umi-lib doc build",
+   "doc:deploy": "umi-lib doc deploy",
  },
```

umi-library 默认将文档部署到 github.io, url 规则是 https://{yourname}.github.io/{your-repo}，我们需要修改.umirc.library.js 配置一下文档静态资源的前缀 base。

```js
export default {
  entry: 'src/index.js',
  esm: 'rollup',
  cjs: 'rollup',
+ doc: {
+   base: '/umi-library-demo'
+ }
}
```

接下来执行命令:

```js
# 打包文档
$ yarn run doc:build

# 部署文档, 速度取决于网速
$ yarn run doc:deploy
```

### 示例代码

https://github.com/xiaowangmm2/my-umi-component-scaffold
