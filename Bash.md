# Shell expansions
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

### String manupulations with `expr`
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

## `read`
By default, read considers a newline character as the end of a line, but this can be changed using the `-d` option.
After reading, the line is split into words according to the value of the special shell variable **IFS**, the internal field separator.
To preserve white space at the beginning or the end of a line, it's common to specify `IFS=` (with no value) immediately before the read command. After reading is completed, the IFS returns to its previous value. **By default, the IFS value is "space, tab, or newline".**

read assigns the first word it finds to name, the second word to name2, etc. If there are more words than names, all remaining words are assigned to the last name specified. If only a single name is specified, the entire line is assigned to that variable. If no name is specified, the input is stored in a variable named **REPLY**.

**Syntax:**
```shell
read [-ers] [-a array] [-d delim] [-i text] [-n nchars] [-N nchars]
     [-p prompt] [-t timeout] [-u fd] [name ...] [name2 ...]
```

**options:**

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

**Examples:**

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

The above commands are the "proper" way to use find and read together to process files. It's especially useful when you want to do something to a lot of files that have odd or unusual names. Let's take a look at specific parts of the above example:

`while IFS=`. IFS= (with nothing after the equals sign) sets the internal field separator to "no value". Spaces, tabs, and newlines are therefore considered part of a word, which preserves white space in the file names.
Note that IFS= comes after while, ensuring that IFS is altered only inside the while loop. For instance, it won't affect find.

`read -r`: Using the -r option is necessary to preserve any backslashes in the file names.

`-d $'\0'`: The -d option sets the newline delimiter. We're using NULL as the line delimiter because Linux file names can contain newlines, so we need to preserve them. (This sounds awful, but yes, it happens.)
However, a NULL can never be part of a Linux file name, so that's a reliable delimiter to use. Luckily, find can use NULL to delimit its results, rather than a newline, if the -print0 option is specified:

`< <(find . -print0)`: Here, find . -print0 creates a list of every file in and under . (the working directory) and delimit all file names with a NULL. When using -print0, all other arguments and options must precede it, so make sure it's the last part of the command.

Enclosing the find command in <( ... ) performs process substitution: the output of the command can be read like a file. In turn, this output is redirected to the while loop using the first "<".

For every iteration of the while loop, read reads one word (a single file name) and puts that value into the variable file, which we specified as the last argument of the read command.

When there are no more file names to read, read returns false, which triggers the end of the while loop, and the command sequence terminates.
