---
layout: single
title:  "Cmd and Conquer: Visualising CMD's Abstract Syntax Tree"
---

In November 2018 FireEye released a report titled [Cmd and Conquer: De-DOSfuscation with flare-qdb](https://www.fireeye.com/blog/threat-research/2018/11/cmd-and-conquer-de-dosfuscation-with-flare-qdb.html).
The report introduces a tool called [De-Dosfuscator](https://github.com/fireeye/flare-qdb/blob/master/flareqdb/scripts/deDOSfuscator.py) which helps solve the problem of analysing highly obfuscated CMD instructions by hooking `CMD.EXE` to extract commands "in the clear".

Towards the end of the article, the author shares an easter egg they discovered while looking through the `CMD.EXE` disassembly.  Buried within `CMD.EXE` is a flag named `fDumpParse` which can be used to dump an [Abstract Syntax Tree (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree).

![CMD AST Output using fDumpParse with De-Dosfuscator]({{ site.url }}/assets/images/posts/cmdastviz/cmdast-example.png)

The above screenshot shows the AST output from De-Dosfuscator.  Given the input expression:

```
echo foo && echo bar
```

De-Dosfuscator produces the AST:
```
&&
  Cmd: echo  Type: 0 Args: ` foo '
  Cmd: echo  Type: 0 Args: ` bar'
```

This is a textual representation of an AST in pre-order, sometimes called [Polish Notation](http://localhost:4000/cmd-and-conquer-visualising-cmds-ast/).  We see the operator `&&` at the top-level, with two equally indented `Cmd: ...` nodes below.  Conceptually, this output represents a tree, with `&&` at the root, and the two `Cmd` nodes forming left and right branches:
```
    &&
   /  \
Cmd1  Cmd2
```

As the author of the FireEye report points out, generating an AST from CMD in this format can be quite tricky to read, given the AST is not presented in the same order as the original command was entered.  Furthermore, eyeballing the AST to make any sense of the input becomes almost impossible when the input command is large, as is often the case with obfuscated `CMD` sequences.

Given this hard-to-read limitation, I thought it may be interesting to convert the AST in to a graphical format to make it slightly easier to see what was going on.

Given the following input:

```
  (for %a in (1 1 50) Do (echo foo && echo bar)) && echo baz
```
`fDumpParse` produces the AST:
```
  &&
    (
      for %a in (1 1 50) Do
        (
          &&
            Cmd: echo  Type: 0 Args: ` foo '
            Cmd: echo  Type: 0 Args: ` bar'
    Cmd: echo  Type: 0 Args: ` baz'
```
...which I feed in to my parser (`CMD-AST-View`), which in turn produces a [GraphViz](https://www.graphviz.org/) drawing:

![Example CMDASTView Output]({{ site.url }}/assets/images/posts/cmdastviz/ex2.ast.png)

While it may not solve the problem of presenting a pre-order AST, `CMD-AST-View` does *hopefully* help make the AST easier to read.  The GitHub project contains an [F#](https://fsharp.org/) library for parsing the `fDumpParse` output, and a command line utility that reads an AST from STDIN, and writes a GraphViz program to STDOUT, similar to:

```sh
cat example.ast | cmdast2dot | dot -Tpng -o example-ast.png
```

The F# library means the utility can be used on Mac, Linux, and Windows.  Full usage and build instructions availalbe from the project's README.md.

![Example CMDASTView Output]({{ site.url }}/assets/images/posts/cmdastviz/ex1.ast.png)
![Example CMDASTView Output]({{ site.url }}/assets/images/posts/cmdastviz/ex3.ast.png)