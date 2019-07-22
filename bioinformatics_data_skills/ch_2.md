# Chapter 2: setting up and managing a bioinformatics project

## shell wild cards and expansion
    - asterisk  matches zero or more characters (ignores hidden files)
    - ? matches one characters (ignore hidden files)
    - [A-Z]  alphanumeric range
    - [0-9]  alphanumeric range

**examples:**

    - ls zmays[AB]_R1.fastq
    - ls zmays*
    - ls zmays?_R1.fastq

## brace expansion `text-{1,2,3}`:
expand the comma separated values in curle braces. There should be o space between values. The -p argument let mkdir to create any subdirecory (here data) if needed. brace expansion will expand irrespective o whether file or directory exist. but wildcards expand to existing files.

       mkdir -p zmays/{data/seqs,analysis/scripts}

## Pandoc: convert between formats: *markdown to HTML*

    sudo apt-get install pandoc
    pandoc --from markdown --to html README.md > README.html
*→*→*→
# Chapter 3: Remedial Unix shell
You may try to use **zsh** which more advance and contain atucomplete that comes handy in bioinformatics.
## Streams and Redirection
Data streaming or text streaming: is a pylosophy in unix and allows us to handle large files without piting them all into memory. Data streaming in Unix contains:
- std in (file descriptor integer: 0)
- std out (file descriptor integer: 1)
- std err (file descriptor integer: 2)
### concatanating multiple files:
    cat file1.fasta file 2.fasta > out.fasta
### redirection:
      > std out. overrite existing content of a file
      >> std out. append to the end of a file
      2> std err. overrites
      2>> std err. append
      < std in
    std in: somethimes we use <. somethime we use pipe (|). some programs need a dash for indicating using std input and others takes an argument for this.
### a tip regarding `ls -lrt`*
  - l: listing
  - r reverse order
  - t: order by time
### redirectong both std out & std err: logfiles
file1 exist and file2 doesnot exists (producing error)
    ls -l file1 file2 >listing 2>listing.err
### redirecting to pseudodevice `/dev/null/`
Unwanted output can be redirected to `/dev/null`. Outputs redirected here will be disappered.
### using tail -f to monitor redirected std err
When redirecting both std out and std err, a long time running command may be finished with no message. To keep yourself updated with wahts going use either:
    tail stderr.txt: print the last 10 lines of the stderr file.
or use `tail -f` which follows the file being updated. lines added to the file will be printed. This can be canceled using `Control+c`, but the file continu being updated.
    tail -f stderrr.txt
## using pipes
There is an example for using pipe: a linux one-liner to take a fasta file and looks for non-nucleotide characters.

    grep -v "^>" file .fasta | grep --colour -i "[^ATCG]"

  - first remove header line
  - then print lines that doesnot match A, T, C or G
  - colour: print the matching in colour
  - i: ignore case (T is the same as t)
  - ^ inside [] means everything that is not one of the carachters in []
### Combining pipes and redirection
handling stderr and stdout of two processes

    program1 input.txt 2> program1.stderr | program2 2> program2.stderr

### Redirecting stderr to stdout `>2&1`
A use case of this approach is when you want to look for "error" both in std out and std out. pipes only works on std out not std err. First merge stderr and std output

    program1 2>&1 | grep "error"
### a tee joint in pipe
We occasionly need to write intermediate files to disk for debuging purposes or to have the output of long term running programs and also use pipe benefits. The tee program works like a plumber tee joint: the output of program1 both wrritten to intermediate.file and redirected as input to program2.

    program1 input.txt | tee intermediate.file | program2 >results.txt

## Managing and interacting with processes
Basics of manipulatidng processes.
### Background processes
By default shell runs commands in **foreground**. we can run a command in background and making shell available for running other commands by appending `&` to the end of the command. This with return a **process ID** or **PID**.

    program1 input > output.txt &

  **to see all background jobs:**
- `jobs`  without args the status of **all active jobs** are diplayed and gives a **job ID** which is different from PID.
- `jobs -l` adds PID to other displated info.

**to bring a job to foreground `fg`**
- `fg` bring the most recent process to foreground
- `fg %<num>` replace <num> with job ID from jobs output (e.g. `fg %1`)

### background processes and **hangeup**
when proccesses from a terminal are send to background and we close the terminal, it sends a `hangeup` signal or `SIGHUP` to all processes started from that terminal and kill the process. To prevent this we should use the tool `nohup` or run the command from within `Tmux`. Chaapter 4 for details.

### puting a foreground command to Background
we need to **suspend** or pause the proccess using `CTRL + z` and then put it to background using `bg` which is the same as `fg`.

    bg %<num>
### kill a foregrounded process using `CTRL+C`
The process is then non-recoverable. Other usefull process management commands `top; ps, kill`

## Exit status
**exit status 0**: command was successfully ran. **nonzero exit status:** command failed. Not all programs handle errors correctly. you should never trust your tools! It is possible that a command return an error and also a zero exit status.To cjeck the exit status of the last command use shell variable `$?`

    program1 input.txt > output.text
    echo $?
    0
**operators that implement exit status:**
 - `&&` will run the next command only if the previous command return a **zero exit** status (secessful)
 - `||` will run the next command onlt if the previous command return a **non-zero exit** status (failure)
 - `true` would return exit success
 - `false` would return exit fail

```
program1 input > intermediate.txt && program2 intermediate.txt > results.txt
program1 input > intermediate.txt || echo "warninng: an error occured"
true; echo $?  # return 0
false; echo $? # return 1
true && echo "first cmd was suceess"  # return the message
true || "first command was suceess"   # return nothing
false && echo "first cmd was ok"      # return nothing
false || echo "first cmd was success" # return the message
```

**to run command sequentially use `;`**
This is irrespective of the exit status.

    false; true; false; echo "none of previous mattered"
