## Embedding SQLite 

I want my Java program to analyze Java code. To do so I'm splitting Java files into separate tokens and I store those tokens in a SQLite database in a table that has fields for token-id, the place of the token in the line, the line number and the file reference. I remember reading a paper of the Google founders in which they argued that this is actually very efficiënt.

Creating my SQLite class felt somewhat familiar, as I have been working a lot with SQLite while doing data analysis with Python. Databases have their own set of instructions, no matter what programming language you use to manipulate them. After having created a connection and with the appropriate name 'connection' I created two tables and then automatically wrote this line:

```connection.commit();```

The compiler returned an error, indicating that 'commit()' wasn't appropriate. It turns out that Java has some 'autocommit' flag built in its library, defaulting to 'true'. It means that each database update is applied immediately, which is terribly inefficient.

I set autocommit to 'false' and for for a short second felt like an expert.
