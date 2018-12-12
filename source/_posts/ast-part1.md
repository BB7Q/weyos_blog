---
title: The Super Tiny Compiler（上）
date: 2018-12-09
categories:
- 技术分享
tags:
- ast
---

# The Super Tiny Compiler（上）

本文是[`The Super Tiny Compiler`](https://github.com/jamiebuilds/the-super-tiny-compiler)的翻译，可以通过本文学习如何实现一个编译器。
本文并非严格按照字面直译，如有理解错误，还请指正。
本篇内容是编译器基本知识介绍。

***[The Super Tiny Compiler（下）](/2018/12/09/ast-part2/)***

## 正文

今天我们来一起编写一个简易编译器，一个非常小的编译器，如果你删除注释，实际代码将只有200行左右。

我们将把一些类`lisp`的函数调用编译成一些类`c`的函数调用。

如果你对其中一个不熟悉。我简单介绍一下。

如果我们有两个函数`add`和`subtract`，他们像这样书写：

```js
/*
 *                  LISP                      C
 *
 *   2 + 2          (add 2 2)                 add(2, 2)
 *   4 - 2          (subtract 4 2)            subtract(4, 2)
 *   2 + (4 - 2)    (add 2 (subtract 4 2))    add(2, subtract(4, 2))
 */
```

很好，因为这正是我们要编译的。虽然这既不是一个完整的LISP语法，也不是一个完整的C语法，但是它已经足够展示现代编译器的许多主要部分了。

大多数编译器分为三个主要阶段:解析（Parsing）、转换（Transformation）和代码生成（Code Generation）

1. 解析：是将原始代码转换为更抽象的代码表示形式。
2. 转换：接受这个抽象形式操作，并操作它去做编译器想要做的事。
3. 代码生成：将转换后的代码表示形式转换成新代码。

### 解析（Parsing）

解析通常分为两个阶段:词法分析（Lexical Analysis）和句法分析（Syntactic Analysis）。

1. 词法分析：获取原始代码，并通过`tokenizer`（或`lexer`）将代码拆分成一个叫做`tokens`的东西。

  -> `tokens`是一组很小的对象，它们描述了语法的一个独立部分。它们可以是数字、标签、标点符号、操作符等等。
2. 语法分析：接受`tokens`并将它们重新格式化为描述语法的每个部分及其相互关系的表示形式。这被称为中间表示或抽象语法树（Abstract Syntax Tree）。

  -> 抽象语法树（Abstract Syntax Tree），或简称AST，是一个嵌套很深的对象，它以一种既容易使用又能告诉我们很多信息的方式表示代码。

对于下列语法

```js
(add 2 (subtract 4 2))
```

`tokens`可能是这样的

```js
[
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'add'      },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'subtract' },
  { type: 'number', value: '4'        },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: ')'        },
  { type: 'paren',  value: ')'        },
]
```

他的抽象语法树（AST）可能是这样的

```js
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}
```

### 转换（Transformation）

编译器的下一个阶段是转换（Transformation）。这里可以获取AST并将其修改，你可以用同一种语言操作AST，也可以将他转换成一个全新语言的AST。

让我们来看看怎么转换AST

您可能注意到，AST中的元素看起来非常相似。这些对象具有类型属性。这些节点都称为AST节点。这些节点在其上定义了描述树的一个独立部分的属性。

我们可以有一个数字文本节点`NumberLiteral`

```js
{
  type: 'NumberLiteral',
  value: '2',
}
```

或有个调用表达式节点`CallExpression`

```js
{
   type: 'CallExpression',
   name: 'subtract',
   params: [...nested nodes go here...],
}
```

在转换AST时，我们可以通过添加/删除/替换属性来操作节点，也可以添加新节点、删除节点，或者不使用现有的AST，而是基于它创建一个全新的AST。

由于我们的目标是一种新语言，所以我们将专注于创建一个针对目标语言的全新AST。

#### Traversal

为了浏览所有这些节点，我们需要能够浏览它们。这个遍历过程首先遍历AST深度中的每个节点。

```js
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2'
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4'
      }, {
        type: 'NumberLiteral',
        value: '2'
      }]
    }]
  }]
}
```

对于以上AST，我们需要做：

1. Program - 从AST的顶层开始
2. CallExpression (add) - 移动到程序（`Program`）主体（`body`）的第一个元素
3. NumberLiteral (2) - 移动到调用表达式（`CallExpression`）参数的第一个元素
4. CallExpression (subtract) - 移动到调用表达式（`CallExpression`）参数的第二个元素
5. NumberLiteral (4) - 移动到调用表达式（`CallExpression`）参数的第一个元素
6. NumberLiteral (2) - 移动到调用表达式（`CallExpression`）参数的第二个元素

如果我们直接操作这个AST，而不是创建一个单独的AST，我们可能会在这里引入各种抽象用于访问（`visiting`）树中的每个节点。

我之所以使用“`visiting`”这个词，是因为它是一种如何表示对象结构元素上的操作的模式。

#### Visitors

这里的基本思想是，我们将创建一个具有接受不同节点类型的方法的“visitor”对象。

```js
var visitor = {
  NumberLiteral() {},
  CallExpression() {},
};
```

遍历AST时，当匹配到相应节点类型时，就调用该访问器`visitor`上的方法，并且触发`entry`。
为了使其有用，我们还将传递节点和父节点。

```js
var visitor = {
  NumberLiteral(node, parent) {},
  CallExpression(node, parent) {},
};
```

但是，也存在调用“exit”上的东西的可能性。想象一下我们之前的树结构，以列表的形式:

```js
- Program
  - CallExpression
    - NumberLiteral
    - CallExpression
      - NumberLiteral
      - NumberLiteral
```

随着我们向下遍历，我们会到达一些分支的尽头，当我们结束分支时我们需要“`exit`”，因此，向下遍历树时我们“`entry`”每个节点，需要返回时我们则“`exit`”。

```js
-> Program (enter)
  -> CallExpression (enter)
    -> Number Literal (enter)
    <- Number Literal (exit)
    -> Call Expression (enter)
       -> Number Literal (enter)
       <- Number Literal (exit)
       -> Number Literal (enter)
       <- Number Literal (exit)
    <- CallExpression (exit)
  <- CallExpression (exit)
<- Program (exit)
```

为了支持这一点，我们的`visitor`的最终形式如下:

```js
var visitor = {
  NumberLiteral: {
    enter(node, parent) {},
    exit(node, parent) {},
  }
};
```

### 代码生成（Code Generation）

编译器的最后一个阶段是代码生成。有时编译器会做一些与转换重叠的事情，但在大多数情况下，代码生成只是意味着将AST和string-ify代码拿出来。

Code generators work several different ways, some compilers will reuse the tokens from earlier, others will have created a separate representation of the code so that they can print node linearly, but from what I can tell most will use the same AST we just created, which is what we’re going to focus on.

实际上我们的代码生成器将会输出所有不同AST节点，并且遍历它自生，输出内部嵌套节点，直到所有内容都打印成一长串代码为止。

介绍完了，这就是一个编译器的各个部分内容。
这并不是说每个编译器都像我在这里描述的那样。
编译器有许多不同的用途，它们可能需要比我详细介绍的更多的步骤。
但是现在您应该对大多数编译器有了一个大致的高级概念。
既然我已经解释了所有这些，那么就可以编写自己的编译器了，对吧?

***[The Super Tiny Compiler（下）](/2018/12/09/ast-part2/)***
