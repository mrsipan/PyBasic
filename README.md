# A BASIC Interpreter - Program like it's 1979!

## Introduction

*This project is still a work in progress and under active development*

A simple interactive BASIC interpreter written in Python 3. It is based heavily on material in the excellent book *Writing Interpreters
and Compilers for the Raspberry Pi Using Python* by Anthony J. Dos Reis. However, I have had to adapt the Python interpreter presented
in the book, both to work with the BASIC programming language and to produce an interactive command line interface. The interpreter
therefore adopts the key techniques for interpreter and compiler writing, the use of a lexical analysis stage followed by a recursive descent parser
which implements the context free grammar representing the target programming language.

The interpreter is a homage to the home computers of the early 1980s, and when executed, presents an interactive prompt ('>')
typical of such a home computer. Commands to run, list, save and load BASIC programs can be entered at the prompt as well as
program statements themselves.

The BASIC dialect that has been implemented is slightly simplified, and naturally avoids machine specific instructions, such as those
concerned with sound and graphics for example. It allows a limited range of arithmetic expressions composed of multiplication, division,
addition and subtraction (including the use of parentheses to change precedence), thus:

```
> 10 PRINT 2 * 3
> 20 PRINT 20 / 10
> 30 PRINT 10 + 10
> 40 PRINT 10 - 10
> RUN
6
2
20
0
>
```

Additional numerical operations may be performed using numeric functions (see below).

There is reasonably comprehensive error checking. Syntax errors will be picked up and reported on by the
lexical analyser as statements are entered. Runtime errors will highlight the cause and the line number of
the offending statement.

The interpreter can be invoked as follows:

```
$ python interpreter.py
```

## Commands

Programs may be listed using the **LIST** command:

```
> LIST
10 LET I = 10
20 PRINT I
>
```

A program is executed using the **RUN** command:

```
> RUN
10
>
```

A program may be saved to disk using the **SAVE** command. Not that the full path must be specified within double quotes:

```
> SAVE "C:\path\to\my\file"
Program written to file
>
```

Saving is achieved by pickling the Python object that represents the BASIC program, i.e. the saved file is *not* a textual copy of
the program statements.

The program may be re-loaded (i.e. unpickled) from disk using the **LOAD** command, again specifying the full path using double quotes:

```
> LOAD "C:\path\to\my\file"
Program read from file
>
```

Since loading is performed by unpickling the program object from a file, only BASIC programs *previously saved
by the interpreter* may be loaded.

Individual program statements may be deleted by entering their line number only:

```
> 10 PRINT "Hello"
> 20 PRINT "Goodbye"
> LIST
10 PRINT "Hello"
20 PRINT "Goodbye"
> 10
> LIST
20 PRINT "Goodbye"
>
```

The program may be erased entirely from memory using the **NEW** command:

```
> 10 LET I = 10
> LIST
10 LET I = 10
> NEW
> LIST
>
```

Finally, it is possible to terminate the interpreter by issuing the **EXIT** command:

```
> EXIT
c:\
```

On occasion, it might be necessary to force termination of a program and return to the
interpreter, for example, because it is caught in an infinite loop. This can be achieved by
using Ctrl-C to force the program to stop:

```
> 10 PRINT "Hello"
> 20 GOTO 10
> RUN
"Hello"
"Hello"
"Hello"
...
...
<Ctrl-C>
Program terminated
> LIST
10 PRINT "Hello"
20 GOTO 10
>
```

## Programming language constructs

### Statement structure

As per usual in old school BASIC, all program statements must be prefixed with a line number which indicates the order in which the
statements may be executed. There is no renumber command to allow all line numbers to be modified. A statement may be modified or
replaced by re-entering a statement with the same line number:

```
> 10 LET I = 10
> LIST
10 LET I = 10
> 10 LET I = 200
> LIST
10 LET I = 200
>
```

### Variables

Variable types follow the typical BASIC convention. *Simple* variables may contain either be strings
or numbers (the latter may be integers or floating point numbers). *Array* variables may contain arrays
of either strings or numbers.

Note that all keywords and variable names are case insensitive (and will be converted to upper case internally by the lexical analyser).
String literals will retain their case however. There is no inherent limit on the length of variable names or string literals,
this will be dictated by the limitations of Python. The range of numeric values is also dependent upon the underlying
Python implementation.

Numeric variables have no suffix, whereas string variables are always suffixed by '$'. Note that 'I' and 'I$' are
considered to be separate variables. Note that string literals must always be enclosed within double quotes (not single quotes).
Using no quotes will result in a syntax error.

Array variables - TBD

### Comments

The **REM** statement is used to indicate a comment, and occupies an entire statement. It has no effect on execution:

```
> 10 REM THIS IS A COMMENT
```

Note that comments will be automatically normalised to upper case.

### Stopping a program

The **STOP** statement may be used to cease program execution:

```
> 10 PRINT "one"
> 20 STOP
> 30 PRINT "two"
> RUN
one
>
```

