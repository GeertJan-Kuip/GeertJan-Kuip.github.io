## Don't pass around SQL ResultSet

My pet project includes the printing of SQLIte tables to a Java Swing JTextPane, for which I use a html format.

My initial idea to make it work was to pass the SQL resultset object from the SQLite class to some other class where I could parse it into a string with the right html tags. That didn't work out well, as it seems that databases connections can only be closed after the resultset has been processed. Having the resultset elsewhere makes it compliceted to close the connection. SOme thread unsafety might be involved as wel.

My workaround was to process the resultset in the SQL class itself, more specifically within the method that generated the resultset. It is not pretty, having a method do two different things instead of one and having the SQL class doing formatting. But for now it works.