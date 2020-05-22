---
title: Shell Scripting Pattern Matching
subtitle: Regular Expressions, Globbing and Brace Expansion
date: '2018-08-27'
slug: shell-scripting-pattern-matching
categories:
  - Linux
tags:
  - Shell
---

Reference:  
[Advanced Bash-Scripting Guide](http://www.tldp.org/LDP/abs/html/)  
[Bash Reference Manual](https://www.gnu.org/software/bash/manual/html_node/)

# [Regular Expressions](http://www.tldp.org/LDP/abs/html/regexp.html)

> An expression is a string of characters. Those characters having an interpretation above and beyond their literal meaning are called metacharacters. Regular Expressions are sets of characters and/or metacharacters that match (or specify) patterns.
> 
> The main uses for Regular Expressions (REs) are text searches and string manipulation. An RE matches a single character or a set of characters -- a string or a part of a string.

首先，Regular Expression 是正则表达式，是一种用来定义搜索模式的字符序列，可以用于字符串查找等操作。其应用比较广泛，很多 commands 和 utilities 可以解释 regex，但是 shell 本身无法解释 regex。shell 引入了多种其他的 expansion 机制：如 globbing 的机制来进行 filename expansion，brace expansion 的机制来生成任意字符串，其与 regex 存在一定的差异。

下边是 WIKIPEDIA 对 Regular Expression 的定义：

> A regular expression, regex or regexp (sometimes called a rational expression) is, in theoretical computer science and formal language theory, a sequence of characters that define a search pattern. Usually this pattern is then used by string searching algorithms for "find" or "find and replace" operations on strings, or for input validation.

# [Globbing](http://www.tldp.org/LDP/abs/html/globbingref.html)

> Bash itself **cannot** recognize Regular Expressions. Inside scripts, it is **commands** and **utilities** -- such as sed and awk -- that interpret RE's.
> 
> Bash does carry out **filename expansion** -- a process known as **globbing** -- but this does not use the standard RE set. Instead, globbing recognizes and expands wild cards. Globbing interprets the standard wild card characters -- * and ?, character lists in square brackets, and certain other special characters (such as ^ for negating the sense of a match). There are important limitations on wild card characters in globbing, however. Strings containing * will not match filenames that start with a dot, as, for example, .bashrc. Likewise, the ? has a different meaning in globbing than as part of an RE.
>
> Bash performs filename expansion on unquoted command-line arguments.

虽然 shell 中调用的一些 commands/utilities 可以解释 regex，但是 shell 本身无法解释 regex。globbing 是 shell 引入的一种 filename expansion 机制。

下边是 WIKIPEDIA 对 glob 的定义：

> In computer programming, glob patterns specify sets of filenames with wildcard characters. For example, the Unix command `mv *.txt textfiles/` moves (mv) all files with names ending in .txt from the current directory to the directory textfiles. Here, * is a wildcard standing for "any string of characters" and *.txt is a glob pattern. The other common wildcard is the question mark (?), which stands for one character.

# [Brace Expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html#Brace-Expansion)

> Brace expansion is a mechanism by which arbitrary strings may be generated. This mechanism is similar to filename expansion (see Filename Expansion), but the filenames generated need not exist. Patterns to be brace expanded take the form of an optional preamble, followed by either a series of comma-separated strings or a sequence expression between a pair of braces, followed by an optional postscript. The preamble is prefixed to each string contained within the braces, and the postscript is then appended to each resulting string, expanding left to right.
>
> Brace expansion is performed before any other expansions, and any characters special to other expansions are preserved in the result.

Brace expanssion 是另一种 **shell expanssion**，是一种用来生成任意字符串的机制。

下边是 WIKIPEDIA 对 barce expansion 的定义：

> Brace expansion, also called alternation, is a feature copied from the C shell. It generates a set of alternative combinations. Generated results need not exist as files. The results of each expanded string are not sorted and left to right order is preserved.

# Regular Expressions, Globbing and Brace Expansion 的对比

1. Regular Expressions 是一种匹配字符串的机制，用来进行字符串的搜索等相关操作。
2. Globbing 和 Brace Expansion 都是 Shell Expansion 的一种，Globbing 专门用来进行 Filename Expanssion，而 Brace Expansion 用来生成任意的字符串。
3. 在进行文件匹配时，Globbing 和 Braces 有时候可以通用，如 `rm file{1,a,X}.txt` 等价于 `rm file[1aX].txt`。但是 Braces 在某些场合更加合适。
4. Globbing only operates on the file names in the local directory. 而 Braces 由于相当于生成了字符串，其可以应用于操作远程电脑上的文件，如 `scp user@computer:~/file{1,2,3}.txt ./`。
5. Braces 实用于备选项长度大于 1 的情况，如 `rm dir1/dir2/{abc,xyz}.txt`，而 Globbing 只适用于备选项长度为 1 的情况。
6. Globbing & Braces both can be used multiple times in one argument, in which case you get all possible pairings of the expansions. For example, `{a,b,c}{1,2,3}` and `rm [abc][123].txt`。

# [Shell Expansions](https://www.gnu.org/software/bash/manual/html_node/Shell-Expansions.html#Shell-Expansions)

Expansion is performed on the command line after it has been split into tokens. There are seven kinds of expansion performed:

* brace expansion
* tilde expansion
* parameter and variable expansion
* command substitution
* arithmetic expansion
* word splitting
* filename expansion

> Brace Expansion:	  	Expansion of expressions within braces.  
> Tilde Expansion:	  	Expansion of the ~ character.  
> Shell Parameter Expansion:	  	How Bash expands variables to their values.  
> Command Substitution:	  	Using the output of a command as an argument.  
> Arithmetic Expansion:	  	How to use arithmetic in shell expansions.  
> Process Substitution:	  	A way to write and read to and from a command.  
> Word Splitting:	  	How the results of expansion are split into separate arguments.  
> Filename Expansion:	  	A shorthand for specifying filenames matching patterns.  
> Quote Removal:	  	How and when quote characters are removed from words.

The order of expansions is: brace expansion; tilde expansion, parameter and variable expansion, arithmetic expansion, and command substitution (done in a left-to-right fashion); word splitting; and filename expansion.

# grep, find 等 programs 对 glob, regular expressions 的支持

Grep searches one or more files (or standard input if you do not specify any file names) for **lines** that include a particular pattern.

**Grep, and a variety of other tools that use similar patterns, describe them as “regular expressions” (“regexps” for short), which is mostly truetechnically speaking, grep’s patterns support features which go beyond the capabilities of true regular expressions.**

Find searches for a particular patƒtern (defined by options),  which takes the criteria to look for, and the path to look in. The criteria can be name of the file, or other things like "find files newer than some specific file."

Grep 和 Find 均支持 pattern matching，但二者的行为有很大的不同。

1. Grep 只支持 "regular expression"。
2. Find 需要通过参数来指定具体支持的规则，如 `-name [expression]` 中的匹配模式类似于 shell glob，若要使用 regular expression，需要用 `-regex` option。（貌似跟文件名相关的一般是 glob，其他地方一般是 regexps）

下边通过例子来演示，其中当前文件夹下有 hello.c, hello0.c, hello1.c 三个文件，文件中均含有一行为：hahaHellohaha

### Grep
**首先 Grep 只支持 "regular expressions"。** 但是可以用以下三种方式来表示：

1. 单引号 `''`
2. 双引号 `""`
3. 通过 `\` 来阻止 shell 对特殊字符进行解释。

<img src="https://i.gyazo.com/aed3c25aee5992d9db4b8fd692cda362.png" height="50%" width="50%"/>

从图中可以看出，`\` 表示的 expression 被直接传入 Grep 后是被当成 regexps 来解释的（`*` 代表重复之前的字符任意次），而非 shell glob 的规则（`*` 代表任意的字符或者字符串）。

### Find
与 Grep 不同的是，Find 需要通过 option 来指定具体采用的匹配机制，如 `-name` 用的是 shell glob，而 `-regex` 用的是 regular expressions。

`-name` 参数用于搜索文件名，具体使用方式如下：

1. 单引号 `''`
2. 双引号 `""`
3. 通过 `\` 来阻止 shell 对特殊字符进行解释。

<img src="https://i.gyazo.com/6d22e4ac06fd4f9ae3e2bc4943ee26eb.png" height="50%" width="50%"/>
