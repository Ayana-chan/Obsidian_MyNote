
# 安装与启用

```bash
npm install -g typescript
```

```bash
tsc --version
```

```bash
tsc file_name.ts
```

[TypeScript: Documentation - 中文](https://www.typescriptlang.org/zh/docs/handbook/2/basic-types.html)

[TypeScript: Documentation - The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)

# 语法

## Optional Properties

```ts
function printName(obj: { first: string; last?: string }) {
	// 'obj.last' is possibly 'undefined'.
	if (obj.last !== undefined) {
	// OK
		console.log(obj.last.toUpperCase());
	}
	
	// A safe alternative using modern JavaScript syntax:
	console.log(obj.last?.toUpperCase());
}
```


## Union Types (combine)

使用`|`表示变量可能是这几个类型的其中一个，于是变量就获得了这些类型的共同接口。想要分辨到底是哪个类型的话就需要用`typeof`进行narrow：

```ts
function printId(id: number | string) {
	if (typeof id === "string") {
		// In this branch, id is of type 'string'
		console.log(id.toUpperCase());
		} else {
		// Here, id is of type 'number'
		console.log(id);
	}
}
```

也有一些类型判定函数：
```ts
function welcomePeople(x: string[] | string) {
	if (Array.isArray(x)) {
		// Here: 'x' is 'string[]'
		console.log("Hello, " + x.join(" and "));
	} else {
		// Here: 'x' is 'string'
		console.log("Welcome lone traveler " + x);
	}
}
```

## 合并对象类型

使用`&`将成员变量合并：

```ts
type Animal = {
  name: string;
}  

type Bear = Animal & { 
  honey: boolean;
}  

const bear = getBear();
bear.name;
bear.honey;
```

## Type Aliases

```ts
type Point = {
	x: number;
	y: number;
};

// Exactly the same as the earlier example
function printCoord(pt: Point) {
	console.log("The coordinate's x value is " + pt.x);
	console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

## Interface

```ts
interface Point {
	x: number;
	y: number;
}

function printCoord(pt: Point) {
	console.log("The coordinate's x value is " + pt.x);
	console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

[TypeScript: Documentation - Differences Between Type Aliases and Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)

可以在任意地方给**原Interface**增加成员变量。

## Type Assertions

使用`as TypeName`或者`<TypeName>`来断言一个变量的类型。

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

如果运行时断言错误不会出现任何报错。TS在编译期间只允许更父或更子的断言。

暴力断言：
```ts
const a = expr as any as T;
```

## Literal Types

类型可以限定string和number的可能值；如果变量是const的话会直接令其类型为其值。可以当成枚举类使用。

```ts
let x: "hello" = "hello";
// OK
x = "hello";
// Type '"howdy"' is not assignable to type '"hello"'.
x = "howdy";
```

>The type `boolean` itself is actually just an alias for the union `true | false`.

有时需要一些强制手段来要求其保持literal type：[TypeScript: Documentation - Literal Inference](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-inference)

## Non-null Assertion Operator (Postfix `!`)

如果打开了`strictNullChecks`，则必须要求检查是否为null。使用`!`来断言其不是null。但这个断言是用来骗编译器的，并不会自动加检测代码。

```ts
function liveDangerously(x?: number | null) {
	// No error
	console.log(x!.toFixed());
}
```


# 规范

[编码规范 | TypeScript手册](https://bosens-china.github.io/Typescript-manual/download/zh/wiki/coding_guidelines.html#%E5%91%BD%E5%90%8D)