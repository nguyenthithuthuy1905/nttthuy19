---
title: "Hiểu sâu về Fetch API — nền tảng của mọi yêu cầu HTTP hiện đại"
date: 2024-01-15T13:00:00+07:00
draft: false
description: "Hướng dẫn chi tiết về Fetch API trong JavaScript, từ cơ bản đến nâng cao với ví dụ thực tế"
tags: ["javascript", "fetch", "api", "http", "async", "network"]
categories: ["javascript-essentials"]
---

## Giới thiệu về Fetch API

Fetch API là một interface hiện đại trong JavaScript để thực hiện các yêu cầu HTTP. Nó được thiết kế để thay thế XMLHttpRequest với cú pháp đơn giản hơn và hỗ trợ Promise natively. Fetch API cung cấp một cách mạnh mẽ và linh hoạt để tương tác với các API REST và các tài nguyên web khác.

### Tại sao Fetch API quan trọng?

- **Promise-based**: Tích hợp tự nhiên với async/await
- **Streaming**: Hỗ trợ đọc dữ liệu theo stream
- **Request/Response objects**: Cung cấp control chi tiết
- **Modern syntax**: Cú pháp đơn giản và dễ hiểu
- **Built-in JSON support**: Xử lý JSON tự động

## Cú pháp cơ bản của Fetch API

### Cú pháp đơn giản nhất

```javascript
// GET request cơ bản
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

### Sử dụng async/await

```javascript
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error('Error:', error);
  }
}

fetchData();
```

## Các phương thức HTTP với Fetch

### GET Request

```javascript
async function getUsers() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const users = await response.json();
    console.log('Users:', users);
    return users;
  } catch (error) {
    console.error('Error fetching users:', error);
  }
}

getUsers();
```

### POST Request với JSON

```javascript
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
    console.log('Created user:', newUser);
    return newUser;
  } catch (error) {
    console.error('Error creating user:', error);
  }
}

// Sử dụng
const userData = {
  name: 'John Doe',
  email: 'john@example.com',
  username: 'johndoe'
};

createUser(userData);
```

### PUT Request

```javascript
async function updateUser(userId, userData) {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const updatedUser = await response.json();
    console.log('Updated user:', updatedUser);
    return updatedUser;
  } catch (error) {
    console.error('Error updating user:', error);
  }
}

// Sử dụng
updateUser(1, {
  name: 'Jane Doe',
  email: 'jane@example.com'
});
```

### DELETE Request

```javascript
async function deleteUser(userId) {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`, {
      method: 'DELETE'
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    console.log('User deleted successfully');
    return true;
  } catch (error) {
    console.error('Error deleting user:', error);
    return false;
  }
}

// Sử dụng
deleteUser(1);
```

## Xử lý Response chi tiết

### Kiểm tra Response Status

```javascript
async function fetchWithStatusCheck(url) {
  try {
    const response = await fetch(url);
    
    // Kiểm tra status code
    if (response.status === 200) {
      console.log('Request successful');
    } else if (response.status === 404) {
      console.log('Resource not found');
    } else if (response.status >= 500) {
      console.log('Server error');
    }
    
    // Kiểm tra response.ok (status 200-299)
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch error:', error);
  }
}
```

### Xử lý các loại Response khác nhau

```javascript
async function handleDifferentResponseTypes(url) {
  try {
    const response = await fetch(url);
    
    // Kiểm tra Content-Type
    const contentType = response.headers.get('content-type');
    
    if (contentType && contentType.includes('application/json')) {
      return await response.json();
    } else if (contentType && contentType.includes('text/')) {
      return await response.text();
    } else if (contentType && contentType.includes('image/')) {
      return await response.blob();
    } else {
      return await response.arrayBuffer();
    }
  } catch (error) {
    console.error('Error handling response:', error);
  }
}
```

## Headers và Authentication

### Thêm Custom Headers

```javascript
async function fetchWithHeaders(url) {
  try {
    const response = await fetch(url, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer your-token-here',
        'X-Custom-Header': 'custom-value',
        'User-Agent': 'MyApp/1.0'
      }
    });
    
    return await response.json();
  } catch (error) {
    console.error('Error with headers:', error);
  }
}
```

### Authentication với Bearer Token

```javascript
class ApiClient {
  constructor(baseURL, token) {
    this.baseURL = baseURL;
    this.token = token;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    
    const defaultOptions = {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.token}`
      }
    };
    
    const config = { ...defaultOptions, ...options };
    
    try {
      const response = await fetch(url, config);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('API request failed:', error);
      throw error;
    }
  }
  
  async get(endpoint) {
    return this.request(endpoint);
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
    return this.request(endpoint, {
      method: 'DELETE'
    });
  }
}

