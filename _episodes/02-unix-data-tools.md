---
title: "Unix Data Tools"
output: html_document
---



# Unix Data Tools

## Introduction

## Dataset
### Downloading Data with wget and curl
Two common command-line programs for downloading data from the Web are `wget` and `curl`. 
Depending on your system, these may not be already installed; you’ll have to install them 
with a package manager (e.g., Homebrew or apt-get). While `curl` and `wget` are similar in 
basic functionality, their relative strengths are different:  
* `wget` is useful for quickly downloading a file from the command line
* `curl` is used in scripts that send data to a system or recieve data through a variate of protocols, including SFTP and SCP.

To get a file with data for human chromosome 22 from the GRCh37 (also known as hg19) assembly version:


```bash
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr22.fa.gz
```

Notice that the link to chromosome 22 begins with “http” (short for Hyper‐Text Transfer Protocol). wget can also handle FTP links (which start with “ftp,” short for File Transfer Protocol). In general, FTP is preferable to HTTP for large files (and is often recommended by websites like the UCSC Genome Browser). 

To download chromosome 22 with curl, we’d use:

```bash
curl http://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr22.fa.gz > chr22.fa.gz
```

```
##   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
##                                  Dload  Upload   Total   Spent    Left  Speed
##   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0  0 10.8M    0 41607    0     0   171k      0  0:01:04 --:--:--  0:01:04  170k 83 10.8M   83 9183k    0     0  7464k      0  0:00:01  0:00:01 --:--:-- 7460k100 10.8M  100 10.8M    0     0  8429k      0  0:00:01  0:00:01 --:--:-- 8425k
```

### Data Integrity
Data we download into our project directory is the starting point of all future analyses and conclusions. 
Although it may seem improbable, the risk of data corruption during transfers is a concern when 
transferring large datasets. So it's important to explicitly check the transferred data’s integrity 
with check‐sums. Checksums are very compressed summaries of data, computed in a way that even if just 
one bit of the data is changed, the checksum will be different. Data integrity checks are also helpful 
in keeping track of data versions. The checksums would differ if the data changed even the tiniest bit, 
so we can use them to calculate the version of the data. Checksums also facilitate reproducibility, 
as we can link a particular analysis and set of results to an exact version of data summarized by the 
data’s checksum value.

### SHA and MD5 Checksums
The two most common checksum algorithms for ensuring data integrity are MD5 and SHA-1. 
SHA-1 is newer and generally preferred. However, MD5 is more common; it’s likely to be 
what you encounter if a server has precomputed checksums on a set of files. To create 
checksums using SHA-1 we can pass arbitrary strings to the program shasum (on some systems, 
it’s sha1sum) through standard in:


```bash
echo "bioinformatics is fun" | shasum
echo "bioinformatic is fun" | shasum
```

```
## f9b70d0d1b0a55263f1b012adab6abf572e3030b  -
## e7f33eedcfdc9aef8a9b4fec07e58f0cf292aa67  -
```

Checksums are reported in hexadecimal format, where each digit can be one of 16 characters: 
digits 0 through 9, and the letters a, b, c, d, e, and f. The trailing dash indicates this 
is the SHA-1 checksum of input from standard in. 

We can also use checksums with file input:

```bash
shasum chr22.fa.gz
```

```
## d012edd46f50d674380460d3b4e91f450688e756  chr22.fa.gz
```

When downloading many files, it can get rather tedious to check each checksum individually. 
The program `shasum` has a convenient solution: it can create and validate against a file 
containing the checksums of files. We can create a SHA-1 checksum file for all FASTQ files 
in the data/ directory as follows:


```bash
shasum data/*fastq > fastq_checksums.sha
```

Then, we can use shasum’s check option (-c) to validate that these files match the original versions:


```bash
shasum -c fastq_checksums.sha
```

```
## shasum: fastq_checksums.sha: No such file or directory
```

The program `md5sum` (or `md5` on OS X) calculates MD5 hashes and is similar in operation to `shasum`. 
However, note that on OS X, the md5 command doesn’t have the -c option, so you’ll need to install the 
GNU version for this option.

### Keeping records

When we download data from the internet, it's important to include checksum numbers in a README.md file
for reproducibility, e.g.,

>    ## Genome and Annotation Data
>    Mouse (*Mus musculus*) reference genome version GRCm38 (Ensembl
>    release 74) was downloaded on Sat Feb 22 21:24:42 PST 2014, using:
>        wget ftp://ftp.ensembl.org/pub/release-74/fasta/mus_musculus/dna/Mus_musculus.GRCm38.74.dna.toplevel.fa.gz
>    Gene annotation data (also Ensembl release 74) was downloaded from Ensembl on
>    Sat Feb 22 23:30:27 PST 2014, using:
>        wget ftp://ftp.ensembl.org/pub/release-74/fasta/mus_musculus/dna/Mus_musculus.GRCm38.74.gtf.gz
>    ## SHA-1 Sums
>     - `Mus_musculus.GRCm38.74.dna.toplevel.fa.gz`: 01c868e22a9815c[...]c2154c20ae7899c5f
>     - `Mus_musculus.GRCm38.74.gtf.gz`: cf5bb5f8bda2803[...]708bff59cb575e379

Although this isn’t a lot of documentation, this is infinitely better than not documenting how data was 
acquired. As this example demonstrates, it takes very little effort to properly track the data that 
enters your project, and thereby ensure reproducibility. The most important step in documenting your work 
is that you’re consistent and make it a habit.

## Inspecting and Manipulating Text Data with Unix Tools
In this lesson, we'll learn how to use core Unix tools to manipulate and explore plain-text 
data formats. Many formats in bioinformatics are simple tabular plain-text files delimited 
by a character. The most common tabular plain-text file format used in bioinformatics is 
tab-delimited. Bioinformatics evolved to favor tab-delimited formats because of the 
convenience of working with these files using Unix tools. Tab-delimited file formats are also 
simple to parse with scripting languages like Python and Perl, and easy to load into R.

> ## Tabular Plain-Text Data Formats
> Tabular plain-text data formats are used extensively in computing. The basic format is incredibly 
> simple: each row (also known as a record) is kept on its own line, and each column (also known as 
> a field) is separated by some delimiter. There are three flavors you will encounter: tab-delimited, 
> comma-separated, and variable space-delimited. Of these three formats, tab-delimited is the most 
> commonly used in bioinformatics. File formats such as BED, GTF/GFF, SAM, tabular BLAST output, and 
> VCF are all examples of tab-delimited files. Columns of a tab-delimited file are separated by a 
> single tab character (which has the escape code \t). A common convention (but not a standard) is to 
> include metadata on the first few lines of a tab-delimited file. These metadata lines begin with # 
> to differentiate them from the tabular data records. Because tab-delimited files use a tab to delimit 
> columns, tabs in data are not allowed. Comma-separated values (CSV) is another common format. CSV is 
> similar to tab-delimited, except the delimiter is a comma character. Lastly, there are space-delimited 
> formats. A few stubborn bioinformatics programs use a variable number of spaces to separate columns. 
> In general, tab-delimited for‐ mats and CSV are better choices than space-delimited formats because 
> it’s quite com‐ mon to encounter data containing spaces. Despite the simplicity of tabular data formats, 
> there’s one major common headache: how lines are separated. Linux and OS X use a single linefeed character 
> (with the escape code \n) to separate lines, while Windows uses a DOS-style line separator of a carriage 
> return and a linefeed character (\r\n). CSV files generally use this DOS-style too, as this is specified 
> in the CSV specification RFC-4180 (which in practice is loosely followed). Occasionally, you might encounter 
> files separated by only carriage returns, too.
{: .callout}

In this lesson, we’ll work with very simple genomic feature formats: BED (three-column) and GTF files. 
These file formats store the positions of features such as genes, exons, and variants in tab-delimited 
format. Don’t worry too much about the specifics of these formats; we’ll cover them later. Our goal here 
is to develop the skills to freely manipulate plain-text files or streams using Unix data tools. We’ll 
learn each tool separately, and cumulatively work up to more advanced pipelines and programs.

### Inspecting Data with `head` and `tail`
Many files we encounter in bioinformatics are much too long to inspect with cat: running cat on a file 
a million lines long would quickly fill your shell with text scrolling far too fast to make sense of. 
A better option is to take a look at the top of a file with `head`. First, download the file 
Mus_musculus.GRCm38.75_chr1.bed:


```bash
curl -O https://raw.githubusercontent.com/Data-Skills/bds-files/master/chapter-07-unix-data-tools/Mus_musculus.GRCm38.75_chr1.bed
curl -O https://raw.githubusercontent.com/Data-Skills/bds-files/master/chapter-07-unix-data-tools/Mus_musculus.GRCm38.75_chr1.gtf
```

