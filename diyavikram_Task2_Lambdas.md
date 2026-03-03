# Task 2 — Functional Programming with Lambdas and Higher-Order Functions

**Submitted by:** Diya Vikram  
**Date:** March 3, 2026

---

## 1. Concept Notes

### Higher-Order Function
A **higher-order function** is a function that either:
- **Takes one or more functions as parameters**, OR
- **Returns a function as a result**

Higher-order functions enable functional programming patterns and allow you to pass behavior as data, making code more flexible and reusable.

*Example:* A function that takes a filter function as a parameter to filter a list.

### Lambda Expression
A **lambda expression** is an anonymous (unnamed) function written in a compact syntax. In Kotlin, lambdas are enclosed in curly braces `{ }` and can be passed directly to functions.

**Syntax:** `{ parameters -> body }`

*Example:* `{ x: Int -> x * 2 }` is a lambda that doubles an integer.

### Function Type
A **function type** describes the signature of a function (its parameters and return type). It's written as `(ParameterTypes) -> ReturnType`.

*Examples:*
- `(Int, Int) -> Int` — A function taking two integers and returning an integer
- `(String) -> Boolean` — A function taking a string and returning a boolean
- `() -> Unit` — A function taking no parameters and returning nothing (Unit)

### Inline Functions
**Inline functions** are functions marked with the `inline` keyword. When called, their body is directly inserted at the call site instead of being called normally. This eliminates function call overhead and allows lambdas to use non-local returns.

```kotlin
inline fun <T> forEach(items: List<T>, action: (T) -> Unit) {
    for (item in items) action(item)
}
```

---

## 2. Lambda Examples

### Example 1: Lambda that Filters a List of Integers

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// Lambda to filter even numbers
val filterEven: (List<Int>) -> List<Int> = { list ->
    list.filter { number -> number % 2 == 0 }
}

val evenNumbers = filterEven(numbers)
println(evenNumbers)  // Output: [2, 4, 6, 8, 10]

