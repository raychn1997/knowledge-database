# Guide for Creating Production Quizzes for the Adaptive French Tutor

This document outlines the process for generating and delivering production quizzes to the Adaptive French Tutor application. This guide is intended for agents or developers responsible for creating new vocabulary-based quizzes.

## 1. Objective

The primary objective is to create structured quiz content that facilitates active learning of French vocabulary and grammatical structures, and to deliver this content to the French Tutor application via its `postMessage` API.

## 2. Project Context & Delivery Mechanism

The French Tutor application expects quiz data in a specific JSON format delivered via `window.postMessage`. The application should be opened first, ideally by the script sending the quiz, to establish a communication channel.

**Key components:**
*   **French Tutor Application:** Runs on `http://localhost:3000`.
*   **`postMessage` API:** The primary method of sending quiz data to the application.
*   **`PRODUCTION_QUIZ` type:** The specific message type for delivering pre-generated quizzes.

A common approach for delivery is to use an HTML page with embedded JavaScript, which first opens the French Tutor app and then sends the quiz data.

## 3. Source of Vocabulary and Structures

Vocabulary and grammatical structures for quizzes are typically derived from markdown files within the project, such as:
*   `/Production/taux_de_nom_verbe.md` (e.g., for "Le taux de [nom] [verbe]" structure).

These files provide the core French terms, example sentences, and usage contexts necessary to formulate effective quiz questions.

## 4. Required Quiz Item Structure

Each quiz item in the `PRODUCTION_QUIZ` JSON payload must adhere to the following structure to ensure pedagogical effectiveness:

```json
{
    "context": "Context sentence in the target language (French) using the topic structure and vocabulary.",
    "question": "Production prompt in English that requires the learner to actively use the structure and vocabulary. Example: 'Translate: "The unemployment rate has decreased this year."'",
    "answer": "The correct answer in French for the prompt."
}
```

**Details for each field:**
*   **`context` (French):** A clear, concise sentence or two that provides context for the vocabulary or structure being tested. It should use the target French language and ideally the vocabulary and grammatical pattern from the lesson.
*   **`question` (English):** This is the prompt given to the learner, typically an English sentence requiring translation into French, specifically designed to elicit the use of the target pattern and vocabulary. It should start with "Translate:".
*   **`answer` (French):** The definitive correct French translation/response to the `question` prompt.

*(Optional: A "smooth transition sentence" connecting to the next idea can be included within the `context` field if it enhances flow, but it is not a separate JSON field.)*

## 5. Quiz Generation Process

To create a new set of production quizzes:

1.  **Identify Target Vocabulary/Structure:** Review the relevant markdown files (e.g., `taux_de_nom_verbe.md`) to understand the lesson's core vocabulary and grammatical patterns.
2.  **Formulate Quiz Items:** For each concept or vocabulary item you wish to test:
    *   Craft a **`context`** sentence in French using the target pattern.
    *   Develop an **`question`** prompt in English that specifically requires the learner to apply the target pattern and vocabulary.
    *   Determine the precise **`answer`** in French.
3.  **Assemble JSON Array:** Collect all generated quiz items into a JSON array, ensuring each item matches the structure described in Section 4.

## 6. Delivering the Quiz (Example: `quizSender.html`)

The recommended method to send the quiz is via an HTML file with embedded JavaScript. This script should perform the following steps:

1.  **Open the French Tutor application:** Use `window.open('http://localhost:3000', 'FrenchTutor')` to launch the application in a new window/tab. Store the reference to this new window.
2.  **Wait for Application Readiness:** The French Tutor application signals its readiness by sending an `APP_READY` message. A robust solution would involve adding an event listener for `message` events on `window` and waiting for the `APP_READY` type. For simpler cases, a `setTimeout` (e.g., 2000ms) can serve as a temporary delay while the app loads.
3.  **Send `PRODUCTION_QUIZ` message:** Once the target window is ready (or the delay has passed), use its `postMessage` method to send the JSON payload. It is crucial to specify the `targetOrigin` for security (e.g., `'http://localhost:3000'`).

**Example `quizSender.html` structure:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Quiz Sender</title>
</head>
<body>
    <h1>Send Production Quiz</h1>
    <button id="sendQuizButton">Open App & Send Quiz</button>
    <script>
        document.getElementById('sendQuizButton').addEventListener('click', function() {
            const frenchTutorWindow = window.open('http://localhost:3000', 'FrenchTutor');
            setTimeout(() => { // Replace with actual APP_READY listener for robustness
                if (frenchTutorWindow && !frenchTutorWindow.closed) {
                    frenchTutorWindow.postMessage({
                        type: 'PRODUCTION_QUIZ',
                        json: [
                            // Your generated quiz items go here
                            {
                                "context": "L'économie connaît des défis. Le taux de chômage est actuellement élevé.",
                                "question": "Translate: 'The unemployment rate is high.'",
                                "answer": "Le taux de chômage est élevé."
                            }
                            // ... more quiz items
                        ]
                    }, 'http://localhost:3000');
                    alert('Quiz message sent!');
                } else {
                    alert('Could not open French Tutor window or window was closed. Please ensure pop-ups are enabled.');
                }
            }, 2000); // Adjust delay as needed
        });
    </script>
</body>
</html>
```

By following these steps, new production quizzes can be effectively created and delivered to the Adaptive French Tutor application.