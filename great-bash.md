# Great `bash`

## Test file contents

1. `file.txt`:
```
There's a big ... machine in the sky,
... some kind of electric snake ...
coming straight at us.
```
## Run

To run a script with bash:

    bash myscript.sh

## IO Redirection

Collectively referred to as standard IO.

* stdin
* stdout `>`
* stderr `2>`

### Redirecting output

You can redirect both `stdout` and `stderr` to the same file like this:

    ls -l file.txt not-here.txt &> output.txt

or like this:

    ls -l file.txt not-here.txt > output.txt 2>&1

This redirects `stderr` to the same location where `stdout` had already been
directed. Note that `2>&1` has to come **after** redirection to the file.

### Redirecting input

Take the `wc` program for example. It counts the words in a file or from
`stdin`. If you choose to input from `stdin`, press `CTRL-D` to end stream
input.

    wc < file.txt

If you use a pipe, you can leave out the intermediary file.

    ls | wc

That works if the `ls` command does not produce an error, but what if it does?
Then you need to use to use this syntax to combine `stdout` and `stderr`.

    ls not-here.txt 2>&1 | wc

### Creating input with here documents

You can feed content to a program reading from `stdin` with the `<<` operator.

    wc << EOF

`>` characters will appear if you enter this line. input your content and stop
reading by entering `EOF`. The text that you enter is saved in a buffer and
supplied as stream to the command that you're invoking.

You don't even really need to input the lines yourself, they may also come from
a file.

`ph` is a file with the following content:

```
grep -i $* << EOF
henk 06-12345
bob 06-13125
sjaak 06-43521
bert 06-48291
EOF
```

Run with bash:

    bash ph bob

The script that you feed as argument to bash, executes the first line, which
pulls in the here document. From there on the file will be read in line by line
until `EOF` is encountered. The second argument (the lookup) is referred to by
`$*`.

### Running tasks in the background

If running a command will take a long time, you can send it to the background.

    ./long-time-taking-command &

This will return a job number and a process id. A notification will appear as
soon as a command completes. Adding the ampersand simply tells the shell not to
wait for a particular command to complete.  

### Running commands in series

To run commands in series, separate the commands with a semicolon.

    pwd ; ls

Separating with a semicolon is like having the commands on separate lines.

If you run a command 'in the background' but decide that you do actually want
to wait for it to complete, you can use `wait [process-id]`. The terminal will
not respond until that process finishes.

In case you're running more than one process in the background and you want to
really wait for all of them to complete. just enter `wait` to wait for all
processes.

### Making scripts executable

To make a script executable, change permissions on so that it may execute.

    chmod 755 test.sh # or
    chmod a+x test.sh

You'll still want to execute a script with `bash` explicitly sometimes.

```
bash -n test.sh # dry run: does syntax checking
bash -v test.sh # prints both the commands and their results as they appear in the script
bash -x test.sh # same as above, but does not print comments
```

You can tell the executing environment about what application you want to use
to run your script on the first line with:

    #!/bin/bash

This is not needed in modern posix systems.

## Making decisions

`bash` allows you to use conditions in your scripts.

### Control operators

You can check if a command succeeded before proceeding to the next by using
control operators `&&` and `||`, which work the way you'd expect them to in
c-like languages.

    cd /temp && rm * # only invokes rm if cd succeeds
    cd /temp || echo does not exist # print message on failure

This type of logic does not stop the program by itself, though you can use
`exit` for that.

    cd /might-not-exist || exit 1

You can group logic inside `{ }`. It's important to have a trailing semicolon
followed by a space after the last command and before the closing `}`

    cd /tmp 2>/dev/null || { echo cd failed ; exit 1 ; }
    echo here comes the rest of the script

Another way to group commands is by enclosing them in parenthesis `( )`. This
will run the enclosed commands in a **subshell**. This means that the contained
logic will only influence the behavior of that subshell and the parent shell will
just continue on its way.

### if statements

A better way to do logic in bash, is by using if statements. They look like this:

```
if cd /tmp 2>/dev/null
then
    echo successfully entered directory
else
    echo failed to enter directory
    exit 1
fi
echo here comes the rest of the script
```

It's possible to compact logic on to one line.

```
if cd /tmp 2>/dev/null ; then echo okidoki ; else exit 1 ; fi
```

You can use any combination of commands as condition. The result of the sequence
of commands will determine if the condition is truthy or falsy.

### Returning a value with `exit`

In `bash`, exiting with a non zero value means a script has failed.

```
exit 0 # success
exit 1 # failure
```

