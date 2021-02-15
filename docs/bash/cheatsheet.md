# Bash Cheatsheet

## `shellcheck`

Verify validity of scripts.

`shellcheck somescript.sh`

## shell variables

var=value `with no spaces in between, else it will run it as a command`

echo "$var"

> Always quote your variables

a=2 and a="2" is same and evaluated as strings. Technically there is a way to do arichmetics, but it's better be avoided.

### interpolation

cat "${var}sometexttoappend"

## environment variables

`env` to list all available env variables

```sh
# To set an env variable

NEWENVVAR=somevalue
export NEWENVVAR
```

> Child processes inherit environment variables but not shell variables

## arguments

`$ svg2png pic1.svg pic1.png`

Access arguments with:

$0 - svg2png $1 pic1.svg $2 pic1.png

When command is run you can access all arguments as array with `$@` except the $0

### loop over arguments

```sh
for i in "$@"
do
...
done
```

### shift

```sh
echo $1 prints first argument
shift removes first argument
echo $1 prints second argument
```

## builtins

Most bash commands are programs but there are builtin functions as well such as:

1. type
2. source
3. read
4. echo
5. cd
6. printf

`which/type command` will tell you what it is.

## quotes

single quotes will not expand variables while double quotes will.

Thus, `echo: 'home: $HOME'` will return `echo: $HOME`

## globs

Is a way to match strings. If you want to match a regular expression though, quote it with single quotes.

`egrep 'b.*' file.txt` vs `grep b.* file.txt`

When you glob like:

`cat *.txt` it will match all text files, `but not` the ones that starts with `.`.

You have to define that explicitly

`cat .*.txt`

## redirects

STDIN
STDOUT
STDERR

### Redirect stderr

`somecommand 2> file.txt` -> redirects stderr to file.txt

### Redirect stderr to stdout

`somecommand 2>&1` -> redirects stderr to stdout

### sudo doesn't affect redirects

`sudo echo x > /etc/xyz` won't work. Instead do:

`echo x | sudo tee /etc/xyz`

See [here](https://stackoverflow.com/questions/82256/how-do-i-use-sudo-to-redirect-output-to-a-location-i-dont-have-permission-to-wr)

## brackets

### arithmetics

`x=$((2+2))` does arithmetics

### subshell

`(cd ~/downloads; pwd)`

### group commands

`{ cd ~/downloads; pwd }` -> groups commands and runs `within` the process

### brace expansion

`command a{.png,.svg}` -> expands to `command a.png a.svg`

### process substitution - similar to pipes

`<(command)`

### arrays

`x=(1 2 3)` creates an array

### [] vs [[]]

[Quick Reference](https://www.assertnotmagic.com/2018/06/20/bash-brackets-quick-reference/)

## non-posix features

1. `[[]]` 
2. arrays - posix shells only have one array: $@ for arguments
3. local keyword - in POSIX all variables are global
4. expansion: `a.{png,svg}`
5. no c style loops eg: for((i=; i<3; i++))
6. {1..5} POSIX alternative ${seq 1 5}
7. `$'\n' POSIX alternative $(printf "\n")

## for loops

```bash
for i in panda swan
do 
  echo "$i"
done
```

```bash
for i in panda swan; do echo "$i"; done
```

```bash
for word in $(cat file.txt)
do 
  echo "$word"
done
```

```bash
while COMMAND
do
...
done
```

```bash
for i in {1..5}
do 
  ...
done
```

## reading input

`read -r variable` -> reads STDIN into a variable

By default read uses `IFS`'s value to strip whitespace. Set `IFS=''` to avoid this.

Same applies to for loops -- for loops will iterate over every `word` in file, which by setting IFS='' can be avoided. Then looping will happen over each line instead

## functions

```bash
sayHello() {
}

function sayHello() {
}

# () may be omitted as in bash params will be used with $1, $2, $3

# local x -> sets local variable x
# y=somevalue -> is global
```

## pipes

OS creates buffer for each pipe and when it gets full, it will pause until there is more space.

### named pipes

We can create named pipes:

```bash
mkfifo mypipe
ls > mypipe &
wc < mypipe
```

## parameter expansion

${} is powerful, few usages:

1. Using default value for the variable:

```bash
${var:-$othervar}
```

2. `${#var}` -> length of string or array `var`

3. `${var:?some error}` -> prints "some error" and exists if variable is unset or null

4. Search and replace

```bash
x="some text"
# replace only first occurrence
${x/text/newtext}

# replace all occurrences
${x//text/newtext}
```

5. ${var} -> use when interpolating within a string etc...

6. ${var:offset:length} -> get a substring of var

7. Remove prefix or suffix of some pattern from variable

```bash
${x#pattern}
${x%pattern}
```

## background processes

Schedule a background process

```bash
somecommand &
```

Wait for processes to be finished

```bash

somecommand &
somecommand2 &

wait
```

Keep processes running even when after shell session is closed with `nohup`

```bash
nohup command &
```

jobs -> list all current bg processes spawned by current shell

fg -> jump into the bg process in foreground

bg -> resumes suspended job

`disown` -> like `nohup` but you do it after process is started

## trap

Callback functions in bash. We can listen to many different events and act on them:

1. unix signals(INT, TERM ...)
2. script exits(EXIT)
3. each line of code(DEBUG)
4. function returns(RETURN)

```BASH
trap 'kill $(jobs -p)' INT
```

Will kill all jobs on SIGINT signal

```bash
function cleanup{
  rm -rf $TMPDIR
}

trap cleanup EXIT
```

## errors

By default bash will continue working through the script even if any of it's commands exited with status code non-zero.

Also by default, unset variables will not error

To avoid this we can:

```bash
set -e # stop scripts on error
set -u # unset variables will error
set -o pipefail # by default when piping commands, if the left side fails, it doesn't make the later stages to fail

# to combine all above
set -euo pipefail
```

## debugging

1. `set -x` -> will print out every line of script as it executes
2. `trap read DEBUG` -> will stop before each line of code

