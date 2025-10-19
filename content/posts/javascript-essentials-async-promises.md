---
title: "JavaScript Essentials - Async Programming & Promises"
date: 2024-12-21T00:00:00+07:00
draft: false
tags: ["javascript", "async", "promises", "async-await", "callbacks"]
categories: ["JavaScript Essentials"]
description: "Khám phá lập trình bất đồng bộ trong JavaScript: callbacks, promises, async/await và error handling"
---

# JavaScript Essentials - Async Programming & Promises

## 🎯 **Mục tiêu học tập**
- Hiểu về lập trình bất đồng bộ trong JavaScript
- Nắm vững Promises và async/await
- Thực hành với API calls và error handling
- Áp dụng async patterns trong thực tế

## ⏰ **1. Callbacks - Cơ bản về Async**

### **Callback Functions**
```javascript
// Synchronous callback
function processArray(arr, callback) {
    const result = [];
    for (let i = 0; i < arr.length; i++) {
        result.push(callback(arr[i]));
    }
    return result;
}

const numbers = [1, 2, 3, 4, 5];
const doubled = processArray(numbers, x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Asynchronous callback
function fetchData(callback) {
    setTimeout(() => {
        const data = { name: "John", age: 30 };
        callback(null, data); // Error-first callback pattern
    }, 1000);
}

fetchData((error, data) => {
    if (error) {
        console.error("Error:", error);
    } else {
        console.log("Data:", data);
    }
});
```

### **Callback Hell Problem**
```javascript
// Callback hell - khó đọc và maintain
function getUserData(userId, callback) {
    setTimeout(() => {
        const user = { id: userId, name: "John" };
        callback(null, user);
    }, 500);
}

function getUserPosts(userId, callback) {
    setTimeout(() => {
        const posts = [{ id: 1, title: "Post 1" }, { id: 2, title: "Post 2" }];
        callback(null, posts);
    }, 300);
}

function getUserComments(postId, callback) {
    setTimeout(() => {
        const comments = [{ id: 1, text: "Great post!" }];
        callback(null, comments);
    }, 200);
}

// Callback hell
getUserData(1, (err, user) => {
    if (err) {
        console.error(err);
        return;
    }
    
    getUserPosts(user.id, (err, posts) => {
        if (err) {
            console.error(err);
            return;
        }
        
        getUserComments(posts[0].id, (err, comments) => {
            if (err) {
                console.error(err);
                return;
            }
            
            console.log("User:", user);
            console.log("Posts:", posts);
            console.log("Comments:", comments);
        });
    });
});
```

## 🎯 **2. Promises - Giải pháp cho Callback Hell**

### **Promise cơ bản**
```javascript
// Tạo Promise
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (userId > 0) {
                const user = { id: userId, name: "John", email: "john@example.com" };
                resolve(user);
            } else {
                reject(new Error("Invalid user ID"));
            }
        }, 1000);
    });
}

// Sử dụng Promise
fetchUserData(1)
    .then(user => {
        console.log("User data:", user);
        return user.id; // Return value cho then tiếp theo
    })
    .then(userId => {
        console.log("User ID:", userId);
    })
    .catch(error => {
        console.error("Error:", error.message);
    })
    .finally(() => {
        console.log("Request completed");
    });
```

### **Promise Chaining**
```javascript
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve({ id: userId, name: "John" });
        }, 500);
    });
}

function fetchUserPosts(userId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve([
                { id: 1, title: "Post 1", userId: userId },
                { id: 2, title: "Post 2", userId: userId }
            ]);
        }, 300);
    });
}

function fetchPostComments(postId) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve([
                { id: 1, text: "Great post!", postId: postId }
            ]);
        }, 200);
    });
}

// Promise chaining - giải quyết callback hell
fetchUserData(1)
    .then(user => {
        console.log("User:", user);
        return fetchUserPosts(user.id);
    })
    .then(posts => {
        console.log("Posts:", posts);
        return fetchPostComments(posts[0].id);
    })
    .then(comments => {
        console.log("Comments:", comments);
    })
    .catch(error => {
        console.error("Error:", error);
    });
```

### **Promise.all và Promise.race**
```javascript
// Promise.all - Chờ tất cả promises hoàn thành
const promise1 = fetchUserData(1);
const promise2 = fetchUserData(2);
const promise3 = fetchUserData(3);

Promise.all([promise1, promise2, promise3])
    .then(users => {
        console.log("All users:", users);
    })
    .catch(error => {
        console.error("One or more requests failed:", error);
    });

// Promise.race - Lấy kết quả của promise hoàn thành đầu tiên
const fastPromise = new Promise(resolve => setTimeout(() => resolve("Fast"), 100));
const slowPromise = new Promise(resolve => setTimeout(() => resolve("Slow"), 1000));

Promise.race([fastPromise, slowPromise])
    .then(result => {
        console.log("Winner:", result); // "Fast"
    });

// Promise.allSettled - Chờ tất cả, không quan tâm success/failure
Promise.allSettled([promise1, promise2, promise3])
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Promise ${index + 1} succeeded:`, result.value);
            } else {
                console.log(`Promise ${index + 1} failed:`, result.reason);
            }
        });
    });
