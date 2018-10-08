**What the `awk` `sed`**
===================
_an introduction to_ `sed` _and_ `awk` _for biologists_
----------------------------------------------------------
_by Nick Kron_

---

This is a cursory introduction to the UNIX utilities `sed` and `awk` that are incredibly useful tools for your day to day data management and cleaning in computational biology. The intent of this introduction is to highlight some practical uses of `sed` and `awk` for common computational biology file types and low level problems.

I assume you have had a basic introduction to UNIX and the Bourne Again Shell (bash) and can navigate between directories, read and write files, and, ideally, use `grep` is. If you have used `sed` or `awk` before, this introduction is likely not for you. If you have no idea what any of that meant, this is also not for you. If, however, you are a newbie UNIX user that wants to expand their toolkit and take their computational biology to the next level, you've come to the right place!

---

## Part 1 - `sed`
---
### What is `sed`?
Yes, `sed` may be a homophone for the past tense of say, but it's much more than that! `sed` is actually a text editor, just like `vi`, atom, BBedit, and notepad. In fact, `sed` is a descendant of the original UNIX text editor, `ed`. However, unlike its predecessor and all those other programs, `sed` is a _stream editor_. While you may be used to popping open a file in atom and changing things around directly, `sed` sucks those files in as a long stream of characters (hence the stream editor) and edits them behind the scenes.

This may not seem intuitive at first, after all you can't really see what's happening, but this functionality makes `sed` incredibly efficient at performing both targeted and repetitive edits on large files. This, as you can imagine, is incredibly useful for anything with large datasets and files, like computational biology.

### Ok, so how do I use `sed`?
If you want a really thorough look at how `sed` works, the GNU has a great manual available [here](https://www.gnu.org/software/sed/manual/sed.html). However, in the spirit of _practical computing for biologists_, I will only discuss those uses of `sed` that I use regularly in a computational biology context (with some relevant/useful examples). That said, first we have to disentangle the `sed` UNIX command call from the "language" of `sed`.

#### The `sed` command
Before we run off to stack overflow to find a kind stranger to write our script for us, we do what any good UNIX user worth their salt should do: RTFM, or "read the ..._friendly_... manual". So, in our shell let's bring up the manual page for `sed` like so:

`sed --help`

You should see something like this pop up:

```
Usage: sed [OPTION]... {script-only-if-no-other-script} [input-file]...
```

followed by a long list of `OPTION`s you can use with the `sed` command call.
What the usage tells us is that to use `sed` we need to provide it a set of instructions (a script) and file(s) to work on. The options section tells us we can provide those instructions two ways:

1) As part of the command call:
```
sed -e 'instructions' input.txt
```
or

2) As a file:
```
sed -f instructions_file.sed input.txt
```

The two ways of providing instructions really do the same thing internally, `sed` just needs to be told what it should do. However, writing the same set of instructions over and over again or writing really long or complex instructions can be tedious/unwieldy/annoying, so you are given the option to store those instructions in a file for later.

Now, if you only want `sed` to do one thing, you can omit the `-e` option entirely and just use something like this:

```
sed 'instructions' input.txt.
```

If you want to tell `sed` to do **more** than one thing, however, that's where the `-e` option comes in. Say you want `sed` to do three things, the command would look as follows:

```
sed -e 'instructions1' -e 'instructions2' -e 'instructions3' input.txt
```

but if you don't want to write out `-e` that many times (and who does!?), then you can pass in the list of instructions inside only one set of `''` by separating each instruction with a `;`, like so:

```
sed 'instructions1 ; instructions2 ; instructions3' input.txt
```


Better yet, if you are using a Bourne Again Shell (bash), and really you should be, then you can just separate each instruction by hitting `return`/`enter`, like so:

```
sed '
instructions1
instructions2
instructions3 ' input.txt
```

One last thing: the input for `sed` can be a file (ie `input.txt`), or several files (ie `input1.txt`, `input2.txt`, `input3.txt`), or not specified. If you don't specify a file, `sed` will look to `stdin` for text to operate on (ie we can use a pipe to pass text to `sed`:

