## Revisting an old project

In 2020 I created an interactive 3D data visualisation of research data related to a paper I wrote with political psychologist Mark Dechesne. The visualisation can be found [here](https://geertjan-kuip.github.io/politicologenetmaal-2020-data-visualisation/) (just go wild on the buttons to see the dots moving and to get different camera views).

Being seriously into programming now, I decided to have a look at what I created more than four years ago. I found the following:

- I put 90% of relevant code, including Javascript and research data, in the index.html file
- The Javascript section in the index.html has the following structure: 64 variables are declared, then 6 functions are called. Subsequently you find 32 function declarations.
- The research data, originally a .csv file, is stored as a string variable named 'csvtext,' as one of the 64.
- Some research data is hardcoded in function bodies.
- Based on file modification dates, creating the whole thing must have taken a little more than two weeks.

Probably this is what you get when you code only to get a presentable result and no one else will have to work with your code. I have no regrets and the audience, used to 2D box- and scatter plots, was actually impressed with the fancy 3D animations. 

One interesting take is that I took a look at the code of the plugin I used, [Three.js](https://threejs.org/). That is something impressive and I want to study it better. The makers have good documentation on their website it seems, lucky me.

