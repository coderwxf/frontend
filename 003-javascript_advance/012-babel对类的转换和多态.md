ES6中的类本质上其实是ES5中类的语法糖写法，同时做了一定程度的增强，我们可以通过[babel](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2010%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=AQYwNghgzlwIIAcHAKYA8AuKB2ATWASihCBgHQDCA9gLYJXY4bADeAUMJ1yA1BgE4BXUlX4AKAJSsOXWVEEIU4iTNmcMACwCWUMnwhZgAXmlq12QTSgAuYAG0AjAAYANMABMAZjeeArG4d3dzcAdgAONwAWT08HNy83ADZ3AF1VNQBfdM4s7OB-HFwlSVMzfJQMQX5sYDE82QB6BuBAU7lAdCVACwjAI31AeH1AVR1ALk9AL8ULK0BaOUABI0BIc0AQt0A3BUAHUwXAO2N6rgAeQTAAPjXZdjKyzR09DAMUMlHdADMtMCxxUeNt4CeAUg9jIxMnCTIaCAIMRPIwvdZgLTbFijDLrBoQ7YqQ6yLLI4Bwra7Q5IzLpXKyHjYPioMAoGjGYBEEjkEAFc4AUVJNCYYgA5Bp3Ky3Ac1OBoFBsBBmbZWdcqFRWXi3IgECp0lTSAARADyAFkyAU8MUUEy3LgqCBLEwyABzCqMslMABCAE8AJK4NmAhCsiQSIA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=true&targets=&version=7.18.5&externalPlugins=&assumptions=%7B%7D)来查看ES6和ES5之间类的转换过程

## 基本转换

`ES6`

```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }

  running() {
    console.log('running')
  }

  static eatting() {
    console.log('eatting')
  }
}

const per = new Person('Klaus', 23)
```



`ES5`

```js
// 经过babel转换后 会默认开启严格模式
"use strict";

function _classCallCheck(instance, Constructor) {
  // 如果当前类被作为普通函数进行调用的时候，instance即this的值为window
  // 所以该方法可以确保我们调用类的时候，一定是使用new关键字来进行调用
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function _defineProperties(target, props) {
  // 循环遍历每一个方法，转换为 由方法的属性描述符和方法名构成的对象
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    // 每一个方法，在遍历的时候，全部都默认不可迭代
    descriptor.enumerable = descriptor.enumerable || false;
    // 每一个方法默认都是可配置的
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    // 挂载方法
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}

function _createClass(Constructor, protoProps, staticProps) {
  // 第二个参数为实例方法列表，挂载到Constructor.prototype上
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  // 第三个参数为静态方法列表，挂载到Constructor上
  if (staticProps) _defineProperties(Constructor, staticProps);
  // 确保构造函数的显示原型是只读的，不会被随意修改
  Object.defineProperty(Constructor, "prototype", { writable: false });
  return Constructor;
}

// class转换后的构造函数本质上是一个立即执行函数
// 这样可以开启一个函数作用域，确保class -> function的过程为纯函数
// 这样 如果某一个类没有被使用的时候，可以方便打包工具对其进行tree-shaking操作
var Person = /*#__PURE__*/ (function () {
  function Person(name, age) {
    // 确保类是通过new关键字进行调用的
    _classCallCheck(this, Person);

    // 初始化属性
    this.name = name;
    this.age = age;
  }


  // 挂载实例方法和静态方法
  _createClass(
    Person,
    [
      {
        key: "running",
        value: function running() {
          console.log("running");
        }
      }
    ],
    [
      {
        key: "eatting",
        value: function eatting() {
          console.log("eatting");
        }
      }
    ]
  );

  // 返回转换后的类
  return Person;
})();

var per = new Person("Klaus", 23);
```



## 继承转换

`ES6`

```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }

  running() {
    console.log('running')
  }

  static eatting() {
    console.log('eatting')
  }
}

class Student extends Person{
  constructor(name, age, sno) {
    super(name, age)
    this.sno = sno
  }

  playing() {
    console.log('playing')
  }

  static swiming() {
    console.log('swiming')
  }
}

const stu = new Student('Klaus', 23, 1810166)
```



