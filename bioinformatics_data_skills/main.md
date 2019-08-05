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
- **create a window:** (CTRL+a) + c
- (CTRL+a) + ?: list of shortcuts

# Chapter-6: Bioinformatics data
## Downloading Data
### `wget`: downloading Data
- For quickly downloading files.
- works for both `ftp` and `http`.
- set username with `--user=` and passwd with `--ask-password`
- `-r` or `--recruisive`: downloading `recruisively`
  - `-l` or `--level`: default:5.
  - we should set ome limits.
  -`wget --accept "*.gtf --no-directories --recruisive --no-parent <url>"`
  `--no-parent`: prevent downloadinghigher directories
  `--accept` or `-A`: download specific filenames.

### `curl`
- For quickly downloading files.
- Capable of using more protocols: `sftp; SCP`.
- by default, prints to std out. Either redirect the output or use `-o <filename>`.
- If `-o` is specified without filename argument, the remote filename will be used.
- `-L` or `--location`: **Can follow page redirects**.
- is implemented in `RCurl` and `pycurl`

### `Rsync` and `Secure Copy` (SCP)
- `Rsynch` is appropriate for hevy-duty tasks. e.g. **synchronizing** entire large directory.
- When a copy in already exist or partially exists, Rsynch will download only **differences between file version** in a compressed form: its **faster**.
- Rsynch has an **Archive option** which preserves file attributes (owner, modifications, timestamps, permissions)), making it deall for **network backup**.
- `rsynch source destination`. Either dource or destination can be a remote host in a format `user@host:directory/to/files`
- `rsynch -avz -e ssh zea_mays/data/ 192.168.237.42:/home/data`
  - `-a`: archiving; `-v`: verbose; `-z`: compression;
  - `-e ssh`: we rae going to connect through ssh. if ssh host aliases were used, this could be replaced by host's aliase.
  - `data/` in source means copy contents, but `data` means copy entire directory (directory itself & its contents)
- rsynch transfers files if they **do not exist or changed**. Use `rsynch` again to check if worked fine.
- Use **exit status** in scripts to check for problems in transferring files.

**Secure Copy (scp)**:
It is used in cases where we need to sust quickly copy a file over ssh. We can use scp using `scp source destination`. Destination is the same as what was used for rsynch.
## Data integrity
Checksum is a compresseed summary of the data and will change if even a bit of data changes. Using `checksums` for evaluating data integrity:
1. To verify file transfered correctly and is not changes or corrupted.
2. To keep track of data versions.
3. Reproducibility: associate a particular analysis to a specific data with checksums.
**SHA and MD5 Checksums**
- `SHA-1` is newer but `md5` is more common.
- `shasum`program or `sha1sum` for SHA-1. `md5sum` for MD5 (in some servers `chsum` or `sum` programs).
- `shasum *.fastq >chechsum.sha` create checksum file for all fastq file.
- `shasum -c checksum.sha` to validate checksums. Returns a nonzero exit status if any file fail.
## Looking at differences between data: `diff`
- `diff -u gene-1.bed gene-2.bed`
- the `-u` option enables unified format for output wich gives more context relative to diff's default output.
- Hunks (i..e changed chunks) are seperated with `@@ .. .. @@`
- changes (- or +) are relative to the current file (i.e. ---).
- The output of diff can be redirected to a file, **patch file**. Patch files are instructions on how to update plain text files and can be used with `patch` unix program to apply changes.
- diff can be computationally expensive on large datasets.
## Compressing Data
- `gzip` and `bzip2` are two most common compression program.
- `gzip` is faster, `bzip2` compress more.
- `gzip` for most bioinformatics jobs, `bzip2` for long-term storages.
**Some usefull applications are:**
- `trimmer in.fastq.gz | gzip > out.fastq`
- `gzip in.fastq`: compress file  in place (replacing the uncompressed file)
- `gunzip in.fastq.gz`: uncompress file  in place (replacing the compressed file)
- `gzip -c in.fastq > infastq.gz`: compress file to std out
- `gunzip -c in.fastq.gz >in.fastq`: uncompress file to std out
- `gzip -c in2.fastq >> in.fastq.gz`: compressing and appending to an existing gz file. Files are **Concatanated** and are no longer seperated.
- `cat in.fastq in2.fastq | gzip >in.fastq.gz`: compressing multiple file together. Files are **Concatanated** and are no longer seperated.
- use `tar` for compressing multiple files and keep then seperated.
- `tar -cvf in.fastq.tar in1.fastq in2.fastq`
**An important advantage of gzip a nd bzip2 is that unix and bioinfo tools can work directly with compressed files.**
- `zcat` instead of `cat`.
- `zdiff` instead of `diff`
- `zgrep` instead of `grep`
- `zless` instead of `less`
- If the program connot handle compressed files, use `zcat in.fastq.gz | program`.

