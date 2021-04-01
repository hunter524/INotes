# react package.js 分析

## scripts 运行脚本区域

```json
  {
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject",
    "flow": "flow"
  }
}
```

右键运行上述 package.js 脚本实际上执行的命令是:

npm: /usr/bin/node /usr/lib/node_modules/npm/bin/npm-cli.js run start --scripts-prepend-node-path=auto

yarn: /usr/bin/node /usr/share/yarn/bin/yarn.js run start

上述的命令等同于使用 npm start,yarn start (*直接执行 yarn start 其实是变相执行了 yarn 这个系统脚本,然后通过yarn 脚本启动了 node /usr/share/yarn/bin/yarn.js build 执行,实际上还是通过 yarn.js 执行 script 部分的 build,start 命令*)

使用 npm 运行和yarn 运行分别通过 yarn.js npm-cli.js 分别进行解析当前根目录下的 package.js scripts 块定义的 start,build,test 等命令执行.

react-scripts 直接鼠标点击指向了 /home/hunter/WebstormProjects/StudyReact/node_modules/.bin/react-scripts

实际其指向的为 /home/hunter/WebstormProjects/StudyReact/node_modules/react-scripts/bin/react-scripts.js

同时在 react-scripts.js 脚本中会解析命令行参数 start build test --env=jsdom eject 等参数,进而继续运行

/home/hunter/WebstormProjects/StudyReact/node_modules/react-scripts/scripts 目录下对应的 start.js,build.js 等.

## create-react-app

facebook 提供的使用命令行创建 react 项目的工具.使用 npm install -g create-react-app 即可在全局安装该工具.

[其内置多种官方预定义的模板,如:cra-template 是 creat-react-app 内置的默认项目模板,cra-template-typescript 则是内置的集成了type-script 的 react 模板][https://github.com/facebook/create-react-app/tree/master/packages]

## build 生成的输出文件

- *.map

如:
2.5d86e6ae.chunk.js.map
3.abc9f1e8.chunk.js.map
main.9d5b29c0.chunk.css.map
main.b3e311dc.chunk.js.map
runtime-main.dc6f02b5.js.map

Todo://暂且理解为 java proguard 的 map 文件功能

- 生成的上线文件

2.5d86e6ae.chunk.js
2.5d86e6ae.chunk.js.LICENSE.txt
3.abc9f1e8.chunk.js
main.b3e311dc.chunk.js
runtime-main.dc6f02b5.js

main.9d5b29c0.chunk.css

css 的大包会自动做拆分

## node 导入类型

- require/export

```js
let ReactDOM = require("react-dom");
```

require 导入的变量需要使用 let 进行赋值操作

- module.require/module.exports

 node 内置的同步导入依赖操作

- import

```js
import ReactDOM from 'react-dom';
import {add} from "./flowTest";
```

导入成为一个对象,或者对其进行解构操作
