---
layout: module
title: "03 GenAi Applications"
permalink: /modules/03-genai-applications/
---

Hello, hello\! Welcome back to the course. I am so excited for this module because today is the day we stop working *harder* and start working *smarter*.

Imagine you are a master chef in a busy kitchen. Until now, you’ve been chopping every single onion, peeling every potato, and kneading every ball of dough by hand. It’s noble work, but it’s exhausting. Today, we are introducing you to your new Sous Chef: **Generative AI**.

This Sous Chef is incredibly fast, knows every recipe in the world, and never gets tired. But—and this is a big "but"—if you don’t give it clear instructions, it might put salt in your coffee instead of sugar.

In this module, we are going to learn how to communicate with this Sous Chef (Prompt Engineering) to automate your test scripts, generate data, and fix broken code, all based on the rigorous standards found in the O'Reilly book *Generative AI: Future-Proof Inputs for Reliable AI Outputs*.

Are you ready to supercharge your testing workflow? Let’s dive in\!

-----

## 1. Prompting

### What are LLMs and How Do They Work?


Before we start typing commands, we need to understand what is happening inside the "brain" of tools like ChatGPT, Claude, or Copilot.

Think about when you are texting a friend. You type: *"I am running five minutes..."*
Your phone suggests: *"...late."*

Your phone is guessing the next word based on probability. It has seen that phrase thousands of times. Large Language Models (LLMs) do this, but on a galactic scale. They haven't just read your text messages; they have read (almost) the entire internet—StackOverflow, documentation, GitHub repositories, and Wikipedia.

However, LLMs are **probabilistic**, not deterministic. They don't "know" facts; they predict patterns.

> **The Technical Takeaway:**<br>
> An LLM treats your input as a sequence of "tokens" (chunks of text). It calculates the statistical likelihood of the next token. This is why **Context** is king. If you don't give enough context, the probability distribution is too flat, and the AI guesses wildly (hallucination). If you provide specific constraints, you narrow the probability path, forcing the AI toward the correct answer.

<br>

### Good Prompts


Many people treat AI like a magic 8-ball. They ask: *"Write me a test for a login page."*

The result? A generic, useless script that doesn't match your framework.

According to the core principles in our reference text, a reliable prompt isn't just a question; it is a **Structured Specification**. We need to move from "Conversational Prompting" to "Systematic Prompting."

Think of this like ordering a custom suit.

  * **Bad Request:** "I want a suit." (You might get a clown suit).
  * **Good Request:** "I want a navy blue, two-button suit, wool fabric, size 42R, with a slim fit cut."

To get reliable outputs for QA, we will use the **C.P.S.O.** framework derived from best practices in reliable AI inputs:

1.  **Context (The Environment):** Who is the AI? What is the task?
2.  **Persona (The Role):** Is it a Junior Dev? A Security Expert? A QA Lead?
3.  **Specifics (The Constraints):** Language, Framework, naming conventions.
4.  **Output (The Format):** Code block? Table? JSON?


**Weak Prompt:**

> "Write a Selenium test for the search bar."

**Reliable Prompt:**

> "Act as a Senior QA Automation Engineer [Persona]. I need a robust automated test for a search feature [Context].
>
> **Constraints:**
>
>   * Language: Python
>   * Framework: Pytest with Selenium WebDriver
>   * Pattern: Page Object Model (POM)
>   * Wait Strategy: Use Explicit Waits (WebDriverWait), strictly no `time.sleep()`.
>
> **Task:** Write the test script assuming the input field ID is `#search-box` and the submit button class is `.btn-search`.
>
> **Output:** Provide only the code in a markdown block."

See the difference? 

-----

## 2. Prompt Engineering for Automation

Now that we know the theory, let's get our hands dirty. How do we use this to write actual automation code?


### Context Setting

**How to paste DOM snippets into LLMs to get correct selectors**

One of the biggest pain points in automation is finding the right **CSS Selector** or **XPath**. It’s like playing Where’s Waldo in a sea of HTML tags.

You might be tempted to just paste the URL of your website into ChatGPT and say "Find the selectors." **Don't do this.** Most LLMs cannot browse the live web in real-time to inspect elements, or if they do, they might see a different version of the site (A/B testing, different regions). (and also you'd probably fall into compliance issues on your company, so please don't...)

