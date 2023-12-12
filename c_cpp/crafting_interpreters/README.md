# Crafting Interpreters 

Let's build a language interpreter, this takes the book 

[Crafting Interpreters](https://www.craftinginterpreters.com/contents.html#top)

## A map of the territory

### Scanning

Scanning or **lexing** takes a linear stream of characters and chunks them in something like words. These are called **tokens**. Some tokens are single characters like __(__ or __,__ other are x characters long like numbers, string literals and identifiers (const).

Some characters are ignores or doesn't mean anything like whitespace or comments. The scanner usually discards these, leaving only important tokens.

```
var average = ( min + max ) / 2 ;
```
> every "word"/character separated by whitespace is a token
> and the full sentence is called "grammar"

### Parsing

A parser takes the flat sequence of tokens and builds a **tree** structure (also called **syntax tree** or **ASTs**).

```
statement var           average
                           |
expr binary                /
                        |     |
expr binary             +     2  expr. literal
                     /    \  
expr variable       min   max    expr. variable
```

### Static analysis

The first bit of analysis the most languages do is called **binding** or **resolution**. For each **identifier**, we find out where is defined. This is where **scope** comes into play.

For example in the expression a + b, we know that we are adding, but we don't know what are we adding.

If the language is statically typed, we do a type check. Once we know where __a__ and __b__ are declared, we can also figure out their types are, then if their types can't support the operation, we report a **type error**.

All this analysis information we need to store, there are a few places where we can:

 * Often it is stored as **attributes** on the syntax tree itself, extra fields in the nodes that are not initialized but filled later.
 * Other times, in a **look at table**, typically the keys to this table are identifiers, in that case we call it a **symbol table** and values are associated with what identifies to.
 * The most powerful bookkeeping tool is to transform the tree into an tirely new data structure that better expresse the semantics of code.

Everything at this point is considered **frontend** of the implementation.

### Intermediate representations

You can think of the compiler as a pipeline. The frontend is specific to the source language the program is written in. The backend is concerned about the final architecture where the program will run.

Between, the code may be stored in some **intermediate representation (IR)** that act as interface between the two ends.

That lets you support different source languages and platforms with only 1 compiler.

### Optimization

When the user's code is parsed, we can swap it up with a different program that has the same semantics but implements them more efficiently.

A simple example is **constant folding**, if some expression evaluates to the exact same value, we can do the evaluation at compile time and replace the code for the expression with its result.

```
pennyArea = 3.14159 * (0.75 / 2) * (0.75 / 2);
- we can change the code in compiler to:
pennyArea = 0.4417860938;
```

### Code generation

Once we have applied all optimization we can think of, the last step is **generating code** to a machine kind (assembly, etc.).

This part is officially the backend as the representation of the ode becomes more and more primitive. Now we have a decission to make, to generate real machine code or virtual code(**bytecode**), the first is native and superfast but not portable.

### Virtual machine

If we decide to opt for virtual code, we need to translate into machine code, we have 2 ways; 

 - Write a little mini-compiler for each target architecture
 - Write a virtual machine (VM), a program that emulates the hypothetical chip supporting the architecture on runtime

A VM is slower since it simulates the instructions on runtime every time it executes.

### Runtime

Now we can execute the user's code. If we compiled to machine code, simply ask the OS to load it. If compiled to bytecode, we need to start the VM and load the program.

In both cases we usually need some service that our language provides while running. For exmaple, if the language automatically manages memory, we need a garbage collector to reclaim unused bits. If the language supports "instance of" tests we need some representation to keep track of type during execution.

All of this is done at runtime, so it's called (oh surprise) the **runtime**. In a fully compiled language, the code is inserted directly into the resulting executable. For example in Go, each compiled app has it's own copy of Go's runtime directly embedded. If the language is run inside an interpreter or VM, then the runtime lives there (Java, Python, JS).

## Shortcuts and Alternate routes

Until now, it's the long path covering every possible phase. Though there are a few shortcuts and alternate paths.

### Single-pass compilers

Some simple compilers interleave parsing, analysis and code feneration so they produce code directly in the parser without allocating any syntax trees or any IRs. These **single-pass compilers** restrict the design of the language, that means that as soon as you see any expression, you need to know enough to correctly compile it (Pascal, C).

### Tree-walk interpreters

Some programming languages begin executing code after parsing it to an AST. To run the program, the interpreter traverses the syntax tree one branch and leaf at a time, evaluating each node as it goes. This is usually not used since it tends to be slow.

### Transpilers

Writing a complete backend for a language can be a lot of work. You can write a frontend and translate it to valid source code for another language. These are called **source-to-source compiler** or **transcompiler**. 

### Just-in-time compilation

Basically the language translates to a VM and when the program is loaded then, it compiles for the target platform, this is called **just-in-time compilation** or **JIT**.

## Compilers and Interpreters

What's the difference between compiler and interpreter. It depends, in terms of languages:
 - **Compiling** is an __implementation technique__ that involves translating a source language to another. When you generate bytecode or machine code, you are compiling. When you transpile to another high-level language, you are compiling too.
 - When we say a language implementatuion is a **compiler** it means that translates source code to anohter, but doesn't execute it.
 - When we say an implementation is an **interpreter** we mean that it takes the source code and executes it immediately.

``` 
Some examples

            COMPILER        MID-TERM        INTERPRETER
            java            c#              jilox
            Typescript      Lua             PHP3
            Rust            Go      
```


