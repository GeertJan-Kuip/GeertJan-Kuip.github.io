# Finding files in Linux

Today I wondered whether any JAR files were still on my server. I had copied some stuff and either deleted it or copied it into the tmp directory that is automatically being emptied, either after a reboot or at some scheduled time.

## Working with stdout and stderror

### The search

Using the find command was not as easy as I thought it was, at least not for a full search in all accessible directories. I ended up with this one that did what I wanted:

```
find . -name '*.jar' 2>&1 | grep -v "Permission denied"
```

I ran it while in the root folder (`/`) and it gave me all jar files, and at the same time excluded all folders that I was not allowed to enter. The line `find . -name '*.jar'` resulted in many output lines like this one:

```
find: ‘./etc/cni/net.d’: Permission denied
```

These lines with their 'Permission denied' message fill the whole terminal, make it impossible to just use `find . -name '*.jar'`. But what does the line with the `2>&1` and the grep part then do?

### The meaning of `2>&1`

I found [this link](https://tldp.org/LDP/abs/html/io-redirection.html) useful. [This one](https://tldp.org/LDP/abs/html/ioredirintro.html) from the same tutorial is useful too. 

From these tutorials: _A command expects the first three file descriptors to be available. The first, fd 0 (standard input, stdin), is for reading. The other two (fd 1, stdout and fd 2, stderr) are for writing._ Here we have the meaning of numbers 1 and 2. The `>` redirects output to something. In this case the output of stderr is redirected to that of stdout. `2>&1` means that both the errors ('Permission denied') and the regular output will now both be in stdout.

The ampersand (`&`) is required because otherwise Linux thinks that '1' is the name of a file that you want to create. 

### About stdout and stderr

The contents of stdout and stderr are streams, not files, and they both write to the terminal. This means that actually the two streams are mixed but you won't see this until you know it. 

Given that stdout and stderr are two different streams, you can do another trick as well to suppress error messages in your terminal:

```
find . -name '*.jar' 2>/dev/null
```

What this does is redirecting the stderr output (2) to `/dev/null`, which is a sort of black hole for output. You use this black hole for output you aren't interested in. Substituting 1 for 2 in the line above will result in the terminal only printing the errors. 

### Different meanings of &

If you write `find . -name '*.jar' &>/dev/null`, the `&` signifies 'both 1 and 2' or 'both stdout and stderr'. This is a different meaning as in '2>&1' where & indicates that 1 is stdout, not a file named 1. Note that when using & as in `cmd &`, the & indicates that the command must run in the background, which is a third possible meaning of the ampersand.

## Skipping directories with -prune

### Using -prune

On some forum someone showed how you could use the `-prune` option to exclude specific directories from your search. I didn't understand it and the documentation is not very verbose on this option so I asked ChatGPT who gave me this line that excludes all /proc folders from the search:

```
find / \( -path /proc -prune \) -o -name '*.jar' -print
```

It is an odd one that needs clarification and this is what ChatGPT learned me:

#### -o stands for OR

The -o divides the line in two true/false statements. If the left is evaluated to true, the right statement will not be evaluated anymore because the endresult is already determined. Only if the left statement is false, the statement on the right will be evaluated.

The statement left is `-path /prc -prune`, the statement right is `-name '*.jar' -print`.

Btw -a means logical AND.

#### `\(` because parentheses need to be escaped

Using parentheses means that you create groups, which is not what you want here according to Chat. So the escaped parentheses have some use but grouping is not what is required here.

#### Evaluating `-path /proc -prune`

`-path /proc -prune` says as much as: if the path of the file is /proc, skip it, don't descend in it and return true. Note that -path and -name are so-called 'tests' (see [manpage](https://www.man7.org/linux/man-pages/man1/find.1.html)) that return true or false. 

What the test thus says is 'if the path contains /proc, skip it. Btw you can do multiple tests with a line like this:

```
\( -path /proc -path /sys -o -path /dev \) -prune
```

#### Evaluating `-name '*.jar' -print`

Here the result is true if the file or directory name ends with '.jar' and in this case, the print command will be executed. Note that in this context the -print command is the opposite of the -prune command. The former prints, the latter dismisses.

Th `-print` at the end can be omitted, but it is safe to use it as it guarantees that the requested output will be printed.


