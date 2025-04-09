## Comparator method chaining

I was not aware of the fact that interface Comparator provdides some static and default methods that can be chained. Such a chain facilitates the creation of more complex Comparator lambdas, in which different variables of the object can be used for sorting. I like the functional and readable style it delivers and it is part of the exam. Below a sample I made in which 8 Box2 objects are being sorted, first by width, then depth, then height. Subsequently the order is reversed.

```
import java.util.*;

class Box2{
    int width, depth,height;

    public Box2(int width, int depth, int height) {
        this.width = width;
        this.depth = depth;
        this.height = height;
    }
    public int getWidth() {return width;}
    public int getDepth() {return depth;}
    public int getHeight() {return height;}
}

public class Compare {

    static ArrayList<Box2> boxes = new ArrayList<>();

    public static void main(String... args) {

        int i = 0;
        Random random = new Random();
        int w=0,d=0,h=0;
        while (i<8){  // create 8 random Box2 objects
            w = random.nextInt(3) + 2;
            d = random.nextInt(3) + 2;
            h = random.nextInt(3) + 2;

            boxes.add(new Box2(w,d,h));
            i++;
        }
        
        // creation of a Comparator lambda. Normally I would create this in traditional lambda style without method references
        Comparator<Box2> comp = Comparator.comparing(Box2::getWidth)
                .thenComparing(Box2::getDepth)
                .thenComparing(Box2::getHeight)
                .reversed(); // reversed() is a method in Comparator

        // streaming the array with 8 boxes of different sizes. The comparator is used as parameter for sorted()
        boxes.stream().sorted(comp)
                .forEach(b->{
            System.out.printf("%s - %s - %s\n", b.getWidth(), b.getDepth(), b.getHeight());
        });
    }
}

-- result
4 - 3 - 2
4 - 2 - 3
4 - 2 - 3
3 - 4 - 3
2 - 4 - 2
2 - 4 - 2
2 - 2 - 3
2 - 2 - 2

```