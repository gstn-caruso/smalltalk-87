# Proposal for 1987 Smalltalk Standard Syntax

**Edited by:** L. Peter Deutsch, ParcPlace Systems  
**Contributors (alphabetical order, partial listing):** George Bosworth (Digitalk), Steve Burbeck (Softsmarts), L. Peter Deutsch (ParcPlace Systems), Barry Haynes (Apple Computer), Ralph Johnson (University of Illinois, Urbana-Champaign), J. Eliot B. Moss (University of Massachusetts, Amherst), Dave Thomas (Carleton University), David Ungar (Stanford University), Steve Vegdahl (Tektronix), Allen Wirfs-Brock (Tektronix).

> Copyright (C) 1987 by ParcPlace Systems. All rights reserved. Nothing in this document constitutes a commitment by ParcPlace Systems to implement or support any facility discussed here.

This is a working document proposing a complete definition of the syntax of the 1987 version of the Smalltalk-80 (TM) language. This language is a revision of the Smalltalk-80 language defined in *Smalltalk-80: The Language and Its Implementation* (Addison-Wesley, 1983), which we will refer to below as the Blue Book.

This document is meant to define the language syntax fully and formally, but only deals partially and informally with the semantics of the language. We intend to augment this document with a complete formal definition of the semantics of the language at some future time. We also recognize that the usefulness of the Smalltalk-80 language depends heavily on the set of defined classes and messages: in this respect it resembles Lisp, and contrasts with most other languages. We specify a very small set of classes and messages which we believe are required to support any useful Smalltalk-80 system, and which we consider a suitable starting point for standardization; however, we recognize that we have not dealt with this subject adequately.

Changes made in each version are not marked in the text, because they interfere too much with the reading of the equations and tables: the version history summarizes the important changes. Comments are solicited.

You will find this document easier to read if you print it in a fixed-pitch or nearly fixed-pitch font.

Smalltalk-80 is a trademark of ParcPlace Systems.

## Version history

- **[6] 1 July 1987:** Incorporates George Bosworth’s cover letter, with some minor additions and clarifications discovered by Glenn Krasner; moves block arguments and assignment operators to lexical syntax, to avoid problems with where whitespace can appear; changes primitives (again) to be specified either by class and selector, or by string. This version was sent to all workshop participants for approval.
- **[5] 5 June 1987:** Incorporates comments from the Feb. 26–27 workshop and subsequent contributions. Special thanks to Steve Vegdahl for debugging the previous version of the syntax by writing a parser for it.
  - **Lexical syntax:** Clarified the assumption that the scanner always takes the longest token in case of ambiguity; made minor changes to number syntax, including removing alternate-radix floats and scientific notation for integers; explicitly reserved braces and backquote for future extensions; changed the role of `~` in binary selectors.
  - **Other syntax:** Removed dynamic array creation syntax and multiple expressions within parentheses; added `#'...'` to denote a symbol containing arbitrary characters; restored classes with both named and indexed instance variables, and changed the syntax of class definitions; specified primitives by string and number.
  - **Semantics:** Removed all messages with fixed meanings; clarified the position on redeclaration of names.
- **[4] 25 February 1987:** Made control messages have full message semantics; made minor changes to number syntax; allowed multiple expressions in parentheses and clarified the role of `.`; provided for extended and contracted character sets; specified primitives by class and selector. This is the version presented at the Feb. 26–27 implementors’ workshop.
- **[3] 2 January 1987:** Moved Blue Book syntax to an appendix; reorganized the summary of changes; removed hook for declarations; removed declarations and multiple expressions within parentheses; made control messages less special; allowed both numbers and strings for primitives; added section on file format; made minor changes reflecting comments from internal review. This is the version sent to Eliot Moss for distribution to the participants in the Dec. 11–12 workshop.
- **[2] 22 December 1986:** Added summary comparison with the Blue Book and class definition syntax.
- **[1] 20 December 1986:** First version; no syntax for class definitions yet.

Syntax equations are given in BNF extended with these constructs:

```text
[x]  optional x
x*   zero or more occurrences of x
x+   one or more occurrences of x
```

## 1. Summary of Changes from the Blue Book

This is a summary only; consult the following sections for details.

### Clarifications

- Some minor errors in classifying characters have been corrected.
- The syntactic role of `super` has been clarified.
- The order of evaluation of receiver and arguments has been defined as left-to-right.
- The syntax of primitives has been made explicit.
- The syntax for defining classes has been made explicit.

