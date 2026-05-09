---
title: "JavaScript Crash Course"
description: "Learn JavaScript fundamentals including functions, objects, promises, and patterns used in AngularJS applications"
date: 2025-01-25
draft: false
weight: 3
tags: ["javascript", "programming", "fundamentals", "angularjs", "promises"]
categories: ["foundations"]
prev: "/advanced/widgets/html_crash_course/"
next: "/advanced/widgets/critical_web_foundations/"
---

## What is JavaScript?
JavaScript is a programming language that makes web pages interactive. It runs in browsers and can manipulate HTML, handle events, make calculations, and communicate with servers.

## Basic Syntax

### Variables
```javascript
// ES5 (AngularJS era)
var name = "John";
var age = 25;
var isActive = true;

// ES6+ (Modern JavaScript)
let username = "jane_doe";    // Block-scoped, can change
const API_URL = "https://api.example.com";  // Block-scoped, cannot change
```

### Data Types
```javascript
// Primitive types
var text = "Hello World";        // String
var number = 42;                 // Number
var decimal = 3.14;              // Number (no separate float type)
var isTrue = true;               // Boolean
var nothing = null;              // Null
var undefined = undefined;       // Undefined

// Complex types
var array = [1, 2, 3, "four"];  // Array
var object = {                   // Object
    name: "John",
    age: 30,
    active: true
};
```

## Functions

### Function Declaration (Hoisted)
```javascript
function greet(name) {
    return "Hello, " + name + "!";
}

function calculateArea(width, height) {
    var area = width * height;
    return area;
}

// Call the function
var message = greet("Alice");
var area = calculateArea(10, 5);
```

### Function Expression
```javascript
var greet = function(name) {
    return "Hello, " + name + "!";
};

// Anonymous function
var numbers = [1, 2, 3];
var doubled = numbers.map(function(num) {
    return num * 2;
});
```

### Arrow Functions (ES6+)
```javascript
// Modern syntax (not in AngularJS era)
const greet = (name) => {
    return `Hello, ${name}!`;
};

// Shorter syntax
const double = (x) => x * 2;
const add = (a, b) => a + b;
```

## Objects

### Creating Objects
```javascript
// Object literal
var person = {
    name: "John",
    age: 30,
    city: "New York",
    
    // Method
    introduce: function() {
        return "Hi, I'm " + this.name;
    }
};

// Constructor function (AngularJS pattern)
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.introduce = function() {
        return "Hi, I'm " + this.name;
    };
}

var john = new Person("John", 30);
```

### Accessing Properties
```javascript
var person = {name: "John", age: 30};

// Dot notation
console.log(person.name);        // "John"
person.age = 31;

// Bracket notation
console.log(person["name"]);     // "John"
person["city"] = "Boston";

// Dynamic property access
var property = "name";
console.log(person[property]);   // "John"
```

## Arrays

### Basic Array Operations
```javascript
var fruits = ["apple", "banana", "orange"];

// Access elements
console.log(fruits[0]);          // "apple"
console.log(fruits.length);      // 3

// Add elements
fruits.push("grape");            // Add to end
fruits.unshift("mango");         // Add to beginning

// Remove elements
var last = fruits.pop();         // Remove from end
var first = fruits.shift();      // Remove from beginning

// Find element
var index = fruits.indexOf("banana");  // Returns index or -1
```

### Array Methods (Important for AngularJS)
```javascript
var numbers = [1, 2, 3, 4, 5];

// Loop through array
numbers.forEach(function(num, index) {
    console.log(index + ": " + num);
});

// Transform array
var doubled = numbers.map(function(num) {
    return num * 2;
});

// Filter array
var evenNumbers = numbers.filter(function(num) {
    return num % 2 === 0;
});

// Find element
var found = numbers.find(function(num) {
    return num > 3;
});

// Check if any/all match condition
var hasEven = numbers.some(function(num) {
    return num % 2 === 0;
});

var allPositive = numbers.every(function(num) {
    return num > 0;
});
```

## Control Flow

