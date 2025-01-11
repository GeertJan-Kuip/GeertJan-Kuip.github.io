## Zlib and Git 

My Java project requires reading the files stored in the hidden .git folder of projects. I did my research and learned a bit about compression techniques and -standards.

With zlib, used for git files, a file is compressed by finding all the duplicate sequences of bytes and then backreferencing all those duplicates to the first occurence. At least that's how I understand it. Subsequently, the system of character encoding is transformed using Huffman encoding in a way that the most commonly used characters get the lowest amount of bits to encode them.

Java is perfectly capable of compressing and decompressing files, but I wonder if I would be able to write an 'inflator' and 'deflator' myself. I might have a look at the Java source code and I just found the Github repository with the actual C code.

Furthermore I wonder if it is possible to unpack the pack files that Git creates to compress things even more. If not I need to use Git itself but it would be preferable to do it in Java. To be continued.