Instead, we use the **DOM Snippet Strategy**. Imagine you want a detective to find a suspect in a crowd. You don't just say "Find the guy." You hand them a **photograph** of the suspect.

In our case, the "Photograph" is the specific chunk of HTML from the Chrome DevTools.

#### Hands-On Workflow:

1.  **Inspect Element:** Right-click your target element (e.g., a "Submit" button) in Chrome.
2.  **Copy Outer HTML:** Don't just copy the line; copy the parent container too. Context matters\!
3.  **Clean the Data:** (Crucial Step\!) If there is a giant SVG icon or a massive block of text inside the HTML, delete it. It distracts the AI.
4.  **The Prompt:**

> "I am providing you with a snippet of HTML from a login modal.
>
> **HTML Snippet:**
>
> ```html
> <div class="auth-form-body">
>    <label for="login_field">Username</label>
>    <input type="text" name="login" id="login_field" class="form-control input-block js-login-field">
>    <input type="password" name="password" id="password" class="form-control form-control input-block">
>    <input type="submit" name="commit" value="Sign in" class="btn btn-primary btn-block" data-disable-with="Signing in…">
> </div>
> ```
>
> **Task:** Generate a robust CSS Selectors for the Username field, Password field, and Sign In button. Prioritize ID over Class. Exclude dynamic classes that look like they might change (e.g., `js-login-field`)."

**Why this works:** You are stripping away the noise. You are forcing the AI to look only at what matters, resulting in selectors that won't break next week.

<br>

### Iterative Refinement

**Correcting AI output when code is deprecated or incorrect**

 **AI writes legacy code.** Because LLMs are trained on data that cuts off at a certain date (or includes old forums from 2018), they *love* to suggest deprecated methods.

**Example:** You ask for a Selenium script in Python. The AI gives you:
`driver.find_element_by_id("username")`

If you run this today, your console will scream red errors. That method was removed in Selenium 4. We need to use `driver.find_element(By.ID, "username")`.

Do we give up? No\! We use **Iterative Refinement**. We treat the AI like a Junior Developer during a code review.

**The Correction Prompt:**

> "Review the code you just generated. I notice you are using `find_element_by_id`. This is deprecated in Selenium 4.
>
> **Refinement Task:** Refactor the code to use the `By` class and the `find_element` method. Ensure all imports (e.g., `from selenium.webdriver.common.by import By`) are included."

**Pro-Tip:** You can even set this as a "System Prompt" or "Custom Instruction" at the start of your chat: *"Always use Selenium 4 syntax. Never use deprecated `find_element_by_*` methods."*

-----

## 3. Code Generation & Explanation

### Using Copilot/ChatGPT to Generate Boilerplate Code


When making a pizza, making the dough from scratch takes 4 hours. Putting on the toppings takes 5 minutes.
In coding, **Boilerplate** (imports, class setup, teardown methods) is the dough. It’s necessary, but boring.

Let’s use AI to buy the dough so we can focus on the toppings (the test logic).

**Scenario:** We need a new Page Object file for a Shopping Cart page.

**The Prompt:**

> "Generate a Python class file for a 'ShoppingCartPage' using the Page Object Model design pattern.
>
> **Requirements:**
>
> 1.  Include a constructor that accepts the `driver`.
> 2.  Define locators at the top of the class as tuples.
> 3.  Create methods for: `get_cart_total()`, `remove_item(item_name)`, and `proceed_to_checkout()`.
> 4.  Use type hinting for all methods."

**The Result:** The AI types out 50 lines of perfect structural code in 10 seconds. You didn't have to remember how to initialize the `__init__` method or look up the import syntax. You just fill in the logic inside the methods.

<br>

### "Explain This Code" Workflows

**Understanding Legacy Scripts**

Have you ever opened a test script written by a guy named "Dave" who left the company 3 years ago? The code is full of variables named `x`, `y`, and `temp`, and there are zero comments. It’s a nightmare.

This is where AI acts as your **Universal Translator**.

**The Workflow:**

