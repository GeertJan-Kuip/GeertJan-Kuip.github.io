## You can ask databases anything 

In my Python days I used to make series of consecutively executed SQLite queries that in the end would give me the desired result. Often I looped through resultsets to refine the search further, all under the assumption that making too difficult requests was either impossible of would take too much thought.

Over the last week I have begun to explore the possibilities of sql a bit further and now I write larger, nested queries that give me the desired result. I see 3 upsides:

- I get better in formulating queries
- Readability
- Execution speed

What I hadn't realized is that you can nest queries as deep as you like, that you can using BETWEEN with a JOINT statement, and that there is a list of so-called [window-functions](https://www.sqlitetutorial.net/sqlite-window-functions/) I didn't know about that extend the range of possibilities quite a bit. Here is one I with JOIN/BETWEEN:

```
SELECT classes.representation, classes.id AS classid, classes.file, tokens.line, tokens.id AS tokensid FROM classes 
INNER JOIN tokens ON tokens.id BETWEEN classes.token_id AND classes.token_id_end 
WHERE tokens.token = ? 
AND 
tokens.id != ? 
AND classes.representation != ? 
```

and here one with double nesting and a windows function (LAG):

```
SELECT id, token, place, line, file, representation FROM 
(
	SELECT *, 
	LAG(token, 1, 0) OVER (PARTITION BY tokens.file ORDER BY tokens.id) AS prevToken, 
	LAG(dictionary.representation,1,"-") OVER (PARTITION BY tokens.file ORDER BY tokens.id) AS prevRepresentation 
	FROM tokens INNER JOIN dictionary ON tokens.token = dictionary.id 
	WHERE (file, line) IN 
	(
		SELECT file, line FROM tokens WHERE token=
		(
                            SELECT id FROM DICTIONARY WHERE representation='class'
		)
	)
)
WHERE prevRepresentation = 'class' 
```

It is actually not that difficult once you start to organize the query in separate parts and believe that any result can be generated. It can.
