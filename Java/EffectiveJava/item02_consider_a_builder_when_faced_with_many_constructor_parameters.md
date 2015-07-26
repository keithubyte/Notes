### Item 02 : Consider a builder when faced with many constructor parameters

----------

Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameter. Consider the case of a class representing the `Nutrition Facts` label that appears on packaged foods. These labels have a few required fields -- serving size, servings per container, and calories per serving -- and over twenty optional fieds -- total fat, saturated fat, trtans fat, cholesterol, sodium, and so on. Most products have nonzero values for only a few of these optional fields. Here is the `Nutrition Facts JavaBean`:

```java
/**
 * JavaBean for Nutrition Facts.
 * Created by keith on 15/3/8.
 */
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            this.calories = val;
            return this;
        }

        public Builder fat(int val) {
            this.fat = val;
            return this;
        }

        public Builder sodium(int val) {
            this.sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            this.carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }

    }

    public NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    @Override
    public String toString() {
        return "NutritionFacts : \n"
                + "servingSize = " + servingSize + "\n"
                + "servings = " + servings + "\n"
                + "calories = " + calories + "\n"
                + "fat = " + fat + "\n"
                + "sodium = " + sodium + "\n"
                + "carbohydrate = " + carbohydrate + "\n";
    }
}
```

And the client is:

```java
/**
 * Client to use the NutritionFacts JavaBean
 * Created by keith on 15/3/8.
 */
public class NutritionFactsClient {
    public static void main(String[] args) {
        NutritionFacts nfs = new NutritionFacts.Builder(240, 8)
                .calories(100)
                .fat(50)
                .sodium(35)
                .carbohydrate(27)
                .build();
        System.out.println(nfs);
    }
}
```

#### Summary

The `Build Pattern` is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, expecially if most of those parameters are optional.
