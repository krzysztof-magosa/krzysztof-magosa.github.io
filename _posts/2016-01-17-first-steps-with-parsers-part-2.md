---
layout: post
title: First steps with parsers - part 2
excerpt: Article about chaining operators in yacc...
tags: [programming, c, parsers]
---

Hi guys.
In the previous article I showed you how to build very simple calculator.
We will improve it, so it can calculate something more than simple 2+2.

First thing we need is ability to do more operations in one formula, like that:

```
1+2+3
```

How to achieve that? We could build grammar for 2, 3, 4... operands,
but I assume you don't want to spend rest of your life writing grammar
for all possible combinations :)

What about splitting "hard" formula into two smaller and simpler ones?

```
1+2|+3
3+3=6
```

This way we still need to just support two operands, and one operator in the middle.
So, what's the difference? Previously we assumed that formula is `integer operator integer`.
Above example shows that in addition to integers, we should accept also another formulas
on the left and right side of operator.

Instead of immediate printing of result, we will calculate result, so it can be used later
as a operand for neighboring formula.

```
case
    : case T_ADD case    { $$ = $1 + $3; }
    | case T_SUB case    { $$ = $1 - $3; }
    | case T_DIV case    { $$ = $1 / $3; }
    | case T_MUL case    { $$ = $1 * $3; }
```

To return value in _yacc_ we use pseudo variable `$$`.
We missed one thing, situation when operand is just integer and not another formula:

```
case
    : T_INT              { $$ = $1; }
    | case T_ADD case    { $$ = $1 + $3; }
    | case T_SUB case    { $$ = $1 - $3; }
    | case T_DIV case    { $$ = $1 / $3; }
    | case T_MUL case    { $$ = $1 * $3; }
```

Because now our formula have variable length we need something saying that's end of it.
I decided to use new line character, so after typing formula and pressing enter we will see the result.

Let's define additional rule in `main.l` file, so it detect new line character:

```
\n    return T_NL;
```

Of course token needs to be defined in `main.y`:

```
%token T_INT T_ADD T_SUB T_DIV T_MUL T_NL
```

Previously our program accepted list of cases on the input,
we will change it to accepting list of lines with case:

```
list
    : line
    | list line

line
    : T_NL
    | case T_NL          { printf("Result = %d\n", $1); }
```

Let's try that:

```
lex main.l
yacc -d main.y
gcc -o program lex.yy.c y.tab.c
```

```
./program
2+2*2
Result = 6
2*2+2
Result = 8
```

Okay, fine, we support more operands, but our calculator doesn't know proper order of operations.
Fortunately _yacc_ supports operators associativity and precedence, so we can easily fix that.

```
%token T_INT T_NL

%left T_ADD T_SUB
%left T_DIV T_MUL
```

Instead of just defining tokens by `%token%`, we use `%left` directive to inform _yacc_ that these operators are left-associative
(I will tell more about that next time). Tokens defined later have higher precedence, so our calculator will
perform division and multiplication first and then addition and subtraction.

```
2+2*2
Result = 6
2*2+2
Result = 6
```

Yeah, now it's correct!  
That's all for today, thank you for reading!  
You can find full example on
[GitHub](https://github.com/krzysztof-magosa/blog-examples/tree/master/parsers/calc2).
