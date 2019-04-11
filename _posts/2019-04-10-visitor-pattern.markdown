---
layout: post
title:  "Visitor Pattern"
categories: gof23 java
date: 2019-04-10 11:01:22
---

## 1. 从Pattern Match说起

&nbsp;&nbsp;&nbsp;&nbsp;函数式语言有一个很方便的特性————`Pattern Match`, 例如:

```haskell
-- haskell example
data Expr = Add Expr Expr  
          | Sub Expr Expr 
          | Lit Int

eval expr = case expr of 
              Add e1 e2 -> eval e1 + eval e2
              Sub e1 e2 -> eval e1 + eval e2
              Lit i     -> i
```
&nbsp;&nbsp;&nbsp;&nbsp;但是上述代码在Java中如何实现呢？首先看数据类型的表示，当然在Java中肯定是class了！

```java
abstract class Expr {}
class Add {
    Expr e1;
    Expr e2;
    Add(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
}
class Sub {
    Expr e1;
    Expr e2;
    Sub(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
}
class Lit {
    int i;
    Lit(int i) {
        this.i = i;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;如何实现`eval`方法?`eval`的类型是明确的，但是如何知道参数的实际类型呢？instanceof!!最后代码的实现如下：
```java
static int eval(Expr e) {
    // how to know what type e actually is ? instanceof !!!
    if (e instanceof Add) {
        Add add = (Add) e;
        return eval(add.e1) + eval(add.e2);
    } else if (e instanceof Sub) {
        // ... Convert to Sub
    } else if (e instanceof Lit) {
        // ... Convert to Lit
    }
}