// Sử dụng
const apiClient = new ApiClient('https://api.example.com', 'your-token');
apiClient.get('/users').then(users => console.log(users));
```

## Xử lý Timeout và Error

### Timeout với AbortController

```javascript
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    
    clearTimeout(timeoutId);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    clearTimeout(timeoutId);
    
    if (error.name === 'AbortError') {
      throw new Error('Request timeout');
    }
    
    throw error;
  }
}

// Sử dụng
fetchWithTimeout('https://api.example.com/slow-endpoint', 3000)
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

### Retry Logic

```javascript
async function fetchWithRetry(url, maxRetries = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.log(`Attempt ${attempt} failed:`, error.message);
      
      if (attempt === maxRetries) {
        throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
      }
      
      // Delay trước khi retry
      await new Promise(resolve => setTimeout(resolve, delay * attempt));
    }
  }
}

// Sử dụng
fetchWithRetry('https://api.example.com/unreliable-endpoint')
  .then(data => console.log('Success:', data))
  .catch(error => console.error('Failed:', error));
```

## So sánh Fetch API với XMLHttpRequest

### XMLHttpRequest (Cách cũ)

```javascript
function fetchWithXHR(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open('GET', url);
    xhr.setRequestHeader('Content-Type', 'application/json');
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        try {
          const data = JSON.parse(xhr.responseText);
          resolve(data);
        } catch (error) {
          reject(new Error('Invalid JSON response'));
        }
      } else {
        reject(new Error(`HTTP error! status: ${xhr.status}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Network error'));
    };
    
    xhr.send();
  });
}
```

### Fetch API (Cách mới)

```javascript
async function fetchWithFetch(url) {
  try {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    throw error;
  }
}
```

### Bảng so sánh

| Đặc điểm | XMLHttpRequest | Fetch API |
|----------|----------------|-----------|
| **Cú pháp** | Phức tạp, callback-based | Đơn giản, Promise-based |
| **Streaming** | Hạn chế | Hỗ trợ tốt |
| **Request/Response** | Không có object riêng | Có Request/Response objects |
| **CORS** | Phức tạp | Đơn giản |
| **Browser Support** | Rộng rãi | Modern browsers |
| **Error Handling** | Manual | Built-in |

## Demo thực tế: Gọi API thật

### GitHub API Example

```javascript
class GitHubAPI {
  constructor(token) {
    this.baseURL = 'https://api.github.com';
    this.token = token;
  }
  
