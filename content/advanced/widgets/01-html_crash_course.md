---
title: "HTML Crash Course"
description: "Master HTML fundamentals including forms, elements, and semantic markup essential for AngularJS development"
date: 2025-01-25
weight: 2
tags: ["html", "web-development", "fundamentals", "angularjs"]
categories: ["foundations"]
prev: "/advanced/widgets/"
next: "/advanced/widgets/javascript_crash_course/"
---

## What is HTML?
HTML (HyperText Markup Language) is the standard markup language for creating web pages. It describes the structure and content of web pages using elements (tags).

## Basic HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Web Page</title>
</head>
<body>
    <h1>Welcome to My Website</h1>
    <p>This is a paragraph.</p>
</body>
</html>
```

## HTML Elements and Tags

### Basic Syntax
```html
<tagname attribute="value">Content</tagname>
```

- **Opening tag**: `<tagname>`
- **Closing tag**: `</tagname>`
- **Self-closing**: `<tagname />`
- **Attributes**: Extra information about elements

## Essential HTML Elements

### Headings
```html
<h1>Main Heading</h1>
<h2>Sub Heading</h2>
<h3>Smaller Heading</h3>
<!-- h4, h5, h6 also available -->
```

### Text Elements
```html
<p>This is a paragraph.</p>
<span>Inline text</span>
<strong>Bold text</strong>
<em>Italic text</em>
<br> <!-- Line break -->
```

### Lists
```html
<!-- Unordered List -->
<ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
</ul>

<!-- Ordered List -->
<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
</ol>
```

### Links and Images
```html
<a href="https://example.com">Visit Example</a>
<img src="image.jpg" alt="Description of image">
```

## Container Elements (Critical for AngularJS)

### Div - Generic Container
```html
<div class="container">
    <p>Content inside a div</p>
</div>
```

### Semantic Containers
```html
<header>Page header content</header>
<nav>Navigation menu</nav>
<main>Main content</main>
<section>A section of content</section>
<article>An article</article>
<aside>Sidebar content</aside>
<footer>Page footer</footer>
```

## Forms (Essential for AngularJS)

### Basic Form Structure
```html
<form action="/submit" method="POST">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" required>
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <button type="submit">Submit</button>
</form>
```

### Input Types
```html
<input type="text" placeholder="Enter text">
<input type="email" placeholder="Enter email">
<input type="password" placeholder="Enter password">
<input type="number" min="1" max="100">
<input type="date">
<input type="checkbox" id="agree">
<input type="radio" name="gender" value="male">
<input type="file">
<input type="hidden" value="secret">
```

### Select Dropdown
```html
<label for="country">Country:</label>
<select id="country" name="country">
    <option value="">Choose a country</option>
    <option value="us">United States</option>
    <option value="ca">Canada</option>
    <option value="uk">United Kingdom</option>
</select>
```

### Textarea
```html
<label for="message">Message:</label>
<textarea id="message" name="message" rows="4" cols="50">
    Default text here
</textarea>
```

## HTML Attributes

### Common Attributes
```html
<div id="unique-id" class="css-class another-class" title="Tooltip text">
    Content
</div>

<input type="text" name="username" value="default" placeholder="Enter username" required disabled>

<a href="https://example.com" target="_blank" rel="noopener">External Link</a>
```

### Data Attributes (Important for AngularJS)
```html
<div data-user-id="123" data-role="admin">User Info</div>
<button data-action="save" data-target="form1">Save</button>
```

## CSS Classes and IDs

### Classes (Reusable)
```html
<div class="header-section primary-color">
<p class="text-large text-bold">Important text</p>
<button class="btn btn-primary btn-small">Click me</button>
```

### IDs (Unique)
```html
<div id="main-navigation">
<form id="login-form">
<button id="submit-button">Submit</button>
```

## Tables
```html
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Age</th>
            <th>City</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>John</td>
            <td>25</td>
            <td>New York</td>
        </tr>
        <tr>
            <td>Jane</td>
            <td>30</td>
            <td>London</td>
        </tr>
    </tbody>
