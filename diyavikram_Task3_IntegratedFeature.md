
# Task 3 — Mini Project: Compose App Feature Combining OOP and Lambdas

**Submitted by:** Diya Vikram  
**Date:** March 3, 2026

---

## 1. Choose Your App Theme: Book Library App

### Theme Description

The **Book Library App** is an interactive mobile application designed for book enthusiasts to browse, discover, and manage their personal library. Users can view a curated list of books with details including title, author, genre, rating, and availability status. The app features dynamic filtering capabilities using lambdas to find books by genre, rating threshold, or availability. Users can mark books as favorites, request borrowing, and see real-time updates to their selections. This project demonstrates how OOP modeling of books combined with functional lambda-based filtering creates a responsive, flexible user interface in Jetpack Compose.

---

## 2. Design Class Structure and Data

### Main Class: `Book`

```kotlin
data class Book(
    val id: String,
    val title: String,
    val author: String,
    val genre: String,
    val rating: Double,
    val yearPublished: Int,
    var isAvailable: Boolean = true,
    var isFavorite: Boolean = false,
    val pageCount: Int
) {
    // Method 1: Get book summary
    fun getBookSummary(): String {
        val availabilityStatus = if (isAvailable) "Available" else "Checked Out"
        val favoriteTag = if (isFavorite) "❤️" else "🤍"
        return "$favoriteTag $title by $author ($genre) - ⭐$rating - $availabilityStatus"
    }
    
    // Method 2: Toggle favorite status
    fun toggleFavorite() {
        isFavorite = !isFavorite
    }
    
    // Method 3: Request book (mark as unavailable)
    fun requestBook(): Boolean {
        return if (isAvailable) {
            isAvailable = false
            true
        } else {
            false
        }
    }
    
    // Method 4: Return book (mark as available)
    fun returnBook() {
        isAvailable = true
    }
    
    companion object {
        const val MAX_RATING = 5.0
        const val MIN_RATING = 1.0
        
        // Factory function to create a book with validation
        fun createBook(
            id: String,
            title: String,
            author: String,
            genre: String,
            rating: Double,
            yearPublished: Int,
            pageCount: Int
        ): Book? {
            return if (rating in MIN_RATING..MAX_RATING && pageCount > 0) {
                Book(id, title, author, genre, rating, yearPublished, pageCount = pageCount)
            } else {
                null
            }
        }
    }
}
```

### List of Book Objects

```kotlin
val bookLibrary = listOf(
    Book(
        id = "B001",
        title = "The Great Gatsby",
        author = "F. Scott Fitzgerald",
        genre = "Fiction",
        rating = 4.8,
        yearPublished = 1925,
        isAvailable = true,
        pageCount = 180
    ),
    Book(
        id = "B002",
        title = "To Kill a Mockingbird",
        author = "Harper Lee",
        genre = "Fiction",
        rating = 4.9,
        yearPublished = 1960,
        isAvailable = false,
        pageCount = 281
    ),
    Book(
        id = "B003",
        title = "1984",
        author = "George Orwell",
        genre = "Dystopian",
        rating = 4.7,
        yearPublished = 1949,
        isAvailable = true,
        pageCount = 328
    ),
    Book(
        id = "B004",
        title = "Sapiens",
        author = "Yuval Noah Harari",
        genre = "Non-Fiction",
        rating = 4.6,
        yearPublished = 2011,
        isAvailable = true,
        isFavorite = true,
        pageCount = 443
    ),
    Book(
        id = "B005",
        title = "Educated",
        author = "Tara Westover",
        genre = "Memoir",
        rating = 4.8,
        yearPublished = 2018,
        isAvailable = true,
        pageCount = 352
    )
)
```

### Higher-Order Function with Lambda Filtering

```kotlin
// Type alias for book filtering predicates
typealias BookFilter = (Book) -> Boolean

// Higher-order function that filters books based on a lambda condition
fun filterBooks(books: List<Book>, predicate: BookFilter): List<Book> {
    return books.filter(predicate)
}

// Alternative: Extension function for more idiomatic Kotlin
fun List<Book>.filterByGenre(genre: String): List<Book> {
    return this.filter { it.genre.equals(genre, ignoreCase = true) }
}

fun List<Book>.filterByAvailability(available: Boolean): List<Book> {
    return this.filter { it.isAvailable == available }
}

fun List<Book>.filterByRating(minRating: Double): List<Book> {
    return this.filter { it.rating >= minRating }
}

fun List<Book>.filterByFavorites(): List<Book> {
    return this.filter { it.isFavorite }
}
```

