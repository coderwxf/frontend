[tsconfig.json](https://www.typescriptlang.org/tsconfig/) 文件指定了编译项目所需的根目录下的文件以及编译选项

当目录中出现了 tsconfig.json 文件，则说明该目录是 TypeScript 项目的根目录

1. 让TypeScript Compiler在编译的时候，知道如何去编译TypeScript代码和进行类型检测

2. 让编辑器(比如VSCode)可以按照正确的方式识别TypeScript代码

   + 默认情况下，vscode对于代码的错误检测和提示拥有自己的默认配置

   + 而如果项目根目录存在对应tsconfig.json或jsconfig.json的时候

     vscode会自动读取这类文件，以结合默认的配置选项

     给予更好的提示和更准确的错误检测



当我们执行`tsc`命令或者使用ts-loader进行自动编译ts文件的时候，会自动读取tsconfig.json文件

> 注意: tsc在使用的时候，后边不能加上文件，如果加上文件只会编译单个文件，而忽略tsconfig.json文件
>
> ```shell
> # 会使用tsconfig.json文件
> tsc
>
> # 此时不会使用tsconfig.json文件
> tsc <文件>
> ```



## 顶层配置项

| 选项            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| compilerOptions | ts的编译配置选项                                             |
| files           | 指定那些文件需要编译，如果不指定就表示所有的文件都需要进行编译<br />所以多数情况下不需要指定 |
| include         | 指定那些文件夹下的文件需要编译，那些不需要进行编译<br />例如`include:[src/**/*]` |
| exclude         | 指定在include编译选项内，有那些文件夹不需要进行编译<br />例如`exclude:[node_modules]` |



### 常见的compilerOptions选项

| 选项              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| target            | 指定目标代码需要编译成那个版本的JS<br />一般会配置为ESNext，也就是说仅仅只是将TS代码编译为JS代码，对于ESNext代码的转换将由babel来进行完成<br />默认值为ES3 |
| lib               | 当我们指定了target的时候，会存在默认的lib选项<br />我们也可以手动指定对应的lib选项，当我们手动指定lib选项后，就是要指定的lib选项，而不使用默认的lib选项 |
| module            | 编译后的代码需要使用哪种显示的模块化，<br />一般设置为ESNext，依旧是交给babel来进行处理 |
| strict            | 值为boolean值，当开启严格检测的时候，实际是开启了包括`noImplicitAny,strictNullChecks.noImplicitThis等`在内的一系列严格检测，我们也可以单独关闭或开启其中的某一个严格检测选项，以进行粒度化管理 |
| allowJS           | 是否允许在TS项目中编写JS代码                                 |
| jsx               | 如果项目中使用了JSX，那么JSX文件是否需要交给TSC来进行编译<br />常设置的值是preserve，也就是不需要转换，一般会交给babel来进行处理 |
| declaration       | 编译后是否需要生成`*.d.ts`文件<br />对于业务项目一般不需要，对于库可能需要 |
| importHelpers     | TS在转换代码的时候，可能会使用到一个工具函数，默认情况下是生成在需要使用的文件中的<br />如果多个文件都需要同一个工具函数，那么这个工具函数会在多个需要使用到的文件中存在<br />当设置该选项为true的时候，会将这些重复使用的工具函数进行抽离，并通过模块机引入的方式进行导入 |
| moduleResolution  | 常见值 node<br />模块采用那种解析规则来进行加载              |
| skipLibCheck      | 只检测项目中使用的类型，对于导入的库中的类型不进行检测<br />当自己定义的类型和导入的库中的类型冲突的时候可以开启这个选项<br />当开启这个选项的时候TS在编译的时候会尝试检测所有声明文件（*,d,ts）中的有效类型，而当声明文件中的内容发生错误的时候，会被认为是any，而不是直接报错 |
| esModuleInterop   | 是否允许CJS和ESM混用<br />一般会设置为true，因为第三方可能会使用CJS，也可能使用ESM |
| sourceMap         | 编译后是否生成sourceMap文件                                  |
| paths             | 例如`paths: { "@/*": ["src/*"] }`<br />用于指定项目中所有使用的路径别名，方面TS在编译的时候可以正确识别和解析对应的文件 |
| baseUrl           | 默认值为当前目录<br />paths中对应相对路径的基准路径，例如`paths: { "@/*": ["src/*"] }`中的src就是相对于baseUrl所在目录的 |
| outDir            | 编译后的文件 输出到那个文件夹中                              |
| resolveJsonModule | 默认情况下，TS并不识别引入的JSON文件<br />开启该选项后，TS可以解析并识别对应的JSON文件<br />也就是说开启该选项后，不仅可以引入对应的JSON文件，而且引入后还可以拥有自己对应的类型 |



### tslib

typescript在编译为javaScript的时候，具体编译成那个版本的JavaScript是取决于target的

而如果我们使用了比较新的语法，而需要编译成的target比较旧的时候

typescript在转换我们代码的时候就需要降级处理

`目标target为ES5`

`ts`

```ts
const foo = [1, 2, 3]
console.log([...foo])
```

`编译后`

```js
"use strict";
var __spreadArray = (this && this.__spreadArray) || function (to, from, pack) {
    if (pack || arguments.length === 2) for (var i = 0, l = from.length, ar; i < l; i++) {
        if (ar || !(i in from)) {
            if (!ar) ar = Array.prototype.slice.call(from, 0, i);
            ar[i] = from[i];
        }
    }
    return to.concat(ar || Array.prototype.slice.call(from));
};
var foo = [1, 2, 3];
console.log(__spreadArray([], foo, true));
```



此时如果我们在多个文件中使用了这些“新语法”的时候

`index.ts`

```ts
import { foo } from './foo'

console.log([...foo])
```



`foo.ts`

```ts
const foo = [1, 2, 3]
const baz = [...foo]

export {
  foo
}
```



此时编译后的代码如下

`index.js`

```js
"use strict";
var __spreadArray = (this && this.__spreadArray) || function (to, from, pack) {
    if (pack || arguments.length === 2) for (var i = 0, l = from.length, ar; i < l; i++) {
        if (ar || !(i in from)) {
            if (!ar) ar = Array.prototype.slice.call(from, 0, i);
            ar[i] = from[i];
        }
    }
    return to.concat(ar || Array.prototype.slice.call(from));
};
Object.defineProperty(exports, "__esModule", { value: true });
var foo_1 = require("./foo");
console.log(__spreadArray([], foo_1.foo, true));
```

`foo.js`

```js
"use strict";
var __spreadArray = (this && this.__spreadArray) || function (to, from, pack) {
    if (pack || arguments.length === 2) for (var i = 0, l = from.length, ar; i < l; i++) {
        if (ar || !(i in from)) {
            if (!ar) ar = Array.prototype.slice.call(from, 0, i);
            ar[i] = from[i];
        }
    }
    return to.concat(ar || Array.prototype.slice.call(from));
};
Object.defineProperty(exports, "__esModule", { value: true });
exports.foo = void 0;
var foo = [1, 2, 3];
exports.foo = foo;
var baz = __spreadArray([], foo, true);
```



此时发现类似于`__spreadArray`这类的工具函数(也就是执行polyfill功能的函数)

在多个文件中被多次定义，为此typescript提供了一个编译选项`importHelpers`

当开启这个选项的时候，需要结合官方库`tslib`一并使用

`tslib`是typescript把一系列的工具函数抽离并合并导出的库，从而减少编译后的代码量



在开启importHelpers选项并引入tslib后，再次进行编译，此时工具函数因为被抽离到了tslib中，因此对应的编译后代码会简洁很多

`index.js`

```js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
var tslib_1 = require("tslib");
var foo_1 = require("./foo");
console.log(tslib_1.__spreadArray([], foo_1.foo, true));
```

`foo.js`

```js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.foo = void 0;
var tslib_1 = require("tslib");
var foo = [1, 2, 3];
exports.foo = foo;
var baz = tslib_1.__spreadArray([], foo, true);
```