```
##   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
##                                  Dload  Upload   Total   Spent    Left  Speed
##   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0100 1658k  100 1658k    0     0  1708k      0 --:--:-- --:--:-- --:--:-- 1708k
##   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
##                                  Dload  Upload   Total   Spent    Left  Speed
##   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0  0 25.3M    0  1929    0     0   2722      0  2:42:54 --:--:--  2:42:54  2720100 25.3M  100 25.3M    0     0  21.9M      0  0:00:01  0:00:01 --:--:-- 21.9M
```

Now look at it with `head`:


```bash
head -n 3 Mus_musculus.GRCm38.75_chr1.bed
```

```
## 1	3054233	3054733
## 1	3054233	3054733
## 1	3054233	3054733
```

Notice the -n argument; it controls how many lines to display (the default is 10).
`head` allows you to quickly inspect a file to see if a column header exists, how many 
columns there are, what delimiter is being used, some sample rows, and so on.
`head` has a related command designed to look at the end, or `tail` of a file:


```bash
tail -n 3 Mus_musculus.GRCm38.75_chr1.bed
```

```
## 1	195240910	195241007
## 1	195240910	195241007
## 1	195240910	195241007
```

We can also use tail to remove the header of a file. Normally the -n argument specifies how many 
of the last lines of a file to include, but if -n is given a number x preceded with a + sign (e.g., +x), 
tail will start from the xth line. So to chop off a header, we start from the second line with -n +2. 
Here, we’ll use the command seq to generate a file of 3 numbers, and chop of the first line:


```bash
seq 3 > nums.txt
cat nums.txt
echo "---"
tail -n +2 nums.txt
```

```
## 1
## 2
## 3
## ---
## 2
## 3
```

Sometimes it’s useful to see both the beginning and end of a file. Foor example, if we have a 
sorted BED file and we want to see the positions of the first feature and last feature. 
We can do this using:


```bash
(head -n 2; tail -n 2) < Mus_musculus.GRCm38.75_chr1.bed
```

```
## 1	3054233	3054733
## 1	3054233	3054733
## 1	195240910	195241007
## 1	195240910	195241007
```

This is a useful trick, but it’s a bit long to type. To keep it handy, we can create a short‐cut 
in your shell configuration file, which is either ~/.bashrc or ~/.profile:


```bash
# inspect the first and last 3 lines of a file
i() { (head -n 2; tail -n 2) < "$1" | column -t}
```

Then, either run source on your shell configuration file, or start a new terminal ses‐
sion and ensure this works. Then we can use i (for inspect) as a normal command.

`head` is also useful for taking a peek at data resulting from a Unix pipeline. 
For example, suppose we want to grep the Mus_musculus.GRCm38.75_chr1.gtf file for 
rows containing the string gene_id "ENSMUSG00000025907". We can pipe the standard out of grep 
directly to head to to see if everything looks correct:


```bash
grep 'gene_id "ENSMUSG00000025907"' Mus_musculus.GRCm38.75_chr1.gtf | head -n 1
```

```
## 1	protein_coding	gene	6206197	6276648	.	+	.	gene_id "ENSMUSG00000025907"; gene_name "Rb1cc1"; gene_source "ensembl_havana"; gene_biotype "protein_coding";
```

After printing the first few rows of your data to ensure your pipeline is working prop‐ erly, the head process exits. This is an important feature that helps ensure your pipes don’t needlessly keep processing data. When head exits, your shell catches this and stops the entire pipe, including the grep process too. Under the hood, your shell sends a signal to other programs in the pipe called SIGPIPE—much like the signal that’s sent when you press Control-c (that signal is SIGINT). When building complex pipelines that process large amounts of data, this is extremely important. It means that in a pipeline like:


```bash
grep "some_string" huge_file.txt | program1 | program2 | head -n 5
```

grep won’t continue searching huge_file.txt, and program1 and program2 don’t continue processing 
input after head outputs 5 lines and exits. 

### `less`
`less` is a terminal pager, a program that allows us to view large amounts of 
text in our terminals by scrolling through long files and standard output a 
screen at a time. Once we start less, it will stay open until we quit it. To 
quit it, press `q`. Some other commands are listed in the table below:

![](./images/table7.1.png)

Let’s review an example—in this chapter’s directory in the book’s GitHub repository, 
there’s a file called contaminated.fastq. Let’s look at this with less: 


```bash
less ./data/contaminated.fastq
```

```
## ./data/contaminated.fastq: No such file or directory
```

One of the most useful features of less is that it allows you to search text and highlights matches. 
For example, let’s use less to get a quick sense of whether there are 3’ adapter contaminants in 
the contaminated.fastq file. In this case, we’ll look for AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC 
(a known adapter from the Illumina Tru‐ Seq® kit1). Instead of trying to find the complete sequence, 
let’s search for the first 11 bases, AGATCGGAAGA. First, we open contaminated.fastq in less, and then 
press / and enter AGATCGG. 

Q: Are the sequeces contaminated?

`less` is also extremely useful in debugging our command-line pipelines. One of the great beauties 
of the Unix pipe is that it’s easy to debug at any point—just pipe the output of the command you want 
to debug to less and delete everything after. When you run the pipe, less will capture the output of 
the last command and pause so you can inspect it.

`less` is also crucial when iteratively building up a pipeline. Suppose we have an imaginary pipeline 
that involves three programs, step1, step2, and step3. Our finished pipeline will look like `step1 
input.txt` | `step2` | `step3` > output.txt. However, we want to build this up in pieces, running `step1 
input.txt` first and checking its output, then adding in `step2` and checking that output, and so forth.
The natural way to do this is with less:


```bash
step1 input.txt | less # inspect output in less 
step1 input.txt | step2 | less
step1 input.txt | step2 | step3 | less
```

A useful behavior of pipes is that the execution of a program with output piped to less will be paused 
when less has a full screen of data. This is due to how pipes block programs from writing to a pipe when 
the pipe is full. When you pipe a program’s output to less and inspect it, less stops reading input 
from the pipe. Soon, the pipe becomes full and blocks the program putting data into the pipe from continuing. 
The result is that we can throw less after a complex pipe processing large data and not worry about wasting 
computing power—the pipe will block and we can spend as much time as needed to inspect the output.

### Plain-Text Data Summary Information with `wc`, `ls`, and `awk`
In addition to peeking at a file with `head`, `tail`, or `less`, we may want other bits of summary 
information about a plain-text data file like the number of rows or columns. With plain-text data 
formats like tab-delimited and CSV files, the number of rows is usually the number of lines. We can 
retrieve this with the program `wc` (for word count):


```bash
wc ./data/Mus_musculus.GRCm38.75_chr1.bed
```

```
## wc: ./data/Mus_musculus.GRCm38.75_chr1.bed: open: No such file or directory
```

By default, wc outputs the number of words, lines, and characters of the supplied file. Often, 
we only care about the number of lines. We can use option -l to just return the number of lines:


```bash
wc -l ./data/Mus_musculus.GRCm38.75_chr1.bed
```

```
## wc: ./data/Mus_musculus.GRCm38.75_chr1.bed: open: No such file or directory
```

