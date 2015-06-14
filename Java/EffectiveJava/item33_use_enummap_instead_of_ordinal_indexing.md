# Use EnumMap instead of oridinal indexing

Occasionally you may see code that uses the `ordinal` method to index into an array. Here is an example:

```java
public class Herb {
    public enum Type {
        ANNUAL, PERENNIAL, BIENNIAL
    }

    private final String name;
    private final Type type;

    public Herb(String name, Type type) {
        this.name = name;
        this.type = type;
    }
}
```

Now suppose you hava an array of herbs representing the plants in a garden, and you want to list these plants organized by type(annual, perenial, or biennial). To do this, you construct three sets, one for each type, and iterate through the garden, placing each herb in the appropriate set. Here is the code to do this:

```java
Herb[] garden = new Herb[10];

Set<Herb>[] herbsByType = (Set<Herb> [])new Set[Type.values().length];
for (int i = 0; i < herbsByType.length; i++) {
    herbsByType[i] = new HashSet<Herb>();
}

for (Herb herb: garden) {
    herbsByType[herb.type.ordinal()].add(herb);
}

for (int i = 0; i < herbsByType.length; i++) {
    System.out.printf("%s: %s%n", Herb.Type.values()[i], herbsByType[i]);
}
```

This technique works, but it is fraught with problems. Because arrays, are not compatible with generic, the program requires an unchecked cast and will not compile cleanly. Because the array does not know what its index represents, yo have to label the output manually. But the most serious problem with this technique is that when you access an array that is indexed by an enum's ordinal, it is your responsibility to use the correct `int` value; `int` do not provide the type safety of enums. If you use the wrong value, the program will silently do the wrong thing or -- if you're lucky -- throw an `ArrayIndexOfBoundsException`.

Luckily, there is a much better way to achieve the same effect:

```java
Herb[] garden = new Herb[10];
Map<Herb.Type, Set<Herb>> herbsByType = new EnumMap<Type, Set<Herb>>(Herb.Type.class);
for (Type type : Herb.Type.values()) {
    herbsByType.put(type, new HashSet<Herb>());
}
for (Herb herb : garden) {
    herbsByType.get(herb.type).add(herb);
}

System.out.println(herbsByType);
```

The program is shorter, clearer, safer, and comparable in speed to the original version. There is no unsafe cast; no need to label the output manually, as the map keys are enums that know how to translate themselves to printable strings; and no possibility for error in computing array indices.

And here is another situation you may encounter -- an array of arrays indexed by orindinals used to represent a mapping from two enum values. For example, this program uses such an array to map two phases to a a phase transition:

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
    }

    private static final Transition[][] TRANSITIONS = {
            {null, Transition.MELT, Transition.SUBLIME},
            {Transition.FREEZE, null, Transition.BOIL},
            {Transition.DEPOSIT, Transition.CONDENSE, null}
    };

    public static Transition from(Phase src, Phase dst) {
        return TRANSITIONS[src.ordinal()][dst.ordinal()];
    }
}
```

This program works and may even appear elegant, but appearances can be deceiving. Like the simpler herb garden example above, the compiler has no way of knowing the relationship between ordinals and array indices. If you make a mistake in the transition table, or forget to update it when you modify the `Phase` or `Phase.Transition` enum type, your program will fail at runtime. The failure may take the form of an `ArrayIndexOfBoundsException`, a `NullPointerException`, or(worse) silent erroneous behavior. And the size of the table is quadratic in the number of phases, even if the number of non-null entries is smaller.

Again, you can do much better with `EnumMap`:

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        final Phase src;
        final Phase dst;

        Transition(Phase src, Phase dst) {
            this.src = src;
            this.dst = dst;
        }
    }

    private static final Map<Phase, Map<Phase, Transition>> m  = new EnumMap<Phase, Map<Phase, Transition>>(Phase.class);

    static {
        for (Phase phase : Phase.values()) {
            m.put(phase, new EnumMap<Phase, Transition>(Phase.class));
        }
        for (Transition transition : Transition.values()) {
            m.get(transition.src).put(transition.dst, transition);
        }
    }

    public static Transition from(Phase src, Phase dst) {
        return m.get(src).get(dst);
    }

}
```

In summary, **it is rarely apropriate to use ordinals to index arrays: use EnumMap instead.** If the reletionship that you are representing is multidimensional, use `EnumMap<..., EnumMap<...>>`. This is a special case of the general principle that application programmers should rarely, if ever, use `Enum.ordinal`.