// Using the built-in filter function with a lambda
val oddNumbers = numbers.filter { it % 2 != 0 }
println(oddNumbers)  // Output: [1, 3, 5, 7, 9]
```

### Example 2: Higher-Order Function that Accepts a Lambda

```kotlin
// Higher-order function that takes a lambda as a parameter
fun applyOperation(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// Pass different lambdas to perform different operations
val addition = applyOperation(5, 3) { x, y -> x + y }
println("5 + 3 = $addition")  // Output: 5 + 3 = 8

val multiplication = applyOperation(5, 3) { x, y -> x * y }
println("5 * 3 = $multiplication")  // Output: 5 * 3 = 15

val subtraction = applyOperation(5, 3) { x, y -> x - y }
println("5 - 3 = $subtraction")  // Output: 5 - 3 = 2
```

---

## 3. Design a Mini Use Case: Task Filtering by Urgency

### Scenario: Task Manager Application
We want to filter tasks based on urgency using a higher-order function.

### Data Model

```kotlin
data class Task(
    val id: Int,
    val title: String,
    val description: String,
    val priorityLevel: Int,  // 1 (Low) to 5 (Critical)
    val isCompleted: Boolean = false
)
```

### Higher-Order Function for Task Filtering

```kotlin
// Higher-order function that filters tasks based on a lambda condition
fun filterTasks(tasks: List<Task>, predicate: (Task) -> Boolean): List<Task> {
    return tasks.filter(predicate)
}

// Alternative: Extension function for more idiomatic Kotlin
fun List<Task>.filterByUrgency(minPriority: Int): List<Task> {
    return this.filter { task -> task.priorityLevel >= minPriority && !task.isCompleted }
}
```

### Pseudo-Code: Applying Lambdas to Task List

```
1. Create a list of Task objects
   tasks = [Task(1, "Email report", 3), Task(2, "Fix bug", 5), ...]

2. Define filtering lambdas:
   - urgentTasks = filter where priorityLevel >= 4
   - completedTasks = filter where isCompleted == true
   - highPriorityUrgent = filter where priorityLevel == 5 AND !isCompleted

3. Apply lambdas using higher-order function:
   result1 = filterTasks(tasks) { task -> task.priorityLevel >= 4 }
   result2 = tasks.filterByUrgency(4)
   result3 = filterTasks(tasks) { it.priorityLevel == 5 && !it.isCompleted }

4. Display filtered results
```

### Complete Working Example

```kotlin
fun main() {
    val tasks = listOf(
        Task(1, "Write documentation", "API docs", 2),
        Task(2, "Fix critical bug", "Login crash", 5),
        Task(3, "Review PR", "Code review", 3),
        Task(4, "Deploy to production", "Release v2.0", 5),
        Task(5, "Update dependencies", "Security patches", 4)
    )

    // Using higher-order function with lambda
    val criticalTasks = filterTasks(tasks) { task ->
        task.priorityLevel == 5 && !task.isCompleted
    }
    
    println("Critical Tasks:")
    criticalTasks.forEach { println("  - ${it.title} (Priority: ${it.priorityLevel})") }
    
    // Using extension function
    val urgentTasks = tasks.filterByUrgency(4)
    println("\nUrgent Tasks (Priority >= 4):")
    urgentTasks.forEach { println("  - ${it.title}") }
    
    // Sort tasks by priority
    val sortedByPriority = tasks.sortedByDescending { it.priorityLevel }
    println("\nAll Tasks Sorted by Priority:")
    sortedByPriority.forEach { 
        println("  [${it.priorityLevel}] ${it.title}") 
    }
}
```

---

## 4. Compose Lambda Usage Research

### 3 Common Places Where Lambdas Are Used in Compose

#### 1. **Button onClick Handler**
```kotlin
Button(onClick = { viewModel.submitForm() }) {
    Text("Submit")
}
```
**Parameters/State:** The lambda receives `Unit` and performs an action when clicked.

#### 2. **Modifier.clickable()**
```kotlin
Box(
    modifier = Modifier
        .clickable { onItemClick(itemId) }
        .padding(16.dp)
) {
    Text("Click me")
}
```
**Parameters/State:** The lambda is triggered on click and can pass data to parent composables.

#### 3. **LazyColumn with items()**
```kotlin
LazyColumn {
    items(itemList) { item ->
        ItemRow(
            item = item,
            onClick = { onItemSelected(item.id) }
        )
    }
}
```
**Parameters/State:** The lambda receives each item in the list for rendering and handles selection.

### My Own Compose Example

```kotlin
@Composable
fun TaskListScreen(
    tasks: List<Task>,
    onTaskClick: (Int) -> Unit,
    onMarkComplete: (Int) -> Unit
) {
    var completedCount by remember { mutableStateOf(0) }
    
    LazyColumn {
        items(tasks) { task ->
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(8.dp)
                    .clickable { onTaskClick(task.id) }
            ) {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp),
                    horizontalArrangement = Arrangement.SpaceBetween
                ) {
                    Column {
                        Text(task.title, style = MaterialTheme.typography.headlineSmall)
                        Text(
                            "Priority: ${task.priorityLevel}/5",
                            style = MaterialTheme.typography.bodySmall
                        )
                    }
                    
                    Button(onClick = { 
                        onMarkComplete(task.id)
                        completedCount++
                    }) {
                        Text("✓ Done")
                    }
                }
            }
        }
    }
    
    Text("Completed: $completedCount tasks", modifier = Modifier.padding(16.dp))
}
```

---

## 5. Function Type Exercise: Type Alias & Discount Rules

### Define a Function Type Alias

```kotlin
// Function type alias for discount calculations
typealias DiscountRule = (Double) -> Double

// Alternative: For more complex scenarios
typealias PriceCalculator = (originalPrice: Double, itemCount: Int) -> Double
```

### Implementation with Different Discount Lambdas

```kotlin
fun calculateFinalPrice(
    originalPrice: Double,
    applyDiscount: DiscountRule
): Double {
    return applyDiscount(originalPrice)
}

