## Abstract classes, implements and @Override

I didn't have much idea about the purpose or definition of abstract classes, but since Intellij started to assert its power things begin to dawn on me.

My idea of Java was that it was more efficient because (1) it was statically-typed and (2) its object-oriented character allowed for well-organized object hierarchies and reusable code. It turns out I missed something important.

Abstract classes and the whole 'implements' thing allows you to force others to create classes according to your specifications. A software architect can create a design for a project, specify all the classes, fields and methods required, and then write an abstract class for each of them. Subsequently others can write the bodies of the classes, implementing the abstract classes handed to them, thus guaranteeing that their work will fit in the grand design.

You can do this in a team but as well in your own pet project. First you design, then you create the abstrct classes, and then you use those abstract classes to keep yourself within the guardrails.

I'm gonna try this in my project. The separation of design and implementation sounds extremely attractive to me. It might not work, but if it works, it might be Java's best feature.