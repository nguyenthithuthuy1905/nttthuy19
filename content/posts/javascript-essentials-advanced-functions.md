---
title: "JavaScript Essentials - Advanced Functions & Closures"
date: 2024-12-20T22:00:00+07:00
draft: false
tags: ["javascript", "functions", "closures", "advanced", "scope"]
categories: ["JavaScript Essentials"]
description: "Khám phá các khái niệm nâng cao về functions trong JavaScript: closures, hoisting, arrow functions và functional programming"
---

# JavaScript Essentials - Advanced Functions & Closures

## 🎯 **Mục tiêu học tập**
- Hiểu sâu về scope và closures trong JavaScript
- Nắm vững arrow functions và functional programming
- Thực hành với higher-order functions
- Áp dụng closures trong thực tế

## 📚 **1. Scope và Lexical Scoping**

### **Global Scope vs Local Scope**
```javascript
// Global scope
var globalVar = "Tôi là global";

function outerFunction() {
    // Local scope của outerFunction
    var outerVar = "Tôi là outer";
    
    function innerFunction() {
        // Local scope của innerFunction
        var innerVar = "Tôi là inner";
        
        console.log(globalVar); // ✅ Có thể truy cập
        console.log(outerVar);  // ✅ Có thể truy cập
        console.log(innerVar);  // ✅ Có thể truy cập
    }
    
    innerFunction();
    // console.log(innerVar); // ❌ Lỗi: innerVar không tồn tại ở đây
}

outerFunction();
```

### **Block Scope với let và const**
```javascript
function demonstrateBlockScope() {
    if (true) {
        var varVariable = "var - function scoped";
        let letVariable = "let - block scoped";
        const constVariable = "const - block scoped";
    }
    
    console.log(varVariable);   // ✅ "var - function scoped"
    // console.log(letVariable);  // ❌ ReferenceError
    // console.log(constVariable); // ❌ ReferenceError
}

demonstrateBlockScope();
```

## 🔒 **2. Closures - Khái niệm quan trọng**

### **Closure cơ bản**
```javascript
function createCounter() {
    let count = 0; // Biến private
    
    return function() {
        count++;
        return count;
    };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 (độc lập với counter1)
console.log(counter1()); // 3
```

### **Closure với tham số**
```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### **Closure trong thực tế - Module Pattern**
```javascript
const Calculator = (function() {
    let result = 0;
    
    return {
        add: function(num) {
            result += num;
            return this;
        },
        subtract: function(num) {
            result -= num;
            return this;
        },
        multiply: function(num) {
            result *= num;
            return this;
        },
        getResult: function() {
            return result;
        },
        reset: function() {
            result = 0;
            return this;
        }
    };
})();

// Sử dụng
Calculator.add(10).multiply(2).subtract(5);
console.log(Calculator.getResult()); // 15
```

## 🏹 **3. Arrow Functions**

### **Cú pháp cơ bản**
```javascript
// Function thông thường
function add(a, b) {
    return a + b;
}

// Arrow function
const addArrow = (a, b) => a + b;

// Với một tham số
const square = x => x * x;

// Không có tham số
const sayHello = () => "Hello World!";

// Với nhiều dòng
const complexFunction = (a, b) => {
    const sum = a + b;
    const product = a * b;
    return { sum, product };
};
```

### **Arrow Functions vs Regular Functions**
```javascript
const obj = {
    name: "JavaScript",
    
    // Regular function - có this riêng
    regularMethod: function() {
        console.log("Regular:", this.name);
        
        // Inner function - mất this
        function inner() {
            console.log("Inner:", this.name); // undefined
        }
        inner();
    },
    
    // Arrow function - không có this riêng
    arrowMethod: function() {
        console.log("Arrow method:", this.name);
        
        // Arrow function - giữ this từ context cha
        const inner = () => {
            console.log("Arrow inner:", this.name); // "JavaScript"
        };
        inner();
    }
};

obj.regularMethod();
obj.arrowMethod();
```

## 🔄 **4. Higher-Order Functions**

### **Functions nhận function làm tham số**
```javascript
// Callback function
function processData(data, callback) {
    console.log("Đang xử lý:", data);
    const result = callback(data);
    console.log("Kết quả:", result);
    return result;
}

