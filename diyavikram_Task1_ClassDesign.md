# Task 1 — Design Your Own Data Model and Blueprint Class

**Submitted by:** Diya Vikram  
**Date:** March 3, 2026

---

## 1. Concept Review & Notes

### What is a Class in Kotlin?
A class in Kotlin is a blueprint or template for creating objects. It defines the structure (properties) and behavior (methods/functions) that objects of that class will have. Classes are fundamental to object-oriented programming and allow us to model real-world entities in code.

### Difference Between a Class and Object
- **Class:** A class is the blueprint or template (the abstract definition).
- **Object:** An object is a concrete instance of a class (a specific realization of that blueprint with actual values).

*Example:* `Recipe` is a class, but "Chocolate Cake" is an object (instance) of the `Recipe` class.

### Primary and Secondary Constructors
- **Primary Constructor:** Defined in the class declaration line with parameters. It's the main constructor used to initialize properties.
- **Secondary Constructor:** Defined using the `constructor` keyword inside the class body. Used when you need multiple ways to construct an object.

### Purpose of Init Blocks and Member Functions
- **Init Block:** Runs when an object is created. Used for initialization logic, validation, or printing messages.
- **Member Functions:** Methods that define the behavior or actions an object can perform.

### Simple Kotlin Data Class Example
```kotlin
data class Person(
    val name: String,
    val age: Int,
    val email: String
)
```

---

## 2. Designed Data Model: Recipe Management App

### Domain: Recipe Manager Application

I've chosen the **Recipe Manager** domain to create a comprehensive class that represents a recipe with multiple properties and methods.

### Class Design: `Recipe`

```kotlin
class Recipe(
    val id: String,
    val name: String,
    val ingredients: List<String>,
    val cookingTimeMinutes: Int,
    var isFavorite: Boolean = false,
    var rating: Double = 0.0
) {
    
    init {
        require(cookingTimeMinutes > 0) { "Cooking time must be positive" }
        require(rating in 0.0..5.0) { "Rating must be between 0 and 5" }
        println("✓ Recipe created: $name (ID: $id)")
    }
    
    // Function 1: Estimate cooking difficulty
    fun estimatedEffort(): String {
        return when {
            cookingTimeMinutes < 20 -> "Quick & Easy"
            cookingTimeMinutes in 20..45 -> "Medium"
            else -> "Advanced/Time-Intensive"
        }
    }
    
    // Function 2: Toggle favorite status
    fun toggleFavorite() {
        isFavorite = !isFavorite
        val status = if (isFavorite) "♥ Added to favorites" else "Removed from favorites"
        println(status)
    }
    
    // Function 3: Update rating
    fun setRating(newRating: Double) {
        if (newRating in 0.0..5.0) {
            rating = newRating
            println("Rating updated to $newRating stars")
        } else {
            println("Rating must be between 0 and 5")
        }
    }
    
    // Function 4: Get recipe summary
    fun getSummary(): String {
        val favoriteTag = if (isFavorite) "♥" else ""
        return "$favoriteTag $name - $cookingTimeMinutes min - ⭐$rating/5 - ${estimatedEffort()}"
    }
}
```

### Properties Explanation:
1. **id** (String): Unique identifier for the recipe
2. **name** (String): Name of the recipe
3. **ingredients** (List<String>): List of ingredients needed
4. **cookingTimeMinutes** (Int): Time required to prepare the dish
5. **isFavorite** (Boolean): Whether the user marked it as favorite
6. **rating** (Double): User's rating from 0 to 5 stars

---

## 3. Diagram and Object Instances

### UML-Style Class Diagram

```
┌─────────────────────────────────────┐
│           Recipe                    │
├─────────────────────────────────────┤
│ Properties:                         │
│ - id: String                        │
│ - name: String                      │
│ - ingredients: List<String>         │
│ - cookingTimeMinutes: Int           │
│ - isFavorite: Boolean               │
│ - rating: Double                    │
├─────────────────────────────────────┤
│ Methods:                            │
│ + estimatedEffort(): String         │
│ + toggleFavorite(): Unit            │
│ + setRating(Double): Unit           │
│ + getSummary(): String              │
└─────────────────────────────────────┘
```

### Object Instances

