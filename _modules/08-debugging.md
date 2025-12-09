---
layout: module
title: "08 Debugging"
permalink: /modules/07-debugging/
---

It is 9:00 AM. You open your laptop, coffee in hand. You check the nightly Jenkins build.

**Status: FAILED.**

Your heart sinks slightly. This is the moment where many testers panic. They stare at the red screen and think, "I broke it." But you are not just a tester anymore; you are a Code Archaeologist.

Maintenance is the "dark matter" of automation. Writing the test takes 1 hour; maintaining it over a year takes 10. If you cannot debug quickly, your automation suite becomes a burden, not an asset.

In this module, we are going to learn how to fix things *fast*. We are going to stop guessing and start diagnosing. We will turn that red "FAILED" status into a green "PASSED" before your coffee gets cold.

---

### **8.1 Root Cause Analysis: The Triage Tent**

When a test fails, it is a cry for help. But who is screaming? Is the application broken? Is your test code wrong? Or is the server just having a bad day?

Distinguishing between these three is the most critical skill in automation.

#### **8.1.1 The "Blame Game": App vs. Auto vs. Env**

Imagine you are a doctor. A patient walks in with a headache.
1.  **Application Bug (The Disease):** The patient actually has the flu. The software is broken. The button doesn't click. The calculation is wrong. *Action: Log a bug for the developer.*
2.  **Automation Bug (The Faulty Thermometer):** The patient is fine, but your thermometer is broken and says they have a fever of 105°F. Your logic is flawed. You used the wrong selector. You expected "User" but the app says "user" (lowercase). *Action: Fix your code.*
3.  **Environment Issue (The Hospital is on Fire):** The patient is fine, the thermometer is fine, but the lights just went out in the hospital. The database is down. The network is slow. The API is returning 503 errors. *Action: Ping the DevOps team.*

**How to tell the difference?**
* *The "Manual Sanity Check":* If the automation fails, try to do exactly the same thing manually.
    * If you *can* do it manually -> It's likely an **Automation Bug**.
    * If you *cannot* do it manually -> It's likely an **Application Bug**.
    * If you cannot even open the website -> It's an **Environment Issue**.

#### **8.1.2 Analyzing Stack Traces Efficiently (Reading the DNA)**

We touched on this in Reporting, but now we go deeper. The Stack Trace is the crime scene map.



* **The Exception Type tells the "What":**
    * `NoSuchElementException`: "I can't find it." (Check your locators/selectors).
    * `StaleElementReferenceException`: "I had it, but it changed." (The page refreshed while you were holding the element).
    * `TimeoutException`: "I waited, but it never came." (Performance issue or network lag).
    * `AssertionError`: "I found it, but it's wrong." (This is usually a real bug!).

* **The Line Number tells the "Where":**
    * Scan past the grey text (Framework code).
    * Look for the **first line** that mentions *your* package name (e.g., `com.company.tests.LoginTest`).
    * *Pro Tip:* In your IDE, this line is blue and clickable. Click it. It transports you exactly to the line of code that pulled the trigger.

---

### **8.2 Debugging Tools: The Surgical Instruments**

Stop using `System.out.println("Here 1");`. Stop it. You are better than that.
Using print statements to debug is like trying to find a needle in a haystack by burning down the haystack. We have laser-precision tools for this.

#### **8.2.1 IDE Breakpoints and "Evaluate Expression" (Freezing Time)**

This is your superpower.

* **Breakpoints (The "Time Stone"):**
    * You click the "gutter" (the margin next to the line numbers) in your IDE (IntelliJ/Eclipse/VS Code). A red dot appears.
    * You run the test in **Debug Mode**.
    * When execution hits that red dot, **Time Stops**.
    * The browser stays open. The variables are frozen in memory. You can look around.

* **Evaluate Expression (The Interrogation):**
    * While time is frozen, you can ask the code questions.
    * Right-click on a variable and select "Evaluate Expression" (or "Watch").
    * *Question:* "What is the value of `loginButton.getText()` right now?"
    * *Answer:* "Log In".
    * *Question:* "Is `loginButton.isDisplayed()`?"
    * *Answer:* `false`.
    * *Aha Moment:* You realized the button is in the DOM (code) but hidden from the user. That’s why the click failed! You solved it without re-running the test 50 times.

#### **8.2.2 Browser DevTools: Under the Hood**

Sometimes the IDE can't see the problem because it's happening in the browser's brain, not Java's.

* **The Console (The Diary):**
    * Open Chrome DevTools (F12) -> Console.
    * Look for Red text. "Uncaught TypeError: Cannot read property of undefined."
    * If you see this, the Javascript on the page crashed. Even if your Selenium/Playwright code is perfect, the button won't work because the brain of the page is dead. Take a screenshot of *this* and send it to the developer. They will love you for it.

* **The Network Tab (The Wiretap):**
    * Your test fails saying "Search results not found."
    * Open the Network tab. Run the search.
    * Look for the request to `/api/search`.
    * Is it Red (404/500)? The server is broken.
    * Is it Green (200 OK) but the `Response` is empty `[]`? The database is empty.
    * This distinguishes a "Frontend Issue" (UI not showing data) from a "Backend Issue" (API not sending data).

---

### **8.3 Handling Flakiness: The Ghost in the Machine**

A "Flaky Test" is a test that passes sometimes and fails sometimes, without any code changes.
It is the enemy of trust. If your tests are flaky, developers will stop listening to you. They will say, "Oh, just run it again, it's probably nothing."
**We must kill flakiness.**

#### **8.3.1 Strategies for Retrying (The Band-Aid)**

Sometimes, the internet hiccups. A packet gets lost. It happens.

* **The Retry Listener:**
    * Most frameworks allow you to implement an `IRetryAnalyzer`.
    * *Logic:* If (Test Failed) AND (Count < 2) -> Run Again.
    * *The Danger:* Do not rely on this to fix bad code! If a test needs 3 tries to pass, it is a bad test. Retries are for *network blips*, not for *race conditions*.
    * *Rule of Thumb:* If it fails twice, it's a fail. Don't loop forever.

#### **8.3.2 Stabilizing Environment Timing (The Cure)**

90% of flakiness comes from **Timing**.
* *Scenario:* The automation clicks "Save." The automation immediately checks for the "Success" message.
* *Reality:* The "Save" takes 500 milliseconds. The check happens at 10ms. The test fails. You re-run it. This time the server is faster (400ms). The check happens at 410ms. The test passes. **This is flakiness.**

* **The Fix: Explicit Waits (Smart Waits)**
    * Never use `Thread.sleep(5000)`. This is "Dumb Waiting." You are forcing the test to sleep even if the page is ready in 1 second. It slows everything down.
    * Use **Explicit Waits**: `Wait until (element is visible)`.
    * *Metaphor:*
        * *Thread.sleep:* You order a pizza and sit by the door for exactly 30 minutes, ignoring the doorbell. If the pizza comes in 10 mins, it gets cold. If it comes in 40, you miss it.
        * *Explicit Wait:* You sit by the door until the doorbell rings. If it rings in 5 mins, you open it immediately. If it doesn't ring after 30 mins (Timeout), you complain.

### **Summary: From Panic to Precision**

By the end of this module, you will stop fearing the red "FAILED" text. You will see it as a puzzle.

1.  You will use **Root Cause Analysis** to triage the problem instantly.
2.  You will use **Debuggers** to perform surgery on running code.
3.  You will eliminate **Flakiness** by synchronizing your robot with the speed of the application.

A broken test is not a failure of the tester. It is an opportunity to fix a hole in the net.
Go fix it.