### Pseudo-Code: Filtering Logic Flow

```
INPUT: bookLibrary (list of Book objects)

FILTERING OPERATIONS:
├── Filter by Genre
│   └── bookLibrary.filterByGenre("Fiction")
│       → Returns: [The Great Gatsby, To Kill a Mockingbird, 1984]
│
├── Filter by Availability
│   └── bookLibrary.filterByAvailability(true)
│       → Returns: [The Great Gatsby, 1984, Sapiens, Educated]
│
├── Filter by Rating (>= 4.7)
│   └── bookLibrary.filterByRating(4.7)
│       → Returns: [The Great Gatsby, To Kill a Mockingbird, 1984, Educated]
│
├── Combined Filtering
│   └── bookLibrary.filterByGenre("Fiction").filterByAvailability(true)
│       → Returns: [The Great Gatsby, 1984]
│
└── Custom Lambda Filtering
    └── filterBooks(bookLibrary) { book ->
        book.genre == "Fiction" && book.rating > 4.5 && book.isAvailable
    }
    → Returns: [The Great Gatsby, 1984]

OUTPUT: Filtered list of books ready for UI display
```

---

## 3. Compose UI Integration Plan

### UI Layout Structure

```
┌─────────────────────────────────────────────────┐
│         📚 Book Library App                     │
├─────────────────────────────────────────────────┤
│  [Filter: Genre ▼] [Filter: Rating ▼]          │
│  [Available Only ☐]  [Show Favorites ☐]        │
├─────────────────────────────────────────────────┤
│  📖 The Great Gatsby                            │
│     F. Scott Fitzgerald | Fiction               │
│     ⭐ 4.8 | 180 pages | Available              │
│     [❤️ Favorite] [Request] [Details]           │
├─────────────────────────────────────────────────┤
│  📖 To Kill a Mockingbird                       │
│     Harper Lee | Fiction                        │
│     ⭐ 4.9 | 281 pages | Checked Out           │
│     [🤍 Add to Favorites] [Waitlist] [Details] │
├─────────────────────────────────────────────────┤
│  [Load More Books...]                           │
└─────────────────────────────────────────────────┘
```

### Compose Code Implementation