# Chapter 7: Unix Data Tools
## line seperator characters
In windows, line seperator is usually `\r\n`, where `\r` is **cariage return**. Some files may also be seperated by a signgle carriage return. In inux and Mac, line seperator is `\n`: **linefeed character**. CSV format usually use `\r\n` as line seperator as suggested by **RFC-4180 guidelines**.
## `head` and `tail`
- `head -n 10`: show the first 10 lines.
- `tail -n 10`: show the last 10 lines.
- `tail -n +2`: truncate the first row. Show from row number 2 to the end.
-`(head -n 3; tail -n 3) < file.bed`: show both head and tail of a file.
- to write a function for the above command and put it into ~/.bashrc and source it:
  - `i(){(head -n 3; tail -n 3) < "$1" | column -t;}` put it into `~/.bashrc`
  - `source ~/.bashrc` to load the function.
  - `i` stands for inspection.
  - `column -t` will print a tabular output.
  - see `man column`.
- `pipeline | head -n 5`: use `head` to peek at data rsulting from a pipeline:
  - `grep 'gene_id "ENSG00000025907"' Mus-muscu;us.gtf | head -n 1`
  - usefull for checking if each step of pipeline is working correctly.
  - When printing the required lines, `head` exits, shell catches this signal i.e. `SIGPIPE` and stops the entire pipe.
  - for example in `grep 'string' huge_file.txt | program1 | program2 | head -n 5`, after head print out five lines and exits, the grep and other programs wont continue searching and running.
  - A program may explicitly, catches and ignore this signal (not required in bioinfo works).
## `less`
Is a terminal pager and more recent than `more`. run it using `less file.fastq`.
Usefull shortcuts:
- `h`: getting help;
- `space`: go down a page; `b`: go up a page.
- `j or k`: forward and backward a line.
- `q`: quite.
- `g`: first line; `G` last line.
- `/ <pattern>` search downward for strings.
- `? <pattern>` search upward for strings.
**Using `less` for debugging pipelines:**
Just pipe the output of the command you want to debug to `less` and delete everything after.
**Using `less` to iteratively construct pipelines:**
Pipelines should be contructed stepwise. If the full pipeline is `step 1 input.txt | step2 | step3 > output.txt`, we shlould first evaluate the output of each step and then add the next step. This can be done with `less`:
1. `step1 input.txt | less`  inspect the results
2. `step1 input.txt | step2 | less`
3. `step1 input.txt | step2 | step3 | less`
A good fiture of pipe is that when the output of a program is piped to `less`, pipes blockes the programm when less has a full scrin of data (i.e. when pipe if full). This enables us to quickly evaluate the pipeline prrocessing large data.
## Plain text summary information with `wc`, `ls`, `awk`
- `wc` by default gives number of words, lines, characters
- `wc file1 file2 file3` works with multiple files
- `wc -l` only number of lines
- lines may differ with rows: when there are some empty line, wc would count them as lines, but the actual number of rows are different.
- `awk -F "\t" '{print NF; exit}' file.bed` Get the number of columns based on first row.
- `grep -v "^#"  file.gtf | awk -F "\t" '{print NF; exit}'` Get the number of columns by first excluding comment lines.
## `cut` working with columns
- By defalut, the delimiter is tab `\t`.
- `cut -f1` cut and retain the first column
- `cut -f1-3` cut columns 1 to 3
- `cut -f1,4,5` cut columns 1, 4 and 5.
- cannot reorder columns with cut.
- `cut -d, -f2,3` specify the dilimiter a comma.
## `column`
- By defalut, the delimiter is tab `\t`
- `-s` specifies delimiter
- `column -s"," -t file.csv`
- `column -t` treat the data as table.

## `GNU` or `BSD` flavors
Some unix tools like `grep cut sort` has two flavores: **GNU or BSD**. BSD is used in OSX and freeBSD, GNU is used in linux. GNU-based flavore of tools are under developement, have better documentations and are more robust and powerfull. GNU tools can be instlled with:
```Shell
sudo apt install linuxbrew-wrapper
brew install coreutils
```
## `grep`
- `grep -w "bioinfo" file.txt`: by default, grep includes partial matches (i.e. bioinformatics, bioinfo, bioinformatician). `-w` restricts matches to words (surronded by spaces). Therefore, bioinformatics will not be in the output.
- `--color==auto`: color matches.
- providing context for matches: `-A` lines after each match; `B`: lines before; `-C`: after and before;
  - `grep -B1 "ATCGA" file.fastq`: include header (one line before) the matches.