1.  Highlight the confusing function.
2.  **Prompt:** *"Explain this Python function to me in plain English. What is the business logic here? Why is there a `try/except` block on line 14?"*

**Deep Dive Idea:**
You can ask the AI to **Refactor and Document** simultaneously.

> "Add Docstrings to the following code following the Google Style Guide. Rename variables `x` and `y` to something descriptive based on how they are used in the logic."

Suddenly, Dave's spaghetti code becomes a clean, well-documented recipe.

-----

## 4. Test Data Generation

Quality Assurance is 20% writing tests and 80% finding the right data to break those tests.

### Generating JSON/CSV Datasets for Edge Cases

Imagine you are testing a bridge. You don't just drive a sedan over it. You drive a tank, a bicycle, and a herd of elephants over it.
In software, "happy path" data (valid emails, short names) is the sedan. We need elephants (edge cases).

Writing JSON files with 50 different edge cases by hand is tedious. AI loves tedious tasks.

**The Prompt:**

> "I need a JSON dataset to test a user registration form.
>
> **Structure:** An array of objects with keys: `username`, `email`, `age`, `password`.
>
> **Task:** Generate 10 entries that test **boundary values and edge cases**:
>
> 1.  Extremely long username (255+ chars).
> 2.  Email with SQL injection characters.
> 3.  Age as a string instead of int.
> 4.  Age as a negative number.
> 5.  Password with emojis.
> 6.  Empty fields.
>
> Output strictly raw JSON."

**Why this is powerful:** The AI knows about "SQL injection characters" (`' OR 1=1 --`). You don't have to look them up. It instantly creates a "Naughty String" list for your automated tests to consume.

<br>

### Creating Synthetic User Profiles (PII-Safe Data)

You should never, ever use real customer data in a test environment. That is a security violation waiting to happen. But you need data that *looks* real. You need **Stunt Doubles**.

If you need a database of 100 users with addresses, phone numbers, and credit card formats, asking AI to generate "Synthetic PII" (Personally Identifiable Information) is the safest route.

**The Prompt:**

> "Generate a CSV format list of 20 synthetic users for a Mexican e-commerce site.
>
> **Columns:** `FirstName`, `LastName`, `Address`, `City`, `PostalCode`, `PhoneNumber`.
>
> **Context:** The addresses must be valid-looking Mexico City addresses (e.g., Colonia Roma, Condesa). The phone numbers must follow the +52 mask.
> **Constraint:** Do NOT use real people. Use completely fictional data."

Now you have a CSV file you can plug into your Data-Driven Testing framework (like Pytest parametrization or TestNG DataProvider) without violating GDPR or local privacy laws.

-----

## 5. Unit Test Generation

### Auto-Generating Unit Tests for Helper Functions


In automation frameworks, we often write helper functions—little tools that calculate dates, parse strings, or format currency. Who tests the testers?

If you write a function to format a date, you need to prove it works.

**The Scenario:** You have a helper function:

```python
def calculate_delivery_date(order_date, business_days):
    # logic to skip weekends
    ...
```

**The Prompt:**

> "I have a Python function `calculate_delivery_date`.
>
> **Task:** Write a `unittest` class to cover this function.
> **Requirements:**
>
> 1.  Test the happy path (standard week).
> 2.  Test the edge case where the delivery falls on a Saturday or Sunday (should move to Monday).
> 3.  Test crossing a month boundary (e.g., Jan 31st to Feb).
> 4.  Test crossing a year boundary."

The AI will generate the `assert` statements for you. It understands calendar logic (Jan has 31 days, Feb has 28) better than we do when we are tired.

-----

## Module Summary & Next Steps

Phew\! We just covered a massive amount of ground. Let’s recap our recipe for success:

1.  **Prompting is Architecture:** We don't just "ask" the AI; we construct prompts using Context, Persona, and Constraints (C.P.S.O.).
2.  **DOM Snippets are Key:** We feed the AI the snippets to get accurate selectors.
3.  **Iterative Refinement:** We are the Senior Devs correcting the AI's "Junior" mistakes (like deprecated code).
4.  **Data Generation:** We use AI to create the "Elephants" (edge cases) to stress-test our apps.



See you in the next lesson\!