```kotlin
// State management
@Composable
fun BookLibraryScreen() {
    var selectedGenre by remember { mutableStateOf<String?>(null) }
    var minRating by remember { mutableStateOf(1.0) }
    var showAvailableOnly by remember { mutableStateOf(false) }
    var showFavoritesOnly by remember { mutableStateOf(false) }
    
    // Apply filters based on state
    val filteredBooks = remember(selectedGenre, minRating, showAvailableOnly, showFavoritesOnly) {
        var books = bookLibrary
        
        if (selectedGenre != null) {
            books = books.filterByGenre(selectedGenre!!)
        }
        
        if (minRating > 1.0) {
            books = books.filterByRating(minRating)
        }
        
        if (showAvailableOnly) {
            books = books.filterByAvailability(true)
        }
        
        if (showFavoritesOnly) {
            books = books.filter { it.isFavorite }
        }
        
        books
    }
    
    Column(modifier = Modifier.fillMaxSize()) {
        // Header
        Text(
            "📚 Book Library",
            style = MaterialTheme.typography.headlineMedium,
            modifier = Modifier.padding(16.dp)
        )
        
        // Filter Controls
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Button(onClick = { /* Show genre dropdown */ }) {
                Text("Genre: ${selectedGenre ?: "All"}")
            }
            
            Button(onClick = { /* Show rating slider */ }) {
                Text("Rating: ${String.format("%.1f", minRating)}")
            }
        }
        
        // Checkboxes for filters
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp),
            horizontalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            Checkbox(
                checked = showAvailableOnly,
                onCheckedChange = { showAvailableOnly = it },
                modifier = Modifier.align(Alignment.CenterVertically)
            )
            Text("Available Only", modifier = Modifier.align(Alignment.CenterVertically))
            
            Checkbox(
                checked = showFavoritesOnly,
                onCheckedChange = { showFavoritesOnly = it },
                modifier = Modifier.align(Alignment.CenterVertically)
            )
            Text("Favorites", modifier = Modifier.align(Alignment.CenterVertically))
        }
        
        // Book List
        LazyColumn(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            items(filteredBooks.size) { index ->
                BookCard(
                    book = filteredBooks[index],
                    onFavoriteClick = { filteredBooks[index].toggleFavorite() },
                    onRequestClick = { filteredBooks[index].requestBook() }
                )
            }
        }
    }
}

// Individual Book Card Component
@Composable
fun BookCard(
    book: Book,
    onFavoriteClick: () -> Unit,
    onRequestClick: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            // Title and Author
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Column(modifier = Modifier.weight(1f)) {
                    Text(
                        book.title,
                        style = MaterialTheme.typography.titleMedium,
                        fontWeight = FontWeight.Bold
                    )
                    Text(
                        "by ${book.author}",
                        style = MaterialTheme.typography.bodySmall,
                        color = Color.Gray
                    )
                }
                
                // Favorite Button with Lambda
                IconButton(onClick = onFavoriteClick) {
                    Icon(
                        imageVector = if (book.isFavorite) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
                        contentDescription = "Toggle Favorite",
                        tint = if (book.isFavorite) Color.Red else Color.Gray
                    )
                }
            }
            
            // Details
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 8.dp),
                horizontalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                Text(
                    book.genre,
                    style = MaterialTheme.typography.labelSmall,
                    modifier = Modifier
                        .background(MaterialTheme.colorScheme.surfaceVariant)
                        .padding(4.dp)
                )
                Text("⭐ ${book.rating}/5", style = MaterialTheme.typography.labelSmall)
                Text("${book.pageCount} pages", style = MaterialTheme.typography.labelSmall)
            }
            
            // Status
            Text(
                if (book.isAvailable) "✅ Available" else "❌ Checked Out",
                style = MaterialTheme.typography.labelSmall,
                color = if (book.isAvailable) Color.Green else Color.Red,
                modifier = Modifier.padding(top = 8.dp)
            )
            
            // Action Buttons
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 12.dp),
                horizontalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                Button(
                    onClick = onRequestClick,
                    enabled = book.isAvailable,
                    modifier = Modifier.weight(1f)
                ) {
                    Text(if (book.isAvailable) "Request" else "On Waitlist")
                }
                
                Button(
                    onClick = { /* Show details */ },
                    modifier = Modifier.weight(1f)
                ) {
                    Text("Details")
                }
            }
        }
    }
}
```

### UI Reactivity to State Changes

```
State Change Flow:
─────────────────

1. User toggles "Available Only" checkbox
   ↓
2. showAvailableOnly state updates
   ↓
3. filteredBooks is recalculated via remember dependency
   ↓
4. LazyColumn recomposes with new filtered list
   ↓
5. BookCard components update to reflect availability
   ↓
6. UI displays updated book list instantly

User Actions:
─────────────
- Click ❤️ button → Book.toggleFavorite() → isFavorite state changes → Icon recomposes
- Click "Request" → Book.requestBook() → isAvailable becomes false → Button disabled + status changes
- Filter by genre → selectedGenre updates → filteredBooks recalculates → LazyColumn refreshes
```

---

## 4. Flow Diagram and Architecture

### Data Flow Diagram

