---
layout: post
title: First steps with parsers - part 3
excerpt: Few words about operators associativity in yacc...
tags: [programming, c, parsers]
---

Hi guys.

In the previous article I've used `%left` directive.  
Now it's time to tell you few words about what it does.

Let's look into example:

```
10 - 5 + 2
```

Because we defined addition and subtraction with the same precedence
we could compute this formula in two ways without breaking the rule:

```
(10 - 5) + 2 = 7 # left-associative (left to right)
10 - (5 + 2) = 3 # right-associative (right to left)
```

> Associativity of operator defines how operators with the same precedence are grouped
> when there are no parentheses. Operators can be also non-associative what prevents
> them from be used in chains.

Basic mathematical operators like `+`, `-`, `/`, `*`
are usually defined as left-associative, while exponential `**` is right-associative.

Now we should be able to add exponential operator to our calculator.

We will use `**`, like in Ruby or Python:

```
\*\*                    return T_POW;
```

Definition of our right-associative exponential operator needs to be added
below `T_DIV` and `T_MUL`, so it has higher precedence - according to mathematical rules.

```
%left T_ADD T_SUB
%left T_DIV T_MUL
%right T_POW // this one
```

And define what our operator actually does:

```
case
    : T_INT              { $$ = $1; }
    | case T_ADD case    { $$ = $1 + $3; }
    | case T_SUB case    { $$ = $1 - $3; }
    | case T_DIV case    { $$ = $1 / $3; }
    | case T_MUL case    { $$ = $1 * $3; }
    | case T_POW case    { $$ = pow($1, $3); } // this one
```

Okay, give it a try:

```
2+2**2
Result = 6

2+2*2**2
Result = 10

-2**2
error: syntax error
```

Results look good, however our calculator failed on negative number - it expects a number before minus sign.

To support negative numbers, we need to create unary minus operator:

```
%left T_ADD T_SUB U_NEG
%left T_DIV T_MUL
%token U_NEG // this one
%right T_POW

[...]

case
    : T_INT              { $$ = $1; }
    | case T_ADD case    { $$ = $1 + $3; }
    | case T_SUB case    { $$ = $1 - $3; }
    | case T_DIV case    { $$ = $1 / $3; }
    | case T_MUL case    { $$ = $1 * $3; }
    | case T_POW case    { $$ = pow($1, $3); }
    | T_SUB case %prec U_NEG { $$ = -$2; } // this one
```

Mathematical rules say that exponential should be calculated before
applying negation to it's result, so we have defined pseudo token
`U_NEG`, and then used `%prec U_NEG` to inform _yacc_ that it should
use its precedence for this rule.

This way it works like that:

```
-3**2 = -(3**2) = -9
```

Let's compile and check:

```
lex main.l
yacc -d main.y
gcc lex.yy.c y.tab.c -o program
```

```
./program
-3**2
Result = -9
-3+3
Result = 0
3--3
Result = 6
3+-3
Result = 0
```

Results exactly as expected.  
You can see complete example on [GitHub](https://github.com/krzysztof-magosa/blog-examples/tree/master/parsers/calc3).

That's all for today folks. To the next time!