- grep uses **BRE** (POSIX basic regular expression) and supports **ERE** (extended regular expression) with `-E` option.
- `egrep` in many systems refers to extended grep (i.e. grep with -E option)
- `grep -E "(olfr413|olfr411)"`: use pipe with `-E` to match any of the patterns.
- `grep -o "olfr.*"` extract only the matching part not the entire line.
- `grep -E -o 'gene_id "\w+"' file.gtf | cut -f2 -d" " | sed 's/"//g' | sort | uniq > mm_gene_id.txt`: capture alll unique gene names in the last fileld of a gtf file.
- `grep -c`: count the number of occurance
- `grep -m 3`: gives the First 3 matched lines
- `grep -m 3 -v`: gives the First 3 unmatched lines
- `grep -L`: only prints name of files that do not have any match. The reverse in `grep -l`

**Options:**
- Matcher Selection:
  - `-E`: ERE (extended);
  - `-G`: BRE (basic): default
  - `-P`: powerfull
  - `-F`: fixed string
- Matching Control:
  - `-i`: ignore case; `-v`: invert selection; `-w`; word.
  - `-x`: matching the whole line (^(pattern)$)
  - `-f filename`: giving a file with a pattern in each line.
- General Output Control:
  - `L` & `-l`
  - `-o`
  - `-c`
  - `--color`
  - `-m`
  - `-n`: also print line number
## decoding plain text data with `hexdump`
Two commonly used character encoding schemes: **ASCII** and **UTF8**. ASCII uses 7 bits to store 128 different values. Modern systems use 8 bits (one byte) to store ASCII characters. ASCII doesnot have special characters and therefore is usefull for bioinformatics analysis. Most data downloaded from famous bioinformatic databases are in ASCII format. **UTF8** is a superset of ASCII (i.e. ascii with specila charater support). Files generated by hand (copy  & paste) may have hiden special characters (i.e. being UTF-8), and therefore giving errors during analysis.
- `file`: this command gives the infered encoding of a file.
- `hexdump -c filename`: return the heximal value of each character. Usefull for insecting file for non-ascii characters.
- `alias nonascii="LC_CTYPE=C grep --color='auto' -n -P '[\80-\xFF]'"`. Add this alias to `~/.bashrc`. this will search for non-ascii characters and print the line with line number.

## Sorting text files with `sort`
- Sort uses a **merge sort** algorithm with sort intermediate sorted files writed on disk and allows us to sort files larger than our memory size.
- By default sort is **unstable** which means that when two lines are identical in terms of all specified sorting keys (-k), they are sorted by whole line (last-resort comparison) and therefore the order of such lines may be different from the original file. To disble this feature, use `-s` option to make sort stable.
- Default sorting is to sort alphanumerically. Sort doesnot sort by numbers in text (chr2, chr10, chr11, chr22). To make sort use a clever alphanumeric algorithm use `-V` for all columns or `k1,1V` for a specific column.
- The locale specified by the environment affects sort order.  Set **LC_ALL=C** to get the traditional sort order that uses native byte values.
-`sort file1 file2 >file12.txt`; Concatanate sorting. This will first concatanate all lines and then sort them as they were from a single file.
- **KEYDEF**  is F[.C][OPTS][,F[.C][OPTS]]: for start,end position;
  - F is field number; .C is character position in the field.
  - `-k2,`: default end position is end of file. column 2 to last column as sorting key.
  - `-k2.2,2.9` sorting key is from column2-character2 to column2-character9
  - OPTS is one or more single-let‐ter ordering options  [bdfgiMhnRrV],  which  override  global  ordering options  for  that key.
- `sort -kstart,end`: specifying start and end column of each sorting key. use `-kstart,start` to specify a specific column.
- `sort -k1,1V k2,2nr`: sort first column clever alphanumerically and then sort by second column reverse numerically.
- `sort -t","`: specift delimiter.
- `sort -n`: sort all columns numerically.
- `sort -k1,1 -k2,2n -c; echo $?`: To validate if a file is sorted by first and second column. Zero exit status mean file is sorted.
- `sort -k1,1 -k4,4n -S2G file.gtf`: `-S` option specifies the fixed-sized memmory buffer. 2G: 2gigabytes; 2M; 2K; `-S 50%` 50% of memory; A larger buffer means less intermediate files  wrriten to disk and more speed.
- `sort -k1,1 -k4,4n --parallel 4`: use 4 cores; use it for extremely large files otherwise may slow down the process.
-`sort -u` or `--unique`: without -c, output only the first of an equal run.

## finding unique values with `uniq`
- `uniq` **does not** return unique values. It only removes consecutive duplicate lines and keeping the first instance.
- To find unique values across all lines: `sort filename | uniq`
- `sort | uniq -i`: case insensitive
- `sort | uniq -c filename`:  give counts as well as unique values.
- Summarizing columns of categorical data:`grep -v '^#' file.gtf | cur -f3,7 | sort | uniq -c`: uniq also works with multiple columns. This gives how many features in column 3 are on each strand (column7)
- count number of duplicated lines: `sort test.bed | uniq -d | wc -l`. `-d` option of uniq return only duplicates.