```
&nbsp;&nbsp;&nbsp;&nbsp;这样的代码中含有大量的instanceof以及强制类型转换，这一点都不Java，那么如何处理的Java点呢？

## 2. Static Dispatch<sup>[1]</sup> vs. Dynamic Dispatch<sup>[2]</sup>

&nbsp;&nbsp;&nbsp;&nbsp;第一种思路会想希望能针对不同Expr的不同eval实现，就会有这样的实现：
```java
class Eval {
    int eval(Add e) {
        return eval(e.e1) + eval(e.e2);
    }
    int eval(Sub e) {
        return eval(e.e1) + eval(e.e2);
    }
    int eval(Lit e) {
        return e.i;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;但是很遗憾这样的代码编译报错，eval方法至少还应该有Expr类型吧，那就加上：
```java
class Eval {
    int eval(Expr e) {
        return eval(e);
    }
    int eval(Add e) {
        return eval(e.e1) + eval(e.e2);
    }
    int eval(Sub e) {
        return eval(e.e1) - eval(e.e2);
    }
    int eval(Lit e) {
        return e.i;
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;上面的代码虽然编译通过了，但是测试下，死递归了～，可以回顾下Java的一些基础知识的Puzzle，下面的函数的会打印什么呢？

```java
static void print(Expr e) {
    System.out.println("Expr");
}
static void print(Lit e) {
    System.out.println("Lit");
}

Expr expr = new Lit(2);
print(expr);
```
&nbsp;&nbsp;&nbsp;&nbsp;虽然e的实际类型是Lit类型，但是静态类型是Expr类型，最终会打印“Expr”，这也就是为何上面的方法行不通。那就换个思路把方法塞到`class`里面呢？
```java
abstract class Expr {
    abstract void print() {
        System.out.println("Expr");
    }
}
class Lit {
    int i;
    Lit(int i) {
        this.i = i;
    }
    @Override void print() {
        System.out.println("Lit");
    }
}
Expr expr = new Lit(2);
expr.print();
```
&nbsp;&nbsp;&nbsp;&nbsp;这样就很有面向对象的味道了，结果也很直白，面向对象的最基本的多态呀！肯定是打印“Lit”啊。这样重写上面的`eval`方法的思路就很简单粗暴————塞到`class`里面去呗～
```java
abstract class Expr {
    abstract int eval();
}
class Add {
    Expr e1;
    Expr e2;
    Add(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override int eval() {
        return e1.eval() + e2.eval();
    }
}
class Sub {
    Expr e1;
    Expr e2;
    Sub(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override int eval() {
        return e1.eval() - e2.eval();
    }
}
class Lit {
    int i;
    Lit(int i) {
        this.i = i;
    }
    @Override int eval() {
        return i;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这样开来仿佛一起都挺好，那么还有什么问题呢？问题在于如果不止`eval`方法呢？不止这几个数据类型呢？那就会产生如下的情况：

|   \    |   \   |     Add   |    Sub    |    Lit    |    Mul    |   ...    | 
|--------|-------|-----------|-----------|-----------|-----------|----------|
|  求值   | eval  | Add.eval  | Sub.eval  | Lit.eval  | Mul.eval  |   ...    |  
|表达式深度| depth | Add.depth | Sub.depth | Lit.depth | Mul.depth |   ...    |  
|打印表达式| print | Add.print | Sub.print | Lit.print | Mul.print |   ...    |  
|  ...   | ...   |    ...    |    ...    |    ...    |    ...    |   ...    | 

&nbsp;&nbsp;&nbsp;&nbsp;每增加/删除/修改一种数据类型，或者一种方法的改动相当于拓宽/删除/修改表格的一列或者一行。当这些数据类型定义在不同的文件下，这样的改动实在是让人蛋疼...

## 3. Visitor

&nbsp;&nbsp;&nbsp;&nbsp;在上章节中，了解到函数重载是静态分派的，重写是动态分派的，静态分派的实现的`Eval`思路上虽然清晰但是死递归了，动态分派的实现的`eval`让代码的改动变得困难。再回到第一种思路，由于静态分派的原因，无法实现我们希望的类型分派调用，但是多态可以实现动态分派，当两者的优势结合起来：

```java
abstract class Expr {
    abstract int accept(Eval eval);
}
class Add {
    Expr e1;
    Expr e2;
    Add(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override int accept(Eval eval) {
        return eval.eval(this); // this will call eval(Add expr)
    }
}
class Sub {
    Expr e1;
    Expr e2;
    Sub(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override int accept(Eval eval) {
        return eval.eval(this); // this will call eval(Sub expr)
    }
}
class Lit {
    int i;
    Lit(int i) {
        this.i = i;
    }
    @Override int accept(Eval eval) {
        return eval.eval(this); // this will call eval(Lit expr)
    }
}
//注意！！！再改动Eval, 继续使用eval还是会造成静态分派的问题，通过accept实现动态分派
class Eval {
    int eval(Expr e) {
        return e.accept(this);
    }
    int eval(Add e) {
        return e.e1.accept(this) + e.e2.accept(this);
    }
    int eval(Sub e) {
        return e.e1.accept(this) - e.e2.accept(this);
    }
    int eval(Lit e) { // basic case
        return e.i;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这样的好处究竟在哪里呢？当增加一种数据类型只需要实现相应的接口，然后在Eval中实现相应的eval方法就行了，那么增加方法呢？以depth为例，只用实现一个class继承Eval即可：
```java
class Depth extends Eval {
    int eval(Expr e) {
        return e.accept(this);
    }
    int eval(Add e) {
        int depth1 = e.e1.accept(this);
        int depth2 = e.e2.accept(this);
        return (depth1 > depth2 ? depth1 : depth2) + 1;
    }
    int eval(Sub e) {
        int depth1 = e.e1.accept(this);
        int depth2 = e.e2.accept(this);
        return (depth1 > depth2 ? depth1 : depth2) + 1;
    }
    int eval(Lit e) { // basic case
        return 1;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这时候再称为`Eval`就不合适了,改成`Visitor`, `eval`方法的名字也不妥当，改成`visit`, 抽离出改动的接口如下：
```java
abstract class Expr {
    abstract int accept(Visitor visitor);
}
class Add {
    Expr e1;
    Expr e2;
    Add(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override int accept(Visitor visitor) {
        return visitor.visit(this); // this will call eval(Add expr)
    }
}
class Sub {
    Expr e1;
    Expr e2;
    Sub(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override int accept(Visitor visitor) {
        return visitor.visit(this); // this will call eval(Sub expr)
    }
}
class Lit {
    int i;
    Lit(int i) {
        this.i = i;
    }
    @Override int accept(Visitor visitor) {
        return visitor.visit(this); // this will call eval(Lit expr)
    }
}

abstract class Visitor {
    int visit(Expr e) {
        return e.accept(this);
    }
    abstract int visit(Add e);
    abstract int visit(Sub e);
    abstract int visit(Lit e);
}
```
&nbsp;&nbsp;&nbsp;&nbsp;剩下的只需要Eval和Depth继承Visitor实现就行了。

## 4. Generic and More ...

&nbsp;&nbsp;&nbsp;&nbsp;可能有人要问了，这里的depth和eval类型签名是一样的，算是投机取巧了，但是print返回的void，不能实现这个visitor了吧！这里第一种办法就是为Expr实现具有不同返回值Visitor满足你的不同需求，如IntVisitor，VoidVisitor。为了实现代码的通用性，第二种办法那就需要借助Generic：
```java
abstract class Expr {
    abstract <R> R accept(Visitor<R> visitor);
}
class Add {
    Expr e1;
    Expr e2;
    Add(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override <R> R accept(Visitor<R> visitor) {
        return visitor.visit(this); // this will call eval(Add expr)
    }
}
class Sub {
    Expr e1;
    Expr e2;
    Sub(Expr e1, Expr e2) {
        this.e1 = e1;
        this.e2 = e2;
    }
    @Override <R> R accept(Visitor<R> visitor) {
        return visitor.visit(this); // this will call eval(Sub expr)
    }
}
class Lit {
    int i;
    Lit(int i) {
        this.i = i;
    }
    @Override <R> R accept(Visitor<R> visitor) {
        return visitor.visit(this); // this will call eval(Lit expr)
    }
}

abstract class Visitor<R> {
    R visit(Expr e) {
        return e.accept(this);
    }
    abstract R visit(Add e);
    abstract R visit(Sub e);
    abstract R visit(Lit e);
}
```
&nbsp;&nbsp;&nbsp;&nbsp;那么`Eval`和`Depth`就是`Visitor<Integer>`, 而`Print`就是`Visitor<Void>`了，那么如果不同的类型的表达式返回的结果不同呢？方法需要额外参数呢？这些问题就留给读者自己思考了。

## 5. Conclusion & Thinking

&nbsp;&nbsp;&nbsp;&nbsp;这里介绍了设计模式中相对比较难懂的`Visitor Pattern`, 本质上是因为`Java`等语言中缺少`Pattern Match`方便的手段，但是`Visitor Pattern`和`Pattern Match`之间并非等价关系。`Visitor`采用的是面向对象的设计，更具有可组合性，比`Pattern Match`更加灵活方便，在代码复用方面可以通过继承轻松实现。如果想详细了解`Visitor`演绎可以阅读`a litte java, a few pattern`[3].


## 6. Reference
[1] [https://en.wikipedia.org/wiki/Static_dispatch](https://en.wikipedia.org/wiki/Static_dispatch)

[2] [https://en.wikipedia.org/wiki/Dynamic_dispatch](https://en.wikipedia.org/wiki/Dynamic_dispatch)

[3] Felleisen, Matthias, and Daniel P. Friedman. A little Java, a few patterns. MIT press, 1998.