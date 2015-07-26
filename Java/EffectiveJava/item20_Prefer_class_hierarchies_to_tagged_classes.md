### Item20 : Prefer class hierarchies to tagged classes

----------

Occasionlly you may run across a class whose instances come in two or more flavors and contain a *tag* field indicating the flavor of the instance. For example:

```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE}
    
    final Shape shape;
    
    // these fields are used only if the shape is rectangle
    double length;
    double width;
    
    // this field is used only if the shape is circle
    double radius;
    
    // constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    
    // constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError();
        }
    }
}
```

Such *tagged classes* hava numerous shortcommings. They are cluttered with boilerplate, including enum declarations, tag fields, and switch statements. Read-ability is further harmed because multiple implementations are jumbled together in a single class. Memory footprint is increased because instances are burdened with irrelevant fields belonging to other flavors. Fields can't be made final unless constructors initialize irrelevant fields, resulting in more boilerplate. Constructors must set the flag tag field and initialize the right data fields with no help from the compiler: if you initialize the wrong fields, the program will fail in runtime. You can't flavor, you must remember to add a case to every switch statement, or the class will fail at runtime. Finally, the data type of an instance gives no clue as to its flavor. In short, **tagged classes are verbose, error-prone, and ineffecient**.

And here is the version uses class hierarchy:

```java
public abstract class Figure {
    
    abstract double area();
    
}

class Circle extends Figure {
    
    final double radius;
    
    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {

    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override
    double area() {
        return length * width;
    }
}
```

This class hirearchy corrects every shortcomming of tagged classes noted previously. The code is simple and clear, containing none of the boilerplate found in the original. The implementation of each flavor is allotted its own class, and none of these classes are encumbered by irrelevant data fields.

Another advantage of class hierarchies is that they can be made to reflect natural hierarchical relationships among types, allowing for increased flexibility and better compile-time checking.

#### Summary

In summary, tagged classes are seldom appropriate. If you are tempted to write a class with an explicit tag field, think about whether the tag could be eliminated and the class replaced by hierarchy. When you encounter an existing class with a tag field, consider refactoring it into a hierarchy.