### Variables
Variable names are case sensitive and should start with an alphabetic character.

```
myVariable="random string of text" # you need the quotes if there are spaces
```

To echo out the value of a variable, you need to prepend `$` to the variable
name.

```
echo myVariable # will print "myVariable"
echo $myVariable # will print "random string of text"
```

With curly braces, you can concatenate variables and other values. enclose a
variable name in curly braces to substitute it with its value.

```
henk=foo
echo ${henk}bar # prints "foobar"
```

If you don't assign any value to a variable and use it in a command, it's the
same as running the command without any argument.

```
ls $NOVALUE # lists all file in directory
```
Because of substitution, you can capture a command in a variable as well.

```
COMMAND=pwd
$COMMAND # prints working directory
```

Variables are not carried over to sub-shell sessions by default. you need to
`export` a variable to make it usable outside your current shell session.

```
foo=bar
export foo # if you open another session now, you can use foo
```

If you would redefine `foo` inside the sub-shell session, it would shadow `foo`
in the parent shell. Once you would exit the sub-shell, `foo` will have the value
you set in the parent shell.

You can combine assigning a value and exporting and once you've exported a
variable, you don't need to do it again.

```
export foo=bar
```

Another way to export a variable is to use `declare`:

```
declare -x foo=bar
```

You can also assign a value to a variable for just the duration of the running
script. an executed script will not have access to variables that have been
declared but not exported.

```
# executable script.sh
echo $foo

# run it
foo=bar ./script.sh # prints "bar"
```

### Special variables

`bash` has variables predefined.

`$?` is set with the result of the last executed command, so you can verify if
it succeeded or not. It will receive the exit code of the last invoked command.
Note that its value changes **each time** you run a command.

`PS1` is the prompt string. Usually `$ ` by default.

`PATH` contains a list of locations where `bash` will look for commands you want
to execute. it's a collection of directory paths separated by colons. The
directories are read left to right.

Not all things you execute in `bash` are programs. There are also **builtins**
like `cd` and `pwd`. To find out what type of executable you're dealing with:

```
type cd # prints "cd is a shell builtin"
type ls # prints "ls is hashed (/bin/ls)"
```

Say you'd want to add a `bin` folder in your home directory that will hold your
own commands.

```
export PATH="$PATH:~/bin"
```

To execute a command that is not in your `PATH`, tell bash to look for it in the
current directory:

```
./mycommand
```

You could add `.` to your path so that `bash` would always be able to execute any
command, but for reasons of security, it's best not to do so.

### Formatting output with `printf`

Remember to insert a newline character `\n`. If you want to use a percent sign,
escape it by prepending another percent sign `%%`.

```
foo=bar
printf "prepare the suite at once, i'll be at the %s\n" $foo

foo=42
printf "the meaning of life = %d\n" $foo

#TODO: doesn't work?
foo=000
printf "%#x is the new white\n" $foo

# specify padding
foo=3
printf "1, 2, and %8d\n" $foo

# and padding character
printf "1, 2, and %08d\n" $foo
```

### Positional parameters

Words on the command line to the right of the command name. You can not reassign
a positional parameter inside a script.

`$0` is the command name, the parameters start at `$1` and are numbered in order
of appearance.

```
# inside executable params.sh
echo hello $1
./params.sh "good sir"

# if you want to see what the script does line by line:
bash -x params.sh "good sir"
```

### All parameters

You can access all parameters to a script with `$*` or `$@`. There is a
difference in behavior.

#### `$*`
if you pass this to a script in quotes (`"$*"`) it will be treated as a single
argument.

If not quoted and the value contains spaces, spaces will act the way they would
in regular command invocation, as separators between commands.

#### `$@`
Parses the arguments, allowing spaces in arguments.

#### `$#`
Returns the number of arguments passed to the script

### Changing values with simple substitutions

When you use the offset curly braces with a variable, you can manipulate its
value.

* `?` matches one character
* `*` matches 0 or more characters

#### Remove from the trailing end with `%`

```
foo=abba
echo ${foo%a} # removes last 'a'
echo ${foo%?} # removes last 'a'
echo ${foo%b*} # removes 'ba'
echo ${foo%%b*} # removes 'bba'. double percent means largest possible match

pic=foo.jpg
echo ${foo%jpg}png # convert a file extension
```

#### Remove from the leading end with `#`

```
foo=abba
echo ${foo#a} # removes first 'a'

email=bob@snork.com
echo ${email#*@} # prints "snork.com"
```

If you are going to save the output of the substitution to a variable, you should
wrap it in quotes to account for any spaces that might be in there.