### Deletions

- Alternate-radix floating-point constants and scientific notation for integers are no longer provided.
- Classes may be defined with instances containing named instance variables and/or indexed object references, or indexed 8-bit bytes only; they may no longer contain “words” (16-bit bytes).

### Incompatibilities

- Embedded doubled quotes may not appear within comments. This does not change which programs are accepted, only how they are parsed.
- The syntax of literal arrays has changed to make the individual elements look exactly like free-standing literals.
- Block arguments and temporaries are properly scoped both lexically and dynamically; they are not stored in the home context. Many existing programs will not execute properly, but can be changed to work under both the current and proposed rules.
- The primitive, if any, comes before the method temporaries, not after.
- Standard primitives are identified, if necessary, by a class name and a selector; non-standard primitives are identified by a string.

### Additions

- Extensions to the character set are provided for, and the possibility of a reduced character set is acknowledged.
- Lower-case letters are allowed within alternate-radix integers.
- Symbol literals may contain arbitrary characters; the new syntax for this is `#` followed by a string.
- Literal arrays may contain `nil`, `true`, and `false` in addition to other literals.
- The sequence `:=` now also means assignment, as an alternative to `_` (left-arrow).
- Blocks may have temporaries.
- Control messages (`ifTrue:`, etc.) must not be forced to take explicit blocks as arguments. All messages, without exception, behave the same semantically and may be redefined by the user.
- Standard primitives may have a string description as well as a number.

## 2. Proposed 1987 standard

The following syntax is organized in the same way as the Blue Book syntax presented in the Appendix, with one major difference: it is specifically designed to be recognized by a recursive-descent parser with no backup and only a one-token buffer (such as the current Smalltalk-80 parser). Differences from the Blue Book are noted with `**`. Deleted constructs are marked with `--`; new ones with `++`.

The syntax presented in the Blue Book does not indicate where separators (whitespace or comments) are allowed. In the sections entitled **Lexical Primitives** below, separators are not allowed between constructs; in the other sections, separators may appear between any two constructs (terminal or non-terminal symbols or groupings).

### Character set

The 1987 Smalltalk standard syntax is based on the ASCII character set. Although this is not mentioned explicitly in the equations below, all non-printing ASCII characters (codes 0–31 and 127) are treated as whitespace characters; all other characters are mentioned explicitly in the following section on lexical primitives.

Other character sets may include otherwise unspecified characters of three kinds: whitespace, alphabetic, and graphic. Whitespace characters are ignored everywhere except in character and string constants; alphabetic characters are lexically equivalent to letters (they may start and appear within identifiers); graphic characters are allowed in character and string literals and are illegal elsewhere. In ASCII, all non-graphic characters with codes below 128 (codes 0–31 and 127) are whitespace; only letters, upper- and lower-case, are alphabetic. An implementation that goes beyond ASCII may designate additional characters as whitespace, alphabetic, and graphic in an implementation-dependent way. No particular mechanism for doing this is defined.

The ISO character set differs from ASCII and redefines certain ASCII characters to have different significance, particularly the square brackets and braces. Alternative representations for these language elements are not specified at this time; this must be addressed in the final version of the standard.

### Lexical Primitives

The lexical syntax is formally ambiguous: for example, `abc:` can be parsed either as an identifier followed by a non-quote character, or as a keyword. The ambiguity is resolved in all cases in favor of the longest token that can be formed starting at a given point in the source text. Thus `abc:` is always considered a keyword if `a` begins the token. The definition of token is supplied only for exposition.

```text
token = number | identifier | special-character | keyword |
        block-argument | assignment-operator | binary-selector |
        character-constant | string

digit = '0' | ... | '9'
digits = digit+
big-digits = (digit | letter)+  "as appropriate for radix"

number = (digits ['r' big-digits] | fraction-and-exponent) |
         fraction-and-exponent
fraction-and-exponent = '.' digits [('e' | 'E') ['-'] digits]

letter = 'A' | ... | 'Z' | 'a' | ... | 'z'
identifier = letter (letter | digit)*

special-character = '+' | '/' | '\\' | '*' | '~' | '<' | '>' |
                    '=' | '@' | '%' | '|' | '&' | '?' | '!' | ','

non-quote-character = digit | letter | special-character |
                      whitespace-character | '[' | ']' | '{' | '}' |
                      '(' | ')' | '_' | '^' | ';' | ':' | '$' | '#'

block-argument = ':' identifier
assignment-operator = '_' | ':='
keyword = identifier ':'

binary-selector = ('-' | special-character)
                  ('~' special-character | special-character)*

character-constant = '$' (non-quote-character | '"' | "'")
symbol = identifier | binary-selector | keyword+
string = '"' (non-quote-character | '"' '"' | "'")* '"'
comment = '"' (non-quote-character | "'")* '"'
separators = (non-printing-character | comment)*
```

