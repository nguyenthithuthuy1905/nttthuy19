---
title: "JavaScript Essentials 2 - Objects, This & Prototype"
date: 2024-12-21T01:00:00+07:00
draft: false
tags: ["javascript", "objects", "this", "prototype", "oop"]
categories: ["JavaScript Essentials"]
description: "Tìm hiểu sâu về Objects, This keyword và Prototype trong JavaScript"
---

# JavaScript Essentials 2 - Objects, This & Prototype

## 🎯 **Mục tiêu học tập**
- Hiểu sâu về Objects và cách hoạt động của `this`
- Nắm vững Prototype chain và inheritance
- Thực hành với Object methods và properties
- Áp dụng OOP patterns trong JavaScript

## 📦 **1. Objects - Cơ bản**

### **Tạo Objects**
```javascript
// Object literal
const person = {
    name: "Nguyễn Văn A",
    age: 25,
    greet: function() {
        return `Xin chào, tôi là ${this.name}`;
    }
};

// Constructor function
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.greet = function() {
        return `Xin chào, tôi là ${this.name}`;
    };
}

const person1 = new Person("Trần Thị B", 30);
```

### **Object Properties**
```javascript
const student = {
    firstName: "Nguyễn",
    lastName: "Thị D",
    age: 22,
    courses: ["JavaScript", "React", "Node.js"]
};

// Truy cập properties
console.log(student.firstName);        // Dot notation
console.log(student["lastName"]);     // Bracket notation

// Thêm properties
student.email = "student@example.com";
student["phone"] = "0123456789";

// Kiểm tra properties
console.log("firstName" in student);           // true
console.log(student.hasOwnProperty("age"));     // true
```

## 🔗 **2. This Keyword**

### **This trong các context khác nhau**
```javascript
// 1. This trong object method
const obj = {
    name: "Object",
    getName: function() {
        return this.name; // this = obj
    }
};

// 2. This trong function thông thường
function regularFunction() {
    console.log(this); // this = window (browser) hoặc global (Node.js)
}

// 3. This trong arrow function
const arrowFunction = () => {
    console.log(this); // this = lexical scope (không có this riêng)
};

// 4. This với call, apply, bind
const person1 = { name: "Alice" };
const person2 = { name: "Bob" };

function greet() {
    return `Hello, I'm ${this.name}`;
}

console.log(greet.call(person1));    // "Hello, I'm Alice"
console.log(greet.apply(person2));   // "Hello, I'm Bob"

const boundGreet = greet.bind(person1);
console.log(boundGreet());           // "Hello, I'm Alice"
```

### **This trong Event Handlers**
```javascript
// HTML: <button id="myButton">Click me</button>

const button = document.getElementById('myButton');

// Regular function - this = button element
button.addEventListener('click', function() {
    console.log(this); // <button id="myButton">Click me</button>
});

// Arrow function - this = window/global
button.addEventListener('click', () => {
    console.log(this); // Window object
});
```

## 🔗 **3. Prototypes - Cốt lõi của JavaScript**

### **Prototype Chain**
```javascript
// Mọi object đều có prototype
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true

// Prototype chain
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    return `${this.name} makes a sound`;
};

function Dog(name, breed) {
    Animal.call(this, name);
    this.breed = breed;
}

// Kế thừa prototype
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    return `${this.name} barks: Woof!`;
};

const myDog = new Dog("Buddy", "Golden Retriever");
console.log(myDog.speak()); // "Buddy makes a sound"
console.log(myDog.bark());  // "Buddy barks: Woof!"
```

### **Modern Inheritance với ES6 Classes**
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
    
    // Static method
    static getSpecies() {
        return "Unknown species";
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name); // Gọi constructor của parent
        this.breed = breed;
    }
    
    speak() {
        return `${this.name} barks: Woof!`;
    }
    
    // Getter
    get info() {
        return `${this.name} is a ${this.breed}`;
    }
    
    // Setter
    set age(value) {
        if (value > 0) {
            this._age = value;
        }
    }
    
    get age() {
        return this._age || "Unknown";
    }
}

const dog = new Dog("Max", "Labrador");
console.log(dog.speak()); // "Max barks: Woof!"
console.log(dog.info);    // "Max is a Labrador"
dog.age = 3;
console.log(dog.age);     // 3
```

