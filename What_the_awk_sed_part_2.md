**What the `awk` `sed`**
===================
_an introduction to_ `sed` _and_ `awk` _for biologists_
----------------------------------------------------------
_by Nick Kron_

---

This is a cursory introduction to the UNIX utilities `sed` and `awk` that are incredibly useful tools for your day to day data management and cleaning in computational biology. The intent of this introduction is to highlight some practical uses of `sed` and `awk` for common computational biology file types and low level problems.

I assume you have had a basic introduction to UNIX and the Bourne Again Shell (bash) and can navigate between directories, read and write files, and, ideally, use `grep` is. If you have used `sed` or `awk` before, this introduction is likely not for you. If you have no idea what any of that meant, this is also not for you. If, however, you are a newbie UNIX user that wants to expand their toolkit and take their computational biology to the next level, you've come to the right place!

---

## Part 2 - `awk`
---
### What is `awk`?
We now know what `sed` is, so then what on earth is `awk`? Is it a homophone for some sort of weird bird? A hip way to say awkward? Yes, yes, and much more! Just like `sed`, `awk` is a stream editor programming language. The programming power of `awk`, which are modeled after `C` are much more powerful than `sed`, hell someone coded [DOOM](https://github.com/TheMozg/awk-raycaster) using purely `awk`, but their basic usage is similar. Don't believe me? Use `awk --help` and RTFM. As you might expect, the GNU has an equally useful [manual](https://www.gnu.org/software/gawk/manual/gawk.html) for `awk`, so check there if you want more.

Where `awk` differs is in how it interprets the input stream. While `sed` works on the whole line, `awk` breaks the line into a series of chunks using a delimiter. This makes `awk` perfect for regularly delimited data, like a spread sheet or a `.gff` file. Though the power of `awk` extends way beyond that of simple field manipulation, your primary use of `awk` will likely be just that, so that is what we will cover next.

### Ok, so how do I use `awk`?
The command usage of is essentially identical to that of `sed` (and well honestly most UNIX utilities). Just like `sed`, `awk` basic usage can be described as:
```
awk [options] 'instructions' input.txt
```
Just as with `sed`, you feed the `awk` command a list of instructions, either typed directly in the command or from a file (with `-f`), and then provide it with a file (or `stdin`) to work on. Sine `awk` works on fields rather than whole lines, you can specify your delimiter using the `-F` flag (note the upper case), like so:
```
awk -F"," 'instructions' input.txt
```
In this case, we set the delimiter `awk` uses to split lines up to a `,`, which is particularly useful for those comma separated values files (`.csv`). If you omit the `-F` flag, `awk` defaults to any white space.

Now where `awk` really starts to look different from `sed` is in the syntax of the instructions:

#### The language of `awk`
Strangely enough, `awk` is a lot less awkward looking than `sed`. Ok, puns aside, `awk` syntax follows a more standard `C` look, which is sueful for anyone who has any programming experience.

The basic structure to an `awk` statment is as follows:
```
condition {action}
```
So essentially with each `awk` line you are saying: given a condition, do something. Notice those curly braces? Those are very `C`, and are important for bounding actions, so don't forget them! You can string along multiple of these within an `awk` command by sepparating them by `;`, just like in `sed`.

The thing is, `awk` can also be very idiomatic to make for some crazy one-liners. We will stay away from those for now, but if you ever need a quick three work fix to your file, you can find a great list [here](http://www.catonmat.net/blog/awk-one-liners-explained-part-one/). For now, we will keep things basic.

#### Example 4: `awk` (easy)
In our directory we can find the file `example.csv`. The file contains two columns: `tx_id` and `pval`. We can take a quick peak with the `head` command:
```
head -n5 example.csv
```
which yields:
```
tx_ID,pval
XM_005088767.1, 0.02
XM_013085418.1, 0.3
XM_005088768.2, 0.005
XM_005088769.2, 0.00065
```

Lets say we don't really care about the p-value for now and just want the names column. We can do this by using the following command:

```
awk -F"," '{print $1}' example.csv
```
which yields:
```
tx_ID
XM_005088767.1
XM_013085418.1
XM_005088768.2
XM_005088769.2
XM_005088772.2
XM_005088771.2
```

Now you may be thinking: "Hey wait! Doesn't `awk` require a condition?". Well, you are right, it should. Since we didn't include a specific condition, `awk` just assumes that all lines pass and it will perform the desired task. Ok lets break the command down:

-`-F","` - this tells `awk` to split the line by `,`, appropriate for a `.csv`.

-`'{print $1}'` - here we tell `awk` that all lines pass (ie no condition is specified) so it should `print` to screen the first field (ie column), denoted by `$1`.

in `awk` we specify which field to perform operations on by using internal variables that correspond to each field's index by using `$` notation: `$1` is the first field, `$2` is the second field, etc.). Like `sed`, this only goes up to `$9`. Unlike `sed`, `awk` is pretty smart so you can include more complex field indexes by hiding them in parentheses. You want field 15? Just used `$(15)`. You want the field at `2 * 5`, just use `$(2*5)`. The only thing you can't do is specify negative field indexes.

Now you may be thinking to yourself: "what about field `$0`?" Well, lets find out with our next example:

#### Example 5: `awk` (easy)
Lets say we only want the lines in `example.csv` in which the p value is less than or equal to 0.01. To do this we would run the following command:
```
awk -F"," '$2 < 0.01 {print $0}' example.csv
```
which gives us:
```
XM_005088768.2, 0.005
XM_005088769.2, 0.00065
```
Cool! That doesn't answer what `$0` does. Ok, well lets take a closer look at those instructions:

-`-F","` - again, this tells `awk` to split the line by `,`.

-`$2 < 0.01` - here is our condition. We could specify it using an explicit `if` statement, but just stating the inequality is enough. This tells awk only to print those values in column 2 that are less than 0.05

-`{print $0}` - this command tells awk to print the 0th field, which really means the whole line. This is essentially an alternative to `grep` if your data is regularly delimited. (note that `awk` can also do regex if you want it to).

So there you have it. Use `awk` to zip through our tables and delimited data files. Now that we have a handle on the basic syntax, lets move on to some more conceptual problems.

#### Example 6: `awk` (easy)
In our directory we can find the file `example.gff`. This is a sample file, built from the first the first 100 lines of the _Aplysia californica_ genome build 3 `.gff` file. The files is in the regularly delimited _gene feature format_ or `.gff`. This is a format used to store genome annotations, so if you are doing any sort of genome or transcriptome work, you will no doubt run into one of these (or an outdate cousin, the `.gtf`).
The `.gff` file format has 9 columns delimited by tabs. The basic structure of a `.gff`'s columns' can be described as:
```
seqid	source	type	start	end	score	strand	phase	attributes
```
The 9th column is made up of another set of regularly delimited fields separated by `;`'s'. As you can see, this file type is just begging to be operated on with `awk`.

for this example, let's take a look at the 12th line of `example.gff` by using:
```
awk 'NR==12 {print $0}' example.gff
```
to get:
```
NW_004797271.1  Gnomon  mRNA    68472   74579   .       +       .       ID=rna0;Parent=gene0;Dbxref=GeneID:101845732,Genbank:XM_005088767.1;Name=XM_005088767.1;gbkey=mRNA;gene=LOC101845732;product=probable cytochrome c oxidase subunit 6A%2C mitochondrial;transcript_id=XM_005088767.1
```
So what did we do here? Notice our condition is `NR == 12`? Well, `NR` is a special, protected variable in `awk`. It stands for record number, and represents the current line being read. Just like `sed`, `awk` keeps track of how many lines its read, allowing you to access specific lines. Cool.
#### Example 7: `awk` (intermediate)
Now let's see if we can extract only the atributes column of `mRNA` lines within the first 50 lines of the file:

```
awk -F"\t" 'NR < 50 && $3 == "mRNA" {print $9}' example.gff
```
result:
```
ID=rna0;Parent=gene0;Dbxref=GeneID:101845732,Genbank:XM_005088767.1;Name=XM_005088767.1;gbkey=mRNA;gene=LOC101845732;product=probable cytochrome c oxidase subunit 6A%2C mitochondrial;transcript_id=XM_005088767.1
ID=rna1;Parent=gene1;Dbxref=GeneID:101855065,Genbank:XM_013085418.1;Name=XM_013085418.1;gbkey=mRNA;gene=LOC101855065;product=uncharacterized LOC101855065;transcript_id=XM_013085418.1
ID=rna2;Parent=gene2;Dbxref=GeneID:101845963,Genbank:XM_005088768.2;Name=XM_005088768.2;gbkey=mRNA;gene=LOC101845963;product=acetyl-coenzyme A transporter 1-like%2C transcript variant X1;transcript_id=XM_005088768.2
ID=rna3;Parent=gene2;Dbxref=GeneID:101845963,Genbank:XM_005088769.2;Name=XM_005088769.2;gbkey=mRNA;gene=LOC101845963;product=acetyl-coenzyme A transporter 1-like%2C transcript variant X2;transcript_id=XM_005088769.2
```
Nice! So what did we do here?

-`-F"\t"` - here we set our delimiter to a tab

-`NR < 50 && $3 == "mRNA"` - this is a little more complex than before. Here we have two conditions, `NR < 50` and `$3 == "mRNA"` stitched together by the logical operator `&&`. What this means is that only those lines that have the value of `mRNA` in the third column **and** a line number less than fifty will pass our criteria.

-`{print $9}` - this prints to screen only the 9th field of our passing lines.

Great! Now that result is pretty ugly. If all we wanted was that product field, we could just pipe the results to another simple `awk` command like so:
```
awk -F"\t" 'NR < 50 && $3 == "mRNA" {print $9}' example.gff | awk -F";" '{print $7}'
```
```
product=probable cytochrome c oxidase subunit 6A%2C mitochondrial
product=uncharacterized LOC101855065
product=acetyl-coenzyme A transporter 1-like%2C transcript variant X1
product=acetyl-coenzyme A transporter 1-like%2C transcript variant X2
```
Excellent! No we're cooking with gas. Now that we are warmed up, we'll do one last "advanced" `awk` command to introduce the idea of arrays.

#### Example 8: `awk` (advanced)
For this one we will use both `example.csv` and `example_products.csv`, so make sure not to get them confused. Here we will integrate all our `awk` skills thus far and more.
We are going to extract the trancripts in `example.csv` from `example_products.csv`:
```
awk -F"," 'NR==FNR && $2 < 0.01 {a[$1] = $1; next} ($1 in a){print $0}' example.csv example_products.csv
```
gives us:
```
XM_005088768.2,gbkey=mRNA,acetyl-coenzyme A transporter 1-like%2C transcript variant X1
XM_005088769.2,gbkey=mRNA,acetyl-coenzyme A transporter 1-like%2C transcript variant X2
```
Ok so what's going on in this command?:

-`NR==FNR && $2 < 0.01` - this is a little bit of code chicanery. The first condition is only satisfied when the record number is equal to the total records in the first file. That means the next section will only be executed when reading the first file and when the second filed is less than 0.01

-`{a[$1] = $1; next}` - here we create an array a[] and store the values of field one in it that passed our criteria.

-`($1 in a){print $0}` - here we check if the value in the first field of file 2 is somewhere in array a, and if it is, we print the whole line from file 2.

Wonderful. We've done some interesting stuff with `awk` and `sed`. Now we haven't discovered what the `awk` `sed`, and we may never known, but I can show you one last example when we use the two together.

#### Example 9: `awk` and `sed` together (advanced)
Lets try to recreate example_products.csv from example.gff like so:

```
awk -F"\t" '$3 == "mRNA" {print $9}' example.gff | sed 's/.*;Name=\(.*\);.*;product=\(.*\);.*/\1,\2/g' > products_replica.csv
```

Easy, right? At this point you are a `sed` and `awk` master, so I'm sure you can dissect the command yourself. While we've only just scratched the surface of what is possible with `awk` and `sed`, you know have the basic understanding to use them to wrangle most of your data. Remember to think creatively with your code, experiment until it works, RTFM, and that Google is your friend. Now get out there and write some scripts!