```
echo "text" | sed 'instructions')
```

Ok, so now we know how to tell the shell that we want `sed` to do stuff, so you must be dying to know what `sed` instructions look like, right? Keep reading!

#### The language of `sed`
When I say "the language of `sed`", I do actually mean programming language. That may sound scary, but all it really means is that `sed` takes instructions that follow a set of rules of how those instructions can be written, aka syntax. Programming languages can be used to do something as mundane as print out `hello world` or as complex as code an entire operating system, and `sed` is no different. While we could code the game [Tetris](https://github.com/uuner/sedtris) with `sed`, in my day to day experience I only use one simple `sed` command (the substitute command) which is made even easier by leveraging any `grep` experience you may have.

(fun fact: `grep` was originally the command `g/re/p` in `sed`'s precursor `ed`)

If you want to learn about the other useful commands in `sed` (like delete or print), go check out that [manual](https://www.gnu.org/software/sed/manual/sed.html) I linked earlier.

##### The substitute command `s`
Most of your work as a computational biologist is data cleaning and management, so you are going to need to be a good data wrangler. Frequently extracting the information you want from each line of 10,000 line file will require pattern matching and substitution. To do this, I use the `s` command in `sed`. The `s` command has the following basic syntax:

```
sed 's/regex/replacement/flags' input.txt
```

Lets break that down:

-`s` - tells `sed` we want to use the substitute function

-`regex` - the pattern we want `sed` to match (think back to `grep`)

-`replacement` - what we want to replace that matched pattern with

-`flags` - ways to modify `s`'s default behavior

Each of these is separated by a `/`. Remember, an `s` command always has 3 `/`, otherwise you will get an error. Let's try some examples:

#### Example 1: `sed` `s` (easy)
As a simple example, lets say I want `sed` to replace every "t" to a "u" in the file DNA.fa which has the following text:
```
>header
actagctagctcgatcgatcgataggatcgatcgatcgctagctagcatcgatcgactggagcttacg
```

the command would look like this:
```
sed 's/t/u/g' DNA.fa
```

Try it! Your shell should print out this:

```
>header
acuagcuagcucgaucgaucgauaggaucgaucgaucgcuagcuagcaucgaucgacuggagcuuacg
```

Congrats! You just converted a DNA sequence to an RNA sequence. Ok, lets break that command down:

-`s` - we're telling `sed` to substitute

-`t` - the pattern we are matching is the latter "t"

-`u` - we are replacing instances of the pattern with "u"

-`g` - this is the "global" flag, which tells `sed` to replace all instances of the pattern. Had we not specified this, it would only have changed the first instance of "t".

##### Capturing in `s` regex
Ok, let's try something a little more advanced. Lets say you only want to modify a line without editing a specific part of it, that is to say keep one part of a line exactly as is. To do this we "capture" that element of the regex using escaped parentheses `\(` and `\)`.

To capture, simply wrap whatever it is you need to keep with these parentheses, like so:

```
r\(eg\)ex
```
We can then access the captured element in the `replacement` field of our `sed` `s` command by using escaped digits (ie `\1`), like so:

```
sed 's/r\(eg\)ex/\1/'
```
This would turn `regex` into `eg`. But we can go a step further and make the replacement more complex, like so:

```
sed 's/r\(eg\)ex/I like fried \1gs'
```
Which turns `regex` into `I like fried eggs`, where that first `eg` in eggs is what we captured. Do note that you can **only** store **9** submatches (ie up to `\9`) with `sed`. If you want to store more than that you will need to use a more advanced program (or be more creative with your use of `sed`). Now that you are a capturing pro, let's test those skills with another example:

#### Example 2: `sed` `s` (intermediate)
Lets say you had some RNAseq libraries sequenced. When you downloaded your files you noticed the files names contained way more information than you need (or is convenient to type). You decide to store all those filenames into a file named `seq_file_list.txt` so that you can plan how you want to change those names.
When you use
```
cat seq_file_list.txt
```
you get the following result:

```
R001-L1-P01-TTTTTTTT-GGGGGGGG-READ-1.txt
R001-L1-P01-TTTTTTTT-GGGGGGGG-READ-2.txt
R002-L1-P02-AAAAAAAA-GGGGGGGG-READ-1.txt
R002-L1-P02-AAAAAAAA-GGGGGGGG-READ-2.txt
R003-L1-P03-CCCCCCCC-GGGGGGGG-READ-1.txt
R003-L1-P03-CCCCCCCC-GGGGGGGG-READ-2.txt
```
We only want the replicate name (ie P01) and the read (ie READ-1). We also would rather use `_` than `-`, and change the file ending from `.txt` to `.fastq` since that's their real file type. To do this we will need to make a regex pattern that matches all the lines, capture the desired information, and then write a replacement that gives us only what we want. The command will look something like this:

```
sed 's/R00.-L.-\(P0.\)-[ACGT]*-[ACGT]*-\(READ\)-\(.\)\.txt/\1_\2_\3.fastq/g' seq_file_list.txt
```
Which will produce an output like this:
```
P01_READ_1.fastq
P01_READ_2.fastq
P02_READ_1.fastq
P02_READ_2.fastq
P03_READ_1.fastq
P03_READ_2.fastq
```
Easy. Let's break that command down starting with the regex:

`/R00.-L.-\(P0.\)-[ACGT]*-[ACGT]*-\(READ\)-\(.\)\.txt/`
If you remember your regex from `grep`, we use some wild cards to deal with the elements that change between the lines.

`R00.-` -uses a `.` to match the changing numbers in the run field

`\(P0.\)-` - uses the `.` again and captures the replicate name

`[ACGT]*-` - both these segments capture any string of nucleotides up to a `-`

`\(READ\)-` - just captures the word read

`\(.\)` - captures the read number

`\.txt` - captures the file ending. Note, we escape the `.` so it doesn't act as a wild card.

Depending on how much information is shared between lines, the pattern can become more or less complex.

The replacement then prints out the captured replicated name, `\1`, the word READ, `\2`, and the read number, `\3`, separated by `_`'s. Finally, `.fastq` is tacked on at the end.

Changing the names from within a file is a good exercise, but in the real world you will likely be using a command like this to change the actual names of files. That will be our final, advanced example before we move on to `awk`.

#### Example 3: `sed` `s` (advanced)
We have the directory `seq_files/` which contains all of our sequencing files, verbose names and all. We will combine `for` loops, other UNIX utilities, and variables to change these file names to what we want. If you are unfamiliar with loops, I recommend taking a moment to do a little reading about loops in bash before attempting this example.
For this example we will use `sed` within a loop to change all the file names. In our shell, lets navigate to the target folder:
```
cd seq_files/
```

Then we will rename each of the files using a `for` loop:

```
for FILE in `ls *`
do
	NAME=`echo $FILE | sed 's/R00.-L.-\(P0.\)-[ACGT]*-[ACGT]*-\(READ\)-\(.\)\.txt/\1_\2_\3.fastq/g'`
	cp $FILE "$NAME"
done
```

Ok let's break this script down:
-`ls *` lists the file in the directory, which we can then  loop over.
-we iterate over each file in the list
-at each iteration we `echo` the name of the file to `sed` to extract the new file name, and save it to the variable `NAME`
-we copy the file and change its name to whatever is in `NAME` that iteration (note the `" "` are important).

Now our files have more tractable names! Note that we used `cp` to copy the files. If you just wanted to rename the files and not create a copy, you would use `mv` instead of `cp`. Before we finish up with `sed`, lets `cd ..` to get back to our original directory so that we are ready for part 2.

Ok! That's it for a basic introduction to `sed`. If you're hungry for more, that `sed` [manual](https://www.gnu.org/software/sed/manual/sed.html) has all you need. With some practice, `sed` can be incredibly powerful, so read up and experiment! When you get stuck, remember: RTFM and Google is your friend.