Another bit of information we usually want about a file is its size. The easiest way to do this is 
with our old Unix friend, `ls`, with the -lh option (l for long, h for human-readable:


```bash
ls -lh ./data/Mus_musculus.GRCm38.75_chr1.bed
```

```
## ls: ./data/Mus_musculus.GRCm38.75_chr1.bed: No such file or directory
```

Here, “M” indicates megabytes; if a file is gigabytes in size, ls -lh will output results in gigabytes, “G.”

There’s one other bit of information we often want about a file: how many columns it contains. 
We could always manually count the number of columns of the first row with head -n 1, but a far 
easier way is to use `awk`. `Awk` is an easy, small programming language great at working with 
text data like TSV and CSV files. We’ll see more of `awk later, but let’s use an awk one-liner 
to return how many fields a file contains:


```bash
awk -F "\t" '{print NF; exit}' ./data/Mus_musculus.GRCm38.75_chr1.bed
```

```
## awk: can't open file ./data/Mus_musculus.GRCm38.75_chr1.bed
##  source line number 1
```

`awk` was designed for tabular plain-text data processing, and consequently has a built-in variable NF 
set to the number of fields of the current dataset. This simple awk oneliner simply prints the number 
of fields of the first row of the Mus_musculus.GRCm38.75_chr1.bed file, and then exits. By default, `awk` 
treats white‐space (tabs and spaces) as the field separator, but we could change this to just tabs by 
setting the -F argument of `awk`.

Finding how many columns there are in Mus_musculus.GRCm38.75_chr1.gtf is a bit trickier because this file 
has a series of comments in the beginning (marked with #) that contain helpful metadata like the genome 
build, version, date, and accession number. One way to fix this is with a tail trick we saw earlier:


```bash
tail -n +6 ./data/Mus_musculus.GRCm38.75_chr1.gtf | head -n 1;
echo "---"
tail -n +6 ./data/Mus_musculus.GRCm38.75_chr1.gtf | awk -F "\t" '{print NF; exit}'
```

```
## tail: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
## ---
## tail: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

While removing a comment header block at the beginning of a file with tail does work, 
it’s not very elegant and has weaknesses as an engineering solution. As you become 
more familiar with computing, you’ll recognize a solution like this as brittle. 
A better solution would be to simply exclude all lines that match a comment line pattern. 
Using the program grep (which we’ll talk more about it later), we can easily exclude lines 
that begin with “#”:


```bash
grep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf | awk -F "\t" '{print NF; exit}'
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

### Working with Column Data with cut and Columns

When working with plain-text tabular data formats like tab-delimited and CSV files, 
we often need to extract specific columns from the original file or stream. For example, 
suppose we wanted to extract only the start positions (the second column) of the 
Mus_musculus.GRCm38.75_chr1.bed file. The simplest way to do this is with `cut`. `Cut` 
cuts out specified columns (also known as fields) from a text file. By default, cut treats 
tabs as the delimiters, so to extract the second column we use:


```bash
cut -f 2 ./data/Mus_musculus.GRCm38.75_chr1.bed | head -n 3
```

```
## cut: ./data/Mus_musculus.GRCm38.75_chr1.bed: No such file or directory
```

The -f argument is how we specify which columns to keep. The argument -f also allows us to specify 
ranges of columns (e.g., -f 3-8) and sets of columns (e.g., -f 3,5,8). Note that it’s not possible 
to reorder columns using using cut (e.g., -f 6,5,4,3 will not work, unfortunately). To reorder columns, 
you’ll need to use awk, which is discussed later.

Using cut, we can convert our GTF for Mus_musculus.GRCm38.75_chr1.gtf to a three-column tab-delimited 
file of genomic ranges (e.g., chromosome, start, and end position). We’ll chop off the metadata rows 
using the grep command covered earlier, and then use cut to extract the first, fourth, and fifth columns 
(chromosome, start, end):


```bash
grep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f1,4,5 | head -n 3
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Note that although our three-column file of genomic positions looks like a BED-formatted file, it’s not 
due to subtle differences in genomic range formats. We’ll learn more about this later.

cut also allows us to specify the column delimiter character. So, if we were to come across a CSV file 
containing chromosome names, start positions, and end positions, we could select columns from it, too:


```bash
cut -d, -f2,3 ./data/Mus_musculus.GRCm38.75_chr1_bed.csv | head -n 3
```

```
## cut: ./data/Mus_musculus.GRCm38.75_chr1_bed.csv: No such file or directory
```

### Formatting Tabular Data with column
As you may have noticed when working with tab-delimited files, it’s not always easy to see which 
elements belong to a particular column. For example:


```bash
grep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f1-8 | head -n 3
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

While tabs are a terrific delimiter in plain-text data files, our variable width data leads our 
columns to not stack up well. There’s a fix for this in Unix: program `column -t` (the -t option 
tells column to treat data as a table). `column -t` produces neat columns that are much easier to read:


```bash
grep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f1-8 | column -t | head -n 3
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Note that you should only use `columnt -t` to visualize data in the terminal, not to reformat data 
to write to a file. Tab-delimited data is preferable to data delimited by a variable number of spaces, 
since it’s easier for programs to parse. Like cut, column’s default delimiter is the tab character (\t). 
We can specify a different delimiter with the -s option. 

`column` illustrates an important point about how we should treat data: there’s no reason to make data 
formats attractive at the expense of readable by programs. This relates to the general recommendation: 
__“write code for humans, write data for computers”__. In general, it’s easier to make computer-readable 
data attractive to humans than it is to make data in a human-friendly format readable to a computer.

### The All-Powerful Grep
`grep` finds a pattern (fixed string or regular expression) in a file and is faster than 
any other searches you can do in other programs/languages. `grep` requires two arguments: 
the pattern (the string or basic regular expression you want to search for), and the file 
(or files) to search for it in. As a very simple example, let’s use grep to find a gene, 
“Olfr418-ps1,” in the file Mus_musculus.GRCm38.75_chr1_genes.txt (which contains all Ensembl 
gene identifiers and gene names for all protein-coding genes on chromosome 1):


```bash
grep "Olfr418-ps1" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

The quotes around the pattern aren’t required, but it’s safest to use quotes so our shells won’t 
try to interpret any symbols. grep returns any lines that match the pattern.

One useful option when using grep is --color=auto. This option enables terminal colors, so the matching part of the pattern is colored in your terminal.


```bash
grep --color=auto "Olfr" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt | head -n 5
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

> ## GNU, BSD, and the Flavors of Grep
> Up until now, we’ve glossed over a very important detail: there are different implementations 
> of Unix tools. Tools like grep, cut, and sort come from one of two flavors: BSD utils and GNU coreutils. 
> Both of these implementations contain all standard Unix tools we use in this chapter, but their features 
> may slightly differ from each other. BSD’s tools are found on Max OS X and other _Berkeley Software 
> Distribution_-derived operating systems like FreeBSD. GNU’s coreutils are the standard set of tools found 
> on Linux systems. It’s important to know which implementation you’re using (this is easy to tell by reading 
> the man page). If you’re using Mac OS X and would like to use GNU coreutils, you can install these through 
> Homebrew with brew install coreutils. Each program will install with the prefix “g” (e.g., cut would be 
> aliased to gcut), so as to not interfere with the system’s default tools.
> Unlike BSD’s utils, GNU’s coreutils are still actively developed. GNU’s coreutils also have many more 
> features and extensions than BSD’s utils, some of which we use in this chapter. In general, you should use 
> GNU’s coreutils over BSD utils, as the documen‐tation is more thorough and the GNU extensions are helpful.
>
{: .callout}

Earlier, we saw how grep could be used to only return lines that do not match the specified pattern—this is 
how we excluded the commented lines from our GTF file. The option we used was -v, for invert. For example, 
suppose you wanted a list of all genes that contain “Olfr,” except “Olfr1413.” Using -v and chaining together 
to calls to grep with pipes, we could use:


```bash
grep Olfr ./data/Mus_musculus.GRCm38.75_chr1_genes.txt | grep -v Olfr1413
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

But beware! Partially matching may bite us here: while we wanted to exclude “Olfr1413,” this command would 
also exclude genes like “Olfr1413a” and “Olfr14130.” To get around this, use -w to match entire words 
(strings of characters surrounded by whitespace). By constraining our matches to be words, we’re using a more 
restrictive pattern. In general, our patterns should always be as restrictive as possible to avoid unintentional 
matches caused by partial matching.

grep’s default output often doesn’t give us enough context of a match when we need to inspect results by eye; 
only the matching line is printed to standard output. There are three useful options to get around this: 
context before (-B), context after (-A), and context before and after (-C). Each of these arguments takes 
how many lines of context to provide:

grep also supports a flavor of regular expression called POSIX Basic Regular Expressions (BRE). If you’re 
familiar with the regular expressions in Perl or Python, you’ll notice that grep’s regular expressions 
aren’t quite as powerful as the ones in these lan‐ guages. Still, for many simple applications they work 
quite well. For example, if we wanted to find the Ensembl gene identifiers for both “Olfr1413” and “Olfr1411,” 
we could use:


```bash
grep "Olfr141[13]" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

Here, we’re using a shared prefix between these two gene names, and allowing the last single character 
to be either “1” or “3”. However, this approach is less useful if we have more divergent patterns to search for. 
For example, constructing a BRE pattern to match both “Olfr218” and “Olfr1416” would be complex and error prone. 
For tasks like these, it’s far easier to use grep’s support for POSIX Extended Regular Expressions (ERE). `grep` 
allows us to turn on ERE with the -E option (which on many systems is aliased to egrep). EREs allow us to use 
alternation (regular expression jargon for matching one of several possible patterns) to match either “Olfr218” 
or “Olfr1416.” The syntax uses a pipe symbol (|):


```bash
grep -E "(Olfr1413|Olfr1411)" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

We don't have time to talk about regular expressions here. The important part is that you recognize there’s 
a difference and know the terms necessary to find further help when you need it.

`grep` has an option to count how many lines match a pattern: -c. For example, suppose we wanted a quick look 
at how many genes start with “Olfr”:


```bash
grep -c "\tOlfr" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

Alternatively, we could pipe the matching lines to wc -l:


```bash
grep "\tOlfr" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt | wc -l
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
##        0
```

Counting matching lines is extremely useful—especially with plain-text data where lines represent rows, 
and counting the number of lines that match a pattern can be used to count occurrences in the data. 
For example, suppose we wanted to know how many small nuclear RNAs are in our Mus_musculus.GRCm38.75_chr1.gtf 
file. snRNAs are annotated as gene_biotype "snRNA" in the last column of this GTF file. A simple way to count 
these features would be:


```bash
grep -c 'gene_biotype "snRNA"' ./data/Mus_musculus.GRCm38.75_chr1.gtf
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Note here how we’ve used single quotes to specify our pattern, as our pattern includes the double-quote characters (").
By default, `grep` is outputting the entire matching line. Sometimes, however, it’s useful to use `grep` to extract 
only the matching part of the pattern. We can do this with -o:


```bash
grep -o "Olfr.*" ./data/Mus_musculus.GRCm38.75_chr1_genes.txt | head -n 3
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1_genes.txt: No such file or directory
```

Or, suppose we wanted to extract all values of the “gene_id” field from the last column of our 
Mus_musculus.GRCm38.75_chr1.gtf file. This is easy with -o:


```bash
grep -E -o 'gene_id "\w+"' ./data/Mus_musculus.GRCm38.75_chr1.gtf | head -n 5
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Here, we’re using extended regular expressions to capture all gene names in the field. However, as you 
can see there’s a great deal of redundancy: our GTF file has multiple features (transcripts, exons, 
start codons, etc.) that all have the same gene name. What if you want just a list of unique, sorted gene 
names?


```bash
grep -E -o 'gene_id "\w+"' ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f2 -d" " | sed 's/"//g' | sort | uniq | head -n 10
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

As you can see, we are about half-way through the tools we need to learn!

### Decoding Plain-Text Data: hexdump
In bioinformatics, the plain-text data we work with is often encoded in ASCII. ASCII is a character encoding 
scheme that uses 7 bits to represent 128 different values, including letters (upper- and lowercase),
numbers, and special nonvisible characters. While ASCII only uses 7 bits, nowadays computers use an 8-bit byte 
to store ASCII characters. More information about ASCII is available in your terminal through man ascii. Because 
plain-text data uses characters to encode information, our encoding scheme matters. When working with a plain-text 
file, 98% of the time you won’t have to worry about the details of ASCII and how your file is encoded. However, 
the 2% of the time when encoding does matter—usually when an invisible non-ASCII character has entered data—
it can lead to major headaches. In this section, we’ll cover the basics of inspecting text data at a low level 
to solve these types of problems. If you’d like to skip this section for now, bookmark it in case you run into 
this issue at some point.

First, to look at a file’s encoding use the program `file`, which infers what the encoding is from the file’s 
content. For example, we see that many of the example files we’ve been working with in this chapter are ASCII-encoded:


```bash
file ./data/Mus_musculus.GRCm38.75_chr1*
```

```
## ./data/Mus_musculus.GRCm38.75_chr1*: cannot open `./data/Mus_musculus.GRCm38.75_chr1*' (No such file or directory)
```

Some files will have non-ASCII encoding schemes, and may contain special characters. The most common character 
encoding scheme is UTF-8, which is a superset of ASCII but allows for special characters. For example, the utf8.txt 
included in the GitHub directory is a UTF-8 file, as evident from file’s output:


```bash
file ./data/utf8.txt
```

```
## ./data/utf8.txt: cannot open `./data/utf8.txt' (No such file or directory)
```

Because UTF-8 is a superset of ASCII, if we were to delete the special characters in this file and save it, 
file would return that this file is ASCII-encoded. Most files you’ll download from data sources like Ensembl, 
NCBI, and UCSC’s Genome Browser will not have special characters and will be ASCII-encoded. Often, the problems 
you'll run into are from data generated by humans, which through copying and pasting data from other sources may 
lead to unintentional special characters. For example, the improper.fa file in the GitHub repository looks like 
a regular FASTA file upon first inspection:


```bash
cat ./data/improper.fa
```

```
## cat: ./data/improper.fa: No such file or directory
```

However, finding the reverse complement of these sequences using `bioawk (don’t worry about the details of this 
program yet—we’ll cover it later) leads to strange results:


```bash
bioawk -cfastx '{print revcomp($seq)}' ./data/improper.fa
```

```
## bioawk: can't open file ./data/improper.fa
##  source line number 1
```

What’s going on? We have a non-ASCII character in our second sequence:


```bash
file ./data/improper.fa
```

```
## ./data/improper.fa: cannot open `./data/improper.fa' (No such file or directory)
```

We can use `hexdump` program to identify which letter is causing this problem. 
The hexdump program returns the hexadecimal values of each character. 
With the -c option, this also prints the character:


```bash
hexdump -c data/improper.fa
```

```
## hexdump: data/improper.fa: No such file or directory
## hexdump: data/improper.fa: Bad file descriptor
```

As we can see, the character after “CGAGCGAG” in the second sequence is clearly not an ASCII character. 
Another way to see non-ASCII characters is using grep with an option to look for 
characters outside a hexadecimal range: `grep --color='auto' -P '[\x80-\xFF] improper.fa`. Note that this 
does not work with BSD grep, the version that comes with Mac OS X.

### Sorting Plain-Text Data with Sort

Very often we need to work with sorted plain-text data in bioinformatics. The two most common reasons to 
sort data are as follows:
* Certain operations are much more efficient when performed on sorted data.
* Sorting data is a prerequisite to finding all unique lines, using the Unix `sort | uniq` idiom.

Sort, like cut, is designed to work with plain-text data with columns. Running
sort without any arguments simply sorts a file alphanumerically by line:


```bash
cat ./data/example.bed
echo "======="
sort ./data/example.bed
```

```
## cat: ./data/example.bed: No such file or directory
## =======
## sort: open failed: ./data/example.bed: No such file or directory
```

Because chromosome is the first column, sorting by line effectively groups 
chromosomes together, as these are “ties” in the sorted order. Grouped data 
is quite useful, as we’ll see.

### Using Different Delimiters with sort

By default, sort treats blank characters (like tab or spaces) as field delimiters. 
If your file uses another delimiter (such as a comma for CSV files), you can specify 
the field separator with -t (e.g., -t",").

Using sort’s defaults of sorting alphanumerically by line doesn’t handle 
tabular data properly. There are two additional features we need:

* The ability to sort by particular columns
* The ability to tell sort that certain columns are numeric values (and not alphanumeric text)

`sort` has a simple syntax to do this:


```bash
sort -k1,1 -k2,2n ./data/example.bed
```

```
## sort: open failed: ./data/example.bed: No such file or directory
```

Here, -k specifies the sorting keys and their order. Each -k argument takes a
range of columns as start, end, so to sort by a single column we use start,start. Sorting by 
the first column alone leads to many ties in rows with the same chromosomes (e.g., “chr1” 
and “chr3”). Adding a second -k argument with a different column tells sort how to break 
these ties. In our example, -k2,2n tells sort to sort by the second column (start position), 
treating this column as numerical data (because there’s an n in -k2,2n). If you need all 
columns to be sorted numerically, you can use the argument -n rather
than specifying which particular columns are numeric.

### Sorting Stability
There’s one tricky technical detail about sorting worth being aware of: sorting stability. 
To understand stable sorting, we need to go back and think about how lines that have 
identical sorting keys are handled. If two lines are exactly identical according to all 
sorting keys we’ve specified, they are indistinguishable and equivalent when being sorted. 
When lines are equivalent, sort will sort them according to the entire line as a last-resort 
effort to put them in some order. What this means is that even if the two lines are identical 
according to the sorting keys, their sorted order may be different from the order they appear 
in the original file. This behavior makes sort an unstable sort. If we don’t want sort to change 
the order of lines that are equal according to our sort keys, we can specify the -s option. 
-s turns off this last-resort sorting, thus making sort a stable sorting algorithm.

Sorting can be computationally intensive. Unlike Unix tools, which operate on a single line a time, 
sort must compare multiple lines to sort a file. If you have a file that you suspect is already 
sorted, it’s much cheaper to validate that it’s indeed sorted rather than resort it. We can check 
if a file is sorted according to our -k arguments using -c:


```bash
sort -k1,1 -k2,2n -c ./data/example_sorted.bed
echo $?
echo "========"
sort -k1,1 -k2,2n -c ./data/example.bed
echo $?
```

```
## sort: open failed: ./data/example_sorted.bed: No such file or directory
## 2
## ========
## sort: open failed: ./data/example.bed: No such file or directory
## 2
```

The first file is already sorted by -k1,1 -k2,2n -c, so sort exits with exit status 0 (true).
The second file is not sorted by -k1,1 -k2,2n -c, so sort returns the first out-of-order row 
it finds and exits with status 1 (false).

It’s also possible to sort in reverse order with the -r argument:

```bash
sort -k1,1 -k2,2n -r ./data/example.bed
sort -k1,1 -k2,2nr ./data/example.bed
```

```
## sort: open failed: ./data/example.bed: No such file or directory
## sort: open failed: ./data/example.bed: No such file or directory
```

There are a few other useful sorting options to discuss, but these are available for GNU sort 
only (not the BSD version as found on OS X). The first is -V, which is a clever alphanumeric 
sorting routine that understands numbers inside strings. why this is useful, consider the file 
example2.bed. Sorting with sort -k1,1 -k2,2n groups chromosomes but doesn’t naturally order 
them as humans would:


```bash
sort -k1,1 -k2,2n ./data/example2.bed
```

```
## sort: open failed: ./data/example2.bed: No such file or directory
```

However, with V appended to -k1,1 we get the desired result:


```bash
sort -k1,1V -k2,2n ./data/example2.bed
```

```
## sort: stray character in field spec: invalid field specification `1,1V'
```

In practice, Unix sort scales well to the moderately large text data we’ll need to sort 
in bioinformatics. `sort` does this by using a sorting algorithm called merge sort. 
One nice feature of the merge sort algorithm is that it allows us to sort files 
larger than fit in our memory by storing sorted intermediate files on the disk. 
Under the hood, sort uses a fixed-sized memory buffer to sort as much data in-memory 
as fits. Increasing the size of this buffer allows more data to be sorted in memory, 
which reduces the amount of temporary sorted files that need to be written and read 
off the disk. For example:


```bash
sort -k1,1 -k4,4n -S2G ./data/Mus_musculus.GRCm38.75_chr1_random.gtf
```

The -S argument understands suffixes like K for kilobyte, M for megabyte, and G for 
gigabyte, as well as % for specifying what percent of total memory to use (e.g., 50% with -S 50%).

Another option (only available in GNU sort) is to run sort with the --parallel option. 
For example, to use four cores to sort Mus_musculus.GRCm38.75_chr1_random.gtf:


```bash
sort -k1,1 -k4,4n --parallel 4 ./data/Mus_musculus.GRCm38.75_chr1_random.gtf
```

But note that Mus_musculus.GRCm38.75_chr1_random.gtf is much too small for either 
increasing the buffer size or parallelization to make any difference. In general, 
don’t obsess with performance tweaks like these unless your data is truly large 
enough to warrant them.

### Finding Unique Values in Uniq
Unix’s uniq takes lines from a file or standard input stream, and outputs all 
lines with consecutive duplicates removed. While this is a relatively simple 
functionality, you will use uniq very frequently in command-line data processing. 
Let’s first see an example of its behavior:


```bash
cat ./data/letters.txt
echo "=========="
uniq ./data/letters.txt
```

```
## cat: ./data/letters.txt: No such file or directory
## ==========
## uniq: ./data/letters.txt: No such file or directory
```

As you can see, `uni`q only removes consecutive duplicate lines (keeping one). If instead 
we did want to find all unique lines in a file, we would first sort all lines using `sort` 
so that all identical lines are grouped next to each other, and then run `uniq`:


```bash
sort ./data/letters.txt | uniq
```

```
## sort: open failed: ./data/letters.txt: No such file or directory
```

If we had lowercase letters mixed in this file as well, we could add the option `-i` to 
`uniq` to be case insensitive. `uniq` also has a tremendously useful option that’s used 
very often in command-line data processing: -c. This option shows the counts of occurrences 
next to the unique lines. For example:


```bash
uniq -c ./data/letters.txt
echo "=========="
sort ./data/letters.txt | uniq -c
```

```
## uniq: ./data/letters.txt: No such file or directory
## ==========
## sort: open failed: ./data/letters.txt: No such file or directory
```

Both `sort | uniq` and `sort | uniq -c` are frequently used shell idioms in bioinformatics 
and worth memorizing. Combined with other Unix tools like `grep` and `cut`, `sort` and `uniq` 
can be used to summarize columns of tabular data:


```bash
grep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Because `sort` and `uniq` are line-based, we can create lines from multiple columns to count 
combinations, like how many of each feature (column 3 in this example GTF) are on each 
strand (column 7):


```bash
grep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f3,7 | sort | uniq -c
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Or, if you want to see the number of features belonging to a particular gene identifier:


```bash
grep "ENSMUSG00000033793" Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c
```

```
##   13 CDS
##    3 UTR
##   14 exon
##    1 gene
##    1 start_codon
##    1 stop_codon
##    1 transcript
```

These count tables are incredibly useful for summarizing columns of categorical data. 
Without having to load data into a program like R or Excel, we can quickly calculate 
summary statistics about our plain-text data files. 

`uniq` can also be used to check for duplicates with the -d option, which tells the program
to output duplicated lines only. For example, the mm_gene_names.txt file (which contains a 
list of gene names) does not have duplicates:


```bash
uniq -d mm_gene_names.txt | wc -l
```

```
## uniq: mm_gene_names.txt: No such file or directory
##        0
```

A file with duplicates, like the test.bed file, has multiple lines returned: 


```bash
uniq -d test.bed | wc -l
```

```
## uniq: test.bed: No such file or directory
##        0
```

### `join`
The Unix tool join is used to join different files together by a common column. This is easiest 
to understand with simple test data. Let’s use our example.bed BED file, and example_lengths.txt, 
a file containing the same chromosomes as example.bed with their lengths. The files look like this:


```bash
cat example.bed
echo "=========="
cat example_lengths.txt
```

```
## cat: example.bed: No such file or directory
## ==========
## cat: example_lengths.txt: No such file or directory
```

Our goal is to append the chromosome length alongside each feature (note that the result will 
not be a valid BED-formatted file, just a tab-delimited file). To do this, we need to join 
both of these tabular files by their common column, the one containing the chromosome names.
To append the chromosome lengths to example.bed, we first need to sort both files by the column 
to be joined on. This is a vital step—Unix’s join will not work unless both files are sorted by 
the column to join on. We can appropriately sort both files with sort:


```bash
sort -k1,1 example.bed > example_sorted.bed
sort -c -k1,1 example_lengths.txt # verifies is already sorted
```

```
## sort: open failed: example.bed: No such file or directory
## sort: open failed: example_lengths.txt: No such file or directory
```

The basic syntax is `join -1 <file_1_field> -2 <file_2_field> <file_1> <file_2>`. So, with 
example.bed and example_lengths.txt this would be:


```bash
join -1 1 -2 1 example_sorted.bed example_lengths.txt > example_with_lengths.txt
cat example_with_lengths.txt
```

```
## join: example_lengths.txt: No such file or directory
```

There are many types of joins. For now, it’s important that we make sure join is working 
as we expect. Our expectation is that this join should not lead to fewer rows than in our 
example.bed file. We can verify this with wc -l:


```bash
wc -l example_sorted.bed example_with_lengths.txt
```

```
##        0 example_sorted.bed
##        0 example_with_lengths.txt
##        0 total
```

We see that we have the same number of lines in our original file and our joined file. 
However, look what happens if our second file, example_lengths.txt, is truncated such 
that it doesn’t have the lengths for chr3:


```bash
head -n2 example_lengths.txt > example_lengths_alt.txt #truncate file
join -1 1 -2 1 example_sorted.bed example_lengths_alt.txt
join -1 1 -2 1 example_sorted.bed example_lengths_alt.txt | wc -l
```

```
## head: example_lengths.txt: No such file or directory
##        0
```

Because chr3 is absent from example_lengths_alt.txt, our join omits rows from 
example_sorted.bed that do not have an entry in the first column of 
example_lengths_alt.txt. In some cases (such as this), we don’t want this behavior. 
GNU join implements the -a option to include unpairable lines—ones that do not have 
an entry in either file. To use -a, we spec‐ ify which file is allowed to have 
unpairable entries:


```bash
gjoin -1 1 -2 1 -a 1 example_sorted.bed example_lengths_alt.txt # GNU join only
```

Unix’s join is just one of many ways to join data, and is most useful for simple 
quick joins. Joining data by a common column is a common task during data analysis; 
we’ll see how to do this in R later.

## Text Processing with Awk
Throughout this chapter, we’ve seen how we can use simple Unix tools like `grep`, 
`cut`, and `sort` to inspect and manipulate plain-text tabular data in the shell. 
For many trivial bioinformatics tasks, these tools allow us to get the job done 
quickly and easily (and often very efficiently). Still, some tasks are slightly 
more complex and require a more expressive and powerful tool. This is where the 
language and tool `Awk` excels: extracting data from and manipulating tabular plain-text 
files. `Awk` is a tiny, specialized language that allows you to do a variety of 
text-processing tasks with ease. The key to using Awk effectively is to reserve 
it for the subset of tasks it’s best at:quick data-processing tasks on tabular 
data. Learning Awk will also prepares us to learn `bioawk`, which we’ll cover later.

### Gawk versus Awk
As with many other Unix tools, Awk comes in a few flavors. First, you can still 
find the original Awk written by Alfred Aho, Peter Weinberger, and Brian Kernighan 
(whose last names create the name Awk) on some systems. If you use Mac OS X, you’ll 
likely be using the BSD Awk. There’s also GNU Awk, known as Gawk, which is based on 
the original Awk but has many extended features (and an excellent manual; see man gawk). 
For the following examples, we'll use a common subset of Awk functionality 
shared by all these Awks. Just take note that there are multiple Awk implementaions. 

To learn Awk, we’ll cover two key parts of the Awk language: how Awk processes records, 
and pattern-action pairs. After understanding these two key parts the rest of the 
language is quite simple.

- First, Awk processes input data a record at a time. Each record is composed of fields, 
separate chunks that Awk automatically separates. Because Awk was designed to work 
with tabular data, each record is a line, and each field is a column’s entry for that 
record. The clever part about Awk is that it automatically assigns the entire record 
to the variable $0, and field one’s value is assigned to $1, field two’s value is assigned 
to $2, field three’s value is assigned to $3, and so forth.

- Second, we build Awk programs using one or more of the following structures: pattern { action }
Each pattern is an expression or regular expression pattern. Patterns are a lot like if 
statements in other languages: if the pattern’s expression evaluates to true or the regular 
expression matches, the statements inside action are run. In Awk lingo, these are pattern-action 
pairs and we can chain multiple pattern-action pairs together (separated by semicolons). 
If we omit the pattern, Awk will run the action on all records. If we omit the action but 
specify a pattern, Awk will print all records that match the pattern. This simple structure 
makes Awk an excellent choice for quick text-processing tasks. This is a lot to take in, 
but these two basic concepts—records and fields, and pattern-action pairs—are the foundation 
of writing text-processing programs with Awk. 

Let’s see some examples.
First, we can simply mimic cat by omitting a pattern and printing an entire record with the variable $0:

```bash
awk '{ print $0 }' example.bed #emulating cat
```

```
## awk: can't open file example.bed
##  source line number 1
```

print prints a string. Optionally, we could omit the $0, because print called without an argument
would print the current record.

Awk can also mimic cut:

```bash
awk '{ print $2 "\t" $3 }' example.bed #emulating cut
```

```
## awk: can't open file example.bed
##  source line number 1
```

Here, we’re making use of Awk’s string concatenation. Two strings are concatenated if 
they are placed next to each other with no argument. So for each record, $2"\t"$3 
concatenates the second field, a tab character, and the third field. This is far more 
typing than cut -f2,3, but demonstrates how we can access a certain column’s value for 
the current record with the numbered variables $1, $2, $3, etc.

Let’s now look at how we can incorporate simple pattern matching. Suppose we wanted to 
write a filter that only output lines where the length of the feature (end position - 
start position) was greater than 18. Awk supports arithmetic with the standard operators 
+, -, *, /, % (remainder), and ^ (exponentiation). We can subtract within a pattern to 
calculate the length of a feature, and filter on that expression:


```bash
awk '$3 - $2 > 18' example.bed
```

```
## awk: can't open file example.bed
##  source line number 1
```

Here are some other Awk comparison and logical operations:  
a==b  a!=b | a < b | a > b | a<=b | a>=b | a ~ b | a!~b | a&&b a||b !a
Description
aisequaltob
ais not equal tob
a is less than b
ais greater thanb
ais less than or equal tob
ais greater than or equal tob
a matches regular expression pattern b adoes not match regular expression patternb logical andaandb
logical oraandb
nota(logical negation)

We can also chain patterns, by using logical operators && (AND), || (OR), and ! (NOT). 

For example, if we wanted all lines on chromosome 1 with a length greater than 10:


```bash
awk '$1 ~ /chr1/ && $3 - $2 > 10' example.bed
```

```
## awk: can't open file example.bed
##  source line number 1
```

The first pattern, $1 ~ /chr1/, is how we specify a regular expression. Regular expressions 
are in slashes. Here, we’re matching the first field, $1$, against the regular expression chr1. 
The tilde, ~ means match; to not match the regular expression we would use !~ (or !($1 ~ /chr1/)).

We can combine patterns and more complex actions than just printing the entire record. For example, 
if we wanted to add a column with the length of this feature (end position - start position) for 
only chromosomes 2 and 3, we could use:


```bash
awk '$1 ~ /chr2|chr3/ { print $0 "\t" $3 - $2 }' example.bed
```

```
## awk: can't open file example.bed
##  source line number 1
```

So far, these exercises have illustrated two ways Awk can come in handy:  
* For filtering data using rules that can combine regular expressions and arithmetic
* Reformatting the columns of data using arithmetic

These two applications alone make Awk an extremely useful tool in bioinformatics, 
and a huge time saver. But let’s look at some slightly more advanced use cases. 
We’ll start by introducing two special patterns: BEGIN and END. Like in a bad novel, 
beginning and end are optional in Awk. The BEGIN pattern specifies what to do before 
the first record is read in, and END specifies what to do after the last record’s 
processing is complete. BEGIN is useful to initialize and set up variables, and END 
is useful to print data summaries at the end of file processing. For example, suppose 
we wanted to calculate the mean feature length in example.bed. We would have to take 
the sum feature lengths, and then divide by the total number of records. We can do this with:


```bash
awk 'BEGIN{ s = 0 }; { s += ($3-$2) }; END{ print "mean: " s/NR };'
```

There’s a special variable we’ve used here, one that Awk automatically assigns in addition 
to $0, $1, $2, etc.: NR. NR is the current record number, so on the last record NR is set 
to the total number of records processed. In this example, we’ve initialized a variable `s` 
to 0 in BEGIN (variables you define do not need a dollar sign). Then, for each record we 
increment `s` by the length of the feature. At the end of the records, we print this sum `s` 
divided by the number of records NR, giving the mean.

### Setting Field, Output Field, and Record Separators
While Awk is designed to work with whitespace-separated tabular data, it’s easy to set a 
different field separator: simply specify which separator to use with the -F argument. 
For example, we could work with a CSV file in Awk by starting with awk -F",".
It’s also possible to set the record (RS), output field (OFS), and output record (ORS) 
separators. These variables can be set using Awk’s -v argument, which sets a variable 
using the syntax awk -v VAR=val. So, we could convert a three-column CSV to a tab file 
by just setting the field separator F and output field separator OFS: 


```bash
awk -F"," -v OFS="\t" {print $1,$2,$3}
```

Setting OFS="\t" saves a few extra characters when outputting tab-delimited results with/ 
statements like print "$1 "\t" $2 "\t" $3.

We can use NR to extract ranges of lines, too; for example, if we wanted to extract all 
lines between 3 and 5 (inclusive):


```bash
awk 'NR >= 3 && NR <= 5' ./data/example.bed
```

```
## awk: can't open file ./data/example.bed
##  source line number 1
```

Awk makes it easy to convert between bioinformatics files like BED and GTF. For example, 
we could generate a three-column BED file from Mus_musculus.GRCm38.75_chr1.gtf as follows:


```bash
awk '!/^#/ { print $1 "\t" $4-1 "\t" $5 }' ./data/Mus_musculus.GRCm38.75_chr1.gtf | head -n 3
```

```
## awk: can't open file ./data/Mus_musculus.GRCm38.75_chr1.gtf
##  source line number 1
```

Note that we subtract 1 from the start position to convert to BED format. This is because BED 
uses zero-indexing while GTF uses 1-indexing. T

Awk also has a very useful data structure known as an associative array. Associative arrays 
behave like Python’s dictionaries or hashes in other languages. We can create an associative 
array by simply assigning a value to a key. For example, suppose we wanted to count the number 
of features (third column) belonging to the gene “Lypla1.” We could do this by incrementing 
their values in an associative array:


```bash
awk '/Lypla1/ { feature[$3] += 1 }; END { for (k in feature) print k "\t" feature[k] }' ./data/Mus_musculus.GRCm38.75_chr1.gtf
```

```
## awk: can't open file ./data/Mus_musculus.GRCm38.75_chr1.gtf
##  source line number 1
```

This example illustrates that Awk really is a programming language: we can use standard programming statements like if, for, and while, and Awk has several useful built-in functions within action blocks. However, when Awk programs become complex or start to span multiple lines, you may want to switch to Python as you’ll have much more functionality at your disposal.

It’s worth noting that there’s an entirely Unix way to count features of a particular gene: grep, cut, sort, and uniq -c:


```bash
grep "Lypla1" ./data/Mus_musculus.GRCm38.75_chr1.gtf | cut -f 3 | sort | uniq -c
```

```
## grep: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

However, if we needed to also filter on column-specific information (e.g., strand), an approach using just base Unix tools would be quite messy. With Awk, adding an additional filter would be trivial: we’d just use && to add another expression in the pattern.

### Bioawk: An Awk for Biological Formats
Bioawk extends Awk’s powerful processing of tabular data to processing tasks involving common 
bioinformatics formats like FASTA/FASTQ, GTF/GFF, BED, SAM, and VCF. Bioawk is written by Heng Li, 
author of other excellent bioinformatics tools such as BWA and Samtools). You can download, compile, 
and install Bioawk from source, or if you use Mac OS X’s Homebrew package manager, Bioawk is also 
in homebrew-science (so you can install with `brew tap homebrew/science; brew install bioawk`).
The basic idea of Bioawk is that we specify what bioinformatics format we’re working with, and Bioawk will automatically set variables for each field (just as regular Awk sets the columns of a tabular text file to $1, $1, $2, etc.). For Bioawk to set these fields, specify the format of the input file or stream with -c. Let’s look at Bioawk’s supported input formats and what variables these formats set:


