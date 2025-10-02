# Java Study Hub - Architecture & Logic Explained

This document provides a behind-the-scenes look at the architectural decisions and logic that power the Java Study Hub. The goal was to create a feature-rich, interactive application using only vanilla HTML, CSS, and JavaScript, without the need for external frameworks or build tools.

## Core Architectural Principles

### 1. Single-File Simplicity

The entire application lives within a single `index.html` file. This was a deliberate choice for several reasons:

* **Portability:** The app can be downloaded and run by anyone, anywhere, simply by opening the file in a browser. There are no dependencies or build steps.
* **Simplicity:** All code (HTML, CSS, and JavaScript) is co-located, making it easy to understand the relationships between structure, style, and behavior.
* **Zero Configuration:** No web server, package manager, or bundler is required, which lowers the barrier to entry for both using and modifying the app.

### 2. Centralized State Management (Vanilla JS)

Instead of a formal state management library (like Redux or Vuex), the application's state is managed by a collection of simple JavaScript variables declared at the top of the `<script>` tag.

```javascript
// --- STATE ---
let currentMode = 'flashcard';
let currentFlashcardIndex = 0;
let quizScore = 0;
// ... and so on for the matching game state
```

* **Why this approach?** For a small-to-medium-sized application, this is the simplest and most direct way to manage state. It avoids boilerplate and makes the flow of data easy to trace.
* **How it works:** All functions in the script read from and write to this central block of state variables. For example, `switchMode` updates `currentMode`, and `selectAnswer` updates `quizScore` and `quizStreak`.

### 3. Data-Driven UI

The application's content (flashcard terms, quiz questions) is not hardcoded in the HTML. Instead, it's stored in two primary arrays in the JavaScript: `flashcards` and `quizQuestions`.

```javascript
const flashcards = [
    { term: "JVM", definition: "...", mastered: false },
    // ... more cards
];
```

* **Why this approach?** This decouples the content from the presentation. To add a new flashcard or quiz question, you only need to add a new object to the appropriate array. No HTML changes are necessary.
* **How it works:** Functions like `updateFlashcard()` and `displayQuizQuestion()` read data from these arrays and use it to dynamically set the `.textContent` or `.innerHTML` of the relevant HTML elements.

## Logic Behind Key Features

### 1. The Mode Switching Engine (`switchMode` function)

The logic for switching between Flashcards, Quiz, and Matching modes is a simple but robust state machine.

* **Step 1: Reset Everything:** The function first hides *all* view containers (`#flashcard-view`, `#quiz-view`, `#matching-view`) and deactivates *all* mode buttons. This is a critical step that prevents visual bugs where multiple views or active buttons could accidentally be shown at once.
* **Step 2: Activate the Target:** Based on the `mode` passed into the function, it selectively un-hides the target view and applies the "active" styling to its corresponding button.
* **Step 3: Initialize the View:** Finally, it calls the appropriate initialization function (`initFlashcards()`, `initQuiz()`, or `initMatchingGame()`) to prepare the content for the newly activated mode.

### 2. The 3D Card Flip Animation (CSS, not JS)

The card flip is achieved almost entirely with CSS, making it smooth and performant. JavaScript's only job is to toggle a class.

* **`perspective`**: The parent `.card-container` has a `perspective` property. This creates the 3D space, telling the browser how "deep" the scene is.
* **`transform-style: preserve-3d`**: The `.card-inner` element has this property, which allows its children (`.card-front` and `.card-back`) to exist in that 3D space.
* **`backface-visibility: hidden`**: This is the magic. It makes the back of an element invisible. The `.card-front` is hidden when it's facing away, and the `.card-back` is hidden when *it's* facing away.
* **The Flip**: JavaScript simply toggles a `.flipped` class on the container. The CSS rule `.card-container.flipped .card-inner` applies a `transform: rotateY(180deg)` to the inner card, which triggers the 3D rotation.

### 3. Progress Persistence (`localStorage`)

The app remembers your progress even after you close the browser tab.

* **`saveProgress()`**: This function doesn't store the entire, complex `flashcards` array. It efficiently creates a simple object containing only the terms of the cards you've mastered (e.g., `{ "JVM": true, "JDK": true }`). This object is converted to a JSON string with `JSON.stringify()` and saved to `localStorage`.
* **`loadProgress()`**: On page load, this function reads the JSON string, parses it back into an object with `JSON.parse()`, and then iterates through the main `flashcards` array to update the `mastered` status of each card accordingly.

### 4. Matching Game Logic

The matching game has the most complex state management.

* **Board Creation (`initMatchingGame`)**: It randomly selects 8 flashcards, creates an array of 16 objects (8 terms, 8 definitions), and critically, adds a common `key` to each pair (e.g., the term "JVM"). After shuffling, it generates the HTML for each card, embedding the `key` into a `data-key` attribute. This `data-key` is how we know which cards match.
* **The Game Loop (`handleMatchCardClick`)**:
    1.  It uses two state variables, `firstCard` and `secondCard`, to track the user's selections.
    2.  A `lockBoard` flag is used to prevent the user from clicking a third card while the first two are being evaluated.
    3.  When the second card is clicked, it calls `checkForMatch()`.
* **Match Evaluation (`checkForMatch`)**: This is the core check. It simply compares `firstCard.dataset.key === secondCard.dataset.key`.
    * If **true**, it calls `disableCards()`, which removes the click event listeners from the matched pair, effectively locking them in the "flipped" state.
    * If **false**, it calls `unflipCards()`, which uses a `setTimeout` to wait 1.2 seconds before flipping the cards back over. This delay is crucial for allowing the user's brain to register the mismatch.