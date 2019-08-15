# Looping through filenames `for f in $(ls *.mp3)`
**Wrong loops:**

```Shell
for f in $(ls *.mp3); do    # Wrong!
    some command $f         # Wrong!
done

for f in $(ls)              # Wrong!
for f in `ls`               # Wrong!

for f in $(find . -type f)  # Wrong!
for f in `find . -type f`   # Wrong!

files=($(find . -type f))   # Wrong!
for f in ${files[@]}        # Wrong!
```
- Yes, it would be great if you could just treat the output of ls or find as a list of filenames and iterate over it. But you **cannot**.
- problems with this approach:
  - If a filename contains whitespace, it undergoes **WordSplitting**.
  - If a filename contains **glob characters**, it undergoes filename expansion ("globbing"). If ls produces any output containing a * character, the word containing it will become recognized as a pattern and substituted with a list of all filenames that match it.
  - If the **command substitution** returns multiple filenames, there is no way to tell where the first one ends and the second one begins.
  - The ls utility may mangle filenames. may randomly decide to replace certain characters in a filename with "?", or simply not print them at all. **Never try to parse the output of ls**.
  - The **CommandSubstitution strips all trailing newline** characters from its output.

**The right way:**
1. If you don't need recursion, you can **use a simple glob**. Instead of ls:

```Shell
for file in ./*.mp3; do    # Better! and...
    some command "$file"   # ...always double-quote expansions!
done
```
Because globbing is the very last expansion step, each match of the ./*.mp3 pattern correctly expands to a separate word, and isn't subject to the effects of an unquoted expansion.

What happens if there are no `*.mp3-files` in the current directory? Then the for loop is executed once, with `i="./*.mp3"`, which is not the expected behavior! The workaround is to **test whether there is a matching file**:

```shell
# POSIX
for file in ./*.mp3; do
    [ -e "$file" ] || continue
    some command "$file"
done
```
Another solution is to use Bash's `shopt -s nullglob` feature (use carefully).

If you need **recursion**:
2. the standard solution is `find`.

```Shell
find . -type f -name '*.mp3' -exec some command {} \;

# Or, if the command accepts multiple input filenames:

find . -type f -name '*.mp3' -exec some command {} +
```

3. or use:

```Shell
while IFS= read -r -d '' file; do
  some command "$file"
done < <(find . -type f -name '*.mp3' -print0)
```
The advantage here is that "some command" (indeed, the entire while loop body) is executed in the current shell. You can set variables and have them persist after the loop ends.

4. `globstar` in BASH >4
globstar, which permits a glob to be expanded recursively
```Shell
shopt -s globstar
for file in ./**/*.mp3; do
  some command "$file"
done
```

5. **if file names contain space:** temporarily set IFS to just tabs and newlines (or better, just newlines). Remember to restore the default IFS!

```Shell
  OFS="$IFS" # store original field separator to restore later
  IFS="\n" # set new IFS
  all_files=($(cut -d"  " -f1 files.txt))
  for file in "${all_files[@]}"; do echo $file; done;
    file_A
    file_B
    file_C
    file D
  IFS="$OFS"  # or you can "unset IFS"
  ```
# Filenames with leading dashes
Filenames with leading dashes can cause many problems. Globs like *.mp3 are sorted into an expanded list (according to your current locale), and - sorts before letters in most locales. The list is then passed to some command, which may incorrectly interpret the -filename as an option. There are two major solutions to this.
1. One solution is to insert `--` between the command (like cp) and its arguments. That tells it to stop scanning for options, and all is well: `cp -- "$file" "$target"`
2. Another option is to ensure that your filenames always begin with a directory by using relative or absolute pathnames.

```Shell
for i in ./*.mp3; do
    cp "$i" /target
    ...
done
```

# Quoting problems
Another quoting problem is when you have a string which you'd like to break up into separate words but you'd also like to honor any quoted substrings in the string.

```Shell
a="hello \"there big\" world"
echo $a  # hello "there big" world
```

and suppose you'd like to break that into its three parts:

- hello
- there big
- world

If you run this:

```shell
a="hello \"there big\" world"

for i in $a
do
    echo $i
done
```

it produces

```Shell
$ bash t4.sh
hello
"there
big"
world
```
The trick is to use `set`:

```Shell
a="hello \"there big\" world"
set -- $a

for i in "$@"
do
    echo $i
done
```

but it doesnot work (produces the same result as before). you have to `eval` the `set` statement:

```Shell
a="hello \"there big\" world"
eval set -- $a

for i in "$@"
do
    echo $i
done
```