`ES5`

```js
"use strict";

function _typeof(obj) {
  "@babel/helpers - typeof";
  return (
    (_typeof =
      "function" == typeof Symbol && "symbol" == typeof Symbol.iterator
        ? function (obj) {
            return typeof obj;
          }
        : function (obj) {
            return obj &&
              "function" == typeof Symbol &&
              obj.constructor === Symbol &&
              obj !== Symbol.prototype
              ? "symbol"
              : typeof obj;
          }),
    _typeof(obj)
  );
}

function _inherits(subClass, superClass) {
  // 父类必须是构造函数或null
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }

  // 原型链继承
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: { value: subClass, writable: true, configurable: true }
  });

  Object.defineProperty(subClass, "prototype", { writable: false });

  // 将子类构造函数的隐式原型设置为父类构造函数
  // 以便于实现静态方法的继承
  if (superClass) _setPrototypeOf(subClass, superClass);
}

function _setPrototypeOf(o, p) {
  _setPrototypeOf = Object.setPrototypeOf
    ? Object.setPrototypeOf.bind()
    : function _setPrototypeOf(o, p) {
        o.__proto__ = p;
        return o;
      };
  return _setPrototypeOf(o, p);
}

function _createSuper(Derived) {
  // 判断当前浏览器是否支持proxy和reflect
  var hasNativeReflectConstruct = _isNativeReflectConstruct();
  return function _createSuperInternal() {
    // 获取子类的原型 -- 因为之前有设置子类的原型为父类构造函数
    // 所以Super的值就是父类构造函数
    var Super = _getPrototypeOf(Derived),
      result;

    // 借用构造函数继承
    if (hasNativeReflectConstruct) {
      var NewTarget = _getPrototypeOf(this).constructor;
      result = Reflect.construct(Super, arguments, NewTarget);
    } else {
      result = Super.apply(this, arguments);
    }

    // 确保返回的值为对象类型或undefined类型
    // 封装成多个单一的函数的目的是为了单一职责原则
    // 即每一个函数只做自己职责范围内的事情即可
    return _possibleConstructorReturn(this, result);
  };
}

function _possibleConstructorReturn(self, call) {
  if (call && (_typeof(call) === "object" || typeof call === "function")) {
    return call;
  } else if (call !== void 0) {
    throw new TypeError(
      "Derived constructors may only return object or undefined"
    );
  }
  return _assertThisInitialized(self);
}

// 确保子类使用this之前使用super进行初始化调用
// 如果没有使用super调用的时候，this的值为undefined
function _assertThisInitialized(self) {
  // void 任意一个数值 === undefined
  if (self === void 0) {
    throw new ReferenceError(
      "this hasn't been initialised - super() hasn't been called"
    );
  }
  return self;
}

function _isNativeReflectConstruct() {
  if (typeof Reflect === "undefined" || !Reflect.construct) return false;
  if (Reflect.construct.sham) return false;
  if (typeof Proxy === "function") return true;
  try {
    Boolean.prototype.valueOf.call(
      Reflect.construct(Boolean, [], function () {})
    );
    return true;
  } catch (e) {
    return false;
  }
}

// 支持Object.getPrototypeOf就使用Object.getPrototypeOf, 否则就使用`__proto__`
function _getPrototypeOf(o) {
  _getPrototypeOf = Object.setPrototypeOf
    ? Object.getPrototypeOf.bind() // => 对原本的Object.getPrototypeOf方法的拷贝
    : function _getPrototypeOf(o) {
        return o.__proto__ || Object.getPrototypeOf(o);
      };
  return _getPrototypeOf(o);
}

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}

function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  Object.defineProperty(Constructor, "prototype", { writable: false });
  return Constructor;
}

var Person = /*#__PURE__*/ (function () {
  function Person(name, age) {
    _classCallCheck(this, Person);

    this.name = name;
    this.age = age;
  }

  _createClass(
    Person,
    [
      {
        key: "running",
        value: function running() {
          console.log("running");
        }
      }
    ],
    [
      {
        key: "eatting",
        value: function eatting() {
          console.log("eatting");
        }
      }
    ]
  );

  return Person;
})();

// 使用IIFE将父类作为参数进行传入的目的是为了尽可能将转换函数写成纯函数
// 以便于tree shaking
var Student = /*#__PURE__*/ (function (_Person) {
  // 实现继承
  _inherits(Student, _Person);

  // 获取一个函数，该函数的功能是为了实现借用构造函数继承
  // 对应的就是super方法
  var _super = _createSuper(Student);

  function Student(name, age, sno) {
    var _this;

    // 子类只能通过new关键字来进行调用
    _classCallCheck(this, Student);

    // 借用构造函数实现继承 并返回实现属性继承后的实例对象
    _this = _super.call(this, name, age);
    _this.sno = sno;
    return _this;
  }

  // 挂载方法
  _createClass(
    Student,
    [
      {
        key: "playing",
        value: function playing() {
          console.log("playing");
        }
      }
    ],
    [
      {
        key: "swiming",
        value: function swiming() {
          console.log("swiming");
        }
      }
    ]
  );

  return Student;
})(Person);

var stu = new Student("Klaus", 23, 1810166);
```