#### Remove from the middle of the string with `/`

The syntax looks like the on that `sed` uses.

```
foo=foobarbaz
echo ${foo/bar} # prints "foobaz"

foo=abba
echo ${foo/bb/tacam} # atacama
```

## Flow of control

### `for` loop

A loop like this in a script will take any parameters you pass to it as
arguments.

You can use `continue` to skip to the next iteration.

```
# in executable file called loop

for VAR
do
    echo $VAR
done

# run it

./loop one two three
prints  "one"
        "two"
        "three"
```

### Looping over a list

with `for ... in`

```
for item in one two three four five
do
    echo "$item"
done
```

### Counting while looping

A regular c-like `for` loop
```
for ((i=0 ; i < 9 ; i++ ))
do
    echo $i
done
```

### Manipulating strings using substrings

You can extract a portion of a string like this:

```
foo=foobar
echo ${foo:0:3} # prints "foo" # :[offset]:[string length]
```

Combined with an indexed `for` loop, you can loop through a string character by
character.

```
# in executable script..
VAR=$1
for ((i=0 ; i<${#VAR} ; i++))
do
    echo ${VAR:$i:1}
done
# run it
./script foobar # will print argument character by character
```

### using `read`

Read is useful for parsing whitespace separated values. You can populate a
variable from `stdin` and use redirection.

```
read foo
foobar # user types in the value of $foo
echo $foo # prints "foobar"

read -p "Enter your name " foo
```

if you use multiple variables with `read`, the input is separated by the spaces
it contains.

```
read foo bar
# user inputs:
make my day
echo $foo # prints "make"
echo $bar # prints "my day"
```

### `while` loops

The while loop below runs as long as the value inside `(( ))` is non-zero.

```
let i=0
while (( i < $1 ))
do
    echo $i
    let i++
done
```

A while loop can also run as long as the command inside `(( ))` executes
successfully.

```
# in executable
while read foo
do
    echo $foo
done

# run it
ls | ./while # list files in directory one by one
```

### `case` statements

Same as `switch` in c-like languages.

```
read -p "gimme some: " condition

case "$condition" in
    foo)    echo "FOO"
            # more statements
            ;;
    bar)    echo "BAR"
            # more statements
            ;;
    baz)    echo "BAZ"
            # more statements
            ;;
esac
```

You can use shell pattern matching to compare input to your cases.

To create a `default` block, for instance, you can write up the case like

```
case "$condition" in
    *)  echo "this is the default"
        ;;
```

* You're allowed to leave off the leading parenthesis from a case block.
* `;;` ends a case.
* `;;&` means fall through to the next case but continue matching.
* `;&` means just fall through without additional pattern matching.

## Shell math and logic

`bash` is, by default, a string based language. Variables hold string values, so
numbers will not have their intended meaning.

```
foo=5
bar=10
baz=$foo+$bar
echo $baz # prints "5+10"
```

To allow a variable to hold an integer, use `declare -i`

```
declare -i counter=0
count+=1 # no $ prepended. Spaces between operator and operands not allowed
echo $count # prints 1
```

A way to let `bash` know you're about to do some math, is to use `let`
followed by the variable name you want to operate on.

```
foo=7
bar=8
let foo+=bar
echo $foo # prints 15

# with the expression in quotes, you can use spaces.
let "foo += bar"
echo $foo # prints 23
```

Yet another way is to use double parenthesis `(( ))` Everything inside the
parenthesis will undergo arithmetic evaluation. No quotes needed!

```
foo=1
bar=2
(( foo += bar ))
echo $foo # prints 3
```

An third way is to use `$(( ))`. This can be used wherever you'd be able to use
a shell variable reference.

```
echo $(( 3 + 4 )) # prints 7
```

These examples demonstrate **integer** arithmetic. It won't work with fractional
values.

### Making mathematical decisions

You can use `(( ))` to wrap if statements.

```
foo=10

if (( foo > 5 ))
then
    echo more than 5
fi
```

### Simple polish calculator deluxe version

```
if(( $# < 3 || $# % 2 == 0))
then
    echo boo
    exit 1
fi

answer=$(( $1 $3 $2 ))
shift 3 # discards the first 3 items in the array and moves the rest to the left

while (( $# > 0 ))
do
    answer=$(( answer $2 $1 ))
    shift 2
done

echo $answer
```

### `if` revisited

Besides arithmetic and command results, you can make conditions with `[ ]` or
`[[ ]]`.

The `&&` and `||` logic is replaced with `-a` and `-o`