</table>
```

## HTML Validation Attributes
```html
<input type="text" required>
<input type="email" required pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$">
<input type="number" min="1" max="100" step="1">
<input type="text" minlength="3" maxlength="20">
```

## Connecting to Your AngularJS Code

### Form Structure from Your Code
```html
<form data-ng-submit="save()" class="noMargin" name="editWidgetForm" novalidate>
    <!-- Form content -->
</form>
```

**Key Points:**
- `data-ng-submit`: AngularJS directive for form submission
- `name="editWidgetForm"`: Creates form object accessible in controller
- `novalidate`: Disables browser validation (AngularJS handles it)

### Input with AngularJS
```html
<input id="title" 
       name="title" 
       type="text" 
       class="form-control" 
       data-ng-model="config.title" 
       required>
```

**Breakdown:**
- `id` and `name`: Standard HTML attributes
- `type="text"`: HTML input type
- `class="form-control"`: CSS classes for styling
- `data-ng-model`: AngularJS two-way binding
- `required`: HTML5 validation attribute

### Select with AngularJS
```html
<select name="customResource" 
        id="customResource" 
        class="form-control" 
        data-ng-options="module.type as module.name for module in modules" 
        data-ng-model="config.customResource" 
        required>
    <option value="">Select an Option</option>
</select>
```

## Best Practices

### 1. Semantic HTML
Use elements for their intended purpose:
```html
<!-- Good -->
<button type="submit">Submit</button>
<nav>Navigation content</nav>

<!-- Avoid -->
<div onclick="submit()">Submit</div>
<div>Navigation content</div>
```

### 2. Accessibility
```html
<label for="username">Username:</label>
<input type="text" id="username" aria-describedby="username-help">
<div id="username-help">Enter your username</div>

<button aria-label="Close dialog">×</button>
```

### 3. Proper Nesting
```html
<!-- Correct -->
<div>
    <p>Text inside paragraph</p>
</div>

<!-- Incorrect -->
<p>
    <div>Block element inside inline</div>
</p>
```

### 4. Self-closing Tags
```html
<!-- HTML5 style -->
<img src="image.jpg" alt="Description">
<br>
<input type="text">

<!-- XHTML style (also valid) -->
<img src="image.jpg" alt="Description" />
<br />
<input type="text" />
```

## Common HTML Patterns in Web Applications

### Modal Dialog Structure
```html
<div class="modal">
    <div class="modal-header">
        <h3>Modal Title</h3>
        <button type="button" class="close">×</button>
    </div>
    <div class="modal-body">
        <p>Modal content goes here</p>
    </div>
    <div class="modal-footer">
        <button type="button" class="btn btn-primary">Save</button>
        <button type="button" class="btn btn-default">Cancel</button>
    </div>
</div>
```

### Form with Validation Messages
```html
<div class="form-group">
    <label for="email">Email</label>
    <input type="email" id="email" class="form-control" required>
    <div class="error-message">Please enter a valid email</div>
</div>
```

### Navigation Menu
```html
<nav class="navbar">
    <ul class="nav-list">
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
    </ul>
</nav>
```

## HTML Comments
```html
<!-- This is a comment -->
<!-- 
    Multi-line comment
    More comment text
-->
```

## Why This Matters for AngularJS

1. **AngularJS extends HTML** - It adds new attributes and behaviors to standard HTML
2. **Form validation** - Understanding HTML form elements is crucial for AngularJS forms
3. **Data binding** - AngularJS binds to HTML elements and attributes
4. **Directives** - Many AngularJS directives work with specific HTML elements
5. **Structure** - Proper HTML structure makes AngularJS applications more maintainable

This foundation will help you understand how AngularJS enhances and works with standard HTML elements!