```bash
bioawk -c help bed
```

```
## bed:
## 	1:chrom 2:start 3:end 4:name 5:score 6:strand 7:thickstart 8:thickend 9:rgb 10:blockcount 11:blocksizes 12:blockstarts 
## sam:
## 	1:qname 2:flag 3:rname 4:pos 5:mapq 6:cigar 7:rnext 8:pnext 9:tlen 10:seq 11:qual 
## vcf:
## 	1:chrom 2:pos 3:id 4:ref 5:alt 6:qual 7:filter 8:info 
## gff:
## 	1:seqname 2:source 3:feature 4:start 5:end 6:score 7:filter 8:strand 9:group 10:attribute 
## fastx:
## 	1:name 2:seq 3:qual 4:comment
```

As an example of how this works, let’s read in example.bed and append a column with the length of 
the feature (end position - start position) for all protein coding genes:


```bash
bioawk -c gff '$3 ~ /gene/ && $2 ~ /protein_coding/ {print $seqname,$end-$start}' ./data/Mus_musculus.GRCm38.75_chr1.gtf | head -n 4
```

```
## bioawk: can't open file ./data/Mus_musculus.GRCm38.75_chr1.gtf
##  source line number 1
```

Bioawk is also quite useful for processing FASTA/FASTQ files. For example, we could use it 
to turn a FASTQ file into a FASTA file:


