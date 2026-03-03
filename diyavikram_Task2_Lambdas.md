# Task 2: Lambdas and Higher-Order Functions

## Concept Notes

### Higher-Order Functions
Higher-order functions are functions that take other functions as parameters or return them as results. This allows for a more abstract and functional way to manipulate behavior in programming.

### Lambda Expressions
A lambda expression is a concise way to represent an anonymous function. They are often used to define inline expressions.

### Function Types
Function types define the types of functions. In Kotlin, function types are represented using parentheses for parameters and the return type after a "->".

### Inline Functions
Inline functions are a special type of higher-order functions. When you mark a function as inline, the compiler replaces the function call with the actual function body, which can improve performance by avoiding the overhead of function calls.

## Examples

### Lambda Filtering Lists
```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val evenNumbers = numbers.filter { it % 2 == 0 }
println(evenNumbers) // Output: [2, 4]
```

### Higher-Order Function Example
```kotlin
fun operateOnNumbers(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

val result = operateOnNumbers(5, 3, { x, y -> x + y })
println(result) // Output: 8
```

## Mini Use Case: Task Filtering with Urgency Evaluation
To filter tasks based on urgency, we can use higher-order functions combined with lambda expressions. 

### Example
```kotlin
data class Task(val name: String, val isUrgent: Boolean)

val tasks = listOf(Task("Task 1", true), Task("Task 2", false))
val urgentTasks = tasks.filter { it.isUrgent }
println(urgentTasks) // Output: [Task(name=Task 1, isUrgent=true)]
```

## Compose Lambda Usage Research
### Example 1: Click Listener
```kotlin
button.setOnClickListener { 
    // Handle click 
}
```

### Example 2: List Comprehension
```kotlin
val items = listOf("Apple", "Banana", "Cherry")
items.forEach { item -> 
    println(item)
}
```

### Example 3: Custom Composable with Lambda
```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(text = "Hello, $name!", modifier = modifier)
}
```

## Function Type Alias Exercises
You can create type aliases for function types for better readability.
```kotlin
typealias Operation = (Int, Int) -> Int

val addition: Operation = { x, y -> x + y }
println(addition(3, 4)) // Output: 7
```

## Reflection on Code Reusability and Event Handling
Using lambdas and higher-order functions promotes code reusability by allowing behaviors to be passed around. This results in more modular and maintainable code, especially in event-driven programming where event handling can be abstracted into reusable components.