The syntax for numbers in the Blue Book, and its interpretation by the current Smalltalk compiler, leads to some unexpected results:

| Input | Result of `doIt` |
|---|---|
| `1e31000` | `1e31000` (`e` for exponent works with both integers and floats) |
| `1.0e3` | `1000.0` |
| `16r1e3` | `4096` (lower-case `e` still means exponent) |
| `16r10.0` | `16.0` (alternate-radix floats work) |
| `16r1E3` | `483` (upper-case `E` means hex digit) |
| `.5` | `5` (initial `.` is interpreted as statement separator) |

Under the new syntax, these examples would have the following interpretations:

| Input | Result |
|---|---|
| `1e3` | Illegal: `e` is reserved for floats and requires a `.` |
| `1.0e3` | `1000.0` |
| `16r1e3` | `483` (no upper/lower-case distinction) |
| `16r10.0` | Illegal: no alternate-radix floats |
| `16r1E3` | `483` |
| `.5` | `0.5` (initial `.` means float) |

As indicated in the syntax above, `e` and `E` are reserved for floats, and a `.` is required for floats as well.

Errors in the Blue Book have been corrected, and the two-character limit on the length of binary selectors has been removed. The character `~` still plays a special role: it cannot be the last character in a binary selector (unless it is the only character), because otherwise expressions like `x+-5` would be ambiguous (`x + -5` or `x +- 5`).

`:=` is allowed for assignment, since the ASCII standard has underscore in place of left-arrow (`_`). This introduces a minor syntactic anomaly, explained in the section on expressions below. Braces and backquote are deliberately reserved for future extensions to the language.

### Atomic Terms

```text
named-constant = 'nil' | 'true' | 'false'
symbol-constant = '#' (symbol | string)
array-constant = '#' '(' literal* ')'
literal = ['-'] number | named-constant | symbol-constant |
          character-constant | string | array-constant
variable-name = identifier  "other than a named-constant,
                             pseudo-variable-name, or 'super'"
```

The new syntax for array constants is simpler to explain than the present Smalltalk-80 syntax, but different and a little more verbose. `nil`, `true`, and `false` are explicitly recognized as constants. A new syntax, `#` followed by a string, is added for writing symbol constants containing arbitrary characters.

### Expressions and Statements

```text
primary = variable-name | pseudo-variable-name | literal |
          block-constructor | subexpression
pseudo-variable-name = 'self' | 'thisContext'

unary-message = unary-selector
unary-selector = identifier
binary-message = binary-selector primary unary-message*
keyword-message = (keyword primary unary-message* binary-message*)+

cascaded-messages = (';' (unary-message | binary-message |
                           keyword-message))*
messages = unary-message+ binary-message* [keyword-message] |
           binary-message+ [keyword-message] |
           keyword-message
rest-of-expression = [messages cascaded-messages]

expression = variable-name
               (assignment-operator expression | rest-of-expression) |
             keyword '=' expression  "see below" |
             primary rest-of-expression |
             'super' messages cascaded-messages

expression-list = expression ('.' expression)* ['.']
temporaries = '|' temporary-list '|' | '||'
subexpression = '(' expression ')'
temporary-list = declared-variable-name*
declared-variable-name = variable-name

statements = ['^' expression ['.'] | expression ['.' statements]]
block-constructor = '[' block-declarations statements ']'
block-declarations = temporaries |
                     block-argument+
                       ('|' [temporaries] | '||' temporary-list '|' | '||')
```

To keep lexical analysis and parsing separate while still allowing constructs like `x:=3`, the alternative `keyword '=' expression` is introduced for assignment. It should be read as though it were `variable-name ':=' assignment`.