```bash
bioawk -c fastx '{print ">"$name"\n"$seq}' ./data/contam.fastq | head -n 4
```

```
## bioawk: can't open file ./data/contam.fastq
##  source line number 1
```

Note that Bioawk detects whether to parse input as FASTQ or FASTA when we use -c fastx.  

Bioawk can also serve as a method of counting the number of FASTQ/FASTA entries:


```bash
bioawk -c fastx 'END{print NR}' ./data/contam.fastq
```

```
## bioawk: can't open file ./data/contam.fastq
##  source line number 1
```

Or Bioawk’s function `revcomp()` can be used to reverse complement a sequence:


```bash
bioawk -c fastx '{print ">"$name"\n"revcomp($seq)}' contam.fastq | head -n 4
```

```
## bioawk: can't open file contam.fastq
##  source line number 1
```

Bioawk is also useful for creating a table of sequence lengths from a FASTA file. 
For example, to create a table of all chromosome lengths of the Mus musculus genome:


```bash
bioawk -c fastx '{print $name,length($seq)}' Mus_musculus.GRCm38.75.dna_rm.toplevel.fa.gz > mm_genome.txt
head -n 4 mm_genome.txt
```

```
## bioawk: can't open file Mus_musculus.GRCm38.75.dna_rm.toplevel.fa.gz
##  source line number 1
```

