---
layout: post
title: First steps with parsers
excerpt: Introduction to the first program using lex and yacc...
tags: [programming, c, parsers]
---

Hi Folks!
This is my very first article, so I count on your feedback.

One of the very popular parts of software is parser.
We use parsers every day, whether we open website, compile our program or load configuration from file.
A lot of people forget about them because they do hard work behind the scenes.
For example, if you want to load configuration file you will definitely find proper library so you can
load it without knowledge about parsers. But what happens if you would like to create
Domain Specific Language or own format of configuration file? Then you need to know how to build
parsers, and in this cycle of articles I will try to show you how to do that.

And when we say _parser_ we usually mean lexical analyser connected with actual parser which understands
the grammar of language we would like to parse.

Commonly used toolset is:

* _lex_ - lexical analyser generator
* _yacc_ - parser generator

We will use these tools to build our software.

_lex_ files have `.l` extension, _yacc_ files use `.y`.
Both of them are structured the same way:

```
Declarations section
%%
Rules section
%%
Code section
```

For the simplicity of tutorial I will paste only
important fragments (usually rules section), complete examples will be published on GitHub.

Our first application will be very simple calculator.
Let's look how simple math operation looks like:

```
2+2
```

There is integer, operator and again integer.
Now let's define tokens needed to build such cases.

* T_INT - integer number
* T_ADD - plus sign (+)
* T_SUB - minus sign (-)
* T_DIV - division sign (/)
* T_MUL - multiplication sign (*)

Defining tokens for operators is quite intuitive:

```
\+    return T_ADD;
-     return T_SUB;
\/    return T_DIV;
\*    return T_MUL;
=     return T_EQ;
```

We've used escape character (`\`) because some characters have special meaning but we wanted to use
them literally. Now let's define T_INT, for simplicity we will support just unsigned integers:

```
[0-9]+  return T_INT;
```

Is that sufficient? No, in addition to information that's a number, we would like to have it's actual value.
We will use C function _atoi_ which converts string into a integer.

```
[0-9]+  yylval = atoi(yytext); return T_INT;
```

Rules for our lexer are ready, that's sufficient to parse very simple mathematical formulas.

Now let's define our grammar - list of rules how tokens can be used with each other.
In the input of our program we expect list of cases to calculate:

```
list
    : case
    | list case
```

It means that _list_ may be composed of single _case_ or it can contain _list_ followed by _case_.
It looks tricky for the first look, but it's just recursion which allows us to parse
variable-length list of tokens.

Now we will define _case_:

```
expr
    : T_INT T_ADD T_INT  { printf("%d\n", $1 + $3); }
    | T_INT T_SUB T_INT  { printf("%d\n", $1 - $3); }
    | T_INT T_DIV T_INT  { printf("%d\n", $1 / $3); }
    | T_INT T_MUL T_INT  { printf("%d\n", $1 * $3); }
```

It means that expression may be one of 4 basic operations:

* number + number
* number - number
* number / number
* number * number

For each combination it performs desired operation and display it's result.
_$1_ and _$3_ points accordingly to first and third argument of our rule.

We need one more thing, declare tokens so _yacc_ knows which ones will be used.

```
%token T_INT T_ADD T_SUB T_DIV T_MUL
```

This list if also used to generate file `y.tab.h` which is used by _lex_.
By using consts defined in this file _lex_ communicates with _yacc_.

I've saved my lexical rules into file `main.l` and grammar into `main.y`.
Let's compile that.

```
lex main.l
yacc -d main.y
gcc -o program lex.yy.c y.tab.c
```

Now let's do the math ;-)

```
./program
2+2
4

10-7
3

10*10
100

100/10
10
```

It's very basic, but it does exactly what we want from it.
In the next article I will show you how to extend our application to calculate more complicated formulas.

You can find complete example on [GitHub](https://github.com/krzysztof-magosa/blog-examples/tree/master/parsers/calc1).

Thank you for reading!