## 多态

多态（英语：polymorphism）指为不同数据类型的实体提供统一的接口，表现出不同的行为，就是多态的体现。

简单而言就是使用不同数据类型去调同一个方法，可以得到不同的结果，这种表现就是多态



在传统的面向对象语言中，实现多态有以下两个先决条件 

1. 必须有继承(或有接口形式表示的类继承)

2. 必须存在父类引用指向子类对象 - 即将子类赋值给父类类型的变量

   这么做的目的是可以保证任何的父类的子类都可以调用对应的接口(方法)

   可以提高方法的通用性

```ts
class Shape {
  // 父类中必须存在对应方法的定义
  // 以便于TS可以进行更好的提示
  getArea() {}
}

class Cricle extends Shape{
  radius: number

  constructor(radius: number) {
    super()
    this.radius = radius
  }

  // overwrite
  getArea() {
    return Math.PI * this.radius * this.radius
  }
}

class Rectange extends Shape{
  width: number
  hieght: number

  constructor(width: number, height: number) {
    super()
    this.width = width
    this.hieght = height
  }

  // overwrite
  getArea() {
    return this.width * this.hieght
  }
}

/*
  getShapeArea方法 就实现了多态
  该方法在传入不同的数据类型的时候，会调用参数自身的getArea方法
  从而执行不同的逻辑，实现对应不同的功能
*/
function getShapeArea(shape: Shape) {
  console.log(shape.getArea())
}

const c1 = new Cricle(10)
const rect1 = new Rectange(10, 20)

// c1和rect1是Shape的子类类型
// 此处就是通过父类引用指向子类对象的形式
// 提高getShapeArea方法的通用性
getShapeArea(c1)
getShapeArea(rect1)
```



在维基百科中，对于多态的定义如下

多态（英语：polymorphism）指为不同数据类型的实体提供统一的接口，或使用一个单一的符号来表示多个不同的类型

```js
// 为不同数据类型的实体提供统一的接口
function sum(arg1, arg2) {
  return arg1 + arg2
}

sum(10, 20)
sum('hello', 'world')
```

```js
// 使用一个单一的符号来表示多个不同的类型
let foo;

foo = 123
console.log(foo.toFixed())

foo = 'hello world'
console.log(foo.split(' '))

foo = [1, 2, 3]
console.log(foo.length)
```

所以在广义角度而言，`在弱数据类型语言中，就处处存在多态`，因此在JS中，处处都是多态

但是在严格角度而言，只有严格按照传统面向对象语言形式实现了多态才叫存在多态

即必须存在继承，且必须存在父类引用指向之类对象这种形式后，我们才可以认为该编程语言支持多态

因此在严格角度而言，JS中不存在多态