The syntax of messages has been rewritten to make it more amenable to parsing and hopefully easier to read. Pseudo-variables and the requirement that `super` be followed by a message are explicitly recognized. Cascaded messages to `super` are allowed, though some existing Smalltalk-80 compilers do not allow this. The syntax for `.` allows it to be considered either a separator, which may be followed by an empty element at the end of a list, or a terminator, which may be optionally elided at the end of a list.

Blocks may declare local temporary variables. This is the one significant addition to the Blue Book syntax and semantics. A block with both arguments and temporaries requires a double `|` between the arguments and the temporaries. This is needed because the scanner accumulates consecutive `|` characters into a single binary selector. For example:

```smalltalk
[:x || t | y & z]
```

Without the double `|`, this could mean either “argument `x`, temporary `t`, return `y & z`” or “argument `x`, return `t | y & z`.” Requiring the double `|` gives the latter parsing of the separator and unambiguously distinguishes the temporary declaration.

Other considered solutions were rejected: disallowing `|` as a binary operator (it is the standard representation for “or”); removing the `|` after the argument list (it reads worse); or making the distinction depend on whether the names after the first `|` are already defined (that would introduce a subtle syntactic dependency on far-away properties).

There is a semantic restriction that cannot be expressed in context-free syntax: valid variable names must be declared within an enclosing scope. These include global, pool, class, or instance variables of the class where the method is defined or a superclass; arguments or temporaries of the enclosing method or block; and temporaries of an enclosing subexpression. A full treatment of name scopes is beyond this proposal. The following rules are specified:

- A local name (method or block argument or temporary) must not conflict with a non-local name accessible in the same scope (global, pool, class, or instance variable).
- A local name may conflict with another local name accessible in the same scope; the inner declaration takes precedence.

Compiler implementors are encouraged to warn when code redeclares a local name. This helps catch the current Smalltalk practice in which a name used as a block argument is also declared in the method temporaries, for example:

```smalltalk
temp |
...
[:temp | ...]
```

### Methods

```text
message-pattern = unary-selector |
                  binary-selector declared-variable-name |
                  (keyword declared-variable-name)+

primitive = '<' 'primitive:' [primitive-identification] '>'
primitive-identification = symbol symbol | string
method = message-pattern [primitive] [temporaries] statements
```

Primitives are identified either by a class and selector, or by a string. The former identify standard primitives; the interpretation of the latter is not defined. If the primitive identification is missing, the class and selector name of the method containing it are used. In general, primitives are not expected to be standardized. What is proposed for standardization is the behavior of certain messages in certain classes, independent of whether they are implemented primitively.

The implementation (compiler) is responsible for checking that primitives are attached only to methods and classes for which they are legal; this correspondence is part of the language definition. An implementation may include polymorphic primitives that can validly be attached to a variety of classes and methods.

The primitive specification precedes, rather than follows, the method temporaries. This seems more intuitive, since the primitive is executed before the temporaries are bound.

### Classes

The Smalltalk-80 system takes quite a different approach to creating and editing classes versus methods: methods are defined by a textual syntax and a message interface for compiling it, while operations on classes are defined as explicit messages taking various kinds of string arguments. The Blue Book does not introduce syntax per se for classes; it assumes they are created using the messages described in Chapter 16. However, class definition is properly part of the language, just like method definition, so this proposal specifies the fundamental message for creating classes. The semantics of modifying existing classes, and what happens to existing instances when a class is modified, are beyond its scope.

Creating a class from a specification is considered just like creating any other object, and unrelated to installing it under a name in any dictionary. The creation of a compiled method is likewise separate from its installation in a class.

A class is created by the following message:

```smalltalk
Behavior
    newSuperclass: "Behavior | nil"
    instanceVariables: "Array of: Symbol"
    classVariables: "Array of: Symbol"
    poolDictionaries: "Array of: Symbol"
```

Giving a class a name, installing it in a dictionary, or classifying it in an organization are outside the scope of the language definition. An interactive interface is presumed to let users define classes without writing out the message above, and to handle naming and organization if relevant.

The superclass specified for a class may be another `Behavior` or `nil`. The latter is required for class `Object` and allows creating classes that are not subclasses of `Object`. This is fraught with peril: for example, if such a class does not define `printOn:`, the Smalltalk system is likely to enter a recursion loop the first time one tries to inspect an instance of the class.

The syntax of the string supplied to describe the instance variables is:

