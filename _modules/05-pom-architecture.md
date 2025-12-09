---
layout: module
title: "05 POM Architecture"
permalink: /modules/04-pom-architecture/
---


**Welcome Back, Super Coders! 👩‍💻👨‍💻**



Grab your coffee (or tea 🍵), stretch those fingers, and get comfortable. Today is a **big** day.



Up until now, we’ve been writing scripts that work, right? We launch the browser, find an element, click it, and celebrate. It feels good\\! But, imagine for a second that you are building a house of cards. The first few cards are easy. But what happens when you try to add the second story? Or the third?



If your foundation is shaky, the whole thing collapses. 💥



In the world of Test Automation, "Spaghetti Code" is that shaky foundation. It’s code where the logic, the locators, and the test data are all mixed up in a giant, tangled bowl of pasta. It tastes good at first, but try separating the noodles later—it’s a nightmare!



Today, we are going to learn the **Page Object Model (POM)**. This is the industry-standard "Clean Up Your Room" protocol. It’s how we take messy, fragile scripts and turn them into robust, professional, and scalable engineering masterpieces.



By the end of this module, you won't just be writing tests; you’ll be architecting solutions. Let’s dive in! 🚀



-----



## **1. Design Pattern Fundamentals**



Before we write a single line of code, we need to change how we **think**.



### **The Problem: The "One-Man Show"**



Imagine a restaurant where the Chef is also the Waiter, the Host, and the Dishwasher. The customer walks in, and the Chef runs out of the kitchen, takes the order, runs back, chops the onions, runs back out to pour water, runs back to flip the steak...



It’s chaos. If the Chef decides to change the menu (the UI changes), the whole process breaks down. This is how beginners write tests:


 * **Test Script:** "Go to login page. Find the box with ID 'user'. Type name. Find button. Click."



Every time the UI changes (e.g., the ID 'user' becomes 'username'), you have to fix **every single test script** that uses the login. That is unmaintainable.



### **The Solution: Separation of Concerns**



POM is about specialization. We are going to split our code into two distinct worlds:



#### **A. The Test Layer**



Think of your Test Class as a **Food Critic**. The Critic doesn't care **how** the food is cooked. They don't care if the Chef used a gas stove or an induction cooktop. They don't want to know where the spoons are kept.



The Critic only cares about the **outcome**:



* "I ordered a steak."

* "Did I get a steak?"

* "Is it medium-rare?"



In code, your Test Layer should read like plain English. It should state **Business Logic** only.



* **Good Test:** `loginPage.loginAs("Angela");`

* **Bad Test:** `driver.findElement(By.id("username")).sendKeys("Angela");`



#### **B. The Page Layer**



The Page Object is the **Kitchen**. This is where the dirty work happens. This is where we store the ingredients (Locators/WebElements) and the recipes (Methods).



If the restaurant management decides to move the spoons from the top drawer to the bottom drawer (UI change), the **Critic** (Test) doesn't need to know! Only the **Kitchen Staff** (Page Object) needs to adjust. The Critic still just asks for a spoon and gets one.



### **The Golden Rule of POM**



> **"Tests should strictly judge the outcome. Pages should strictly perform the actions."**



-----



## **2. Structure Implementation**



Now that we have the theory, let's look at the anatomy of a POM framework. We are going to build this like a Lego set, piece by piece.



### **Part 1: The BasePage**



Imagine you are Batman. You have a utility belt. No matter what mission you are on—whether you are fighting the Joker or saving a cat—you always need your grappling hook and your smoke bombs.



In automation, every single page in your application interacts with the browser driver. Every page needs to click, type, and wait. We don't want to write `driver.findElement` and `wait.until` a thousand times.



So, we create a **BasePage**. This is a parent class that all other Page Classes will extend (inherit from).



**What lives here?**

The BasePage wraps the raw Selenium/Appium commands into safer, more stable methods.



