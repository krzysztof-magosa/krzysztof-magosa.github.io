---
layout: post
title: First steps with parsers - part 4
excerpt: Introduction to different data types in lex and yacc...
tags: [programming, c, parsers]
---

Hi guys.

This time I will tell you few words about handling different data types in our parser.

By default _yacc_ gets token's value from _lex_ as an integer (in `yylval` variable).
While it was fine for our integer-only calculator there is a lot of situations when we
may need other data types like double, float or string.

Fortunately _yacc_ gives us ability to use different type for `yylval`:

```
%union {
    double dval;
    char cval;
    // type name;
}
```

This way `yylval` becomes union and we can use its fields to pass different data types.

We will use `dval` for storing double numbers, so our calculator can do floating point math,
and `cval` for storing single letter name of variables.

Rule responsible for catching numbers needs to be adjusted, so it handles also numbers with point.
I will also add three rules:

* first for catching single letters
* second for catching equals sign
* last one for ignoring white characters, so we can write formulas like `1 + 100`

`T_INT` has been changed to `T_NUM` so its name says what it actually contains.

```
[0-9]+(\.[0-9]+)? yylval.dval = atof(yytext); return T_NUM;
[a-zA-Z]          yylval.cval = yytext[0]; return T_LETTER;
=                 return T_EQ;
[ \t]
```

As you can see we just use proper field in union to store appropriate type of data.
In _yacc_ we also need to correlate tokens to proper fields, but it's done with different syntax.
I've added also our new token `T_EQ`.

```
%token <dval> T_NUM
%token <cval> T_LETTER
%token T_NL T_EQ
```

Now when we use `T_NUM` or `T_LETTER` _yacc_ knows which field of union should be used.
We need also to specify types returned by our rules:

```
%type <dval> case
```

Just one thing and our calculator will support floats - format used to display result:

```
line
    : T_NL
    | case T_NL          { printf("Result = %f\n", $1); }
```

Okay, it's time for quick test:

```
2.5+2.5
Result = 5.000000

10+10.0
Result = 20.000000

10/3
Result = 3.333333
```

Looks good!  
Now we will add support for variables.

We will need array for storing them.
I've used 256 elements variable for simplicity - ASCII code of letter will be used as index.

```
double variables[256];
```

And inside main() we will clear it, so initially all variables are set to 0.

```
int main(void) {
    memset(variables, 0, sizeof(variables));
    yyparse();
}
```

We I will modify `case` rule, so formula can contain variable:

```
case
    : T_NUM              { $$ = $1; }
    | T_LETTER           { $$ = variables[$1]; } // here
    | case T_ADD case    { $$ = $1 + $3; }
    | case T_SUB case    { $$ = $1 - $3; }
    | case T_DIV case    { $$ = $1 / $3; }
    | case T_MUL case    { $$ = $1 * $3; }
    | case T_POW case    { $$ = pow($1, $3); }
    | T_SUB case %prec U_NEG { $$ = -$2; }
```

The only missing thing is ability to set variable's value:

```
line
    : T_NL
    | case T_NL      { printf("Result = %f\n", $1); }
    | T_LETTER T_EQ case T_NL { variables[$1] = $3; }
```

That's all, now we can check how it works:

```
./program
a = 100
b = 10

c = a / b

90 + c
Result = 100.000000
```

It works like a charm ;-)  
You can find complete example on [GitHub](https://github.com/krzysztof-magosa/blog-examples/tree/master/parsers/calc4).

Thanks for reading, to the next time!