## 🛠️ **4. Object Methods và Utilities**

### **Object Methods**
```javascript
const user = {
    firstName: "Nguyễn",
    lastName: "Văn E",
    age: 25,
    email: "user@example.com",
    isActive: true
};

// Object.keys() - Lấy tất cả keys
console.log(Object.keys(user));
// ["firstName", "lastName", "age", "email", "isActive"]

// Object.values() - Lấy tất cả values
console.log(Object.values(user));
// ["Nguyễn", "Văn E", 25, "user@example.com", true]

// Object.entries() - Lấy key-value pairs
console.log(Object.entries(user));
// [["firstName", "Nguyễn"], ["lastName", "Văn E"], ...]

// Object.assign() - Copy properties
const userCopy = Object.assign({}, user);
const userWithId = Object.assign({}, user, { id: 1 });

// Spread operator (ES6+)
const userSpread = { ...user, id: 2, role: "admin" };
```

### **Object Destructuring**
```javascript
const person = {
    name: "Trần Thị F",
    age: 30,
    address: {
        city: "Hồ Chí Minh",
        district: "Quận 1"
    },
    hobbies: ["reading", "coding", "gaming"]
};

// Destructuring cơ bản
const { name, age } = person;
console.log(name, age); // "Trần Thị F" 30

// Destructuring với rename
const { name: fullName, age: personAge } = person;

// Destructuring nested objects
const { address: { city, district } } = person;
console.log(city, district); // "Hồ Chí Minh" "Quận 1"

// Destructuring với default values
const { name, age, phone = "N/A" } = person;

// Destructuring trong function parameters
function greetUser({ name, age = 0 }) {
    return `Xin chào ${name}, bạn ${age} tuổi`;
}

console.log(greetUser(person)); // "Xin chào Trần Thị F, bạn 30 tuổi"
```

## 🎨 **5. Advanced Object Patterns**

### **Factory Pattern**
```javascript
function createUser(type, name, email) {
    const user = {
        name,
        email,
        createdAt: new Date(),
        isActive: true
    };
    
    switch(type) {
        case 'admin':
            return {
                ...user,
                role: 'admin',
                permissions: ['read', 'write', 'delete'],
                canDelete: true
            };
        case 'user':
            return {
                ...user,
                role: 'user',
                permissions: ['read'],
                canDelete: false
            };
        case 'moderator':
            return {
                ...user,
                role: 'moderator',
                permissions: ['read', 'write'],
                canDelete: false
            };
        default:
            throw new Error('Invalid user type');
    }
}

const admin = createUser('admin', 'Admin User', 'admin@example.com');
const user = createUser('user', 'Regular User', 'user@example.com');
```

### **Module Pattern**
```javascript
const UserModule = (function() {
    let users = [];
    let nextId = 1;
    
    return {
        addUser: function(name, email) {
            const user = {
                id: nextId++,
                name,
                email,
                createdAt: new Date()
            };
            users.push(user);
            return user;
        },
        
        getUser: function(id) {
            return users.find(user => user.id === id);
        },
        
        getAllUsers: function() {
            return [...users]; // Return copy
        },
        
        deleteUser: function(id) {
            users = users.filter(user => user.id !== id);
        },
        
        getUserCount: function() {
            return users.length;
        }
    };
})();

// Sử dụng
UserModule.addUser("Nguyễn Văn G", "user1@example.com");
UserModule.addUser("Trần Thị H", "user2@example.com");
console.log(UserModule.getAllUsers());
```

## 🔧 **6. Object Property Descriptors**

### **Property Descriptors**
```javascript
const obj = {};

// Thêm property với descriptor
Object.defineProperty(obj, 'name', {
    value: 'Test Object',
    writable: true,      // Có thể thay đổi
    enumerable: true,    // Xuất hiện trong Object.keys()
    configurable: true   // Có thể xóa hoặc thay đổi descriptor
});

// Getter và Setter
Object.defineProperty(obj, 'age', {
    get: function() {
        return this._age;
    },
    set: function(value) {
        if (value >= 0) {
            this._age = value;
        } else {
            throw new Error('Age must be positive');
        }
    },
    enumerable: true,
    configurable: true
});

obj.age = 25;
console.log(obj.age); // 25

// Kiểm tra descriptor
console.log(Object.getOwnPropertyDescriptor(obj, 'name'));
```