```text
inst-var-names = declared-variable-name* [indexed-refs] |
                 indexed-bytes
indexed-refs = '*' 'Object'
indexed-bytes = '*' 'Byte'
```

Thus a class may contain named instance variables holding object references, indexed instance variables holding object references (e.g. `Array`), both (e.g. `OrderedCollection`), or information that is not object references (e.g. `ByteArray`). An implementation is anticipated to provide various kinds of primitive access to bit-type objects, e.g. by 8-, 16-, or 32-bit bytes, or perhaps arbitrary bit sequences. Such objects are called “byte” rather than “bit” objects because the proposal does not require implementations to quantize their space in units smaller than 8 bits.

### File syntax

The form in which Smalltalk programs are stored on external files is defined in Chapter 3 (pp. 29–37) of the Green Book, not the Blue Book. This proposal standardizes enough of the external format that program files can be parsed even by systems unable to interpret all their contents. In the syntax equations below, separators are **not** implicitly allowed between elements; the equations must be taken exactly as they appear.

```text
marker = '!'
non-marker = any character except the marker
separators = non-printing-character*
chunk = (non-marker | marker marker)+ marker
special-read-section = marker chunk (separators chunk)* separators marker
program-file = (separators (special-read-section | chunk))* separators
```

Information appears on a program file in “chunks” terminated by a marker, with embedded markers doubled. A chunk not preceded by a marker is simply an expression to be evaluated. A chunk preceded by a marker indicates the start of special syntax: the expression is evaluated to produce some kind of reader or parser object, which is then sent `scanFrom:` with the file stream itself as the argument. The reader is expected to read and process chunks from the file until it encounters an empty chunk. In other words, the following might represent the algorithm for reading in a program file:

```smalltalk
[self skipSeparators.
 self atEnd]
    whileFalse:
        [(self peekFor: $!)
            ifTrue: [(Object evaluate: self nextChunk) scanFrom: self]
            ifFalse: [(Object evaluate: self nextChunk)]]
```

The purpose of the special-read section is primarily to allow classes to read in method definitions without having them copied two extra times (once for chunk parsing and once for parsing as a string literal to be passed as an argument). The current Smalltalk-80 system copies the definition one extra time, since it reads it in as a chunk before parsing; this can clearly be avoided if desired.

At a minimum, a file parser must be able to identify method definitions. This is proposed by defining the message `<Behavior> methodReader` to return an object whose `scanFrom:` method reads and defines methods for the receiver. Additional messages may be sent to this object without compromising its function, for example:

```smalltalk
!aBehavior methodReader category: 'something'!
```

By this convention, an implementation can define additional properties for methods being read without compromising general parsability of source files.

## Semantics

### Order of Evaluation

Expressions in an expression list are evaluated left-to-right. Message sends in a cascade are evaluated left-to-right. The receiver of a message is evaluated before the arguments; arguments are evaluated left-to-right.

### Standard Classes

The following classes are conceptually required to support the language defined above; they are the classes of literal objects:

- `Integer`
- `Float`
- `Symbol`
- `String`
- `Array`
- `Character`
- `Block`
- `True`, `False`, `Nil`
- `Behavior` (for classes)

These classes need not have these specific names, nor must their functionality be divided up exactly this way. For example, integers might be implemented by separate `SmallInteger` and `LargeInteger` classes, or `True` and `False` might be instances of a single class `Boolean`. These names and this division of functionality are used in describing the standard messages in the next section.

### Standard Messages

The language as described has no messages with fixed meanings. This is regarded as a unique strength of Smalltalk (and related languages such as Hewitt’s Actor languages); experience has indicated the utility of this concept for such things as transparent message forwarders. On the other hand, any useful language must provide basic functions such as arithmetic and control structures, and any commercially viable language must implement some of these functions very efficiently. A small set of messages is therefore defined that all implementations of Smalltalk-80 are expected to provide. The pragmatics of these and other messages are discussed in the next section.

The messages below are the proposed absolute minimum for language support. The marks in the left margin refer to the section on Pragmatics and should be ignored here.

**Arithmetic**

```text
P (Integer) + - * / < > <= >= = ~= (Integer, Float)
P (Integer) // \\ (Integer)
P (Integer) / (Float)
P (Float) + - * / < > <= >= = ~= (Integer, Float)
```

**Control**

