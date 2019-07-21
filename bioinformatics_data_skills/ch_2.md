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