#### Does a file exist?
Will search relative to the current working directory, unless you specify a full
path.

```
if [ -e $myFile ]
then
    echo the file exists!
fi

if [ ! -e $myFile ]
then
    echo the file does not exist!
fi
```

#### Is it a directory?

```
if [ -d $someDir ]
then
    echo it is indeed a directory
fi
```

#### What kind of permissions does the file have?

```
if [ -r $readableFile ]
then
    echo the file is readable
fi

if [ -w $writableFile ]
then
    echo the file is writable
fi

if [ -x $executableFile ]
then
    echo the file is executable
fi
```

#### Note about logic

Comparisons with `<` and `>`in square bracket notation are always string based.
If you want numeric comparison you can either use `-eq`, `-lt`, `-gt` (covered
in the `bash` manual), or compare inside double parenthesis.

### `test`

An executable that handles logic. Much like the logic described above.

### `if` with double square brackets

Conditions inside double brackets `[[ ]]` provide all the functionality that
single square bracket notation provides, but includes additional syntax.
For example, you can do this in double square bracket notation.

```
if [[ $1 == *.txt ]]
then
    echo this is a txt file
fi
```

This will not compare to a literal `*.txt` like single square bracket notation
will, but instead will shell-pattern-match to see if the file ends in `.txt`.

Regular expressions are available when you use this syntax. This is the only
place in `bash` where this is possible. `=~` allows you to use a regex pattern
to do logic. Do not use quotes to delimit the regex pattern.

```
if [[ $1 =~ ^foo.*\.txt ]]
then
    echo match!
fi
```

## Functions in `bash`

Define a function like this:

```
# define:

function foo ()
{
    echo inside foo
}

# invoke:

foo # prints "inside foo"
```


There's some flexibility in the syntax you use to define the function. For
instance, you could lose the `function` keyword OR the parenthesis to the right
of the function name `()`.

```
function foo
{
    echo bar
}

foo ()
{
    echo baz
}
```

The characters you use to wrap the function body don't have to be curly braces,
but may also be `( )`, `[ ]`, `(( ))` or `[[ ]]`. These will have different
implications than the regular `{}` enclosure of the function body.

### `( )`
Runs the code in the function body in a sub-shell. Means that you will not be
able to reference some variables you have defined in a parent shell.

### `(( ))`
Interpret the function body as arithmetic.

Bash has no concept like **hoisting** function definitions, so you'd be best off
declaring your functions at the top of your script.

### Function parameters

The positional parameters are redefined while you're inside a function. When the
function returns, the values are restored.

The only positional parameter that is not subject to this rule, is `$0`. This
parameter will always contain the name of the running script, inside or outside
the function.
```
# executable file myfunc
function foo()
{
    echo $1
}
echo $1
foo "redefine this"

# run it
./myfunc bar
```

### Function scope

Regularly declared variables are always global in bash, even inside functions.
If you want to declare a local variable, you can do

```
function foo ()
{
    local bar=foobar
}
foo # if you don't run foo, bar wouldn't be defined at all, locally or globally
echo $bar # can't touch this
```

`declare foo` will also declare a variable `foo` which is **local** to its
containing function.

To explicitly define a **global** function, you can use `declare -g foo` and
later on, assign a value to it, which will be globally available.

### Parsing lines to get max file size

This script parses the output of `ls -l` line by line and extracts the **size**,
while keeping a counter on how many lines (files) were read.

1. The `max` function stores the size globally if it is larger than a
previously set value.
2. `lsparse` gets fed one line at a time of space-separated strings; the result
of reading the output of `ls -l` from **stdin** with `read`.

```
function max ()
{
    if (( ${1} > MAX )) # the curly braces are not really needed
    then
        MAX=$1
    fi
}

function lsparse ()
{
    if(( $# < 5 ))
    then
        SIZ=-1
    else
        SIZ=$5
    fi
}
declare -i CNT MAX=-1
while read lsline
do
    let CNT++
    lsparse $lsline
    max $SIZ
done

printf "largest of %d files was: %d\n" $CNT $MAX
```

This script illustrates that a space separated string, when fed to a function,
is translated into positional parameters for you to work with.

### Dynamic variable names

To do indirect substitution, use an exclamation mark `!`.

```
foo=bar
bar="party time"
echo ${!foo}
```

Here is a script that leverages this technique to get the absolute value of the
given list of numbers. Note that the current iteration variable `num` is
**global**, which is why you are able to reference it in the condition.

