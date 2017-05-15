
js4m -- JavaScript for Macro Processing
---------------------------------------
js4m is a macro processor that uses JavaScript as its basis.  Text is
preprocessed using a limited number of directives, but with the full power
of JavaScript.  Text is preprocessed, converting it to JavaScript with
appropriate macro substitutions, and then the JavaScript is evaluated.  js4m
can be invoked from the command line or called from a JavaScript program.

The problem with many macro languages is that they don't use a well-known
language with a large number of libraries and utilities to support it.  This
adds unnecessary complexity and a learning curve that's hard to justify.
Those kinds of macro languages will always be very limited.

Node.js can be used to run js4m and the JavaScript code it generates.
Note that Node.js provides many asynchronous functions, but only synchronous
functions should be used here.


js4m Console Program (using Node.js)
------------------------------------
    node js4m.js -i input-file -o output-file args
    node js4m.js args <input-file >output-file
    
    Options:
        -i, --input-file input-file
            Specify an input file (instead of stdin)
        -j, --js
            Output JavaScript, but don't evaluate.
        -o, --output-file output-file
            Specify an output file (instead of stdout)
        -h, --help
            Get help.


Variables and Expressions
-------------------------

There are only a few types of macro substitutions performed.

| Macro     | Description                                                    |
|-----------|----------------------------------------------------------------|
| `$$`      | Is substituted with `$`.                                       |
| `$ ...`   | When `$` and a space begin the line, the rest is JavaScript.   |
| `$x`      | The value of the variable x is placed in the text.             |
| `$expr`   | Very simple expression with no operators.                      |
| `$(expr)` | The result of the JavaScript expression is placed in the text. |


### `$$`

Two dollar-signs in a row are replaced with a single dollar-sign (`$`)


### `$ javascript...`

When a dollar-sign ($) and a space character occur at the beginning of a line,
the rest of the line is used as JavaScript to execute.

    $ x = 5        // set x to 5
    $ if (x==y) {
    some text used when x = y
    $ }

    
### `$x`

The value of `x` replaces `$x` in the text.  If x is a number, the value of
argv[x] is substituted.  Either x is an integer or the symbol must be a valid
JavaScript variable name (that doesn't start with a dollar-sign).

    $   var x = 'Hi'
    $   var y = 'There'
    Just say $x $y


### `$expr`

A simple expression starts with a variable name and is followed by any number
of the following in succession:

   - `.x      ` A period followed by a variable name.
   - `(expr)  ` A parameter list/expressions in parenthesis (`()`).
   - `[expr]  ` An array index expression in square brackets (`[]`).

    $   var obj = { x:{a:1, b:2}, y:'hi' }
    $   function f() { return 'a + b = ' }
    $f() $obj.x.a + $obj.x.b = $(obj.x.a + obj.x.b)


### `$(expr)`

The result of the JavaScript expression `expr` is placed in the text.
The JavaScript expression must have a balanced number of parenthesis, `()`,
square brackets, `[]`, and curly braces `{}`.  Also, quoted string literals
must be terminated properly.  The JavaScript expression itself will
not be preprocessed.

    $   var x = 3
    $   var y = 2
    $x plus $y = $(x+y)


Conversion to JavaScript
------------------------
Each line of input text is preprocessed separately generating a single line
of JavaScript output.  This means that an error occurring at a particular line
in the generated file will correspond to the same line in the source file.


### `$ javascript...`

Generates a line containing the JavaScript `javascript...`

Other lines can contain a mix of normal text and macro expressions to be
replaced.  These are all appended to the `out` variable.

Normal text is converted to a string, and $$ is converted to a single
dollar-sign within such text.

    out += "text\n..."

### `$x` 

    out += x


### `$expr`

    out += expr
    
For example, the line...

    $f() $obj.x.a
    
...results in the generated JavaScript...

    out += f() + " " + obj.x.a + "\n"


### `$(expr)`

    out += expr

For example, the lines...

    $ var i = 1
    $ var j = 2
    i plus j = $i + $j = $(i+j)
    The first command line argument is $0

...result in the generated JavaScript...

    var i = 1
    var j = 2
    out += "i plus j = " + i + " + " + j + " = " + (i+j) + "\n"
    out += "The first command line argument is " + argv[0] + "\n"


JavaScript Evaluation
---------------------
After the JavaScript is generated, it is evaluated (if desired).  Before it's
evaluated, the following variables are defined, so they can be used in the
generated program:

    var fs = require('fs')
    var argv = []               // contains the arguments being processed
    var globalData = {}
    var out = ""


Support Functions
-----------------
js4m provides functions that can be called directly from JavaScript, including
the generated JavaScript of a preprocessed file.


### `var js4m = require('js4m')`

Loads js4m for use in a JavaScript program.  In a text file being preprocessed,
this is not necessary because the functions can be called directly
(i.e. without `js4m.` in front of each funciton name).


### `js4m.env(environment_variable_name, default_value)`
The environment variable name is read and returned if it exists.
Otherwise, the default_value will be returned, but if it's undefined
an error will be displayed.


### `js4m.get('x', default)`
Gets `globalData` in strict mode.  The global variable will be set from
the default if it's undefined.


### `js4m.set('x', value)`
Sets `globalData`.  The global variable will be set to `value`.


### `js4m.include(filename [, argv])`

Preprocesses the contents of the specified file, and returns the evaluated
results (as a string).  


### `js4m.processString(s, argv)`

Preprocesses the specified string (expanding macros, etc.), returning
the result as a string.


### `js4m.processFile(filename, argv)`
### `js4m.processFile(filename, outputfilename, argv)`

Preprocesses the contents of the specified file, and writes them to
outputfilename.  outputfilename is first deleted if it exists.
If outputfilename is omitted, the filename with a '.' at the beginning
of the name will be used.  argv is a single array variable.


### js4m.text2js(s)

Processes text into JavaScript.


LICENSE
-------

The MIT License (MIT)

Copyright (c) 2015, Scot Dietz

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.