```text
P (Block) value
P (Block) value: (Object)
F (Block) whileTrue: (Block)
F (Block) whileFalse: (Block)
F (Block) whileTrue
F (Block) whileFalse
F (Block) repeat
P (Integer) to: (Integer) do: (Block)
(Integer) timesRepeat: (Block)
F (True, False) ifTrue: (Block)
F (True, False) ifTrue: (Block) ifFalse: (Block)
F (True, False) ifFalse: (Block)
F (True, False) ifFalse: (Block) ifTrue: (Block)
F (True, False) and: (Block)
F (True, False) or: (Block)
```

**Miscellaneous**

```text
F (Object) == (Object)
```

### Contexts

Contexts are the one area in which several minor changes in Smalltalk semantics are proposed. All are backward-compatible given some minor changes in the Virtual Image.

The first change concerns the scope and lifetime of block arguments (and temporaries, which are new). In the current Smalltalk-80 definition, block arguments are stored in the home context. This prevents blocks from being used recursively or by more than one Process, and leads to anomalous error messages if a process is interrupted (“Block already active”). In the proposed definition, blocks are closures in the sense of Scheme or other modern lexically scoped Lisps: executing the `[]` construct creates a `BlockClosure`, which encapsulates only the current (home) context and the code; invoking a `BlockClosure` creates a `BlockContext`. An implementation may optimize this as long as semantics are maintained. For example, a block that refers to no variables in outer scopes and does not do a return may not need to hold a reference to the outer scope. As a result, the debugger may have less information available; this is explicitly allowed.

The second change concerns `^`. When control returns from a method but the context being returned to is anomalous (e.g. it has already been returned from), the current system sends `cannotReturn: theValue` to the context being returned from. The proposal changes this so the system sends `resumeWith: theValue` to the object being returned to (presumably, but not necessarily, a context). For convenience in implementing non-standard control structures, this message should be defined primitively in class `Context`.

The third change also affects `^`. Currently, `^` from within a block invokes a complex algorithm that users have no control over. The proposal defines `^` within a block as sending the message `thisContext remoteReturn: theValue`. Normally this message will be defined in class `BlockContext` as a primitive that carries out the current built-in algorithm, though future evolution of the system to incorporate exception handling with unwind-protection might affect the definition. Implementations may disallow returns to contexts whose sender chain terminates somewhere other than the root of the current process; such situations should be handled with explicit use of `resumeWith:`.

As indicated below, compilers may be able to avoid creating blocks in certain circumstances, such as with the standard conditional message `ifTrue:ifFalse:`. As a consequence, `thisContext` would then be an outer context rather than the actual (textual) current context. The proposal requires absolutely faithful implementation, which may require constructing an actual context for a conditional message if a `thisContext` appears within one of the alternatives.

### Pragmatics

Absolutely uniform message semantics are required, but more efficient implementation of messages whose meaning is very unlikely to change must be allowed. Certain messages, while retaining the same semantics as all others, are therefore allowed to have substantially different pragmatics:

- Certain messages, if sent to receivers whose classes are not in a specified set, may execute much slower.
- Certain messages, if defined in new classes, or redefined or undefined in existing classes, may suffer a substantial performance penalty for some or all receiver classes. In exchange, under normal circumstances these messages execute substantially faster than others.

In the list of standard messages above, messages marked `P` have the first pragmatic property (`P` indicates that the set of classes for that message is partially fixed); messages marked `F` have both properties (`F` indicates that the set of classes for that message must stay fully fixed to avoid losing performance).

“Changing a definition” means that the new definition is not operationally equivalent to the old one. A sufficient, but not necessary, condition for testing this is that the new definition compiles into the same object code as the old one. Implementors are encouraged to use such a test so that, for example, changing variable names or comments is not considered changing the definition. The pragmatic consequences of not compiling `ifTrue:ifFalse:` inline are so severe that the user should be given an opportunity to confirm that this was actually intended. This is a user-interface question, not a matter of language definition. No existing Smalltalk-80 compiler known to the authors handles this possibility properly.

The `notBoolean` exception, which results from non-Boolean conditions in the present Smalltalk-80 definition, does not conform to the proposed standard. If an implementation uses a mechanism like a `notBoolean` message internally, it must automatically convert this to a correct `ifTrue:ifFalse:` (or equivalent) message with two appropriate blocks as arguments, without user intervention.

Current Smalltalk-80 compilers that adopt the Blue Book’s concept of “special arithmetic selectors” do not properly handle redefinition of these messages; this does not conform to the proposed standard.

