<!-- TOC -->

- [Shell expansions & string manupulations](#shell-expansions--string-manupulations)
  - [parameter or variable expansions](#parameter-or-variable-expansions)
  - [String manupulations with `expr`](#string-manupulations-with-expr)
- [`read`](#read)
  - [Syntax:](#syntax)
  - [options:](#options)
  - [Examples:](#examples)
- [`test` command (`[]`)](#test-command-)
  - [Syntax](#syntax)
    - [File tests:](#file-tests)
    - [String tests:](#string-tests)
    - [Shell options and variables:](#shell-options-and-variables)
    - [Simple logic (test if values are null):](#simple-logic-test-if-values-are-null)
    - [Numerical comparisons:](#numerical-comparisons)
  - [Options](#options)
  - [examples](#examples)
  - [Combine multiple test conditions](#combine-multiple-test-conditions)
- [Parsing bash script options with getopts](#parsing-bash-script-options-with-getopts)
  - [Shifting processed options](#shifting-processed-options)
  - [Parsing options with arguments](#parsing-options-with-arguments)
  - [An example](#an-example)
- [`split` function](#split-function)
- [`body() function in NGSeasy`](#body-function-in-ngseasy)
- [bioawk](#bioawk)
- [Parallel](#parallel)
- [xargs](#xargs)

<!-- /TOC -->

# Shell expansions & string manupulations
## parameter or variable expansions
- Basic form is `$a` or `${b}cd`
- Set the **defalut value** of a variable: `test=${test:-0}`. Examples:

  ```Shell
  #test.sh
  TEST_MODE={$TEST_MODE:-0}
  if [[ TEST_MODE -eq 0 ]]
  then
    echo "Running in live mode"
  else
    echo "Running in test mode"
  fi
  ```
  Normally scrit is running in live mode. To change it:

  ```Shell
  env TEST_MODE=1 sh test.sh
  ```

  or use the default value with command line argument or from a config file:

  ```Shell
  if [[ ${cmd_arg_x:-0} -eq 0 ]]
  then
    echo "-x not specified"
  else
    echo "-x is specified"
  fi
  ```

- **default value of positional parameters:**

  ```Shell
  var=${1:-defaultValue}
  ```
  ```Shell
  #!/bin/bash
  #test.sh
  var=${1:-/home/phpcgi}
  echo "Setting php-cgi at ${var}..."
  # rest of script
  ```

Run the script as:
  - `test.sh` /my_dir  <--- set php jail at /my_dir dir
  - `test.sh` <--- set php jail at /home/phpcgi dir (the default value)

  A handy example:

  ```Shell
  _mkdir(){
          local d="$1"               # get dirname
          local p=${2:-0755}         # get permission, set default to 0755
          [ $# -eq 0 ] && { echo "$0: dirname"; return; } # $0:expands to'bash'
          [ ! -d "$d" ] && mkdir -m $p -p "$d"
  }
  ```

- `${var:=value}`: he assignment (:=) operator is used to assign a value to the variable if it doesnot already have one; if variable hsa already a value, the assignment doesnot change it; This will not work for positional parameters.

- Display an **Error Message** If $VAR Not Passed:

  ```Shell
  ${varName?Error varName is not defined}
  ${varName:?Error varName is not defined or is empty}
  ${1:?"mkjail: Missing operand"}
  MESSAGE="Usage: mkjail.sh domainname IPv4"      ### define error message
  _domain=${2?"Error: ${MESSAGE}"}  ### you can use $MESSAGE too
  ```

  `_domain="${1:?Usage: mknginxconf domainName}"`: if the $1 command line arg is not passed, stop executing the script with an error message.

- **string substitution**
  - `${var/search/replace}`: replace the first instance
    ```Shell
    var=aabbcc
    ${var/b/-dd-}
    # output: aa-dd-bcc

    Arr=(milAAA milBBB milCCC)
    echo ${Arr[@]/mil/ziba} # zibaAAA zibaBBB zibaCCC

    str=milAAA milBBB milCCC
    set -- $str
    echo ${@/mil/ziba} # zibaAAA zibaBBB zibaCCC

    str="f11:f12|f21:f22|f31:f32"
    IFS='|'
    set -- $str
    echo $1 # f11:f12

    ```


  - `${var//search/replace}`: replace all instances
     ```Shell
     var=aabbcc
     ${var//b/-dd-}
     ```
     output: aa-dd--dd-cc

- **removing prefixes and suffixes**:
  - `${VAR#pattern}`: Removes any prefix from the expanded value that matches the pattern.
  The removed prefix is the shortest matching prefix, if you use double pound-signs/hash-marks the longest matching prefix is removed `${VAR##pattern}`
  - `${VAR%pattern}`: removes a matching suffix (single percent for the shortest suffix, double for the longest).

  ```Shell
  file=data.txt
  echo ${file%.*} #file base: data
  echo ${file#*.} #file extension: txt

  stringZ=abcABC123ABCabc

  echo ${stringZ#a*C}  # 123ABCabc
  #strip out shortest path between a and C
  echo ${stringZ##a*C} # abc

  #parametrize the substrings
  X='a*C'
  echo ${stringZ#$X}  # 123ABCabc
  echo ${stringZ##$X} # abc

  declare -A arr=( [idx1]=milAAA [idx2]=milBBB [idx3]=milCCC)
  echo ${!arr[@]}    # idx1 idx2 idx3
  echo ${!arr[*]}    # idx1 idx2 idx3
  echo ${#arr[@]}    # 3
  echo ${#arr[idx1]} # 6
  echo ${arr[@]#mil} # AAA BBB CCC
  echo ${arr[idx1]#mil} # AAA
  echo ${arr[idx1]#m*A} # AA

  str="milAAA milBBB milCCC"
  set -- $str
  echo ${@#mil}  # AAA BBB CCC
  ```

  ```Shell
  # Remove all filenames in $PWD with "TXT" suffixes to a
  # "txt" suffix. For example, "file.TXT" becomes "file.txt"
  SUFF=txt
  suff=txt

  for i in $(ls *.$SUFF)
  do
    mv -f $i ${i%.$SUFF}.$suff
  done
  ```

  Also note that these are **glob patterns** not regular expressions.
- `${str/#search/replacement}`: if search matched in front of str, replace it.
- `${str/%search/replacement}`: if search matched in end of str, replace it.

  ```Shell
  stringZ=abcABC123ABCabc
  echo ${stringZ/#abc/XYZ} # XYZABC123ABCabc
  echo ${stringZ/%abc/XYZ} # abcABC123ABCXYZ
  ```
  **The awk equivalent:**
  ```Shell
  string=23skid001
  # Bash: zero based positions
  # awk: 1 based positions

  echo ${string:2:4} # skid

  echo | awk '{print substr("'"${string}"'",3,4)}' # skid
  # piping an empty echo to awk gives it dummy input, and makes it unnecessary # to supply a flename.
  ```

- **extract substrings**: `${VAR:offset:length}`
offsets start at zero, if you don't specify a length it goes to the end of the string.

  ```Shell
  str=abcdefgh
  echo ${str:0:1} # a
  echo ${str:1}   # bcdefgh
  ```

  Negative offsets which count backwards from the end of the string. But in the below example, Bash misinterpret the code because it is like default value expansion.

  ```Shell
  str=abcdefgh
  echo ${str:-3:2} ##abcdefgh
  ```cdef

  To solve it and select from the end of sting:

  ```Shell
  str=abcdefgh
  echo ${str:$((-3)):2} ## fg

  # or
  i=-3
  echo ${str:$-3:2} ## fg
 ```

 The length can also be negative which means from offset to the last minus the last characters specified by length:

 ```Shell
 str=abcdefgh
 echo ${str:2:-2} # cdef
 ```

 **Reset the positional parameters**
 You can reset positional parameter using `set`.
 Here the first number (i.e. 1) refers to the whole str string, because there is only one whole string here.

 ```Shell
 str=abcdefgh
 set -- $str
 echo ${1:4}   # efgh
 echo ${1:4:2} # ef

 str="abcd efgh ijklm"
 set -- $str
 echo ${2:2} # gh
 echo $3     # ijklm
 echo ${@:2} # efgh ijklm
 echo ${@:2:3} # 3 positional param starting at $2

 arr[0]=abcdefgh
 echo ${arr[0]:2} # cdefgh
 echo ${arr[0]: -7:2} # bc
 ```

- Expand to the length of a variable: `${#VAR}`
- Can also be parametrized: `POS=4; $LEN=3; echo ${str:$POS:$LEN}`
- **Changing the case of matched patterns**:
  - `${parameter^pattern}`: lowercase to uppercase; only first match
  - `${parameter^^pattern}`: lowercase to uppercase; all matches
  - `${parameter,pattern}`: uppercase to lowercase; only first match
  - `${parameter,,pattern}`: uppercase to lowercase; all matches

  ```Shell
  str="milad Bastami"
  echo ${str^m}  # Milad Bastami
  echo ${str^^m} # Milad BastaMi
  echo ${str,,B} # milad bastami
  echo ${str^^} # MILAD BASTAMI
  echo ${str^} # Milad Bastami

  also works for arrays and positional parameters:
  echo ${array[@]^m}
  echo ${@^m}
  ```

- **operator expansion**: `${parameter@operator}`
  - `${var@Q}`: var will be single quoted; with eny special character being escaped;
  - `${var@E}`: var will be expanded; returning the special meaning of characters.


```Shell
foo="one\ntwo\n\tlast"
echo "$foo"    # one\ntwo\n\tlast
echo -e "$foo" # will expand it
echo ${foo@Q}  # 'one\ntwo\n\tlast'
echo ${foo@E}  # will expand it
done
two
  last

# display a file with multiple lines as a single string
# with escape chars (\n)
foo=$(<file.txt)
echo "${foo@Q}"
'line1\nline2'
```

## String manupulations with `expr`
- String length:
  ```shell
  echo `expr length $string`
  echo `expr  "$string" : '.*'`
  ```
- Length of matching substring at the beginning of string: `expr match "$string" 'substring'`
  ```Shell
  echo `expr match "$string" 'abc[A-Z]*.2'` # 8
  ```
- index: `expr index "$string" '$substring'`: numerical position of first matched character of substring in string.
  ```Shell
  string=abcABC123ABCabc
  echo `expr index "$string" C12` # 6
  echo `expr index "$string" 1c`  # 3
  # c in position#3 matches before 1
  ```

# `read`
By default, read considers a newline character as the end of a line, but this can be changed using the `-d` option.
After reading, the line is split into words according to the value of the special shell variable **IFS**, the internal field separator.
To preserve white space at the beginning or the end of a line, it's common to specify `IFS=` (with no value) immediately before the read command. After reading is completed, the IFS returns to its previous value. **By default, the IFS value is "space, tab, or newline".**

read assigns the first word it finds to name, the second word to name2, etc. If there are more words than names, all remaining words are assigned to the last name specified. If only a single name is specified, the entire line is assigned to that variable. If no name is specified, the input is stored in a variable named **REPLY**.

## Syntax:
```shell
read [-ers] [-a array] [-d delim] [-i text] [-n nchars] [-N nchars]
     [-p prompt] [-t timeout] [-u fd] [name ...] [name2 ...]
```

## options:

- `-a array`	Store the words in an indexed array named array. Numbering of array elements starts at zero.
- `-d delim`	Set the delimiter character to delim. This character signals the end of the line. If -d is not used, the default line delimiter is a newline.
- `-e`	Get a line of input from an interactive shell. The user manually inputs characters until the line delimiter is reached.
- `-i text`	When used in conjunction with -e (and only if -s is not used), text is inserted as the initial text of the input line. The user is permitted to edit text on the input line.
- `-n nchars`	Stop reading after an integer number nchars characters have been read, if the line delimiter has not been reached.
- `-N nchars`	Ignore the line delimiter. Stop reading only after nchars characters have been read, EOF is reached, or read times out (see -t).
- `-p prompt`	Print the string prompt, without a newline, before beginning to read.
- `-r`	Use "raw input". Specifically, this option causes read to interpret backslashes literally, rather than interpreting them as escape characters.
- `-s`	Do not echo keystrokes when read is taking input from the terminal.
- `-t timeout`	Time out, and return failure, if a complete line of input is not read within timeout seconds. If the timeout value is zero, read will not read any data, but returns success if input was available to read. If timeout is not specified, the value of the shell variable TMOUT is used instead, if it exists. The value of timeout can be a fractional number, e.g., 3.5.
- `-u fd`	Read from the file descriptor fd instead of standard input. The file descriptor should be a small integer. For information about opening a custom file descriptor, see opening file descriptors in bash.

## Examples:

```Shell
while read; do echo "$REPLY"; done
```

read takes data from the **terminal**. Type whatever you'd like, and press Enter. The text is echoed on the next line. This loop continues until you press **Ctrl+D (EOF)** on a new line. Because no variable names were specified, the entire line of text is stored in the variable **REPLY**.

```Shell
while read text; do echo "$text"; done
```
Same as above, using the variable name text.

```Shell
while read -ep "Type something: " -i "My text is " text; do
  echo "$text";
done
```
Provides a prompt, and initial text for user input. The user can erase "My text is ", or leave it as part of the input. Typing Ctrl+D on a new line terminates the loop.

```shell
echo "Hello, world!" | (read; echo "$REPLY")
```

Enclosing the read and echo commands in parentheses executes them in a dedicated subshell. **This allows the REPLY variable to be accessed by both read and echo**. But this will not work `echo "Hello, world!" | read | echo "$REPLY"`

```Shell
echo "one two three four" | while read word1 word2 word3; do
  echo "word1: $word1"
  echo "word2: $word2"
  echo "word3: $word3"
done
```

word1: one
word2: two
word3: three four

```Shell
echo "one two three four" | while read -a wordarray; do
  echo ${wordarray[1]}
done```
two

```shell
while IFS= read -r -d $'\0' file; do
  echo "$file"
done < <(find . -print0)
```

The above commands are **the "proper" way to use find and read together to process files**. It's especially useful when you want to do something to a lot of files that have odd or unusual names. Let's take a look at specific parts of the above example:

`while IFS=`. IFS= (with nothing after the equals sign) sets the internal field separator to "no value". Spaces, tabs, and newlines are therefore considered part of a word, which preserves white space in the file names.
Note that IFS= comes after while, ensuring that IFS is altered only inside the while loop. For instance, it won't affect find.

`read -r`: Using the -r option is necessary to preserve any backslashes in the file names.

`-d $'\0'`: The -d option sets the newline delimiter. We're using NULL as the line delimiter because Linux file names can contain newlines, so we need to preserve them. (This sounds awful, but yes, it happens.)
However, a NULL can never be part of a Linux file name, so that's a reliable delimiter to use. Luckily, find can use NULL to delimit its results, rather than a newline, if the -print0 option is specified:

`< <(find . -print0)`: Here, find . -print0 creates a list of every file in and under . (the working directory) and delimit all file names with a NULL. When using -print0, all other arguments and options must precede it, so make sure it's the last part of the command.

Enclosing the find command in <( ... ) performs process substitution: the output of the command can be read like a file. In turn, this output is redirected to the while loop using the first "<".

For every iteration of the while loop, read reads one word (a single file name) and puts that value into the variable file, which we specified as the last argument of the read command.

When there are no more file names to read, read returns false, which triggers the end of the while loop, and the command sequence terminates.

# `test` command (`[]`)
test provides no output, but returns an exit status of 0 for "true" (test successful) and 1 for "false" (test failed).
**Examples**
```shell
num=4; if (test $num -gt 5); then echo "yes"; else echo "no"; fi  # no
file="/etc/passwd"; if [ -e $file ]; then echo "whew"; else echo "uh-oh"; fi
```

## Syntax
### File tests:
```shell
test [-a] [-b] [-c] [-d] [-e] [-f] [-g] [-h] [-L] [-k] [-p] [-r] [-s] [-S]
     [-u] [-w] [-x] [-O] [-G] [-N] [file]
```

```shell
    test -t fd
```

```shell
    test file1 {-nt | -ot | -ef} file2
```

### String tests:
```shell
test [-n | -z] string
```

```shell
test string1 {= | != | < | >} string2
```

### Shell options and variables:
```shell
test -o option
```

```shell
test {-v | -R} var
```

### Simple logic (test if values are null):
```shell
test [!] expr

test expr1 {-a | -o} expr2
```
### Numerical comparisons:
for integer values only; bash doesn't do floating point math

```Shell
test arg1 {-eq | -ne | -lt | -le | -gt | -ge} arg2
```
## Options

- `a file`	Returns true if **file exists**. Does the same thing as -e. Both are included for compatibility reasons with legacy versions of Unix.
- `b file`	Returns true if file is **"block-special"**. Block-special files are similar to regular files, but are stored on block devices — special areas on the storage device that are written or read one block (sector) at a time.
- `c file`	Returns true if file is **"character-special."** Character-special files are written or read byte-by-byte (one character at a time), immediately, to a special device. For example, /dev/urandom is a character-special file.
- `d file`	Returns true if file is a **directory**.
- `e file`	Returns true if **file exists**. Does the same thing as -a. Both are included for compatibility reasons with legacy versions of Unix.
- `f file`	Returns true if **file exists**, and is a regular file.
- `g file`	Returns true if file has the setgid bit set.
-`h file`	Returns true if file is a **symbolic link**. Does the same thing as **-L**. Both are included for compatibility reasons with legacy versions of Unix.
- `-k file`	Returns true if file has its sticky bit set.
- `-p file`	Returns true if the file is a named pipe, e.g., as created with the command mkfifo.
- `-r file`	Returns true if file is readable by the user running test.
- `-s file`	Returns true if **file exists, and is not empty**.
- `-S file`	Returns true if file is a socket.
- `-t fd`	Returns true if file descriptor fd is opened on a terminal.
- `-u file`	Returns true if file has the setuid bit set.
- `-w file`	Returns true if the user running test has **write permission to file**, i.e. make changes to it.
- `-x file`	Returns true if file is **executable by the user** running test.
- `-O file`	Returns true if file is **owned by the user** running test.
- `-G file`	Returns true if file is **owned by the group** of the user running test.
- `-N file`	Returns true if file was **modified** since the last time it was read.
- `file1 -nt file2`	Returns true if file1 is **newer** (has a newer modification date/time) than file2.
- `file1 -ot file2`	Returns true if file1 is **older** (has an older modification date/time) than file2.
- `file1 -ef file2`	Returns true if file1 is a **hard link** to file2.
- `test [-n] string` Returns true if **string is not empty**. Operates the same with or without -n.
For example, if mystr="", then test "$mystr" and test -n "$mystr" would both be false. If mystr="Not empty", then test "$mystr" and test -n "$mystr" would both be true.
- `-z string`	Returns true if string *string* is **empty**, i.e. "".
- `string1 = string2`	Returns true if string1 and string2 are equal, i.e. contain the same characters.
- `string1 != string2`	Returns true if string1 and string2 are not equal.
- `string1 < string2`	Returns true if string1 sorts before string2 lexicographically, according to ASCII numbering, based on the first character of the string. For instance, test "Apple" < "Banana" is true, but test "Apple" < "banana" is false, because all lowercase letters have a lower ASCII number than their uppercase counterparts.

  **Tip:** Enclose any variable names in double quotes to protect whitespace. Also, escape the less than symbol with a backslash to prevent bash from interpreting it as a redirection operator. For instance, use test "$str1" \< "$str2" instead of test $str1 < $str2. The latter command will try to read from a file whose name is the value of variable str2. For more information, see redirection in bash.
- `string1 > string2`	Returns true if string1 sorts after string2 lexicographically, according to the ASCII numbering. As noted above, use test "$str1" \> "$str2" instead of test $str1 > $str2. The latter command creates or overwrites a file whose name is the value of variable str2.
- -`o option`	Returns true if the **shell option** opt is enabled.
- `-v var`	Returns true if the **shell variable** var is set.
- -`R var`	Returns true if the **shell variable** var is set, and is a name reference. (It's possible this refers to an indirect reference, as described in Parameter expansion in bash.)
- `! expr`	Returns true if and only if the expression expr is null.
- `expr1 -a expr2`	Returns true if expressions **expr1 and expr2 are both not null**.
- `expr1 -o expr2`	Returns true if **either of the expressions expr1 or expr2 are not null**.
- `arg1 -ne arg2`	true if argument arg1 is **not equal to** arg2.
- `arg1 -lt arg2`	true if numeric value arg1 is **less than** arg2.
- `arg1 -le arg2`	true if numeric value arg1 is **less than or equal** to arg2.
- `arg1 -gt arg2`	true if numeric value arg1 is **greater than** arg2.
- `arg1 -ge arg2`	true if numeric value arg1 is **greater than or equal** to arg2

## examples

```Shell
if test false; then
  echo "Test always returns true for only one argument, unless it is null.";
fi
# Test always returns true for only one argument, unless it is null.

[ false ]; echo $? # 0
[ true ]; echo $? # 0

# works for files and directories
test -e /sys; echo $?   # true if file or directory exists
# for directories
if test -d /home; then echo "/home is a directory"; fi
# for regular files (i.e not directories)
test -f /etc/shadow; echo $?   # true for regular files

touch newfile; if [ -O newfile ]; then echo "I own newfile"; fi
str=""; test -z "$str"; echo $?   # true if variable str is empty string ""
str=""; test -n "$str"; echo $?   # true if variable str is not empty string ""
str=""; test "$str"; echo $?      # same as above command; -n is optional
str="Not empty"; test -z "$str"; echo $?
```
## Combine multiple test conditions

```Shell
[ $x -gt 5 ] && [ $x -lt 8 ] && [ $x -ne 6 ]
[ $x -gt 5 ] || [ $x -lt 8 ]
```

```Shell
[ $x -gt 5 -a $x -lt 8 ] # -a  for AND
[ $x -gt 5 -o $x -lt 8 ] # -o for OR
```

```Shell
[[ $x -gt 5 ]] && [[ $x -lt 8 ]] && [[ $x -ne 6 ]]

[[ $x -gt 5 && $x -lt 8 && $x -ne 6 ]]
```

# Parsing bash script options with getopts
This tutorial explains how to use the `getopts` built-in function to parse arguments and options to a bash script.
The getopts function takes three parameters:
1. The first is a specification of **which options are valid**, listed as a sequence of letters. For example, the string `'ht'` signifies that the options -h and -t are valid.
2. The second argument to getopts is a **variable** that will be populated with the option or argument to be processed next. In the following loop, `opt` will hold the value of the current option that has been parsed by `getopts`.

  ```shell
  while getopts ":ht" opt; do
    case ${opt} in
      h ) # process option a
        ;;
      t ) # process option t
        ;;
      \? ) echo "Usage: cmd [-h] [-t]"
      ;;
    esac
  done
  ```
  - if an invalid option is provided, the option variable is assigned the value `?`.
  - Second, this behaviour is only true when you prepend the list of valid options with `:` to disable the default error handling of invalid options. It is **recommended** to always disable the default error handling in your scripts.
- The third argument to getopts is the list of arguments and options to be processed. When not provided, this defaults to the arguments and options provided to the application (`$@`)

## Shifting processed options
The variable `OPTIND` holds the number of options parsed by the last call to getopts. It is common practice to call the shift command at the end of your processing loop to remove options that have already been handled from $@ by:

```shell
shift $((OPTIND -1))
```

## Parsing options with arguments
Options that themselves have arguments are signified with a `:`. The argument to an option is placed in the variable `OPTARG`. In the following example, the option `t` takes an argument. When the argument is provided, we copy its value to the variable target. If no argument is provided getopts will set opt to `:`. We can recognize this error condition by catching the : case and printing an appropriate error message.

## An example
check that -s exists, if not return error; check that the value after the -s is 45 or 90; check that the -p exists and there is an input string after; if the user enters ./myscript -h or just ./myscript then display help.

```Shell
#!/usr/bin/env bash
usage() { echo "$0 usage:" && grep " .)\ #" $0; exit 0; }
[ $# -eq 0 ] && usage
while getopts ":hs:p:" arg; do
  case $arg in
    p) # Specify p value.
      echo "p is ${OPTARG}"
      ;;
    s) # Specify strength, either 45 or 90.
      strength=${OPTARG}
      [ $strength -eq 45 -o $strength -eq 90 ] \
        && echo "Strength is $strength." \
        || echo "Strength needs to be either 45 or 90, $strength found instead."
      ;;
    h | *) # Display help.
      usage
      exit 0
      ;;
  esac
done
```
# `tee`: Redirect output to multiple files or processes
The tee command copies standard input to standard output and also to any files given as arguments. This is useful when you want not only to send some data down a pipe, but also to save a copy.
- The tee command is useful when you happen to be transferring a large amount of data and also want to summarize that data without reading it a second time.
  - you are downloading a DVD image, you often want to verify its signature or checksum right away. The **inefficient way** to do it is simply: `wget https://example.com/some.iso && sha1sum some.iso`. it makes you wait for the download to complete before starting the time-consuming SHA1 computation and reads DVD two times.
  - The efficient way: `get -O - https://example.com/dvd.iso | tee >(sha1sum > dvd.sha1) > dvd.iso`. That makes tee write not just to the expected output file, but also to a pipe running sha1sum and saving the final checksum in a file named dvd.sha1.
  - Note also that if any of the process substitutions (or piped stdout) might exit early without consuming all the data, the `-p` option is needed to allow tee to continue to process the input to any remaining outputs.
  - A more conventional and portable use of `tee`: `wget -O - https://example.com/dvd.iso | tee dvd.iso | sha1sum > dvd.sha1`
- You can extend this example to **make tee write to two processes**:
    ```Shell
    wget -O - https://example.com/dvd.iso \
    | tee >(sha1sum > dvd.sha1) \
          >(md5sum > dvd.md5) \
    > dvd.iso
      ```

- This technique is also useful when you want to make a compressed copy of the contents of a pipe. For a
large hierarchy, ‘du -ak’ can run for a long time, and can easily produce terabytes of data, so you won’t want to rerun the command unnecessarily. Nor will you want to save the uncompressed output.
  - Inefficient way:
    ```shell
    du -ak | gzip -9 > /tmp/du.gz
    gzip -d /tmp/du.gz | xdiskusage -a
      ```
  - Efficient way:
    ```shell
    du -ak | tee >(gzip -9 > /tmp/du.gz) | xdiskusage -a
    ```
- Another example
-
  ```Shell
  tardir=your-pkg-M.N
  tar chof - "$tardir" \
  | tee >(gzip -9 -c > your-pkg-M.N.tar.gz) \
  | bzip2 -9 -c > your-pkg-M.N.tar.bz2
  ```
- If you want to further process the output from process substitutions, and those processes write atomically (i.e., write less than the system’s PIPE BUF size at a time), that’s possible with a construct like:
-
  ```shell
  tardir=your-pkg-M.N
  tar chof - "$tardir" \
    | tee >(md5sum --tag) > >(sha256sum --tag) \
    | sort | gpg --clearsign > your-pkg-M.N.tar.sig
  ```

# Other notes
- `set -o errexit` or `set -e` in a script: `-e` Exit immediately if a pipeline, which may consist of a single simple command, a subshell command enclosed in parentheses, or one of the commands executed as part of a command list enclosed by braces returns a non-zero status. **The shell does not exit if** the command that fails is part of the command list immediately following a while or until keyword, part of the test in an if statement, part of any command executed in a && or || list except the command following the final && or ||, any command in a pipeline but the last, or if the command’s return status is being inverted with !. Err exit doesnot work inside functions, but find a solution at this **[link][b4bf5877].**
# `split` function
# `body() function in NGSeasy`
https://unix.stackexchange.com/questions/11856/sort-but-keep-header-line-at-the-top/11859#11859
# bioawk
# Parallel
# xargs
# `find`
# `sed`
  [b4bf5877]: https://stackoverflow.com/questions/19789102/why-is-bash-errexit-not-behaving-as-expected-in-function-calls?noredirect=1&lq=1 "Link"