```
┌───────────────────────────────────��─────────────────────────────────┐
│                    BOOK LIBRARY APP ARCHITECTURE                     │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ DATA LAYER (Immutable)                                               │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  bookLibrary: List<Book>                                             │
│  ├── Book(id="B001", title="The Great Gatsby", ...)                 │
│  ├── Book(id="B002", title="To Kill a Mockingbird", ...)            │
│  ├── Book(id="B003", title="1984", ...)                             │
│  └── ...                                                             │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
                   (Passed through)
                              ↓
┌─────────────────────────────��────────────────────────────────────────┐
│ BUSINESS LOGIC LAYER (Functional with Lambdas)                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  filterBooks(books, predicate: (Book) -> Boolean): List<Book>       │
│                                                                       │
│  Extension Functions:                                                │
│  ├── filterByGenre(genre: String): List<Book>                      │
│  ├── filterByAvailability(available: Boolean): List<Book>          │
│  ├── filterByRating(minRating: Double): List<Book>                 │
│  └── filterByFavorites(): List<Book>                               │
│                                                                       │
│  Lambda Filters:                                                     │
│  ├── { book -> book.genre == "Fiction" }                           │
│  ├── { book -> book.isAvailable }                                  │
│  ├── { book -> book.rating >= 4.5 }                                │
│  └── { book -> book.isFavorite }                                   │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
                   (Filtered Output)
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ STATE MANAGEMENT LAYER (Mutable State)                              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  var selectedGenre: String?          (UI State)                     │
│  var minRating: Double               (UI State)                     │
│  var showAvailableOnly: Boolean      (UI State)                     │
│  var showFavoritesOnly: Boolean      (UI State)                     │
│                                                                       │
│  val filteredBooks: List<Book>       (Derived State)                │
│  - Recalculates whenever any filter state changes                   │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
                   (UI Input Parameters)
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ PRESENTATION LAYER (Jetpack Compose UI)                             │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  BookLibraryScreen()                                                 │
│  ├── FilterControls()         ← Lambda: onGenreSelect, onRatingSet  │
│  ├── FilterCheckboxes()       ← Lambda: onAvailableToggle, etc.     │
│  └── LazyColumn {                                                    │
│      └── items(filteredBooks) {                                     │
│          └── BookCard()        ← Lambda: onFavoriteClick,           │
│                                     onRequestClick               │
│      }                                                               │
│  }                                                                    │
│                                                                       │
│  User Interactions:                                                  │
│  • Click Favorite Button → Book.toggleFavorite() → State updates    │
│  • Select Genre Filter  → selectedGenre updates → Recompose         │
│  • Click Request Button → Book.requestBook() → isAvailable changes  │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│ KEY PATTERNS                                                          │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ OOP Elements:                                                        │
│  • Book class with properties and methods                           │
│  • Encapsulation of book state (isAvailable, isFavorite)           │
│  • Companion object with factory function                          │
│                                                                       │
│ Functional Elements:                                                 │
│  • Lambda predicates for filtering                                  │
│  • Higher-order functions (filterBooks)                            │
│  • Extension functions for chaining filters                        │
│  • Function type aliases (BookFilter)                              │
│                                                                       │
│ Reactive Elements:                                                   │
│  • Compose state dependency tracking                                │
│  • Automatic recomposition on state change                         │
│  • Lambda callbacks for user interactions                          │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 5. Error & Edge Case Planning

### Edge Case 1: Empty Book List

**Scenario:** The library has no books, or all books are filtered out.

**Implementation:**

```kotlin
@Composable
fun BookLibraryScreen() {
    // ... existing code ...
    
    LazyColumn(modifier = Modifier.fillMaxSize()) {
        if (filteredBooks.isEmpty()) {
            item {
                Column(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(32.dp),
                    horizontalAlignment = Alignment.CenterHorizontally,
                    verticalArrangement = Arrangement.Center
                ) {
                    Icon(
                        imageVector = Icons.Default.Search,
                        contentDescription = "No books found",
                        modifier = Modifier.size(64.dp),
                        tint = Color.Gray
                    )
                    Text(
                        "No books found",
                        style = MaterialTheme.typography.headlineSmall,
                        modifier = Modifier.padding(top = 16.dp)
                    )
                    Text(
                        "Try adjusting your filters",
                        style = MaterialTheme.typography.bodySmall,
                        color = Color.Gray
                    )
                }
            }
        } else {
            items(filteredBooks.size) { index ->
                BookCard(book = filteredBooks[index], /* ... */)
            }
        }
    }
}
```

### Edge Case 2: Null or Invalid Book Data

**Scenario:** A book object has null values or invalid data (e.g., negative rating).

**Implementation:**

```kotlin
// Use companion object factory function with validation
val newBook = Book.createBook(
    id = "B006",
    title = "New Book",
    author = "Author Name",
    genre = "Fiction",
    rating = 4.5,  // Must be between 1.0 and 5.0
    yearPublished = 2024,
    pageCount = 250  // Must be > 0
)