Finally, Bioawk has two options that make working with plain tab-delimited files easier: 
-t and -c hdr. -t is for processing general tab-delimited files; it sets Awk’s field 
separator (FS) and output field separator (OFS) to tabs. The option -c hdr is for 
unspecific tab-delimited formats with a header as the first line. This option sets 
field variables, but uses the names given in the header. Suppose we had a simple 
tab-delimited file containing variant names and genotypes for individuals (in columns):


```bash
head -n 4 ./data/genotypes.txt
```

```
## head: ./data/genotypes.txt: No such file or directory
```

If we wanted to return all variants for which individuals ind_A and ind_B have identical genotypes
(note that this assumes a fixed allele order like ref/alt or major/minor):


```bash
bioawk -c hdr '$ind_A == $ind_B {print $id}' ./data/genotypes.txt
```

```
## bioawk: can't open file ./data/genotypes.txt
##  source line number 1
```


## Stream Editing with Sed

Unix pipes are fast because they operate on streams of data (rather than data written to disk). 
Additionally, pipes don’t require that we load an entire file in memory at once; instead, we can 
operate one line at a time. We’ve used pipes throughout this chapter to transform plain-text 
data by selecting columns, sorting values, taking the unique values, and so on. Often we need 
to make trivial edits to a stream, usually to prepare it for the next step in a Unix pipeline. 
The stream editor, or `sed`, allows you to do exactly that. sed is remarkably powerful, a
nd has capabilities that overlap other Unix tools like `grep` and `awk`. As with awk, it’s best 
to keep your sed commands simple at first. We’ll cover a small subset of sed’s vast functionality 
that’s most useful for day-to-day bioinformatics tasks.