* **The Wrapper Concept:** instead of calling `element.click()` directly, we create a method `click(element)`. Why? Because inside our custom method, we can say: "Before you click, **wait** for the element to be visible. If it fails, try one more time. Then log a message saying 'Clicked element'."

* **Centralized Driver Management:** The BasePage holds the reference to the `WebDriver`. It ensures that the browser instance is passed correctly from page to page.



**An Analogy:**

Think of the BasePage as the **Universal Remote Control**. You don't get up and manually press buttons on the TV, the DVD player, and the Sound System. You use the remote. The remote handles the signal sending; you just press "Play."



### **Part 2: The Page Classes**



This is the core of POM. For every unique screen in your application (Login Page, Dashboard, Settings, Cart), we create a specific Java/Python class.



**Encapsulation: The "Private" Members Club**

One of the most important concepts in Object-Oriented Programming (OOP) is **Encapsulation**. In POM, this means:



* **Locators (WebElements) should be PRIVATE.**

* **Methods (Actions) should be PUBLIC.**



**Why?**

Let’s go back to the Restaurant. You (the Test) are not allowed to walk into the kitchen and grab a tomato (the WebElement). You might grab a rotten one, or you might slip and fall. You must ask the Chef to "Prepare the Salad" (Public Method). The Chef handles the tomato privately.



If your Test Class can access a WebElement directly (e.g., `loginPage.usernameBox.sendKeys("Hi")`), you have broken the pattern\\! You are leaking implementation details.



**The Structure of a Page Class:**



1\.  **Variables:** The Locators (IDs, XPaths, CSS Selectors) are defined at the top.

2\.  **Constructor:** It initializes the driver (connecting it to the BasePage).

3\.  **Action Methods:** These represent user behaviors.

* `enterUsername(String user)`

* `clickSubmit()`

* `getErrorMessageText()`



### **Part 3: The Test Classes**



This is where the magic happens. Because we did all the hard work in the BasePage and Page Classes, our Test Class is clean, beautiful, and easy to read.



A non-technical Manager should be able to look at your Test Class and understand exactly what is being tested.



**The "Fluent" Style**

We want our tests to flow.



**Bad:**

```java

LoginPage login = new LoginPage(driver);

login.enterUser("Tim");

login.enterPass("123");

login.clickBtn();

DashboardPage dash = new DashboardPage(driver);

dash.checkBalance();

```

**Good:**

```java

new LoginPage(driver)
.loginAs("Tim", "123")
.verifySuccessfulLogin()
.navigateToSettings();

```


See the difference? The second one tells a story. It hides the "plumbing" and focuses on the "user journey."



-----



## **3. Page Factory vs. By Locators**



In the Selenium world, there are two main ways to identify your elements inside the Page Class. This is often a heated debate\\! Let’s break it down so you can choose the right tool.



### **Option A: The `By` Locator Strategy**



This is the classic approach. You define your locators as simple `By` objects.

`By usernameId = By.id("user\_123");`



* **How it works:** You have the address of the element, but you don't go look for it until the exact moment you need it.

* **The Metaphor:** This is like a **Scavenger Hunt List**. You have a piece of paper that says "Find the Red Ball." You don't have the ball yet. You only look for the ball when the game starts.

* **Pros:** It is very stable. Because you find the element **fresh** every time you interact with it, you rarely get the dreaded `StaleElementReferenceException`.

* **Cons:** It can look a bit verbose (lots of `driver.findElement(usernameId)` calls).



### **Option B: Page Factory**



This is the "Magic" approach provided by Selenium. You use annotations like `@FindBy`.

`@FindBy(id = "user\_123") WebElement username;`



* **How it works:** You initialize the elements when the Class starts using `PageFactory.initElements`.

* **The Metaphor:** This is like a **Lazy Butler**. You give the Butler a list of items you **might** need. The Butler stands there, waiting. As soon as you say "I need the username," the Butler magically hands it to you.