// Sử dụng
processData([1, 2, 3, 4, 5], function(arr) {
    return arr.map(x => x * 2);
});

// Với arrow function
processData([1, 2, 3, 4, 5], arr => arr.filter(x => x > 2));
```

### **Functions trả về function**
```javascript
function createValidator(rule) {
    return function(value) {
        return rule(value);
    };
}

const isEmail = createValidator(value => 
    value.includes('@') && value.includes('.')
);

const isLongEnough = createValidator(value => 
    value.length >= 8
);

console.log(isEmail("test@example.com")); // true
console.log(isLongEnough("password123")); // true
```

## 🎨 **5. Functional Programming với JavaScript**

### **Pure Functions**
```javascript
// Pure function - không có side effects
function pureAdd(a, b) {
    return a + b;
}

// Impure function - có side effects
let total = 0;
function impureAdd(a) {
    total += a; // Thay đổi biến global
    return total;
}
```

### **Map, Filter, Reduce**
```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Map - biến đổi từng phần tử
const doubled = numbers.map(x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

// Filter - lọc các phần tử
const evens = numbers.filter(x => x % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// Reduce - tích lũy kết quả
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 55

// Reduce phức tạp hơn
const stats = numbers.reduce((acc, curr) => {
    acc.sum += curr;
    acc.count++;
    acc.average = acc.sum / acc.count;
    return acc;
}, { sum: 0, count: 0, average: 0 });

console.log(stats); // { sum: 55, count: 10, average: 5.5 }
```

### **Chaining Methods**
```javascript
const products = [
    { name: "Laptop", price: 1000, category: "Electronics" },
    { name: "Book", price: 20, category: "Education" },
    { name: "Phone", price: 800, category: "Electronics" },
    { name: "Pen", price: 2, category: "Office" }
];

const expensiveElectronics = products
    .filter(product => product.category === "Electronics")
    .filter(product => product.price > 500)
    .map(product => ({
        ...product,
        discountedPrice: product.price * 0.9
    }))
    .sort((a, b) => b.price - a.price);

console.log(expensiveElectronics);
```

## 🛠️ **6. Thực hành nâng cao**

### **Bài tập 1: Tạo Calculator với Closures**
```javascript
function createAdvancedCalculator() {
    let history = [];
    
    return {
        add: (a, b) => {
            const result = a + b;
            history.push(`${a} + ${b} = ${result}`);
            return result;
        },
        
        subtract: (a, b) => {
            const result = a - b;
            history.push(`${a} - ${b} = ${result}`);
            return result;
        },
        
        getHistory: () => [...history],
        clearHistory: () => history = []
    };
}

const calc = createAdvancedCalculator();
calc.add(5, 3);
calc.subtract(10, 4);
console.log(calc.getHistory());
```

### **Bài tập 2: Debounce Function**
```javascript
function debounce(func, delay) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Sử dụng cho search input
const searchInput = document.getElementById('search');
const debouncedSearch = debounce(function(event) {
    console.log('Searching for:', event.target.value);
}, 300);

searchInput.addEventListener('input', debouncedSearch);
```

## 🎯 **Tóm tắt kiến thức**

### **Điểm quan trọng cần nhớ:**
1. **Scope**: Hiểu rõ global, function, và block scope
2. **Closures**: Giữ lại quyền truy cập vào biến của function cha
3. **Arrow Functions**: Cú pháp ngắn gọn, không có this riêng
4. **Higher-Order Functions**: Functions nhận hoặc trả về functions
5. **Functional Programming**: Pure functions, immutability

### **Best Practices:**
- Sử dụng `const` và `let` thay vì `var`
- Tận dụng closures để tạo private variables
- Arrow functions cho callbacks ngắn
- Pure functions khi có thể
- Method chaining để code dễ đọc

### **Ứng dụng thực tế:**
- Module pattern với closures
- Event handlers với debounce/throttle
- Data transformation với map/filter/reduce
- Configuration objects với factory functions

---

**Chúc bạn học tập hiệu quả! 🚀**