> ## GNU Sed versus BSD Sed
> As with many other Unix tools, the BSD and GNU versions of sed differ considerably in behavior. 
> In this section and in general, the GNU version of sed is preferred. GNU sed has some additional 
> features, and supports functionality such as escape codes for special characters like tab (\t) 
> we expect in command-line tools.
{: .callout}

`sed` reads data from a file or standard input and can edit a line at a time. Let’s look at a very 
simple example: converting a file (chroms.txt) containing a single column of chromosomes in the 
format “chrom12,” “chrom2,” and so on to the format “chr12,” “chr2,” and so on:


```bash
head -n 3 chroms.txt # before sed
echo "=========="
sed 's/chrom/chr/' chroms.txt | head -n 3
```

```
## head: chroms.txt: No such file or directory
## ==========
## sed: chroms.txt: No such file or directory
```

It’s a simple but important change: although chroms.txt is a mere 10 lines long, it could 
be 500 gigabytes of data and we can edit it without opening the entire file in memory. This 
is the beauty of stream editing, sed’s bread and butter. Our edited out‐put stream is then 
easy to redirect to a new file (don’t redirect output to the input file; this causes bad 
things to happen).

In the preceding code works we uses sed’s substitute command, by far the most popular use 
of sed. sed’s substitute takes the first occurrence of the pattern between the first two 
slashes, and replaces it with the string between the second and third slashes. In other words, 
the syntax of sed’s substitute is s/pattern/replacement/.

By default, sed only replaces the first occurrence of a match. Very often we need to 
replace all occurrences of strings that match our pattern. We can enable this behavior 
by setting the global flag g after the last slash: `s/pattern/replacement/g`. If we 
need matching to be case-insensitive, we can enable this with the flag i (e.g., `s/pattern/ replacement/i`).
Note that by default, sed’s substitutions use POSIX Basic Regular Expressions (BRE). As with 
`grep`, we can use the -E option to enable POSIX Extended Regular Expressions (ERE).  

Perhaps most important is the ability to capture chunks of text that match a pattern, 
and use these chunks in the replacement (often called grouping and capturing). For 
example, suppose we wanted to capture the chromo‐ some name, and start and end positions 
in a string containing a genomic region in the format "chr1:28427874-28425431", and 
output this as three columns. We could use:


```bash
echo "chr1:28427874-28425431" | gsed -E 's/^(chr[^:]+):([0-9]+)-([0-9]+)/\1\t\2\t\3/'
```

```
## chr1	28427874	28425431
```

The first component of this regular expression is ^(chr[^:]+):. This matches the text 
that begins at the start of the line (the anchor ^ enforces this), and then captures 
everything between ( and ). The pattern used for capturing begins with “chr” and matches 
one or more characters that are not “:”, our delimiter. We match until the first “:” 
through a character class defined by everything that’s not “:”, [^:]+.
The second and third components of this regular expression are the same: match and capture 
more than one number. Finally, our replacement is these three captured groups, interspersed 
with tabs, \t. Here are a few more ways to do the same thing:


```bash
echo "chr1:28427874-28425431" | sed 's/[:-]/\t/g'
echo "chr1:28427874-28425431" | sed 's/:/\t/' | sed 's/-/\t/' # or sed -e 's/:/\t/' -e 's/-/\t/'
echo "chr1:28427874-28425431" | tr ':-' '\t'
```

```
## chr1t28427874t28425431
## chr1t28427874t28425431
## chr1	28427874	28425431
```

Note, that in the last example we use `tr` to translate both delimiters to a tab character. 
`tr` translates all occurrences of its first argument to its second (see man tr for more details).

By default, sed prints every line, making replacements to matching lines. Sometimes this behavior 
isn’t what we want (and can lead to erroneous results). To print only the lines that match a pattern, 
we need to disable sed from outputting all lines with `-n` option and than append the `p` flag after 
the last slash:


```bash
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | head -n 3 | sed -E -n 's/.*transcript_id "([^"]+)".*/\1/p'
```

```
## ENSMUST00000160944
## ENSMUST00000160944
```

This example uses an important regular expression idiom: capturing text between delimiters 
(in this case, quotation marks): `"([^"]+)"`. In regular extension jargon, the brackets make 
up a character class. Character classes specify what characters the expression is allowed to 
match. Here, we use a caret (^) inside the brackets to match anything except what’s inside these 
brackets.

