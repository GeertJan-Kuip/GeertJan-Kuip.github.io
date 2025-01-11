## Drawing, animating in Swing

For my pet project I want to draw a network graph on a canvas in the UI. From network sociology I knew there are algorithms that can draw beautiful network graphs with many nodes, so I tried to find such an algorithm.

I found one [here](https://editor.p5js.org/JeromePaddick/sketches/bjA_UOPip). The maker wrote an algorithm in Javascript using the p5.js library and it results in a rather smooth animation of a 'force-directed' network graph with 100 nodes. I'm going to try to recreate it in Java, using the Graphics2D class, and I'm curious how the performance will be. I fear a lower framerate but it might work out well. 