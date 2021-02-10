##### 预设
###### @babel/preset-env -- 自动适配设定的目标环境需要的js特性
> 1 - @bable/preset-env主要作用是对浏览器中缺失的功能进行代码转化和加载ployfill，在不进行任何配置的情况下，@babel/preset-env所包含的插件将支持所有最新的js特性（ES2015、ES2016等，不包括stage阶段），将其转化为ES5代码  
> 2 - 启用顺序为数组内从右至左
```javascript
配置:
1. targets: 项目的浏览器环境
{ "targets": "> 0.25%, not dead" }
或
{
    "targets": {
        "chrome": "58",
        "ie": "11"
    }
}
[更多targets配置](https://github.com/browserslist/browserslist)

2. modules: 设置转化后代码的导出方式
value: amd | umd | commonjs | systemjs | cjs | auto | false

3. useBuiltIns: 设置如何去处理polyfill 
value: entry | usage | false
false  default
entry  根据目标环境抽取需要的ployfill
usage  根据当前代码使用到的进行polyfill抽取
( 配置为usage时，可以减小代码打包体积，但必须同时设置corejs )

4. corejs
value: 2(default) | 3 | { "version": 2 | 3, "proposals": boolean }
(
    1. 当useBuiltIns为entry或者usage时才会有效果
    2. core-js@2不会添加新特性，新的特性会被添加到core-js@3中。为了可以使用到更多的新特性，推荐使用core-js@3
)
```
###### @babel/preset-stage-x
```javascript
stage阶段:
stage-0: 设想，只是一个想法，可能有babel插件
stage-1: 建议，这是值得跟进的
stage-2: 草案，初始规范
stage-3: 候选，完成规范并在浏览器上初步实现
stage-4: 完成，将添加到下一个年度版本中
```
###### @babel/preset-react -- 自动引入处理react的插件

###### @babel/preset-typescript -- 自动引入处理typescript的插件
```javascript
配置:
1. isTSX
value: true|false(defalut)
设置为true时强制开启jsx解析。否则，尖括号将被视为 typescript 的类型断言
isTSX设置为true时需要 allExtensions: true

2. allExtensions 
value: true|false(defalut)
将每个文件都作为 TS 或 TSX (取决于 isTSX 参数)进行解析
```
##### 插件
> 1 - babel不进行任何配置开箱即用不会任何效果，也就是说一段ES6的代码经过刚开箱的 babel处理，输出的还是之前的代码  
> 2 - 启用顺序为数组内从左至右

###### @babel/polyfill
```javascript
1. babel不会转化ES6的新API，例如Promise、WeakMap等，因此在低版本浏览器中执行是有问题的，所以需要用到@babel/polyfill。@babel/polyfill包含core-js和regenerator-runtime，可以完整的模拟ES6环境

2. 从7.4.0版本开始，@babel/polyfill已弃用，需单独安装core-js和regenerator-runtime
```
###### @babel/plugin-transform-runtime
```javascript

```
```javascript

```
###### @babel/polyfill对比@babel/plugin-transform-runtime
```javascript
1. polyfill是全局下的包，会将原生方法重写，从而污染全局。引入polyfill会增大打包后的体积，如果只是使用几个新特性，则没必要引入(设置useBuiltIns为usage可以按需引入)

2. transform-runtime是利用plugin自动识别并替换代码中的新特性，按需替换，打包体积会比polyfill小很多。而且transform-runtime也不会污染全局对象、方法，所以更适合开发工具包、库
```
##### 测试代码
```shell
npm instal @babel/core @babel/cli -D
```
```shell
npx babel script.js -o script-compiled.js
```
1. 无配置
```javascript
// script.js
const promise = new Promise((resolve, reject) => { })

class Storage { 
    constructor() { }
}

// .babelrc: 
{}

// script-compiled.js
const promise = new Promise((resolve, reject) => {});

class Storage {
  constructor() {}

}
```
2. 添加@babel/preset-env预设
```javascript
// script.js
const promise = new Promise((resolve, reject) => { })

class Storage { 
    constructor() { }
}

// .babelrc:
{
    "presets": [
        ["@babel/preset-env", {
            "targets": [
                "> 0.25%",
                "last 2 versions, not dead",
                "ie >= 11"
            ]
        }]
    ]
}

// script-compiled.js
"use strict";

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var promise = new Promise(function (resolve, reject) {});

var Storage = function Storage() {
  _classCallCheck(this, Storage);
};
```
3. 引入@babel/polyfill
```javascript
// 不设置useBuiltIns和corejs会将@babel/polyfill全部引入，使用webpack打包时代码量很大
// script.js
import "@babel/polyfill";
const promise = new Promise((resolve, reject) => { })

class Storage { 
    constructor() { }
}

// .babelrc:
{
    "presets": [
        ["@babel/preset-env", {
            "targets": [
                "> 0.25%",
                "last 2 versions, not dead",
                "ie >= 9"
            ]
        }]
    ]
}

// script-compiled.js
"use strict";

require("@babel/polyfill");

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var promise = new Promise(function (resolve, reject) {});

var Storage = function Storage() {
  _classCallCheck(this, Storage);
};
```
4. 将useBuiltIns设置为usage，同时将corejs设置为3
```javascript
// 设置useBuiltIns和corejs时只引入代码中用到的特性，减少webpack打包体积
// webpack打包时会为每个文件生成_classCallCheck，增大代码体积
// script.js
import "@babel/polyfill";
const promise = new Promise((resolve, reject) => { })

class Storage { 
    constructor() { }
}

// .babelrc:
{
    "presets": [
        ["@babel/preset-env", {
            "targets": [
                "> 0.25%",
                "last 2 versions, not dead",
                "ie >= 11"
            ]
            "useBuiltIns": "usage",
            "corejs": 3
        }]
    ]
}

// script-compiled.js
"use strict";

require("core-js/modules/es.object.to-string.js");

require("core-js/modules/es.promise.js");

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var promise = new Promise(function (resolve, reject) {});

var Storage = function Storage() {
  _classCallCheck(this, Storage);
};
```
5. 移除@babel/polyfill，添加@babel/plugin-tranform-runtime插件
```javascript
// script.js
const promise = new Promise((resolve, reject) => { })

class Storage { 
    constructor() { }
}

// .babelrc
{
    "presets": [
        ["@babel/preset-env", {
            "targets": [
                "> 0.25%",
                "last 2 versions, not dead",
                "ie >= 11"
            ]
        }]
    ],
    "plugins": [
        ["@babel/plugin-transform-runtime", {
            "corejs": 3
        }]
    ]
}

// script-compiled.js
"use strict";

var _interopRequireDefault = require("@babel/runtime-corejs3/helpers/interopRequireDefault");

var _classCallCheck2 = _interopRequireDefault(require("@babel/runtime-corejs3/helpers/classCallCheck"));

var _promise = _interopRequireDefault(require("@babel/runtime-corejs3/core-js-stable/promise"));

var promise = new _promise["default"](function (resolve, reject) {});

var Storage = function Storage() {
  (0, _classCallCheck2["default"])(this, Storage);
};
```
