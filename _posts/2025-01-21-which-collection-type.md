## Which collection type

Since I discovered Java ArrayList, I use it for almost anything, mainly because it can do almost anything. But being single minded in this might be a bit unsophisticated so I explored the topic.

What I concluded about collections is that the most important reason to use one over the other type is code readability. If you declare a list, you tell the person that reads your code that you want to be able to iterate over it and, in case of ArrayList, be able to add values to it, whether duplicates or not. If you declare a set, you tell others that it will be used to store unique values. You won't iterate over the set but you will do lookups to see wether a certain value is present. And if you declare a map you signal to the reader of the code that you needed a dictionary.

Oh, and there is also performance. But that is almost never an issue.