It’s also possible to select and print certain ranges of lines with sed. In this case, we’re 
not doing pattern matching, so we don’t need slashes:


```bash
sed -n '1,5p' ./data/Mus_musculus.GRCm38.75_chr1.gtf
echo "============"
sed -n '40,45p' ./data/Mus_musculus.GRCm38.75_chr1.gtf
```

```
## sed: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
## ============
## sed: ./data/Mus_musculus.GRCm38.75_chr1.gtf: No such file or directory
```

Substitutions make up the majority of sed’s usage cases, but this is just scratching the surface 
of sed’s capabilities. sed has features that allow you to make any type of edit to a stream of 
text, but for complex stream processing tasks it can be easier to write a Python script than a 
long and complicated sed command. Remember the KISS principal: Keep Incredible Sed Simple.

## Advanced Shell Tricks
With both the Unix shell refresher in Chapter 3 and the introduction to Unix data tools in this 
chapter, we’re ready to dig into a few more advanced shell tricks. 

### Subshells

The first trick we’ll cover is using Unix subshells. Before explaining this trick, it’s helpful 
to remember the difference between sequential commands (connected with && or ;), and piped commands 
(connected with |). Sequential commands are simply run one after the other; the previous command’s 
standard output does not get passed to the next program’s standard in. In contrast, connecting two 
programs with pipes means the first program’s standard out will be piped into the next program’s 
standard in.

The difference between sequential commands linked with && and ; comes down to exit status: ff we run 
two commands with `command1 ; command2`, command2 will always run, regardless of whether command1 
exits successfully (with a zero exit status). In contrast, if we use `command1 && command2`, command2 
will only run if command1 completed with a zero-exit status. Checking exit status with pipes 
unfortunately gets tricky. So how do subshells fit into all of this? 

Subshells allow us to execute sequential com‐ mands together in a separate shell process. This is 
useful primarily to group sequential commands together (such that their output is a single stream). 
This gives us a new way to construct clever oneliners and has practical uses in command-line data 
processing. Let’s look at a toy example first:


```bash
echo "this command"; echo "that command" | sed 's/command/step/'
```

```
## this command
## that step
```



```bash
(echo "this command"; echo "that command") | sed 's/command/step/'
```

```
## this step
## that step
```

In the first example, only the second command’s standard out is piped into sed. But grouping both 
echo commands together using parentheses causes these two com‐mands to be run in a separate subshell, 
and both commands’ combined standard output is passed to sed. Combining two sequential commands’ 
standard output into a single stream with a subshell is a useful trick, and one we can apply to shell 
problems in bioinformatics.

Consider the problem of sorting a GTF file with a metadata header. We can’t simply sort the entire 
file with sort, because this header could get shuffled in with rows of data. Instead, we want to 
sort everything except the header, but still include the header at the top of the final sorted file. 
We can solve this problem using a subshell to group sequential commands that print the header to 
standard out and sort all other lines by chromosome and start position, printing all lines to standard 
out after the header:


```bash
(zgrep "^#" Mus_musculus.GRCm38.75_chr1.gtf.gz; zgrep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf.gz | sort -k1,1 -k4,4n) | less
```

```
## zgrep: Mus_musculus.GRCm38.75_chr1.gtf.gz: No such file or directory
## zgrep: Mus_musculus.GRCm38.75_chr1.gtf.gz: No such file or directory
```

We can redirect the output to gzip to compress this stream before writing it to disk:


```bash
(zgrep "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf.gz; zgrep -v "^#" ./data/Mus_musculus.GRCm38.75_chr1.gtf.gz | sort -k1,1 -k4,4n) | ` | gzip > ./data/Mus_musculus.GRCm38.75_chr1_sorted.gtf.gz
```

### Named Pipes and Process Substitution

Throughout this chapter, we’ve used pipes to connect command-line tools to build custom 
data-processing pipelines. However, some programs won’t interface with the Unix pipes 
we’ve come to love and depend on. For example, certain bioinformatics tools read in multiple 
input files and write to multiple output files:


```bash
processing_tool --in1 in1.fq --in2 in2.fq --out1 out2.fq --out2.fq
```

In this case, the imaginary program processing_tool requires two separate input files, 
and produces two separate output files. Because each file needs to be provided separately, 
we can’t pipe the previous processing step’s results through processing_tool’s standard in. 
Likewise, this program creates two separate output files, so it isn’t possible to pipe 
processing_tool’s standard output to another program in the processing pipeline. In addition 
to the inconvenience of not being able to use a Unix pipe to interface processing_tool with 
other programs, there’s a more serious problem: we would have to write and read four intermediate 
files to disk to use this program. If processing_tool were in the middle of a data-processing 
pipeline, this would lead to a significant computational bottleneck. Fortunately, Unix provides 
a solution: named pipes. A named pipe, also known as a FIFO (First In First Out, a concept in 
computer science), is a special sort of file. Regular pipes are anonymous—they don’t have a name, 
and only persist while both processes are running. Named pipes behave like files, and are persistent 
on your filesystem. We can create a named pipe with the program mkfifo:


```bash
mkfifo fqin
ls -l fqin
```

```
## prw-r--r--  1 dlavrov  staff  0 Mar  7 22:39 fqin
```

You’ll notice that this is indeed a special type of file: the p before the file permissions is 
for pipe. Just like pipes, one process writes data into the pipe, and another process reads 
data out of the pipe. As a toy example, we can simulate this by using echo to redirect some text 
into a named pipe (running it in the background, so we can have our prompt back), and then cat to 
read the data back out:


```bash
echo "hello, named pipes" > fqin &
cat fqin
rm fqin
```

```
## hello, named pipes
```

Treating the named pipe just as we would any other file, we can access the data we wrote to it 
earlier. Any process can access this file and read from this pipe. Like a standard Unix pipe, 
data that has been read from the pipe is no longer there. Like a file, we clean up by using `rm` 
to remove it. Although the syntax is similar to shell redirection to a file, we’re not actually 
writing anything to our disk. Named pipes provide all of the computational benefits of pipes with 
the flexibility of interfacing with files. In our earlier example, in1.fq and in2.fq could be 
named pipes that other processes are writing input to. Additionally, out1.fq and out2.fq could be 
named pipes that processing_tool is writing to, that other processes read from.m However, creating 
and removing these file-like named pipes is a bit tedious. So there’s a way to use named pipes 
without having to explicitly create them. This is called process substitution, or sometimes known 
as anonymous named pipes. These allow you to invoke a process, and have its standard output go 
directly to a named pipe. However, your shell treats this process substitution block like a file, 
so you can use it in commands as you would a regular file or named pipe. This is a bit confusing 
at first, but it should be clearer with some examples. If we were to re-create the previous toy e
xample with process substitution, it would look as follows:


```bash
cat <(echo "hello, process substitution")
```

```
## hello, process substitution
```

The chunk <(echo "hello, process sub stition") runs the echo command and pipes the output to an 
anonymous named pipe. Your shell then replaces this chunk (the <(...) part) with the path to 
this anonymous named pipe. No named pipes need to be explicitly created, but you get the same 
functionality. Process substitution allows us to connect two (or potentially more) programs, 
even if one doesn’t take input through standard input. This is illustrated in the following figure:

![](./images/fig7.3.png)

In the program example we saw earlier, two inputs were needed (--in1 and --in2). For the sake of 
this example, assume that a program called makein is cre‐ating the input streams for --in1 and --in2. 
We can use process substitution to create two anynyous pipes:


```bash
program --in1 <(makein raw1.txt) --in2 <(makein raw2.txt) --out1 out1.txt --out2 out2.txt
```

The last thing to know about process substitution is that it can also be used to capture an 
output stream. This is used often in large data processing to compress output on the fly before 
writing it to disk. In the preceding program example, assume that we wanted to take the output 
that’s being written to files out1.txt and out2.txt and compress these streams to files out1.txt.gz 
and out2.txt.gz. We can do this using the intuitive analog to <(...), >(...):


```bash
program --in1 in1.txt --in2 in2.txt --out1 >(gzip > out1.txt.gz) --out2 >(gzip > out2.txt.gz)
```

This creates two anonymous named pipes, and their input is then passed to the `gzip` command. 
`gzip` then compresses these and writes to standard out, which we redirect to our gzipped files.

## The Unix Philosophy Revisited

Throughout this chapter, the Unix philosophy—equipped with the power of the Unix pipe—allowed 
us to rapidly stitch together tiny programs using a rich set of Unix tools. Not only are Unix 
piped workflows fast to construct, easy to debug, and versa‐ tile, but they’re often the most 
computationally efficient solution, too. It’s a testament to the incredible design of Unix that 
so much of the way we approach modern bioinformatics is driven by the almighty Unix pipe, a 
piece of technology invented over 40 years ago in “one feverish night” by Ken Thompson (as 
described by Doug McIlroy).
