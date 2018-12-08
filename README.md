sed.py
======

A full implementation of sed in Python
--------------------------------------

* * *

### Contents

* * *

[General Information](#GeneralInformation)

[Usage as a command line utility](#UsageConsole)

[Usage as a Python module](#UsageModule)

[Sed dialect](#Dialect)

[Testing](#Testing)

[Timing](#Timing)

[To do list](#Todo)

* * *

### General Information

* * *

#### Description

`sed.py` is a full and working Python implementation of sed. Its reference is GNU sed 4.2 of which it implements almost all commands and features. It may be used as a command line utility or it can be used as a module to bring sed functionality to Python scripts.

A complete set of tests is available as well as a testing utility. These tests include scripts from various origins and cover all aspects of sed functionalities.

* * *

#### Platform

`sed.py` is a Python script and should run on any platform where a recent version of Python is installed.

| Version                    | Compatibility status                                         |
| -------------------------- | ------------------------------------------------------------ |
| Python 3.7                 | Fully compatible except s///g with zero length matches. See this [question at stackoverflow](https://stackoverflow.com/questions/53642571/retrieving-python-3-6-handling-of-re-sub-with-zero-length-matches-in-python-3) |
| Python 3                   | Fully compatible                                             |
| Python 2.7.4 and above     | Fully compatible                                             |
| Python 2.7 to Python 2.7.3 | Fully compatible except regexps of the form ((.\*)\*). This causes one of the script from Chang suite to fail. |
| Python 2.6                 | Fully compatible except regexps of the form ((.\*)\*). argparse module must be installed. |
| Python 2.5 and below       | Not tested                                                   |

Compatibility status applies also to the testing utility `test-suite.py`.

* * *

#### License

`sed.py` is released under the MIT license.

* * *

### Usage as a command line utility

* * *

`sed.py` may be used as console program receiving information from the command line. The format of the command line is:

`sed.py \[options\] -e<script expression> <input text file>`
`sed.py \[options\] -f<script file> <input text file>`

Note that `sed.py` accepts only one script file or expression, and only one input file. `options` may be one or both of:

`-n` disable automatic printing

`-r`use extended regular expressions

`sed.py` may also use redirection to receive its input or send its output with the usual syntax:

`cat myfile | sed.py -f myscript1 | sed.py -f myscript2 > myresultfile`

It is also possible for `sed.py` to receive its input from the keyboard by omitting any input file:

`sed.py -f myscript`

It is a Windows command line limitation that redirection does not work when calling directly a python script on the command line (check [this](https://mail.python.org/pipermail/python-bugs-list/2004-August/024920.html) for explanation). In that case, it is required to explicitly call python. Assuming python is in the path:

`> type myfile | python sed.py -f myscript > myresultfile`

It is also possible to hide the call in a batch file and even call it without any extension:

```
> type sed.batpython sed.py %1 %2 %3 %4 %5
> type myfile | sed -f myscript > myresultfile
> ...
```
* * *

### Usage as a Python module

* * *

An example covering all necessary symbols:

```python
from sed import Sed, SedException

sed = Sed()
try:
    sed.no_autoprint = True
    sed.regexp_extended = False
    sed.load_script('myscript.sed')
    sed.apply('myinput.txt')
except SedException as e:
    print(e.message)
except:
    raise
```

Note that `sed.apply()` returns the list of lines printed by the script. As a default, these lines are printed to stdout. `sed.apply()` has an output parameter which enables to inhibit printing the lines (`output=None`) or enables to redirect the output to some text file (`output=somefile.txt`).

The script may also be read from a string by using `sed.load_string(my_script_string)`.

* * *

### sed dialect

* * *

`sed.py` implements all standard commands and regular expression features of sed. Its reference is GNU sed 4.2. It implements almost all its features except the most specific ones.

GNU sed [manual](http://www.gnu.org/software/sed/manual/sed.html) page can serve as a reference for `sed.py` given the differences described in the following.

* * *


| Addresses         | status            |
| :---------------- | ----------------- |
| `number`          | standard behavior |
| `$`               | standard behavior |
| `/regexp/`        | standard behavior |
| `/regexp/I`       | implemented       |
| `\%regexp%`       | standard behavior |
| `address,address` | standard behavior |
| `address!`        | standard behavior |
| `0,/regexp/`      | not implemented   |
| `first~step`      | not implemented   |
| `addr1,+N`        | not implemented   |
| `addr1,~N`        | not implemented   |
| **Regular expressions** | **status**                                                   |
| `char`                 | standard behavior                                            |
| `*`                    | standard behavior                                            |
| `\+`                   | standard behavior                                            |
| `\?`                   | standard behavior                                            |
| `\{i\} \{i,j} \{i,\}`  | standard behavior                                            |
| `\(regexp\)`           | standard behavior                                            |
| `.`                    | standard behavior                                            |
| `^`                    | standard behavior. When not at start of regexp, matches as itself |
| `$`                    | standard behavior. When not at end of regexp, matches as itself |
| `[list] [^list]`       | standard behavior. \[.ch.\], \[=a=\], \[:space:\] are not implemented |
| `regexp1`&#124;`regexp2`  | standard behavior                                            |
| `regexp1regexp2`       | standard behavior                                            |
| `\digit`               | standard behavior (back reference)                           |
| `\n \t`                | standard behavior (extensions \\s\\S etc. are not handled)   |
| `\char`                | standard behavior (disable special regexp characters)        |
Note that for any combination of quantifiers (\*, +, ?, {}), consecutive quantifiers or a quantifier starting a regexp will launch an error. This is true in basic or extended regular expression modes.


    <table>
        <tr>
            <td width=150><code>number</code></td><td>standard behavior</td>
        </tr>
        <tr>
            <td><code>$</code></td><td>standard behavior</td>
        </tr>
        <tr>
            <td><code>/regexp/</code></td><td>standard behavior</td>
        </tr>
        <tr>
            <td><code>/regexp/I</code></td><td>implemented</td>
        </tr>
        <tr>
            <td><code>\%regexp%</code></td><td>standard behavior</td>
        </tr>
        <tr>
            <td><code>address,address</code></td><td>standard behavior</td>
        </tr>
        <tr>
            <td><code>address!</code></td><td>standard behavior</td>
        </tr>
        <tr>
            <td><code>0,/regexp/</code></td><td>not implemented</td>
        </tr>
        <tr>
            <td><code>first~step</code></td><td>not implemented</td>
        </tr>
        <tr>
            <td><code>addr1,+N</code></td><td>not implemented</td>
        </tr>
        <tr>
            <td><code>addr1,~N</code></td><td>not implemented</td>
        </tr>
    </table>

    
* * *

#### Extended regular expressions

Using the -r switch enables to simplify regular expressions by removing the antislah character before the special characters +, ?, (, ), |, { and }. If these characters must appear as regular characters in a regexp, they must be slashed.

* * *

#### Commands

| command         | comment                                                      |                                                              |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `a\ <rc> text`  | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (comments start anywhere in the line.)                       |
| `b label`       | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Programming-Commands.html#Programming-Commands) |                                                              |
| `: label`       | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Programming-Commands.html#Programming-Commands) |                                                              |
| `c\ <rc> text`  | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (including single line and double address extensions)        |
| `d`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Common-Commands.html#Common-Commands) |                                                              |
| `D`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `=`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (including double address extension)                         |
| `g`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `G`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `h`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `H`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `i\  <br/>text` | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (including single line and double address extensions)        |
| `l`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (length parameter not implemented)                           |
| `n`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Common-Commands.html#Common-Commands) |                                                              |
| `N`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `p`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Common-Commands.html#Common-Commands) |                                                              |
| `P`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `q`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Common-Commands.html#Common-Commands) | (except exit code extension)                                 |
| `r filename`    | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (including double address extension but not reading from stdin) |
| `s`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/The-_0022s_0022-Command.html#The-_0022s_0022-Command) | (except escape sequences in replacement (\\L, \\l, \\U, \\u, \\E), modifiers e and M/m, and combination of modifier g and number) |
| `t label`       | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Programming-Commands.html#Programming-Commands) |                                                              |
| `w filename`    | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) | (including double address extension but not writing to stdout or stderr) |
| `x`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `y`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Other-Commands.html#Other-Commands) |                                                              |
| `#`             | Compliant with [GNU sed description](https://www.gnu.org/software/sed/manual/html_node/Common-Commands.html#Other-Commands). | (comments start anywhere in the line.)                       |

The other commands specific to GNU sed are not implemented.

* * *

### Testing

* * *

#### Description

The working of `sed.py` is tested and compared to the behavior of GNU sed with a set of tests and a testing utility.

The tests are either coded in text files with .suite extension or may be stored in test directories as standard sed scripts.

The test suites are:

`unit.suite`

a text file containing unitary tests

`chang.suite`

a text file containing scripts from [Roger Chang web site](http://www.rtfiber.com.tw/~changyj/sed/)

`test-suite1`

a set of scripts from [GNU sed test suite](ftp://ftp.gnu.org/gnu/sed/)

`test-suite2`

a set of scripts from the [seder's grab-bag](http://sed.sourceforge.net/grabbag/), [Rosetta code web site](http://rosettacode.org/wiki/Rosetta_Code) and GitHub ([lisp](http://github.com/shinh/sedlisp)!)

`test-suite3`

additional unitary tests better stored in a folder with some extra data text files

`test-suite4`

a set of scripts from the [sed $HOME](http://http://sed.sourceforge.net/)

Note that the goal of these tests is not to check the correctness of the scripts but to verify that `sed.py` and GNU sed have the same behavior.

* * *

#### Testing utility

Tests are launched and checked with the `test-suite.py` Python script. This script uses either `sed.py` to run the sed scripts, or any sed executable. This enables to compare the working of `sed.py` with the one of GNU sed.

The calling syntax is:

```
test-suite.py <testsuite> \[number\] \[-b executable\] \[-x list of script references\]
```

| Parameters                  |                                                              |
| --------------------------- | ------------------------------------------------------------ |
| `testsuite`                 | either a text file with .suite extension or a test directory |
| `number`                    | an optional reference number of a test, when present only this tests is run |
| `executable`                | an optional name or path of a sed executable to use for testing |
| `list of script references` | an optional list of tests to exclude for instance when a feature is not implemented. A script reference is either the title of the test for tests stored in modules, or the the name of the script file. |

* * *

#### Text file test suites

When tests are stored in a text file (with .suite extension), they are made of four elements:

*   the title of the test
*   the script itself
*   the input list of lines
*   the expected result

The four elements of a test are separated with lines made of three identical characters, for instance:

```
---
Test substitution with global flag
---
s/an/AN/g
---
In Xanadu did Kubhla Khan
---
In XANadu did Kubhla KhAN
---
```

Note also that:

*   the script section may be empty, enabling to test a script on various data without repeating the script.
*   The input and output sections may be empty, enabling to test various scripts on the same data, without repeating the data.
*   Flags are set with a comment on the first line. As usual, #n stops autoprint mode and extended regexp mode is set with #r or #nr.
*   The expected result may be ??? when the test has no result and ends with an error.
*   All text outside the test, i.e. before first delimiter or after last delimiter, is ignored and acts like a comment.


#### Directory test suites

When tests are stored in a directory, they are represented by three or four files with same name but different extensions:

*   the script itself, with '.sed' extension
*   the input of the script, with '.inp' extension
*   the expected result of the script, with '.good' extension
*   possibly a file, with '.flags' extension, containing the sed switches -n and/or -r.

Some other files may be used when using reading or writing commands in scripts. In that case, the expected written files must be named with extension '.wgoodN' where N is the number of the expected written file.

* * *

### Timing

* * *

A python implementation of sed has to face legitimate questions about timing. Fortunately, results are not bad. Unfortunately, they seem correlated with version number. Timings are given in seconds.

| Platform                                     | GNU sed 4.2.1 | sed.py python 2.6 | sed.py python 2.7 | sed.py python 3.4 |
| -------------------------------------------- | ------------- | ----------------- | ----------------- | ----------------- |
| Windows7, Intel Xeon 3.2 GHz, 6 Gb RAM       | 19.4          | 19.1              | 22.6              | 26.9              |
| Windows XP, Intel Pentium4 3.2 GHz, 4 Gb RAM | 47.5          | 50.7              | 56.5              | 71.2              |
| Linux, Intel Pentium4 3.2 GHz, 4 Gb RAM      | \-            | \-                | 51.0              | \-                |

Test conditions:

*   Only script files are used (scripts from folders testsuiteN). This is to avoid measuring the time to extract scripts, inputs and results from .suite files.
*   The given values are averaged from three consecutive test runs.


### To do list

* * *

At one moment, one has to decide what will be in the release to come, and what can be delayed. Here are some features which would be nice to have but can be delayed to a future version.

*   Better POSIX compliance:

*   multiple scripts on the command line (-e, -f)
*   multiple input file
*   character classes

*   Better error handling (display of the number of the line in error)
*   Better error handling when testing (the error message could be tested)
*   Use sed.py as a basis for a sed debugger.
*   ...




![](http://www.godrago.net/piwik/piwik.php?idsite=1)