Nothing in this standard precludes an implementation from adding or removing messages or receiver classes from the lists above. Such differences may be built into the implementation or under user control with a sufficiently sophisticated compiler. Since enhancements are required to leave semantics unchanged, they are not specified here.

### Static checking

Supplying receivers or arguments of the wrong class to the messages above will almost certainly result in a runtime error. Compilers may choose to issue warnings if they believe the user has written a program likely to result in an error. For example, a programmer unused to Smalltalk syntax might write:

```smalltalk
a < b ifTrue: trueStuff ifFalse: falseStuff
```

rather than:

```smalltalk
a < b ifTrue: [trueStuff] ifFalse: [falseStuff]
```

A compiler might plausibly ask for user confirmation if an argument to `ifTrue:ifFalse:` is not an explicitly written block. However, the proposed standard requires all compilers to be willing to compile programs containing questionable constructs of this kind. Most current Smalltalk-80 compilers do not do this.

### Contexts

All existing compilers for Smalltalk-80 treat some or all control messages specially by compiling them in a way that avoids creating Context objects during execution. Since the language standard does not specify anything about creating contexts, a compiler is free to compile **any** message in a way that avoids creating Contexts, provided the message’s semantics are unaffected. Since Contexts are visible to the programmer at the meta-level, programs at the meta-level must be prepared for the possibility that a given message send at the source level may not create a Context at the object level.

## Appendix: The Blue Book

The following syntax is the one that appears on the endpaper of the Blue Book, slightly rearranged. A few notes on errors and omissions are interspersed. This material is reprinted by permission of Xerox Corporation.

### Lexical Primitives

```text
digit = '0' | ... | '9'
digits = digit+
number = [digits 'r'] ['-'] digits [',' digits] ['e' ['-'] digits]
letter = 'A' | ... | 'Z' | 'a' | ... | 'z'
identifier = letter (letter | digit)*
special-character = '+' | '/' | '\\' | '*' | '~' | '<' | '>' |
                    '=' | '@' | '%' | '|' | '&' | '?' | '!' | ','
character = digit | letter | special-character |
            '[' | ']' | '{' | '}' | '(' | ')' | '_' | '^' | ';' | ':' | '$' | '#'
keyword = identifier ':'
unary-selector = identifier
binary-selector = '-' | special-character [special-character]
character-constant = '$' (character | '"' | "'")
symbol = identifier | binary-selector | keyword+
string = '"' (character | '"' '"' | "'")* '"'
comment = '"' (character | '"' | "'")* '"'
separators = (non-printing-character | comment)*
```

The syntax for `special-character` and `character` has several errors. `!` appears in both, but should appear only in `special-character`; `,` appears in `character`, but should appear in `special-character`; `.` appears in neither, but should appear in `character`; `-` and backquote appear in neither, but should appear in `special-character`.

There does not appear to be a good reason for limiting the length of binary selectors to two characters. `-` is apparently singled out because of its special role in indicating negative numbers.

### Atomic Terms

```text
symbol-constant = '#' symbol
array = '(' (number | symbol | string | character-constant | array)* ')'
array-constant = '#' array
literal = number | symbol-constant | character-constant | string | array-constant
variable-name = identifier
```

### Expressions and Statements

```text
primary = variable-name | literal | block | '(' expression ')'
unary-object-description = primary | unary-expression
binary-object-description = unary-object-description | binary-expression
unary-expression = unary-object-description unary-selector
binary-expression = binary-object-description binary-selector unary-object-description
keyword-expression = binary-object-description ':' (keyword binary-object-description)+
message-expression = unary-expression | binary-expression | keyword-expression
cascaded-message-expression = message-expression
                             (';' (unary-selector |
                                   binary-selector unary-object-description |
                                   (keyword binary-object-description)+))+
expression = (variable-name '_')*
             (primary | message-expression | cascaded-message-expression)
statements = ['^' expression ['.'] | expression ['.' statements]]
block = '[' [ (':' variable-name)+ '|' statements | statements ] ']'
```

### Methods

```text
temporaries = '|' variable-name* '|'
message-pattern = unary-selector | binary-selector variable-name |
                  (keyword variable-name)+
method = message-pattern [temporaries [statements] | statements]
```

The Blue Book omits the syntax for indicating primitive methods. This may be deliberate.