  async getUser(username) {
    try {
      const response = await fetch(`${this.baseURL}/users/${username}`, {
        headers: {
          'Authorization': `token ${this.token}`,
          'Accept': 'application/vnd.github.v3+json'
        }
      });
      
      if (!response.ok) {
        if (response.status === 404) {
          throw new Error('User not found');
        }
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Error fetching user:', error);
      throw error;
    }
  }
  
  async getUserRepos(username, page = 1, perPage = 10) {
    try {
      const response = await fetch(
        `${this.baseURL}/users/${username}/repos?page=${page}&per_page=${perPage}&sort=updated`,
        {
          headers: {
            'Authorization': `token ${this.token}`,
            'Accept': 'application/vnd.github.v3+json'
          }
        }
      );
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Error fetching repos:', error);
      throw error;
    }
  }
  
  async searchRepositories(query, page = 1) {
    try {
      const response = await fetch(
        `${this.baseURL}/search/repositories?q=${encodeURIComponent(query)}&page=${page}&per_page=10`,
        {
          headers: {
            'Authorization': `token ${this.token}`,
            'Accept': 'application/vnd.github.v3+json'
          }
        }
      );
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Error searching repos:', error);
      throw error;
    }
  }
}

// Sử dụng GitHub API
const github = new GitHubAPI('your-github-token');

async function displayUserInfo(username) {
  try {
    const user = await github.getUser(username);
    console.log('User Info:', {
      name: user.name,
      login: user.login,
      bio: user.bio,
      followers: user.followers,
      following: user.following,
      public_repos: user.public_repos
    });
    
    const repos = await github.getUserRepos(username);
    console.log('Recent Repositories:');
    repos.forEach(repo => {
      console.log(`- ${repo.name}: ${repo.description || 'No description'}`);
    });
    
  } catch (error) {
    console.error('Error:', error.message);
  }
}

// Sử dụng
displayUserInfo('octocat');
```

### Weather API Example

```javascript
class WeatherAPI {
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
        if (response.status === 404) {
          throw new Error('City not found');
        }
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Error fetching weather:', error);
      throw error;
    }
  }
  
  async getForecast(city, days = 5) {
    try {
      const response = await fetch(
        `${this.baseURL}/forecast?q=${city}&appid=${this.apiKey}&units=metric&cnt=${days * 8}`
      );
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.error('Error fetching forecast:', error);
      throw error;
    }
  }
}

// Sử dụng Weather API
const weather = new WeatherAPI('your-openweather-api-key');

async function displayWeather(city) {
  try {
    const currentWeather = await weather.getCurrentWeather(city);
    
    console.log(`Weather in ${currentWeather.name}:`);
    console.log(`Temperature: ${currentWeather.main.temp}°C`);
    console.log(`Description: ${currentWeather.weather[0].description}`);
    console.log(`Humidity: ${currentWeather.main.humidity}%`);
    console.log(`Wind Speed: ${currentWeather.wind.speed} m/s`);
    
  } catch (error) {
    console.error('Error:', error.message);
  }
}

// Sử dụng
displayWeather('Ho Chi Minh City');
```

## Debug Network Request với DevTools

### 1. Network Tab

```javascript
// Mở DevTools (F12) -> Network tab
// Thực hiện request và xem chi tiết

async function debugFetch() {
  console.log('Starting fetch request...');
  
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
    console.log('Response status:', response.status);
    console.log('Response headers:', [...response.headers.entries()]);
    
    const data = await response.json();
    console.log('Response data:', data);
    
  } catch (error) {
    console.error('Fetch error:', error);
  }
}

debugFetch();
```

### 2. Console Logging

```javascript
// Tạo wrapper function để log tất cả requests
function createLoggedFetch() {
  const originalFetch = window.fetch;
  
  window.fetch = async function(...args) {
    console.log('Fetch request:', args);
    
    const startTime = performance.now();
    
    try {
      const response = await originalFetch.apply(this, args);
      const endTime = performance.now();
      
      console.log('Fetch response:', {
        url: args[0],
        status: response.status,
        statusText: response.statusText,
        duration: `${(endTime - startTime).toFixed(2)}ms`
      });
      
      return response;
    } catch (error) {
      console.error('Fetch error:', error);
      throw error;
    }
  };
}

// Sử dụng
createLoggedFetch();

// Bây giờ tất cả fetch requests sẽ được log
fetch('https://api.example.com/data');
```

### 3. Request/Response Interceptors

```javascript
class FetchInterceptor {
  constructor() {
    this.requestInterceptors = [];
    this.responseInterceptors = [];
    this.setupInterceptors();
  }
  
  setupInterceptors() {
    const originalFetch = window.fetch;
    
    window.fetch = async (...args) => {
      // Apply request interceptors
      for (const interceptor of this.requestInterceptors) {
        args = await interceptor(...args);
      }
      
      const response = await originalFetch(...args);
      
      // Apply response interceptors
      let modifiedResponse = response;
      for (const interceptor of this.responseInterceptors) {
        modifiedResponse = await interceptor(modifiedResponse);
      }
      
      return modifiedResponse;
    };
  }
  