A program will automatically cease execution when it reaches the final statement, so a **STOP** may not be necessary. However
a **STOP** *will* be required if subroutines have been defined at the end of the program, otherwise execution will continue
through to those subroutines without a corresponding subroutine call. This will cause an error when the **RETURN**
statement is processed and the interpreter attempts to return control back to the caller.

### Assignment

Assignment may be made to number variables (which can contain either integers or floating point numbers) and string variables
(string variables are distinguished by their dollar suffix). The interpreter will enforce this division between the two types:

```
> 10 LET I = 10
> 20 LET I$ = "Hello"
```

The **LET** keyword is also optional:

```
> 10 I = 10
```

### Printing to standard output

The **PRINT** statement is used to print to the screen:

```
> 10 PRINT 2 * 4
> RUN
8
> 10 PRINT "Hello"
> RUN
Hello
>
```

Multiple items may be printed by providing a comma separated list. The items in the list will be printed immediately after one
another, so spaces must be inserted if these are required:

```
> 10 PRINT 345, " hello ", 678
> RUN
345 hello 678
>
```

A blank line may be printed by using the **PRINT** statement without arguments:

```
> 10 PRINT "Here is a blank line:"
> 20 PRINT
> 30 PRINT "There it was"
> RUN
Here is a blank line:

There it was
>
```

### Unconditional branching

Like it or loath it, the **GOTO** statement is an integral part of BASIC, and is used to transfer control to the statement with the specified line number:

```
> 10 PRINT "Hello"
> 20 GOTO 10
> RUN
Hello
Hello
Hello
...
```

### Subroutine calls

The **GOSUB** statement is used to generate a subroutine call. Control is passed back to the program at the
next statement after the call by a **RETURN** statement at the end of the subroutine:

```
> 10 GOSUB 100
> 20 PRINT "This happens after the subroutine"
> 30 STOP
> 100 REM HERE IS THE SUBROUTINE
> 110 PRINT "This happens in the subroutine"
> 120 RETURN
> RUN
This happens in the subroutine
This happens after the subroutine
>
```

Note that without use of the **STOP** statement, execution will run past the last statement
of the main program (line 30) and will re-execute the subroutine again (at line 100).

Subroutines may be nested, that is, a subroutine call may be made within another subroutine.

A subroutine may also be called using the **ON-GOSUB** statement (see Conditional branching
below).

### Loops

Bounded loops are achieved through the use of **FOR-NEXT** statements. The loop is controlled by a numeric
loop variable that is incremented or decremented from a start value to an end value. The loop terminates when
the loop variable reaches the end value. The loop variable must also be specified in the **NEXT**
statement at the end of the loop.

```
> 10 FOR I = 1 TO 3
> 20 PRINT "hello"
> 30 NEXT I
> RUN
hello
hello
hello
>
```

Loops may be nested within one another.

The **STEP** statement allows the loop variable to be incremented or decremented by
a specified amount. For example, to count down from 5 in steps of -1:

```
> 10 FOR I = 5 TO 1 STEP -1
> 20 PRINT I
> 30 NEXT I
> RUN
5
4
3
2
1
>
```

Note that the start value, end value and step value need not be integers, but can be floating
point numbers as well. If the loop variable was previously assigned in the program, its value will
be replaced by the start value, it will not be evaluated.

### Conditional branching

Conditional branches are implemented using the **IF-THEN-ELSE** statement. The expression is evaluated and the appropriate jump
made depending upon the result of the evaluation.

```
> 10 REM PRINT THE GREATEST NUMBER
> 20 LET I = 10
> 30 LET J = 20
> 40 IF I > J THEN 50 ELSE 70
> 50 PRINT I
> 60 GOTO 80
> 70 PRINT J
> 80 REM FINISHED
> RUN
20
>
```

Note that the **ELSE** clause is optional and may be omitted. In this case, the **THEN** branch is taken if the
expression evaluates to true, otherwise the following statement is executed.

It is also possible to call a subroutine depending upon the result of a relational expression
using the **ON-GOSUB** statement. If the expression evaluates to true, then the subroutine is
called, otherwise execution continues to the next statement without making the call:

```
> 10 LET I = 10
> 20 LET J = 5
> 30 ON I > J GOSUB 100
> 40 STOP
> 100 REM THE SUBROUTINE
> 110 PRINT "I is greater thn J"
> 120 RETURN
> RUN
I is greater than J
>
```

Allowable relational operators are:

* '=' (equal, note that in BASIC the same operator is used for assignment)
* '<' (less than)
* '>' (greater than)
* '<=' (less than or equal)
* '>=' (greater than or equal)
* '<>' (not equal)

### User input

The **INPUT** statement is used to solicit input from the user:

```
> 10 INPUT A
> 20 PRINT A
> RUN
? 22
22
>
```

The default input prompt of '? ' may be changed by inserting a prompt string, which must be terminated
by a colon, thus:

```
> 10 INPUT "Input a number - ": A
> 20 PRINT A
> RUN
Input a number - 22
22
>
```