```
function abs()
{
    local VN=$1
    if(( ${!VN} < 0 )) # translates to ${num}, which amounts to the value of the current loop iteration
    then
        let ${VN}=0-${!VN} # sets num to 0 - loop iteration value, which makes a negative number positive
    fi
}
for num
do
    printf "ABS(%d) = " $num
    abs "num"
    printf "%d\n" $num
done
```

### Make your own library

With `source filename.sh` you can include files. This is useful if you want to
load some functions into a script you're working on. Be aware that the script
you're including only contains function definitions, no executable code.
`. filename.sh` is an older, alternative syntax to achieve this. Some advice:

1. Comment your code
2. Use local variables to avoid name conflicts

## Arrays

You need to use curly braces for accessing arrays, because square brackets on
their own mean that you intend to start some shell expansion.

```
# Initialize a new array variable
declare -a myArray

# Populate array with values (normal zero-indexed)
myArray=(one two three four five)

# Assign a value to a specific position
myArray([6]=bar)
myArray([sjaak]=bert)

# Set value for index
foo[3]=123 # set value of index 3
echo ${arr[3]} # read the value from index 3
```

### Using arrays in loops

The `read` program has a flag for working with arrays. Here we're going to read
the output from `ls -l`. The words in the resulting string are going to become
items in an array. We will print the 6th position, which is the day the file
was created. A new array is created that counts how many time a particular day
was found.

```
# @todo: revisit this
# example output from ls -l
# -rwxr-xr-x  1 thadeetrompetter  staff  183 Jan  8 15:02 test

# in executable script

while read -a lsout
do
    dom=${lsout[6]}
    let counter[dom]++
done
```

An alternative way to loop over a collection, is to use the following syntax.
`"${counter[@]}"` gets transformed in to an array of words. Obviously you can
only get a hold of the current item's value this way.

```
for nm in "${counter[@]}"
{
    printf "%d\n" $nm
}
```

Using an exclamation mark, you can retrieve the indices that are **set**:
`"${!counter[@]}"`. Indices that have not been set, are skipped.

```
for nmIndex in "${!counter[@]}"
{
    printf "%d - %d\n" $nmIndex ${counter[nmIndex]}
}
```

### Associative arrays

**Note**: only works with bash version 4 and up.

You declare an associative array with `declare -A myAssociativeArray`

```
declare -A arr
arr["foo"]=bar
echo ${arr["foo"]}
```

## Parsing options

To parse options in a `bash` script, use the `getopts` builtin. Getopts
expects the flags you want to allow, as well as the variable to store them in.
The arguments are not stored in any particular order.
in `opts.sh`

```bash
while getopts "abc" FLAG
do
    case $FLAG in
    a) AFLAG=set # though you can use any truthy value here
        ;;
    b) BFLAG=set
        ;;
    c) CFLAG=set
        ;;
    *) echo "some default or error message signaling improper use of flags"
        exit 1
        ;;
    esac >&2 # this sends any output from inside the switch to stderr
done # you could also redirect any and all loop output to stderr.
```

`getopts` will generate an error if you supply an option that is not allowed.
You can suppress this behavior by prepending a colon to the string that
contains the allowed options.

```
":abc" # will silence getopts' error reporting
```

### Doing something with a parsed option

You can use a parsed option in a conditional in a wordy way like this:

```
if [[ $SOME_FLAG ]]; then echo something fi
```

Or use a shorthand

```
[[ $SOME_FLAG ]] && echo something
```

### Discarding flags from the script parameters

`OPTIND` when using `getopts` gives you the number of arguments that were used.
Because arguments take up the space of positional parameters, you can get rid
of them like so:

```
shift $((OPTIND - 1))
```

Now other data you might have in positional parameters will be available

### Flags with a value

You want this:

```
somecommand -x somevalue

# in getopts.sh
# Add a colon after the flag that holds the value
while getopts "ax:bc" FLAG # ...
```

In the case that handles the `-x` flag, you can now make use of a special
variable named `$OPTARG`. It will get overridden by the next flag with a value,
So take care to store it somewhere safe.

### Using `$OPTARG`

In the switch statement up there, add a case:

```bash
x) XVAL="$OPTARG"
    ;;
```

## Pipeline issues / grouping

You might be tempted to instead of piping a command to a script, like:

```
ls -l | ./somescript

# instead, inside ./somescript

ls -l | while read FOO BAR BAZ ...etc
```

This will not work because the ls and while loop are both being run in their own sub-shell. As soon as the sub-shell commands finish, their variables with values will not be available to the rest of the script anymore.

You can enclose a group of statement that need to be executed in the same shell session in braces `{}`. All variables in this delimited group share the same environment.