### Conditionals
```javascript
var age = 18;

// If statement
if (age >= 18) {
    console.log("Adult");
} else if (age >= 13) {
    console.log("Teenager");
} else {
    console.log("Child");
}

// Ternary operator
var status = age >= 18 ? "Adult" : "Minor";

// Switch statement
var day = "Monday";
switch (day) {
    case "Monday":
        console.log("Start of work week");
        break;
    case "Friday":
        console.log("TGIF!");
        break;
    default:
        console.log("Regular day");
}
```

### Loops
```javascript
// For loop
for (var i = 0; i < 5; i++) {
    console.log("Count: " + i);
}

// For...in loop (objects)
var person = {name: "John", age: 30, city: "NYC"};
for (var key in person) {
    console.log(key + ": " + person[key]);
}

// While loop
var count = 0;
while (count < 3) {
    console.log("Count: " + count);
    count++;
}

// Do...while loop
var num = 0;
do {
    console.log("Number: " + num);
    num++;
} while (num < 3);
```

## Scope and Context

### Variable Scope (var vs let/const)
```javascript
function example() {
    if (true) {
        var functionScoped = "I'm function scoped";
        let blockScoped = "I'm block scoped";
    }
    
    console.log(functionScoped);  // Works - function scoped
    console.log(blockScoped);     // Error - block scoped
}

// Global scope
var globalVar = "I'm global";

function test() {
    console.log(globalVar);  // Accessible
}
```

### The 'this' Keyword
```javascript
var person = {
    name: "John",
    greet: function() {
        console.log("Hello, I'm " + this.name);
    },
    
    delayedGreet: function() {
        var self = this;  // Common AngularJS pattern
        setTimeout(function() {
            console.log("Hello, I'm " + self.name);
        }, 1000);
    }
};

person.greet();         // "Hello, I'm John"
person.delayedGreet();  // "Hello, I'm John" (after 1 second)
```

## Error Handling
```javascript
try {
    var result = riskyOperation();
    console.log("Success: " + result);
} catch (error) {
    console.log("Error occurred: " + error.message);
} finally {
    console.log("This always runs");
}

// Throwing errors
function divide(a, b) {
    if (b === 0) {
        throw new Error("Division by zero!");
    }
    return a / b;
}
```

## Asynchronous JavaScript

### Callbacks
```javascript
function fetchData(callback) {
    setTimeout(function() {
        var data = {id: 1, name: "John"};
        callback(data);
    }, 1000);
}

fetchData(function(data) {
    console.log("Received:", data);
});
```

### Promises (Modern, but important for AngularJS)
```javascript
function fetchUser(id) {
    return new Promise(function(resolve, reject) {
        setTimeout(function() {
            if (id > 0) {
                resolve({id: id, name: "User " + id});
            } else {
                reject(new Error("Invalid ID"));
            }
        }, 1000);
    });
}

// Using promises
fetchUser(1)
    .then(function(user) {
        console.log("User:", user);
        return fetchUser(2);  // Chain another promise
    })
    .then(function(user2) {
        console.log("User 2:", user2);
    })
    .catch(function(error) {
        console.log("Error:", error.message);
    });
```

## DOM Manipulation (Important Background)

### Selecting Elements
```javascript
// By ID
var element = document.getElementById("myId");

// By class name
var elements = document.getElementsByClassName("myClass");

// By tag name
var paragraphs = document.getElementsByTagName("p");

// CSS selectors (modern)
var element = document.querySelector("#myId");
var elements = document.querySelectorAll(".myClass");
```

### Manipulating Elements
```javascript
var element = document.getElementById("myDiv");

// Change content
element.innerHTML = "New content";
element.textContent = "Plain text";

// Change attributes
element.setAttribute("class", "newClass");
element.style.color = "red";

// Add event listener
element.addEventListener("click", function() {
    console.log("Element clicked!");
});
```

## JavaScript Patterns Used in AngularJS

### Module Pattern (IIFE)
```javascript
(function() {
    'use strict';
    
    // Private variables
    var privateVar = "secret";
    
    // Private function
    function privateFunction() {
        return "private";
    }
    
    // Public API
    window.MyModule = {
        publicMethod: function() {
            return privateFunction();
        }
    };
})();
```

### Dependency Injection Pattern
```javascript
// Function with dependencies
function MyController($scope, $http, MyService) {
    $scope.data = {};
    
    MyService.getData().then(function(response) {
        $scope.data = response;
    });
}

// Explicit dependency declaration (AngularJS pattern)
MyController.$inject = ['$scope', '$http', 'MyService'];
```

