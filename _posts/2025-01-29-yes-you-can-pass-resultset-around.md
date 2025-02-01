## Yes, you can pas ResultSet around 

In an earlier post I mentioned the trouble I got into when passing a reference to a SQLIte ResultSet object to another class. My advice to myself was to never do it again.

As I didn't listen to myself, I tried it again and now it worked ok. The thing is probably that you need to be carefull not to close connections or the ResultSet itself while it is till being accessed by some process.

I read some more about the ResultSet class in the Oracle documentation and created a visual metaphor in my head that makes it more understandable, at least for me. A ResultSet is a two-dimensional crop field living in your RAM that can be harvested using "while (resultSet.next())" by a machine that iterates the set row by row, only picking the crops that have the types and names that you specify. The machine is lightning fast and can skip rows or you can tell it to go to some specific row. SQLIte doesn't allow you to go backwards in the set, other database types do. The ResultSet needs to be closed after using it.
