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

This will return an index and a process id. A notification will appear as soon
as a command completes. Adding the ampersand simply tells the shell not to wait
for a particular command to complete.  