  addRequestInterceptor(interceptor) {
    this.requestInterceptors.push(interceptor);
  }
  
  addResponseInterceptor(interceptor) {
    this.responseInterceptors.push(interceptor);
  }
}

// Sử dụng interceptors
const interceptor = new FetchInterceptor();

// Request interceptor - thêm auth header
interceptor.addRequestInterceptor(async (url, options) => {
  console.log('Request interceptor:', url);
  
  const newOptions = {
    ...options,
    headers: {
      ...options?.headers,
      'Authorization': 'Bearer your-token'
    }
  };
  
  return [url, newOptions];
});

// Response interceptor - log response
interceptor.addResponseInterceptor(async (response) => {
  console.log('Response interceptor:', response.status);
  return response;
});
```

## Best Practices cho Fetch API

### 1. Error Handling

```javascript
async function robustFetch(url, options = {}) {
  try {
    const response = await fetch(url, options);
    
    // Kiểm tra response status
    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`HTTP ${response.status}: ${errorText}`);
    }
    
    // Kiểm tra content type
    const contentType = response.headers.get('content-type');
    if (!contentType || !contentType.includes('application/json')) {
      throw new Error('Response is not JSON');
    }
    
    return await response.json();
  } catch (error) {
    if (error.name === 'TypeError') {
      throw new Error('Network error - check your connection');
    }
    throw error;
  }
}
```

### 2. Request Caching

```javascript
class CachedFetch {
  constructor(cacheTime = 5 * 60 * 1000) { // 5 minutes
    this.cache = new Map();
    this.cacheTime = cacheTime;
  }
  
  async fetch(url, options = {}) {
    const cacheKey = `${url}-${JSON.stringify(options)}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTime) {
      console.log('Returning cached data');
      return cached.data;
    }
    
    const response = await fetch(url, options);
    const data = await response.json();
    
    this.cache.set(cacheKey, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
}

// Sử dụng
const cachedFetch = new CachedFetch();
cachedFetch.fetch('https://api.example.com/data');
```

### 3. Request Queue

```javascript
class RequestQueue {
  constructor(maxConcurrent = 3) {
    this.queue = [];
    this.running = 0;
    this.maxConcurrent = maxConcurrent;
  }
  
  async add(fetchFunction) {
    return new Promise((resolve, reject) => {
      this.queue.push({
        fetchFunction,
        resolve,
        reject
      });
      
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.maxConcurrent || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { fetchFunction, resolve, reject } = this.queue.shift();
    
    try {
      const result = await fetchFunction();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
}

// Sử dụng
const queue = new RequestQueue(2);

async function makeRequest(url) {
  return queue.add(() => fetch(url).then(r => r.json()));
}

// Tất cả requests sẽ được xử lý tuần tự với tối đa 2 concurrent
makeRequest('https://api.example.com/1');
makeRequest('https://api.example.com/2');
makeRequest('https://api.example.com/3');
```

## Kết luận

Fetch API là một công cụ mạnh mẽ và linh hoạt để thực hiện các yêu cầu HTTP trong JavaScript. Với cú pháp đơn giản, hỗ trợ Promise native, và khả năng xử lý nhiều loại dữ liệu khác nhau, Fetch API đã trở thành tiêu chuẩn cho việc giao tiếp với API trong các ứng dụng web hiện đại.

Những điểm quan trọng cần nhớ:
- Luôn xử lý lỗi một cách đúng đắn
- Sử dụng async/await để code dễ đọc hơn
- Thêm timeout và retry logic cho các request quan trọng
- Sử dụng DevTools để debug và monitor network requests
- Implement caching và request queuing cho hiệu suất tốt hơn

Với những kiến thức này, bạn đã có thể sử dụng Fetch API một cách hiệu quả trong các dự án thực tế!