```

## ⚡ **3. Async/Await - Cú pháp hiện đại**

### **Async/Await cơ bản**
```javascript
// Chuyển đổi từ Promise sang async/await
async function getUserDataAsync(userId) {
    try {
        const user = await fetchUserData(userId);
        console.log("User:", user);
        
        const posts = await fetchUserPosts(user.id);
        console.log("Posts:", posts);
        
        const comments = await fetchPostComments(posts[0].id);
        console.log("Comments:", comments);
        
        return { user, posts, comments };
    } catch (error) {
        console.error("Error:", error);
        throw error;
    }
}

// Sử dụng async function
getUserDataAsync(1)
    .then(result => {
        console.log("Complete result:", result);
    })
    .catch(error => {
        console.error("Failed:", error);
    });
```

### **Parallel Execution với Async/Await**
```javascript
async function fetchAllUserData(userIds) {
    try {
        // Parallel execution
        const userPromises = userIds.map(id => fetchUserData(id));
        const users = await Promise.all(userPromises);
        
        console.log("All users:", users);
        return users;
    } catch (error) {
        console.error("Error fetching users:", error);
        throw error;
    }
}

// Sequential vs Parallel
async function sequentialFetch() {
    const user1 = await fetchUserData(1);
    const user2 = await fetchUserData(2);
    const user3 = await fetchUserData(3);
    return [user1, user2, user3];
}

async function parallelFetch() {
    const [user1, user2, user3] = await Promise.all([
        fetchUserData(1),
        fetchUserData(2),
        fetchUserData(3)
    ]);
    return [user1, user2, user3];
}
```

## 🌐 **4. Fetch API - HTTP Requests**

### **Fetch cơ bản**
```javascript
// GET request
async function fetchUserFromAPI(userId) {
    try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const user = await response.json();
        return user;
    } catch (error) {
        console.error("Fetch error:", error);
        throw error;
    }
}

// POST request
async function createUser(userData) {
    try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(userData)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const newUser = await response.json();
        return newUser;
    } catch (error) {
        console.error("Create user error:", error);
        throw error;
    }
}

// Sử dụng
fetchUserFromAPI(1)
    .then(user => console.log("User:", user))
    .catch(error => console.error("Error:", error));
```

### **Advanced Fetch với Error Handling**
```javascript
class ApiClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        const config = {
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            },
            ...options
        };
        
        try {
            const response = await fetch(url, config);
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            const data = await response.json();
            return data;
        } catch (error) {
            console.error(`API request failed: ${endpoint}`, error);
            throw error;
        }
    }
    
    async get(endpoint) {
        return this.request(endpoint, { method: 'GET' });
    }
    
    async post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    async put(endpoint, data) {
        return this.request(endpoint, {
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    async delete(endpoint) {
        return this.request(endpoint, { method: 'DELETE' });
    }
}

// Sử dụng ApiClient
const api = new ApiClient('https://jsonplaceholder.typicode.com');

async function demonstrateApiClient() {
    try {
        const users = await api.get('/users');
        console.log("Users:", users);
        
        const newUser = await api.post('/users', {
            name: "New User",
            email: "newuser@example.com"
        });
        console.log("Created user:", newUser);
        
    } catch (error) {
        console.error("API error:", error);
    }
}
```

## 🔄 **5. Error Handling Patterns**

### **Try-Catch với Async/Await**
```javascript
async function robustDataFetch() {
    try {
        const response = await fetch('https://api.example.com/data');
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const data = await response.json();
        
        // Validate data
        if (!data || !Array.isArray(data)) {
            throw new Error('Invalid data format');
        }
        
        return data;
    } catch (error) {
        if (error.name === 'TypeError' && error.message.includes('fetch')) {
            console.error('Network error:', error);
            throw new Error('Network connection failed');
        } else if (error.name === 'SyntaxError') {
            console.error('JSON parsing error:', error);
            throw new Error('Invalid response format');
        } else {
            console.error('Unknown error:', error);
            throw error;
        }
    }
}
```

### **Retry Pattern**
```javascript
async function fetchWithRetry(url, maxRetries = 3, delay = 1000) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const response = await fetch(url);
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            console.log(`Attempt ${attempt} failed:`, error.message);
            
            if (attempt === maxRetries) {
                throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
            }
            
            // Exponential backoff
            const waitTime = delay * Math.pow(2, attempt - 1);
            console.log(`Waiting ${waitTime}ms before retry...`);
            await new Promise(resolve => setTimeout(resolve, waitTime));
        }
    }
}

