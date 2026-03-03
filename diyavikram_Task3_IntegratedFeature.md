# Task 3 - Mini Project: Book Library App

## Overview
This document outlines the design and implementation of a Book Library application that combines Object-Oriented Programming (OOP) principles with Kotlin's Lambda functions. The project aims to provide comprehensive functionality for managing books in a library.

## 1. Book Class
The core of our application will be the `Book` class. It will represent a book entity with the following properties:
- **Title**: The title of the book.
- **Author**: The author of the book.
- **ISBN**: The unique identifier for the book.
- **Year**: The year the book was published.

### Book Class Implementation
```kotlin
data class Book(val title: String, val author: String, val isbn: String, val year: Int)
```

## 2. Objects
We will create a few instances of the `Book` class to demonstrate the functionality of our app.
```kotlin
val book1 = Book("Kotlin Programming", "Author A", "1234567890", 2021)
val book2 = Book("Advanced Kotlin", "Author B", "0987654321", 2020)
```

## 3. Lambda Functions
### Filtering Books
We will leverage lambda functions to filter books based on various criteria such as title, author, or year:
```kotlin
val books = listOf(book1, book2)
val filteredBooks = books.filter { it.year > 2019 }
```

## 4. Compose UI Integration Plan
The UI will be built using Jetpack Compose, enabling us to define the application’s UI in a declarative manner. Here’s a brief overview:
- **Main Screen**: Displays the list of books.
- **Add Book Screen**: Form to input new book details.
- **Details Screen**: Shows details of the selected book.

### Example Composable Function
```kotlin
@Composable
fun BookListScreen(books: List<Book>) {
    LazyColumn {
        items(books) { book ->
            Text(text = book.title)
        }
    }
}
```

## 5. Flow Diagram
(Here you would include a flow diagram of the app's architecture and data flow.)

## 6. Edge Case Handling
It is essential to handle edge cases to ensure the robustness of our application. Some potential edge cases include:
- Invalid ISBN numbers.
- Duplicate book entries.
- Empty or null values for book properties.

## 7. Reflection
In this project, we will also explore the use of reflection in Kotlin. Reflection allows us to inspect classes, properties, and functions at runtime. This could be used for validation or serialization purposes.

### Example of Reflection Use
```kotlin
val kClass = Book::class
val properties = kClass.memberProperties
```