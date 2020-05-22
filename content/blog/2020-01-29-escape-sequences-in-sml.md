---
title: Char.toString vs String.str in SML/NJ
subtitle: How to handle escape sequences in ML-Lex
date: '2020-01-29'
slug: escape-sequences-in-sml
categories:
  - Compiler
tags:
  - compiler
---

References:  
[The CHAR signature](http://sml-family.org/Basis/char.html)  
[The STRING signature](http://sml-family.org/Basis/string.html)

## 1. The signature of the two functions

### 1.1 Char.toString

> `val toString : char -> String.string`
> returns a printable string representation of the character, using, if necessary, SML escape sequences.

The specific rules are as follows.

1. Printable characters, except for `#"\\"` and `#"\""`, are left unchanged.
2. Backslash `#"\\"` becomes `"\\\\"`; double quote `#"\""` becomes `"\\\""`.
3. The common control characters are converted to two-character escape sequences:
    * Alert (ASCII 0x07)                `"\\a"`
    * Backspace (ASCII 0x08)            `"\\b"`
    * Horizontal tab (ASCII 0x09)       `"\\t"`
    * Linefeed or newline (ASCII 0x0A)	`"\\n"`
    * Vertical tab (ASCII 0x0B)         `"\\v"`
    * Form feed (ASCII 0x0C)            `"\\f"`
    * Carriage return (ASCII 0x0D)      `"\\r"`
4. The remaining characters whose codes are less than 32 are represented by three-character strings in *control character* notation, e.g., `#"\000"` maps to `"\\^@"`, `#"\001"` maps to `"\\^A"`, etc. For characters whose codes are greater than 999, the character is mapped to a six-character string of the form `"\\uxxxx"`, where `xxxx` are the four hexadecimal digits corresponding to a character's code. All other characters (i.e., those whose codes are greater than 126 but less than 1000) are mapped to four-character strings of the form `"\\ddd"`, where ddd are the three decimal digits corresponding to a character's code.

### 1.2 String.str

> `val str : char -> string`
> is the string of size one containing the character `c`.

## 2. Comparison

From the signatures/definitions of the two functions, we can find the `String.str` returns the string representation that we expect like in Java or C++. And `Char.toString` takes one step further - it returns the string which can print the escape sequences of the character, instead of the character itself. For example, for `char #"a"`, both of the two functions return `string "a"`. But for `char #"\n"`, `String.str` returns `string "\n"`, while `Char.toString` returns `string "\\n"`.

```bash
lexer git:(master) ✗ sml
Standard ML of New Jersey v110.79 [built: Tue Aug  8 23:21:20 2017]
- val nl : char = #"\n";
val nl = #"\n" : char
- String.str #"a";
val it = "a" : string
- Char.toString #"a";
val it = "a" : string
- String.str #"\n";
val it = "\n" : string
- Char.toString #"\n";
val it = "\\n" : string
- String.str #"\\";
val it = "\\" : string
- Char.toString #"\\";
val it = "\\\\" : string
- String.str #"\"";
val it = "\"" : string
- Char.toString #"\"";
val it = "\\\"" : string
- String.str #"\";
= ;
stdIn:6.12-6.17 Error: unclosed string
-
lexer git:(master) ✗
```

## 3. Char.fromString

> `val fromString : String.string -> char option`

This function is corresponding to `Char.toString` instead of `String.str`. (But for `\ddd` it seems both `\103` and `\\103` will work. ^-^)

```bash
lexer git:(master) ✗ sml
Standard ML of New Jersey v110.79 [built: Tue Aug  8 23:21:20 2017]
- Char.fromString "a";
val it = SOME #"a" : char option
- Char.fromString "\n";
val it = NONE : char option
- Char.fromString "\\n";
val it = SOME #"\n" : char option
- Char.fromString "\\\"";
val it = SOME #"\"" : char option
- Char.fromString "\"";
val it = NONE : char option
- Char.fromString "\103";
val it = SOME #"g" : char option
- Char.fromString "\\103";
val it = SOME #"g" : char option
-
kz75: lexer git:(master) ✗
```

## 4. How to handle escape sequences in ML-Lex

For a single character, we can just write the escape sequences to match it.

```
<INITIAL> \" => (YYBEGIN STRING; state := stringState; str := ""; strLeftPos := yypos; continue());
<STRING> \" => (YYBEGIN INITIAL; state := initialState; Tokens.STRING(!str, !strLeftPos, yypos));
<STRING> \\ => (YYBEGIN FORMATTING; continue());
<FORMATTING> \\ => (YYBEGIN STRING; continue());
```

For a escape sequence in the string literal, we need match each character in the sequence separately. For example, if we want to match `"\n"`, we need RegExp `\\n` and the `yytext` is `\\n` instead of `\n`. So, we should handle the escape sequences in string literals as follows.

```
...
%%
...
escape = \\n|\\t|\\\"|\\\\|\\{digit}{digit}{digit};
...
%%
<STRING> {escape} => (str := !str ^ (String.str (valOf (Char.fromString yytext))); continue());
```

Here, when we see `"hello\nworld"`, what we want to add in the string is `string "\n"`. But `yytext` is `string "\\n"`. So, first we use `valOf (Char.fromString yytext)` to get `char #"\n"`. Then, we use `String.str` to convert it to `string "\n"`.