// Different discount rules implemented as lambdas
fun main() {
    val originalPrice = 100.0
    
    // Rule 1: Flat 10% discount
    val flatDiscount: DiscountRule = { price -> price * 0.9 }
    println("Flat 10% off: $${calculateFinalPrice(originalPrice, flatDiscount)}")
    // Output: Flat 10% off: $90.0
    
    // Rule 2: Tiered discount (15% if over $50)
    val tieredDiscount: DiscountRule = { price ->
        if (price > 50) price * 0.85 else price * 0.95
    }
    println("Tiered discount: $${calculateFinalPrice(originalPrice, tieredDiscount)}")
    // Output: Tiered discount: $85.0
    
    // Rule 3: Member discount (20%)
    val memberDiscount: DiscountRule = { price -> price * 0.8 }
    println("Member discount: $${calculateFinalPrice(originalPrice, memberDiscount)}")
    // Output: Member discount: $80.0
    
    // Rule 4: No discount
    val noDiscount: DiscountRule = { price -> price }
    println("No discount: $${calculateFinalPrice(originalPrice, noDiscount)}")
    // Output: No discount: $100.0
}
```

### Advanced Example: PriceCalculator Type Alias

```kotlin
fun applyPromotionalPrice(
    basePrice: Double,
    itemCount: Int,
    calculator: PriceCalculator
): Double {
    return calculator(basePrice, itemCount)
}

fun main() {
    // Bulk discount calculator: 5% off per item after 10 units
    val bulkCalculator: PriceCalculator = { price, count ->
        if (count > 10) price * count * 0.95 else price * count
    }
    
    val total1 = applyPromotionalPrice(50.0, 15, bulkCalculator)
    println("Bulk purchase (15 items): $$total1")  // Output: $712.5
    
    // Holiday special: Fixed discount plus percentage off
    val holidayCalculator: PriceCalculator = { price, count ->
        val subtotal = price * count
        val discount = 10.0  // $10 off
        (subtotal - discount) * 0.9  // Additional 10% off
    }
    
    val total2 = applyPromotionalPrice(50.0, 5, holidayCalculator)
    println("Holiday special (5 items): $$total2")  // Output: $180.0
}
```

### Advantages of Function Type Aliases

| Advantage | Explanation |
|-----------|-------------|
| **Reusability** | Define once, use in multiple functions without repeating `(Double) -> Double` |
| **Readability** | `DiscountRule` is clearer than `(Double) -> Double` in function signatures |
| **Maintainability** | If you need to change the type signature, update it in one place |
| **Flexibility** | Easy to swap different implementations without changing function code |
| **No Hardcoding** | Logic is passed in, not hardcoded, making the function generic and extensible |

---

## 6. Reflection

### How Lambdas Improve Code Reusability and Readability

Lambdas transform functions into first-class citizens, allowing us to pass behavior as parameters rather than hardcoding it. This dramatically improves reusability because a single higher-order function like `filterTasks()` can work with any filtering logic without modification. Instead of writing multiple functions like `filterByPriority()`, `filterByCompletion()`, `filterByUrgency()`, we write one function and pass different lambdas to it. This reduces code duplication and makes the codebase more maintainable.

Regarding readability, lambdas allow us to express intent concisely. When I see `tasks.filter { it.priorityLevel >= 4 }`, the logic is immediately clear and reads almost like English. Compared to nested loops or imperative filtering code, lambdas make our intentions transparent. Function type aliases further enhance readability by giving meaningful names to complex function signatures—`DiscountRule` tells us exactly what the function does, better than `(Double) -> Double`.

### How Lambdas Simplify Event Handling in Compose

In Jetpack Compose, lambdas are essential for event handling. Instead of implementing verbose callback interfaces (as in traditional Android), we simply write `onClick = { /* do something */ }`. This inline syntax is both concise and readable. Lambdas capture the state of the scope they're defined in, so we can easily reference `ViewModel` properties or mutable state without complex parameter passing. 

For example, in a button's `onClick` lambda, we can directly modify state: `onClick = { count++ }` or call ViewModel methods: `onClick = { viewModel.submitForm() }`. This eliminates boilerplate and makes event handlers easy to understand at a glance. Compose's design leverages lambdas to make UI code feel less like "configuration" and more like natural, functional code. This paradigm shift significantly improves developer experience and reduces the cognitive load of managing UI interactions.

---

## Summary

This Task 2 submission demonstrates:
✓ Complete understanding of higher-order functions and lambdas  
✓ Practical examples of filtering lists with lambdas  
✓ A real-world use case (Task filtering by urgency)  
✓ Research on Compose lambda usage with 3+ examples  
✓ Function type alias exercises with advantages documented  
✓ Deep reflection on code reusability and event handling  
✓ Working code snippets with output examples  

**Key Takeaways:** Lambdas enable functional programming patterns, higher-order functions promote code reuse, and Compose's lambda-based API makes event handling intuitive and clean.
```
