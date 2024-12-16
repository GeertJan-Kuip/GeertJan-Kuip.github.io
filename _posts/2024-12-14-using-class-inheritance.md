## Class inheritance

My current pet project is a static website that allows users to click the name of a Javascript method name to see what it actually does. Given the large number of methods this project can be scaled up until it covers the totality of all methods.

Right now it only covers methods applicable to arrays, and I wrote a special class for that. I can add extra classes for other types, such as maps, strings and objects.

Given the similarities between these classes, there is logic in creating a parent class containing the shared methods and properties. I'm planning to do that, but right now I'm reading the book 'Design Patterns' which sums up the downsides of using class inheritance. 'Favor object composition over class inheritance,' it states. 

I thought I was being smart using class inheritance. Now I'm a bit confused.