* **The "CacheLookup" Trap:** You can tell the Butler to keep holding the item (Caching). This is faster, but if the page refreshes, the item in the Butler's hand is no longer valid (Stale\\!).

* **Pros:** The code looks cleaner and more readable. No `driver.findElement` calls cluttering your methods.

* **Cons:** It can be buggy with dynamic elements (elements that appear/disappear/reload). The Selenium team has actually discussed deprecating it in future versions because it adds complexity.



**My Advice?**

Start with **By Locators**. It gives you more control. Once you are a master, you can play with Page Factory. But remember: Control > Magic.



-----



## **4. Component Objects**



As your application grows, you will notice something annoying.

You have a **Header** (with search bar, profile icon, logout).

You have a **Footer** (with copyright, social links).

You have a **Navigation Bar**.



These elements appear on **every single page**.



**The Copy-Paste Trap ⚠️**

If you put the `logout()` method in the `DashboardPage`, and then also in the `SettingsPage`, and also in the `ProfilePage`... you are violating the **DRY Principle (Don't Repeat Yourself)**.

If the Logout button ID changes, you have to fix it in 20 different files. 😫



**The Solution: Component Objects**

We treat these shared sections not as full Pages, but as **Components**.



Think of your website like a customized **Mr. Potato Head**.



* The Body is the specific Page (Dashboard, Settings).

* The Hat (Header) is a Component.

* The Shoes (Footer) is a Component.



We build the "Header" logic **once** in a `HeaderComponent` class. Then, we plug that component into any Page that needs it.



**Composition over Inheritance**

Instead of the `DashboardPage` **inheriting** the Header (which gets messy), the `DashboardPage` **has a** Header.



* `dashboard.header().searchFor("Products");`

* `settings.header().searchFor("Products");`



This makes your framework incredibly powerful. You are building reusable Lego blocks. If the Header changes, you fix the `HeaderComponent` once, and boom! 💥 It’s fixed everywhere.



-----



## **Deep Dive: The "Why" behind the "How"**



I want to pause here and talk about **Maintainability**. I know, it sounds like a boring corporate word. But let me frame it differently.



Maintainability = Your Weekend Freedom. 🏖️



**If you write a script without POM:**



1\.  The developer changes the ID of the 'Submit' button on Friday at 4 PM.

2\.  Your suite of 500 tests fails.

3\.  You spend Friday night and Saturday morning finding every line of code where you clicked that button and changing it. 😭



**If you use POM:**



1\.  The developer changes the ID.

2\.  Your 500 tests fail.

3\.  You go to `LoginPage.java`.

4\.  You change **one line**: `By submitBtn = By.id("new\_id");`

5\.  You run the tests. They pass.

6\.  You go home at 5 PM and enjoy your pizza. 🍕



POM is not just about code quality; it's about respecting your own time as an engineer. It is an investment. It takes longer to set up initially (writing the classes, planning the structure), but the return on investment (ROI) is massive.



-----



## **Summary**



Let's recap what we've learned in this deep dive:



1\.  **Separation of Concerns:** The Test Layer is the "Client" ordering food; the Page Layer is the "Kitchen" making it. Never mix them.

2\.  **BasePage:** Your utility belt. The central place for common actions (click, wait, type) and driver management.

3\.  **Page Classes:** Encapsulated blueprints of your screens. Private WebElements, Public Actions.

4\.  **Test Classes:** Clean, readable business logic. No technical jargon allowed here.

5\.  **Page Factory vs. By:** Choose stability (By) over magic (Factory) until you are ready for advanced caching.

6\.  **Components:** Don't repeat code for Headers and Footers. Modularize them like Lego blocks.







In the next module, we are going to look at **Assertions & Reporting**. But for now, master the architecture.



You are doing great. Keep coding, keep breaking things, and keep fixing them. See you in the next lesson\\!


