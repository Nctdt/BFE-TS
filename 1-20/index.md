# 前言

`BFE`是一个前端面试及学习网站，这篇文章将带你学习并理解`BFE-TS`的`1-20`题，进行`TS`能力进阶。

需要一定的`TS`基础

[BFE]: https://bigfrontend.dev/typescript

# 效果

学习并理解这些题目可以让你的`TS`能力飙升。

# 题目

## 1-15 Utility types 实现

`1-15`题都是`TS`的内置`Utility types`，有用但简单，不过多描述。

### 1. implement Partial

> 映射类型添加 ? 标识
```ts
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```
### 2. implement Required

>映射类型删除 ? 标识
```ts
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```
### 3. implement Readonly

> 映射类型添加 readonly 标识
```ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```
### 4. implement Record

> 一般用于快速建立简单对象类型，K 是创建对象类型的索引类型，T 则是创建对象类型的值类型

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```
### 5. implement Pick

> 选择部分 T 中的索引类型后将其映射成新的对象类型

```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```
### 6. implement Omit

> 选择部分 T 中的索引类型后将其从原来的 keyof T 中剔除映射成新的对象类型

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
### 7. implement Exclude

> 通过分布式条件类型(Distributive Conditional Types) 进行筛选，将满足条件的返回 never，因为联合类型忽略 never 从而达到剔除的作用

```ts
type Exclude<T, U> = T extends U ? never : T;
```
### 8. implement Extract

> 同上，返回相反，只将满足条件的返回，不满足则被剔除

```ts
type Extract<T, U> = T extends U ? T : never;
```
### 9. implement NonNullable

>  剔除 null | undefined 类型，表明非空

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```
### 10. implement Parameters

> 通过 infer 推断函数参数并返回

```ts
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```
### 11. implement ConstructorParameters

> 通过 infer 推断 new 构造函数参数并返回，
> TS 中通过 new (args) => any 声明这个函数被 new 调用时所需的参数和返回值

```ts
type ConstructorParameters<T extends new (...args: any) => any> = T extends new (...args: infer P) => any ? P : never;
```
### 12. implement ReturnType

> 通过 infer 推断函数返回值

```ts
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```
### 13. implement InstanceType

> 通过 infer 推断 new 后返回的类型

```ts
type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;
```
### 14. implement ThisParameterType

> 通过 infer 推断函数的 this 类型

```ts
type ThisParameterType<T> = T extends (this: infer U, ...args: any[]) => any ? U : unknown;
```

### 15. implement OmitThisParameter
> 因推断时自动忽略函数的 this 类型，因此直接通过推断后的参数类型和返回值类型包装成个函数返回即可

```ts
type OmitThisParameter<T> = unknown extends ThisParameterType<T> ? T : T extends (...args: infer A) => infer R ? (...args: A) => R : T;
```
> 上面为 TS 源码，但实际通过 BFE 测试还可以这么写，Parameters 本质还是推断，因此会忽略 this：

```ts
type MyOmitThisParameter<T> = T extends (...args: any) => any ? (
  ...args: Parameters<T>
) => ReturnType<T> : T
```

## 16. implement FirstChar\<T>

实现`FirstChar<T>`类型：

```ts
type A = FirstChar<'BFE'> // 'B'
type B = FirstChar<'dev'> // 'd'
type C = FirstChar<''> // never
```

### 答案

目前`TS`支持字符串使用`infer`推断，语法和模板字符串相似，因此使用`infer`即可完成。

```ts
type FirstChar<T extends string> = T extends `${infer T}${any}` ? T : never
```

## 17. implement LastChar\<T>

实现`LastChar<T>`类型：

```ts
type A = LastChar<'BFE'> // 'E'
type B = LastChar<'dev'> // 'v'
type C = LastChar<''> // never
```

### 答案

同样使用`infer`推断字符串中的类型，模板字符串中的推断必须保证前面的`${any}`匹配一个，身下的都丢给最后的`${any}`匹配。

举个栗子：

用`${infer A}${infer B}${infer C}`匹配`'ab'`，`A`在前面匹配了`'a'`，`B`也在前面因此匹配一个`'b'`，剩下的都丢给`C`，因为没有剩余的了，所以`C`匹配到了`''`。

晓得如何匹配了，那么再加上递归即可做出来

```ts
type LastChar<T extends string> = T extends `${infer F}${infer R}`
  ? R extends ''
    ? F
    : LastChar<R>
  : never
```

过程：

```
'BFE' => 
	F -> 'B', R -> 'FE'
	R isn't '' 
	LastChar<'FE'>
		F -> 'F', R -> 'E'
		R isn't ''
		LastChar<'E'>
			F -> 'E', R -> ''
			R is ''
				F -> 'E'
```

即通过一直拿出当前字符串类型的第一位然后判断剩下的是否为`''`是的话就是最后一个了，不是就继续递归剩下的。

## 18. implement TupleToUnion\<T>

```ts
type Foo = [string, number, boolean]

type Bar = TupleToUnion<Foo> // string | number | boolean
```

### 答案

这题考察的是索引访问类型`Indexed Access Types`，我们可以使用索引访问类型来查找另一种类型上的特定属性，`keyof`可以获得某类型的**所有**可访问的索引类型。

```ts
type TupleToUnion<T extends any[]> = T[number]
```

数组类型有个特殊的索引类型`number`，可以通过`arr[number]`获取一个数组中元素的类型。

## 19. implement FirstItem\<T>

```ts
type A = FirstItem<[string, number, boolean]> // string
type B = FirstItem<['B', 'F', 'E']> // 'B'
```

### 答案

如今元组可以通过`infer`推断其中类型，和`16. FirstChar<T>`同理

```ts
type FirstItem<T extends any[]> = T extends [infer T, ...any] ? T : never
```

## 20. implement IsNever\<T>

```ts
type A = IsNever<never> // true
type B = IsNever<string> // false
type C = IsNever<undefined> // false
```

### 答案

粗略一看，霍，介不容易嘛？自信写出

```ts
type IsNever<T> = T extends never ? true : false
```

但是呢，`never`当泛型直接传入时，会直接返回`never`，即`IsNever<never>`返回`never`和需要的`true`不符合，因此行不通。

那有没有办法可以避开这个现象呢，有，和避开启动分布式条件类型的方法一致：

```ts
type Blah<T> = Box<T> extends Whatever ? A : B
type Blah<T> = Whatever extends T ? A : B
```

包装`T`或`T`作为被继承的。

`never extends T`是我们想要的么？`never`是所有的类型基类因此它继承谁都是`true`，所以不符合要求。

所以我们要选择包装`T`，包装`T`的方式很多种，这里选择了最简单的一种，包装成元组：

```ts
type IsNever<T> = [T] extends [never] ? true : false
```

## 总结

前面20题的难度说不上高，但是能提升对`infer`以及一些现象和用法的理解。

[github]: https://github.com/Nctdtman/BFE-TS

如果本文对你有所帮助，麻烦点个赞支持一些，谢谢:)



