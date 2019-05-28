[![Build Status](https://travis-ci.com/jimmort25/jimlang.svg?branch=master)](https://travis-ci.com/jimmort25/jimlang)

## Summary
This language can be thought of as "Mini-Haskell". It aims to provide a set of powerful primitives for
enabling the programmer to write code in a pure, functional style. The 'vanilla' features, especially with regard to the `EnvUnsafeLog`
monad, form the plumbing for the language and allow practical things like persisting the print buffer between function calls. The mix-in
features we chose aim to provide the functionality expected out of a Turing-complete programming language, while simultaneously giving the
programmer the comfort they would normally receive from a scaled-down version of Haskell (see "Mini-Haskell").

### Instructions
Since this project has a configured Travis-CI instance, the `project.cabal` file is located in the directory entitled
`BU-CS320/project-avengers-infinity-list`. Hence, you must `cd` into this directory in order to run the tests using
`cabal new-test`. Further, if you would like to interact with the project directly, first `cd` into the directory mentioned
previously, and then run `cabal new-repl`. From here, load the root module through the command: `:load Exec` and then simply
type `exec <string to parse>` and you will be able to parse and evaluate a wide array of different expressions. The result returned
by this call has three distinct forms. The first, indicating success, is of the form: `Ret <result of computation> <print buffer> <scope warnings>`.
The second, indicating a successful parse but an unsuccessful evaluation (potentially because your expression is malformed), is of the form:
`RuntimeError <error message> <print buffer> <scope warnings>`. Finally, the third is simply `ParseError`, which occurs when the parser is unable
to make sense of the string you've given.

### Modules

#### Ast.hs
This module outlines the abstract syntax tree for the language and should give sufficient background for the types of expressions that are supported by default.
Mainly, our AST consists of atomic values (chars, booleans, integers, floats, strings, etc.) and then binary operators which can be chained together and nested at
arbitrary depths. Our `Let`, `Lam`, `App`, and `Compose` data form the basis for the functional aspect of our language and they are treated as first-class citizens
throughout the parsing and evaluation logic.

#### ParserMonad.hs
`ParserMonad` forms the monadic plumbing for our parser. It is essentially a modified version of the Maybe Monad in which `Nothing` represents a total failure of the
parser and `Just (<parsed string>, <rest of input string>)` allows for a chaining of subparsers that are applied according to a precedence ordering.

#### Parser.hs
The `Parser` module contains all of the parsing logic for the language. The subparsers are chained together according to precedence laws which vary between mirroring
those of Haskell and those of the C/C++ family. The main export of this API is the `parser` which, when combined with the `parse` execution method, can be used in the
`repl` to show what an input string would be transformed into in terms of our abstract syntax tree via the command `parse parser <input string>`. 

#### Eval.hs
This module defines the evaluation logic for our AST and contains the data type `Val` which consists of definitions corresponding to 6 different values:
<pre>
I               Integer Types
F               Floating-Point Types
B               Boolean Types
C               Char Types
S               String
Ls              List Types
Fun             Function Types
Error           Error Type (**)
------------------------------------------------------
(**) For when values don't conform to our `Val` system.
</pre>

#### Check.hs
The `Check` module performs static checking on the abstract syntax tree returned by a successful parse on the input string. The main aspect of the `Check` module's
API is the `check` method, which, given an abstract syntax tree, will warn if a variable is used in an expression but no accompanying definition exists in the scope.

#### Exec.hs
`Exec` serves as the main entry point to our API. It combines `Parser`, `Check`, and `Eval`, applying them in sequence to generate a result from an input string
with contextual error reporting, static variable checking, as well as much more. In combination with a means of I/O (perhaps the I/O monad), the method `exec` can be
integrated to parse entire files with little extra work required.



