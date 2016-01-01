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