## `join`
- Both files should be first sorted on the column we want to join by.
- `join -1 <file-1-field> -2 <file-2-field> file1 file2`
- `join -1 1 -2 1 file1.sorted.bed file2.sorted.bed`
- `-a FILENUM`: print unpairable lines coming from file FILENUM, where FILENUM is 1 or 2, corresponding to FILE1 or FILE2. `join -1 1 -2 2 -a 1 file.1 file.2`: only includes unpaired lines from file 1; Use `-a 1 -a 2`: to include unpaired lines from both file (will mix lines from both files!!).
- `join -j 1 file1  file2` is equivalent to `join -1 1 -2 1 file1 file2`
- `-v FILENUM`: like -a FILENUM, but suppress joined output lines.
- `join -j1 -v1 file1 file2`: return only unpaired lines of first file.
- `join -v1 -v2`: unpaired lines from both files
-  dealing with unpaired lines:
  - `join -a 1 -a 2 file1 file2` prints all lines (common and unpaired) from both files.  Without '-o auto' it is not easy to discern which fields originate from which file.
  - `join -a 1 a 2 -o auto -e X file1 file2` is usefull in dealing with unpaired lines;
- `-i`: --ignore-case
- `-t CHAR`: use CHAR as input and output field separator
- Either FILE1 or FILE2 (but not both) can be '-'
- `-o FIELD-LIST`: construct each output line according to the format in FIELD-LIST.  Each element in FIELD-LIST is either the single character '0' or has the form M.N where the file number, M, is '1' or '2' and N is a positive field number. A field specification of '0' denotes the join field. The elements in FIELD-LIST are separated by commas or blanks. `join -o 1.2,2.2`
- recommended sorting option is `sort -k 1b,1`
- To avoid any locale-related issues, it is recommended to use the 'C'
locale for both commands:

      LC_ALL=C sort -k 1b,1 file1 > file1.sorted
      LC_ALL=C sort -k 1b,1 file2 > file2.sorted
      LC_ALL=C join file1.sorted file2.sorted > file3

- Both 'sort' and 'join' operate of whitespace-delimited fields.  To specify a different delimiter, use '-t' in _both_.
- To specify a tab (ASCII 0x09) character instead of whitespace, use `-t$'\t'` in both.
- `--header`: first line as header.
- To sort a file with a header line, use GNU 'sed -u'. The following example sort the files but keeps the first line of each file in place:

      $ ( sed -u 1q ; sort -k2b,2 ) < file1 > file1.sorted
      $ ( sed -u 1q ; sort -k2b,2 ) < file2 > file2.sorted
      $ join --header -o auto -e NA -a1 -a2 file1.sorted file2.sorted > file3

## Union, Intersection and Difference of files: 'sort', 'uniq' and 'join'
`sort` without `-k` and `join -t''` both consider entire lines as the key. Therefore, All examples below operate on entire lines and not on specific fields.

### For sorted files

    join -t'' -a1 -a2 file1 file2  # Union of sorted files
    join -t'' file1 file2          # Intersection of sorted files
    join -t'' -v2 file1 file2      # Difference of sorted files
    join -t'' -v1 -v2 file1 file2  # Symmetric Difference of sorted files

In each of above examples, if `-t` is set to the seperator, and joining field (i.e `-1` & `-2` or `-j`) is given, then ***union, intersection, difference and symetric difference** would be based on a joining field but the whole line (like above).

    # Union of sorted files (common rows + unjoined in file1 and file2)
    join -j1 -a1 -a2 file1 file2
    join -j1 -a1 -a2 -o auto -e NA file1 file2
    # Intersection of sorted files (only joined lines from both files)
    join -j1 file1 file2
    # Difference of sorted files (unjoined rows in file2)
    join -j1 -v2 file1 file2
    # Symmetric Difference of sorted files (unjoined rows of both files)
    join -j1 -v1 -v2 file1 file2
    join -j1 -v1 -v2 -o auto -e NA file1 file2

# For unsorted Files

    # Union of unsorted files
    sort -u file1 file2

    # Intersection of unsorted files
    #(work if only each file does not contain duplicated line)
    sort file1 file2 | uniq -d

    # Difference of unsorted files
    # only lines of file2 that are not in file1
    sort file1 file1 file2 | uniq

    # Difference of unsorted files
    # only lines of file1 that are not in file2
    sort file1 file2 file2 | uniq

    # Symmetric Difference of unsorted files
    sort file1 file2 | uniq -u