if (newBook != null) {
    // Add to library
} else {
    // Handle invalid data
    showErrorMessage("Invalid book data. Please check ratings and page count.")
}

// Safe access in UI
@Composable
fun BookCardSafe(book: Book?) {
    book?.let {
        // Display book safely
        BookCard(
            book = it,
            onFavoriteClick = { it.toggleFavorite() },
            onRequestClick = { it.requestBook() }
        )
    } ?: run {
        // Handle null book
        Text("Book data unavailable")
    }
}
```

### Edge Case 3: User Input Errors and Filter Edge Cases

**Scenario:** Invalid filter inputs (e.g., rating > 5.0, empty genre string, unknown book ID).

**Implementation:**

```kotlin
// Defensive filtering with safe operations
fun List<Book>.safeFilterByGenre(genre: String?): List<Book> {
    return if (genre.isNullOrBlank()) {
        this  // Return all books if genre is empty
    } else {
        this.filter { it.genre.equals(genre, ignoreCase = true) }
    }
}

fun List<Book>.safeFilterByRating(minRating: Double): List<Book> {
    // Clamp rating between valid range
    val clampedRating = minRating.coerceIn(
        Book.MIN_RATING,
        Book.MAX_RATING
    )
    return this.filter { it.rating >= clampedRating }
}

fun findBookById(id: String?): Book? {
    return if (id.isNullOrBlank()) {
        null
    } else {
        bookLibrary.find { it.id == id }
    }
}

// UI handling with error messages
@Composable
fun BookLibraryScreen() {
    var filterError by remember { mutableStateOf<String?>(null) }
    
    fun applyFilters() {
        try {
            // Validate inputs
            if (selectedGenre?.isBlank() == true) {
                filterError = "Genre cannot be empty"
                return
            }
            
            if (minRating < 1.0 || minRating > 5.0) {
                filterError = "Rating must be between 1.0 and 5.0"
                return
            }
            
            // If validation passes, clear error
            filterError = null
            
            // Apply filters
            val filtered = bookLibrary
                .safeFilterByGenre(selectedGenre)
                .safeFilterByRating(minRating)
            
        } catch (e: Exception) {
            filterError = "An unexpected error occurred: ${e.message}"
        }
    }
    
    // Display error messages
    if (filterError != null) {
        Snackbar(
            modifier = Modifier.padding(16.dp),
            action = {
                TextButton(onClick = { filterError = null }) {
                    Text("Dismiss")
                }
            }
        ) {
            Text(filterError ?: "")
        }
    }
}
```

### Edge Case Summary Table

| Edge Case | Issue | Solution |
|-----------|-------|----------|
| **Empty List** | No books to display | Show empty state with icon and message |
| **Null Book Data** | Invalid book object | Use factory function with validation + safe calls `?.let{}` |
| **Invalid Filter Input** | User enters invalid genre/rating | Input validation, clamping, and error messages |
| **Concurrent Modifications** | List modified during iteration | Use immutable List<Book>, avoid direct mutations |
| **Book Not Found by ID** | Safe call returns null | Use `?.let{}` or provide fallback UI |

---

## 6. Reflection & Improvement Plan

### How OOP and Lambdas Were Combined

In this Book Library App, I successfully combined object-oriented and functional programming paradigms. The **OOP side** provided structure through the `Book` data class with well-defined properties (title, author, genre, rating) and methods (toggleFavorite(), requestBook()). This encapsulation ensured that book state was managed cohesively and safely. 

The **functional side** leveraged lambdas to create flexible filtering logic. Instead of hardcoding separate filter functions like `filterByGenre()`, `filterByRating()`, etc., I designed higher-order functions and extension functions that accept lambda predicates. This allowed different combinations of filters (genre + rating + availability) without code duplication. The `remember` composable combined with mutable state variables created reactive UI that automatically updates when filter lambdas produce new results.

The integration was clean: the data model (Book class) remained pure and testable, while the filtering logic (lambdas) remained flexible and reusable. Compose's lambda-based callback pattern (`onClick = { }`, `onFavoriteClick = { }`) made event handling intuitive and connected the UI directly to object methods.

### Trade-Offs Between OOP and Functional Styles

**Advantages of OOP Approach:**
- Encapsulation keeps book data and related methods together
- Methods like `toggleFavorite()` are easy to understand and maintain
- The Book class serves as a clear contract for what a book entity looks like
- Companion object factory function provides safe object creation

**Advantages of Functional Approach:**
- Lambdas enable flexible, composable filtering without creating many intermediate classes
- Higher-order functions reduce code duplication
- Functional composition (chaining filters) is elegant and readable
- Pure functions are easier to test and reason about

**Trade-Offs Identified:**
1. **Mutable State vs. Pure Functions:** Book objects have mutable fields (`isFavorite`, `isAvailable`), which contradicts pure functional principles. A more functional approach would create new Book instances with updated values (immutability). However, Kotlin's `var` makes mutations simpler and more performant for UI updates.

2. **Imperative Filtering vs. Declarative UI:** The filter controls are imperative (manually building `remember` state), whereas Compose's UI is declarative. A more cohesive approach might use a dedicated state management library like MVI or Redux pattern.

3. **Extension Functions vs. Static Methods:** Extension functions feel more natural in Kotlin but can scatter related logic across the codebase. A more OOP approach would keep all filtering methods in a dedicated `BookRepository` class.

### Improvements for a Real App Feature

**1. State Management Architecture:**
- Introduce a `ViewModel` to separate UI state from business logic
- Use `StateFlow` or `LiveData` for reactive state management
- Implement a `Repository` pattern to abstract book data source

```kotlin
class BookLibraryViewModel(private val bookRepository: BookRepository) : ViewModel() {
    private val _filterState = MutableStateFlow(FilterState())
    val filterState: StateFlow<FilterState> = _filterState.asStateFlow()
    
