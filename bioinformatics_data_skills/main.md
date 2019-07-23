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
  - a: list all (including hidden ones)
  - h: human readable sizes (1k, 1M, ...)
  - `ls -ltrha` is commonly used
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

```Shell
program1 input > intermediate.txt && program2 intermediate.txt > results.txt
program1 input > intermediate.txt || echo "warninng: an error occured"
true; echo $?  # return 0
false; echo $? # return 1
true && echo "first cmd was suceess"  # return the message
true || "first command was suceess"   # return nothing
false && echo "first cmd was ok"      # return nothing
false || echo "first cmd was success" # return the message
```

**to run command sequentially use`;`**
This is irrespective of the exit status.

    false; true; false; echo "none of previous mattered"

## Command substitution `$(CMD)`
Allows to run a command inline and return the output as string and use this string in another command.

    echo "There are $(grep -c "^>" input.fasta) in my FASTA file"

This will make directories with current date. +%F is a standard and usefull format (2015-04-13). When directories are sorted by name using `ls -l`, directories are also sorted chronologically.

    mkdir results-$(date +%F)

# Chapter 4: Working with remote machines (`SSH`)
SSH: secure shell. You can connect to a host by `ssh hostname` or `ssh hostIP` address. If your local username differs with the remote username you should specify username in the `user@domain` format. The default port is 22. The `-p` flag defines the port number. The `-v` or `-vv` or `-vvv` flags define the verbose and it level.

    ssh biocluster.myuniversity.edu
    ssh -p 50432 cdarwin@biocluster.myuniversity.edu
    ssh 192.169.273

## SSH config file: store your frequent hosts
An SSH config file may be created to store frequently connected hosts. Hosts in this file can be used with SSH and two usefull programs: `rsync` and `scp` (Chapter 6).
You can easily creat the file at `~/.ssh/config`. Each entry takes the following form:

```
host bio-serv
     HostName 192.168.237.42
     User cdarwin
     Port 50432
```
Then you can connect to host using `ssh bio-serv`. In terminal you can ask for the host name by `hostname` commaand. If you have multiple account at server check the current account by `whoami`.
## SSH keys
An alternative to entering passwd in each session in to use a public/private key pair (just like the one we used with git that was created with gpg program). Generate a ssh key using `ssh-keygen`:

    ssh-keygen -b 2048

  This creates a private akey at `~/.ssh/id-rsa` and public key at `~/.ssh/id-rsa.pub`. To use passwd less authentication:
  1. first login to hist using passwd.
  2. change to ~/.ssh
  3. append the public key to remote `~./ssh/autherized_keys` by copying and pasting or using `ssh-copy-id
  4. login again. if will ask for key's passwd.
  5. `ssh-agent` which can be started with `eval ssh-agent` allows to use ssh keys without enteing it's password each time.
  6. `ssh-add` will add the keys to ssh-agent
  7. for the next login no passwd is reguired.
## Maintaining long-running jobs with `nohup` and `tmux`
In local machine, by closing terminal the proccess will be terminated by SIGHUP or hangup signal. The same is ture abot remote connections. When you get disconnected or network temporarily lost, the process will be lost.
**Using `nohup` local terminal:**
nohup will return the PID which is the only way to terminate the process in this case.

```Shell
nohup program1 input >output &
```

- **Using `Tmux` or GNU `Screen` for remote machines**
*Terminal multiplexers* like Tmux (well developed) or Screen (popular alternative), create a session with multiple windows. Sessions are stable to hangeups and connection loses. You need to just relogin to host (ssh) and reattach the session to the current terminal. `~/.tmux.conf` is the configuration file of Tmux. Download a copy from book's github and paste it in the relevant directory it adds nice features to Tmux.

- **working with `Tmux`:**
With Tmux, you can create multiple Sessions (one for each project), each session can have multiple windows open (e.g. one for shell, one for text documentation). All these windows are within Tmux. It is not the same as tabs and multiple windows in regular terminals. You can start Tmux by creating nex sessions either in local machine or in remote host (after ssh to the remote).

  tmux nex-session -s <SessionName>

- **Interacting with Tmux:** though shortcuts, by default (Control+B) + (release Control B) + press another key. We changes the Ctrl+b to CTRL+a in configuration file.
- **attaching detaching sessions:**
    detach: (Control+a) + d # detach session
    tmux list-sessions      # see the list of running  Sessions
    tmux attach or tmux attach-session
    tmux attach -t <SessionName>
- **create a windowL:** (CTRL+a) + c
- (CTRL+a) + ?: list of shortcuts