```kotlin
// Instance 1: Chocolate Chip Cookies
val recipe1 = Recipe(
    id = "R001",
    name = "Chocolate Chip Cookies",
    ingredients = listOf(
        "2 cups flour",
        "1 cup butter",
        "1 cup chocolate chips",
        "2 eggs",
        "1 tsp vanilla"
    ),
    cookingTimeMinutes = 25,
    isFavorite = true,
    rating = 4.8
)

// Instance 2: Pasta Carbonara
val recipe2 = Recipe(
    id = "R002",
    name = "Pasta Carbonara",
    ingredients = listOf(
        "400g pasta",
        "200g guanciale",
        "4 eggs",
        "100g Pecorino Romano",
        "Black pepper"
    ),
    cookingTimeMinutes = 20,
    isFavorite = false,
    rating = 4.5
)
```

### Kotlin Snippet: Calling Methods on Objects

```kotlin
fun main() {
    // Display summaries
    println(recipe1.getSummary())
    // Output: ♥ Chocolate Chip Cookies - 25 min - ⭐4.8/5 - Quick & Easy
    
    println(recipe2.getSummary())
    // Output: Pasta Carbonara - 20 min - ⭐4.5/5 - Quick & Easy
    
    // Toggle favorite on recipe2
    recipe2.toggleFavorite()
    // Output: ♥ Added to favorites
    
    // Update rating
    recipe1.setRating(5.0)
    // Output: Rating updated to 5.0 stars
    
    // Check effort level
    println("Recipe 1 difficulty: ${recipe1.estimatedEffort()}")
    // Output: Recipe 1 difficulty: Quick & Easy
}
```

---

## 4. Compose Integration Plan

### How to Display Recipe Objects in Compose UI

The Recipe class can be seamlessly integrated into a Jetpack Compose UI using a LazyColumn to display a list of recipes:

```kotlin
@Composable
fun RecipeListScreen(recipes: List<Recipe>) {
    LazyColumn {
        items(recipes) { recipe ->
            RecipeCard(recipe)
        }
    }
}

@Composable
fun RecipeCard(recipe: Recipe, onFavoriteClick: (Recipe) -> Unit = {}) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp),
        elevation = 4.dp
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text(recipe.name, style = MaterialTheme.typography.headlineSmall)
                IconButton(onClick = { onFavoriteClick(recipe) }) {
                    Icon(
                        imageVector = if (recipe.isFavorite) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
                        contentDescription = "Toggle favorite"
                    )
                }
            }
            
            Text("⏱️ ${recipe.cookingTimeMinutes} min - ${recipe.estimatedEffort()}")
            Text("⭐ ${recipe.rating}/5.0", style = MaterialTheme.typography.bodySmall)
            Text("Ingredients: ${recipe.ingredients.size} items", style = MaterialTheme.typography.bodySmall)
        }
    }
}
```

This design allows users to:
- View all recipes in a scrollable list
- See cooking time and difficulty at a glance
- Toggle favorite status with a heart icon
- View ratings and ingredient counts
- Easily extend with additional features (search, filter by difficulty, etc.)

---

## 5. Reflection

### What I Learned Designing This Class

Designing the `Recipe` class taught me the importance of thoughtful data modeling. By identifying key properties upfront, I created a flexible structure that accurately represents a recipe while remaining simple to use. The inclusion of validation in the `init` block showed me how encapsulation protects data integrity. Writing multiple methods (`estimatedEffort()`, `toggleFavorite()`, `getSummary()`) demonstrated how behavior should be tied to the object itself, not scattered throughout the app. I also realized that good naming conventions and clear method purposes make code self-documenting and easier to maintain.

### Why Modeling Data Early Helps

Modeling data early in the development process is crucial because it acts as the foundation for the entire application. When the data structure is well-designed, the UI logic becomes straightforward—I can easily bind Recipe objects to Compose components without convoluted transformations. Early modeling also prevents the need for costly refactoring later; if I define properties and methods correctly from the start, UI, business logic, and data persistence layers align naturally. Additionally, a clear data model makes it easier for team members to understand the app's structure and collaborate effectively. Finally, proper encapsulation through well-designed classes makes testing simpler and the codebase more maintainable.

---

## Summary

This Task 1 submission demonstrates:
✓ Comprehensive understanding of Kotlin classes and OOP concepts  
✓ A real-world data model (Recipe) with 6 meaningful properties  
✓ 4 functional methods with practical use cases  
✓ Validation logic in the init block  
✓ 2 concrete object instances with realistic data  
✓ A practical Compose UI integration example  
✓ Critical reflection on the learning process  

**Total Learning Outcomes:** Classes structure data, encapsulation protects it, and well-designed models make UI integration seamless.
```