    private val _filteredBooks: StateFlow<List<Book>> = 
        filterState.flatMapLatest { filters ->
            bookRepository.getBooksFiltered(filters)
        }.stateIn(viewModelScope, SharingStarted.WhileSubscribed(), emptyList())
    
    fun setGenreFilter(genre: String) { /* ... */ }
    fun setRatingFilter(rating: Double) { /* ... */ }
}
```

**2. Data Persistence:**
- Add Room database integration for local storage
- Implement network calls to sync with backend API
- Cache favorite books and user preferences

**3. Enhanced UX:**
- Add search functionality (not just filters)
- Implement sorting options (by rating, title, date added)
- Add book recommendations based on reading history
- Create a wishlist feature
- Add review and rating submission from users

**4. Error Handling & Logging:**
- Implement proper exception handling for network errors
- Add analytics tracking for user behavior
- Create comprehensive error messages and recovery flows

**5. Testing:**
- Unit tests for Book class methods
- Unit tests for filter lambda predicates
- Integration tests for ViewModel
- UI tests for Compose components

**6. Performance Optimization:**
- Use `@Stable` and `@Immutable` annotations
- Implement pagination for large book lists
- Cache filtered results to avoid recalculation
- Use `remember` with proper key parameters

**7. Accessibility:**
- Add semantic descriptions to composables
- Ensure proper contrast ratios for text
- Support screen readers
- Make all interactive elements keyboard accessible

---

## Summary

This Task 3 submission demonstrates:
✓ Coherent integration of OOP (Book class) and functional (lambdas) paradigms  
✓ Complete data model with validation and factory functions  
✓ Practical filtering logic with higher-order functions  
✓ Fully designed Compose UI with state management  
✓ Clear architecture diagram showing data flow  
✓ Comprehensive edge case handling with defensive programming  
✓ Thoughtful reflection on design trade-offs and real-world improvements  

**Key Insights:** Combining OOP for data modeling with functional programming for flexible logic creates maintainable, testable, and reactive applications. Jetpack Compose's lambda-based API naturally aligns with functional patterns, making this combination particularly powerful for modern Android development.
```