### **Seal, Freeze, và PreventExtensions**
```javascript
const obj = {
    name: "Test",
    value: 100
};

// Object.seal() - Không thể thêm/xóa properties
Object.seal(obj);
obj.name = "Modified"; // ✅ OK
obj.newProp = "New";   // ❌ Không hoạt động
delete obj.value;       // ❌ Không hoạt động

// Object.freeze() - Không thể thay đổi gì
Object.freeze(obj);
obj.name = "Changed";   // ❌ Không hoạt động

// Object.preventExtensions() - Chỉ không thể thêm properties
Object.preventExtensions(obj);
obj.name = "Modified"; // ✅ OK
obj.newProp = "New";   // ❌ Không hoạt động
```

## 🎯 **7. Thực hành nâng cao**

### **Bài tập 1: Tạo Class Library**
```javascript
class Library {
    constructor(name) {
        this.name = name;
        this.books = [];
        this.members = [];
    }
    
    addBook(book) {
        this.books.push({
            ...book,
            id: this.books.length + 1,
            isAvailable: true
        });
    }
    
    addMember(member) {
        this.members.push({
            ...member,
            id: this.members.length + 1,
            borrowedBooks: []
        });
    }
    
    borrowBook(memberId, bookId) {
        const member = this.members.find(m => m.id === memberId);
        const book = this.books.find(b => b.id === bookId);
        
        if (!member || !book) {
            throw new Error('Member or book not found');
        }
        
        if (!book.isAvailable) {
            throw new Error('Book is not available');
        }
        
        book.isAvailable = false;
        member.borrowedBooks.push(bookId);
        
        return `Member ${member.name} borrowed ${book.title}`;
    }
    
    returnBook(memberId, bookId) {
        const member = this.members.find(m => m.id === memberId);
        const book = this.books.find(b => b.id === bookId);
        
        if (!member || !book) {
            throw new Error('Member or book not found');
        }
        
        book.isAvailable = true;
        member.borrowedBooks = member.borrowedBooks.filter(id => id !== bookId);
        
        return `Member ${member.name} returned ${book.title}`;
    }
}

// Sử dụng
const myLibrary = new Library("City Library");
myLibrary.addBook({ title: "JavaScript Guide", author: "John Doe" });
myLibrary.addMember({ name: "Alice", email: "alice@example.com" });
console.log(myLibrary.borrowBook(1, 1));
```

### **Bài tập 2: Observer Pattern**
```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, callback) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(callback);
    }
    
    emit(event, data) {
        if (this.events[event]) {
            this.events[event].forEach(callback => callback(data));
        }
    }
    
    off(event, callback) {
        if (this.events[event]) {
            this.events[event] = this.events[event].filter(cb => cb !== callback);
        }
    }
}

// Sử dụng
const emitter = new EventEmitter();

emitter.on('userLogin', (user) => {
    console.log(`User ${user.name} logged in`);
});

emitter.on('userLogin', (user) => {
    console.log(`Welcome back, ${user.name}!`);
});

emitter.emit('userLogin', { name: 'John', id: 1 });
```

## 🎯 **Tóm tắt kiến thức**

### **Điểm quan trọng cần nhớ:**
1. **Objects**: Literal, constructor, Object.create()
2. **This**: Context-dependent, call/apply/bind, arrow functions
3. **Prototypes**: Chain inheritance, ES6 classes
4. **Object Methods**: keys, values, entries, assign
5. **Destructuring**: Basic, nested, default values
6. **Patterns**: Factory, Module, Observer

### **Best Practices:**
- Sử dụng ES6 classes cho OOP
- Tận dụng destructuring để code ngắn gọn
- Sử dụng Object methods thay vì for...in
- Áp dụng design patterns phù hợp
- Immutable objects khi có thể

### **Ứng dụng thực tế:**
- Component state management
- API response handling
- Configuration objects
- Event systems
- Data modeling

---

**Chúc bạn học tập hiệu quả! 🚀**