// Sử dụng
fetchWithRetry('https://api.example.com/data', 3, 1000)
    .then(data => console.log("Success:", data))
    .catch(error => console.error("Failed:", error));
```

## 🎯 **6. Thực hành nâng cao**

### **Bài tập 1: Weather App với Async**
```javascript
class WeatherService {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseURL = 'https://api.openweathermap.org/data/2.5';
    }
    
    async getCurrentWeather(city) {
        try {
            const response = await fetch(
                `${this.baseURL}/weather?q=${city}&appid=${this.apiKey}&units=metric`
            );
            
            if (!response.ok) {
                throw new Error(`Weather API error: ${response.status}`);
            }
            
            const data = await response.json();
            return {
                city: data.name,
                temperature: data.main.temp,
                description: data.weather[0].description,
                humidity: data.main.humidity,
                windSpeed: data.wind.speed
            };
        } catch (error) {
            console.error('Weather fetch error:', error);
            throw error;
        }
    }
    
    async getForecast(city, days = 5) {
        try {
            const response = await fetch(
                `${this.baseURL}/forecast?q=${city}&appid=${this.apiKey}&units=metric`
            );
            
            if (!response.ok) {
                throw new Error(`Forecast API error: ${response.status}`);
            }
            
            const data = await response.json();
            return data.list.slice(0, days).map(item => ({
                date: new Date(item.dt * 1000),
                temperature: item.main.temp,
                description: item.weather[0].description
            }));
        } catch (error) {
            console.error('Forecast fetch error:', error);
            throw error;
        }
    }
}

// Sử dụng
const weatherService = new WeatherService('your-api-key');

async function getWeatherInfo(city) {
    try {
        const [currentWeather, forecast] = await Promise.all([
            weatherService.getCurrentWeather(city),
            weatherService.getForecast(city)
        ]);
        
        console.log("Current Weather:", currentWeather);
        console.log("5-Day Forecast:", forecast);
        
        return { currentWeather, forecast };
    } catch (error) {
        console.error("Weather service error:", error);
        throw error;
    }
}
```

### **Bài tập 2: File Upload với Progress**
```javascript
async function uploadFileWithProgress(file, onProgress) {
    return new Promise((resolve, reject) => {
        const formData = new FormData();
        formData.append('file', file);
        
        const xhr = new XMLHttpRequest();
        
        // Track upload progress
        xhr.upload.addEventListener('progress', (event) => {
            if (event.lengthComputable) {
                const percentComplete = (event.loaded / event.total) * 100;
                onProgress(percentComplete);
            }
        });
        
        xhr.addEventListener('load', () => {
            if (xhr.status === 200) {
                resolve(JSON.parse(xhr.responseText));
            } else {
                reject(new Error(`Upload failed: ${xhr.status}`));
            }
        });
        
        xhr.addEventListener('error', () => {
            reject(new Error('Upload failed'));
        });
        
        xhr.open('POST', '/api/upload');
        xhr.send(formData);
    });
}

// Sử dụng
const fileInput = document.getElementById('fileInput');
fileInput.addEventListener('change', async (event) => {
    const file = event.target.files[0];
    if (!file) return;
    
    try {
        const result = await uploadFileWithProgress(file, (progress) => {
            console.log(`Upload progress: ${progress.toFixed(2)}%`);
        });
        
        console.log('Upload successful:', result);
    } catch (error) {
        console.error('Upload failed:', error);
    }
});
```

## 🎯 **Tóm tắt kiến thức**

### **Điểm quan trọng cần nhớ:**
1. **Callbacks**: Error-first pattern, callback hell
2. **Promises**: then/catch/finally, chaining, all/race
3. **Async/Await**: Cú pháp đồng bộ cho async code
4. **Fetch API**: HTTP requests với error handling
5. **Error Handling**: Try-catch, retry patterns

### **Best Practices:**
- Sử dụng async/await thay vì callbacks
- Luôn handle errors với try-catch
- Sử dụng Promise.all cho parallel requests
- Implement retry logic cho network requests
- Validate API responses

### **Ứng dụng thực tế:**
- API integration
- File uploads với progress
- Real-time data fetching
- Error recovery mechanisms
- Performance optimization

---

**Chúc bạn học tập hiệu quả! 🚀**
