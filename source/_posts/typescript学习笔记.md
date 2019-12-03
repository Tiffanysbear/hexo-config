---
title: TypeScript学习笔记
date: 2019-10-20 16:00:28
tags: TypeScript
categories: TypeScript
---


> TypeScript学习笔记
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

1、类型注解
2、接口interface：使用interface可以申明一个类型
3、类

 * 创建类时，在构造函数的参数上使用public等同于创建了同名的成员变量。

```
class Student {
    fullName: string;
    constructor(public firstName, public middleInitial, public lastName) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}
// 编译后
var Student = /** @class */ (function () {
    function Student(firstName, middleInitial, lastName) {
        this.firstName = firstName;
        this.middleInitial = middleInitial;
        this.lastName = lastName;
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
    return Student;
}());

```
<!-- more -->

4、基本类型

```
// bool
let isDone: boolean = false;

// number
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;

// string
let name: string = "bob";

// array
let list: number[] = [1, 2, 3];
let list: Array<number> = [1, 2, 3];


```

5、元组 Tuple

```

// Declare a tuple type
let x: [string, number];
// Initialize it
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error

```


6、枚举

```
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```


7、Any 任意类型的值

```

let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean

```

在对现有代码进行改写的时候，any类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为 Object有相似的作用，就像它在其它语言中那样。 但是 Object类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法。
当你只知道一部分数据的类型时，any类型也是有用的。 比如，你有一个数组，它包含了不同的类型的数据：

```

let list: any[] = [1, true, "free"];
list[1] = 100;
```

8、void 没有任何类型
当一个函数没有返回值时，你通常会见到其返回值类型是 void：

```
function warnUser(): void {
    console.log("This is my warning message");
}
```
声明一个void类型的变量没有什么大用，因为你只能为它赋予undefined和null：

```
let unusable: void = undefined;
```

9、Null 和 Undefined
TypeScript里，undefined和null两者各自有自己的类型分别叫做undefined和null。 和 void相似，它们的本身的类型用处不是很大：

```
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

默认情况下null和undefined是所有类型的子类型。 就是说你可以把 null和undefined赋值给number类型的变量。

然而，当你指定了 `--strictNullChecks` 标记，`null` 和 `undefined` 只能赋值给void和它们各自。 这能避免 很多常见的问题。 也许在某处你想传入一个 `string或null或undefined`，你可以使用联合类型`string | null | undefined`


10、Never
never类型表示的是那些永不存在的值的类型。 例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never类型，当它们被永不为真的类型保护所约束时。
never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 即使 any也不可以赋值给never。

```
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}

// 推断的返回值类型为never
function fail() {
    return error("Something failed");
}

// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
    while (true) {
    }
}
```

11、object
object表示非原始类型，也就是除number，string，boolean，symbol，null或undefined之外的类型。

```
declare function create(o: object | null): void;

create({ prop: 0 }); // OK
create(null); // OK

create(42); // Error
create("string"); // Error
create(false); // Error
create(undefined); // Error
```

12、可选属性

```
interface SquareConfig {
  color?: string;
  width?: number;
}
```

13、只读属性
一些对象属性只能在对象刚刚创建的时候修改其值。 你可以在属性名前用 readonly来指定只读属性:

```
interface Point {
    readonly x: number;
    readonly y: number;
}
```


14、函数类型
为了使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。


```
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}

// 或者
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  let result = src.search(sub);
  return result > -1;
}
```

15、联合类型

```
function padLeft(value: string, padding: string | number) {
    // ...
}
```

16、交叉类型

```
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let id in first) {
        (<any>result)[id] = (<any>first)[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            (<any>result)[id] = (<any>second)[id];
        }
    }
    return result;
}

```

17、可以为null的类型
默认情况下，类型检查器认为 null与 undefined可以赋值给任何类型。 `null` 与 `undefined` 是所有其它类型的一个有效值。

`--strictNullChecks` 标记可以解决此错误：当你声明一个变量时，它不会自动地包含 null或 undefined。


18、接口 Interface vs. 类型别名 type

- 接口创建了一个新的名字，可以在其它任何地方使用。 类型别名并不创建新名字—比如，错误信息就不会使用别名。 在下面的示例代码里，在编译器中将鼠标悬停在 interfaced上，显示它返回的是 Interface，但悬停在 aliased上时，显示的却是对象字面量类型。
- 类型别名不能被 extends和 implements（自己也不能 extends和 implements其它类型）
- 如果你无法通过接口来描述一个类型并且需要使用联合类型或元组类型，这时通常会使用类型别名


19、字符串字面量类型
字符串字面量类型允许你指定字符串必须的固定值。 在实际应用中，字符串字面量类型可以与联合类型，类型保护和类型别名很好的配合。 通过结合使用这些特性，你可以实现类似枚举类型的字符串。


```
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

字符串字面量类型还可以用于区分函数重载：

```
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
    // ... code goes here ...
}

```













```
```