### Factory Pattern
```javascript
function createUser(name, email) {
    return {
        name: name,
        email: email,
        introduce: function() {
            return "I'm " + this.name;
        }
    };
}

var user1 = createUser("John", "john@example.com");
var user2 = createUser("Jane", "jane@example.com");
```

## Examples from Your AngularJS Code

### Controller Function
```javascript
function editC3Charts110DevCtrl($scope, $uibModalInstance, config, appModulesService, Entity, SORT_ORDER, CommonUtils) {
    // Set initial data
    $scope.config = config;
    $scope.config.customFilters = $scope.config.customFilters || {'limit':1, 'sort': ['ASC']};
    
    // Attach functions to scope
    $scope.cancel = cancel;
    $scope.save = save;
    $scope.customResourceReset = customResourceReset;
    $scope.SORT_ORDER = SORT_ORDER;

    // Private function
    function cancel() {
        $uibModalInstance.dismiss('cancel');
    }

    function save() {
        if ($scope.editWidgetForm.$invalid) {
            $scope.editWidgetForm.$setTouched();
            $scope.editWidgetForm.$focusOnFirstError();
            return;
        }
        $scope.processing = true;
        
        // Generate UUID
        if (!$scope.config.correlationValue) {
            var uniqueValue = CommonUtils.generateUUID();
            $scope.config['correlationValue'] = uniqueValue;
        }
        
        $uibModalInstance.close($scope.config);
    }
}
```

### Working with Promises
```javascript
appModulesService.load().then(function(modules) {
    $scope.modules = modules;
    
    // Object initialization
    $scope.moduleFields = {};
    $scope.moduleFieldsArrays = {};
    
    if ($scope.config.customResource && !$scope.moduleFields[$scope.config.customResource]) {
        populateFieldLists($scope.config.customResource);
    }
});
```

### Dynamic Object Property Access
```javascript
function populateFieldLists(resource) {
    let crEntity = new Entity(resource);
    crEntity.loadFields().then(function() {
        // Loop through object properties
        for (var key in crEntity.fields) {
            if (crEntity.fields[key].type === 'datetime') {
                crEntity.fields[key].type = 'datetime.quick';
            }
        }
        
        // Dynamic property assignment
        $scope.moduleFields[resource] = crEntity.fields;
        $scope.moduleFieldsArrays[resource] = crEntity.getFormFieldsArray();
    });
}
```

## Common JavaScript Gotchas

### 1. Truthy/Falsy Values
```javascript
// Falsy values
if (!false) console.log("false is falsy");
if (!0) console.log("0 is falsy");
if (!"") console.log("empty string is falsy");
if (!null) console.log("null is falsy");
if (!undefined) console.log("undefined is falsy");

// Everything else is truthy
if ("0") console.log("String '0' is truthy");
if ([]) console.log("Empty array is truthy");
if ({}) console.log("Empty object is truthy");
```

### 2. Type Coercion
```javascript
"5" + 3        // "53" (string concatenation)
"5" - 3        // 2 (numeric subtraction)
"5" == 5       // true (loose equality)
"5" === 5      // false (strict equality)
```

### 3. Variable Hoisting
```javascript
console.log(x);  // undefined (not error)
var x = 5;

// Is equivalent to:
var x;
console.log(x);  // undefined
x = 5;
```

## Best Practices for AngularJS Development

### 1. Use Strict Mode
```javascript
(function() {
    'use strict';
    // Your code here
})();
```

### 2. Avoid Global Variables
```javascript
// Bad
var globalVar = "avoid this";

// Good
(function() {
    var localVar = "better";
})();
```

### 3. Use Meaningful Names
```javascript
// Bad
function d(x, y) {
    return x * y;
}

// Good
function calculateArea(width, height) {
    return width * height;
}
```

### 4. Handle Errors Properly
```javascript
function saveData(data) {
    try {
        return apiService.save(data);
    } catch (error) {
        console.error('Save failed:', error);
        throw error;
    }
}
```

This JavaScript foundation will help you understand how AngularJS works under the hood and write better AngularJS applications!