Multiple items may be input by supplying a comma separated list. Input variables will be assigned
to as many input values as supplied at run time. If there are more input values supplied than input
variables, the excess input values will be ignored. Conversely, if not enough input values are
supplied, then the excess input variables will not be initialised (and will trigger an error if
an attempt is made to evaluate those variables later in the program).

Further, numeric input values must be valid numbers (integers or floating point), and must
be unquoted. String input values must be surrounded by double quotes:

```
> 10 INPUT "Num, Str, Num: ": A, B$, C
> 20 PRINT A, B$, C
> RUN
Num, Str, Num: 22, " hello ", 33
22 hello 33
>
```

A mismatch between the input value and input variable type will trigger an error.

### Numeric functions

Selected numeric functions are provided, and may be used in any numeric expression. For example,
the square root function, **SQR**, can be applied expressions consisting of both literals and variables:

```
> 10 LET I = 6
> 20 PRINT SQR(I - 2)
> RUN
2.0
>
```

Allowable numeric functions are:

* **ABS**(x) - Calculates the absolute value of *x*

* **POW**(x, y) - Calculates *x* to the power *y*

* **SQR**(x) - Calculates the square root of *x*

## Example programs

A number of example BASIC programs have been supplied in the repository:

* *REGRESSION.BAS* - A program to exercise the key programming language constructs
in such a way as to allow verification that the interpreter is functioning correctly.

* *FACTORIAL.BAS* - A simple BASIC program to take a number, *N*, as input from the user and
calculate the corresponding factorial *N!*.

## Grammar

**ABS**(*numerical-expression*) - Calculates the absolute value of *x*

**EXIT** - Exits the interpreter

**FOR** *loop-variable* = *start-value* **TO** *end-value* [**STEP** *increment*] - Bounded loop

**GOSUB** *line-number* - Subroutine call

**GOTO** *line-number* - Unconditional branch

**IF** *relational-expression* **THEN** *line-number* [**ELSE** *line-number*] - Conditional branch

**INPUT** [*input-prompt*:] *variable-list* - Processes user input presented as a comma separated list

[**LET**] *variable* = *numeric-expression* | *string expression* - Assigns a value to a variable

**LIST** - Lists the program

**LOAD** *filename* - Loads a program from disk

**NEW** - Clears the program from memory

**NEXT** *loop-variable* - See **FOR** statement

**ON** *relational-expression* **GOSUB** *line-number* - Conditional subroutine call

**POW**(*base*, *exponent*) - Calculates the result of raising the base to the power of the exponent

**PRINT** *print-list* - Prints a comma separated list of literals or variables

**REM** *comment* - Internal program documentation

**RETURN** - Return from a subroutine

**RUN** - Runs the program

**SAVE** *filename* - Saves a program to disk

**SQR**(*numerical-expression*) - Calculates the square root of the expression

**STOP** - Terminates a program

## Architecture

The interpreter is implemented using the following Python classes:

* basictoken.py - This implements the tokens that are produced by the lexical analyser. The class mostly defines token categories
and provides a simple token pretty printing method.

* lexer.py - This class implements the lexical analyser. Lexical analysis is performed on one statement at a time, as each statement is
entered into the interpreter.

* basicparser.py - This class implements a parser for individual BASIC statements. This is somewhat inefficient in that statements,
for example those in a loop, must be re-parsed every time they are executed. However, such a model allows us to develop an
interactive interpreter where statements can be gradually added to the program between runs.
Since the parser is oriented to the processing of individual statements, it uses a
signalling mechanism (using FlowSignal objects) to its caller indicate when program level actions are required, such as recording the return address
following a subroutine jump. However, the parser does maintain a symbol table (implemented as a dictionary) in order to record
the value of variables as they are assigned.

* program.py - This class implements an actual basic program, which is represented as a dictionary. Dictionary keys are
statement line numbers and the corresponding value is the list of tokens that make up the statement with that line number.
Statements are executed by calling the parser to parse one statement at a time. This class
maintains a program counter, an indication of which line number should be executed next. The program counter is incremented to the next line
number in sequence, unless executed a statement has resulted in a branch. The parser indicates this by signalling to the program object that
calls it using a FlowSignal object.

* interpreter.py - This class provides the interface to the user. It allows the user to both input program statements and to execute
the resulting program. It also allows the user to run commands, for example to save and load programs, or to list them.

* flowsignal.py - Implements a FlowSignal object that allows the parser to signal a change in control flow. or example, as
the result of a jump defined in the statement just parsed (GOTO, conditional branch evaluation), a loop decision,
a subroutine call, or program termination. This paradigm of using the parser to simply parse individual statements, the Program
object to make control flow decisions and to track execution, and a signalling mechanism to allow the parser to signal
control flow changes to the Program object, is used consistently throughout the implementation.

## Open issues

* It is not possible to renumber a program. This would require considerable extra functionality.
* Negative values are printed with a space (e.g. '- 5') in program listings because of tokenization. This does not affect functionality.

