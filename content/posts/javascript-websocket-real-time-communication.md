---
title: "Tạo ứng dụng real-time với WebSocket"
date: 2025-10-01T14:00:00+07:00
draft: false
description: "Hướng dẫn chi tiết về WebSocket trong JavaScript, từ cơ bản đến xây dựng ứng dụng real-time hoàn chỉnh"
tags: ["javascript", "websocket", "real-time", "nodejs", "communication"]
categories: ["javascript-essentials"]
---

## Giới thiệu về WebSocket

WebSocket là một giao thức truyền thông hai chiều (bidirectional) cho phép client và server giao tiếp real-time mà không cần phải gửi request liên tục như HTTP. Khác với HTTP request-response model, WebSocket duy trì kết nối liên tục, cho phép server push dữ liệu đến client bất cứ lúc nào.

### Tại sao cần WebSocket?

**Vấn đề với HTTP truyền thống:**
- **Polling**: Client phải liên tục hỏi server "có dữ liệu mới không?"
- **Latency cao**: Delay giữa các lần polling
- **Tốn bandwidth**: Gửi nhiều request không cần thiết
- **Không real-time**: Không thể push dữ liệu ngay lập tức

**Ưu điểm của WebSocket:**
- **Real-time**: Dữ liệu được gửi ngay lập tức
- **Hiệu quả**: Chỉ một kết nối duy trì
- **Bidirectional**: Client và server đều có thể gửi dữ liệu
- **Low latency**: Độ trễ thấp hơn HTTP

### Khi nào nên sử dụng WebSocket?

- **Chat applications**: Tin nhắn real-time
- **Live notifications**: Thông báo tức thì
- **Online gaming**: Game multiplayer
- **Trading platforms**: Giá cổ phiếu real-time
- **Collaborative editing**: Google Docs, Figma
- **Live streaming**: Comments, reactions

## WebSocket cơ bản trong JavaScript

### Tạo WebSocket Connection

```javascript
// Tạo kết nối WebSocket
const socket = new WebSocket('ws://localhost:8080');

// Lắng nghe sự kiện kết nối
socket.addEventListener('open', function(event) {
    console.log('Đã kết nối đến WebSocket server');
    
    // Gửi tin nhắn chào mừng
    socket.send('Hello Server!');
});

// Lắng nghe tin nhắn từ server
socket.addEventListener('message', function(event) {
    console.log('Nhận được từ server:', event.data);
    
    // Hiển thị tin nhắn lên UI
    displayMessage(event.data);
});

// Lắng nghe lỗi
socket.addEventListener('error', function(event) {
    console.error('WebSocket error:', event);
});

// Lắng nghe khi đóng kết nối
socket.addEventListener('close', function(event) {
    console.log('WebSocket đã đóng:', event.code, event.reason);
});
```

### Gửi dữ liệu qua WebSocket

```javascript
// Gửi text message
function sendMessage(message) {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(message);
        console.log('Đã gửi:', message);
    } else {
        console.error('WebSocket chưa kết nối');
    }
}

// Gửi JSON data
function sendJSON(data) {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(JSON.stringify(data));
    }
}

// Gửi binary data
function sendBinary(buffer) {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(buffer);
    }
}
```

### Kiểm tra trạng thái kết nối

```javascript
function getConnectionStatus() {
    switch(socket.readyState) {
        case WebSocket.CONNECTING:
            return 'Đang kết nối...';
        case WebSocket.OPEN:
            return 'Đã kết nối';
        case WebSocket.CLOSING:
            return 'Đang đóng kết nối...';
        case WebSocket.CLOSED:
            return 'Đã đóng kết nối';
        default:
            return 'Trạng thái không xác định';
    }
}

// Sử dụng
console.log('Trạng thái:', getConnectionStatus());
```

## Xây dựng WebSocket Server với Node.js

### Server cơ bản

```javascript
const WebSocket = require('ws');
const http = require('http');

// Tạo HTTP server
const server = http.createServer();

// Tạo WebSocket server
const wss = new WebSocket.Server({ server });

// Lắng nghe kết nối mới
wss.on('connection', function connection(ws, request) {
    console.log('Client mới kết nối từ:', request.socket.remoteAddress);
    
    // Gửi lời chào đến client
    ws.send('Chào mừng bạn đến với WebSocket server!');
    
    // Lắng nghe tin nhắn từ client
    ws.on('message', function incoming(message) {
        console.log('Nhận được:', message.toString());
        
        // Echo lại tin nhắn
        ws.send(`Server echo: ${message}`);
        
        // Broadcast đến tất cả client khác
        wss.clients.forEach(function each(client) {
            if (client !== ws && client.readyState === WebSocket.OPEN) {
                client.send(`Broadcast: ${message}`);
            }
        });
    });
    
    // Lắng nghe khi client đóng kết nối
    ws.on('close', function close() {
        console.log('Client đã ngắt kết nối');
    });
    
    // Lắng nghe lỗi
    ws.on('error', function error(err) {
        console.error('WebSocket error:', err);
    });
});

// Khởi động server
const PORT = 8080;
server.listen(PORT, function() {
    console.log(`WebSocket server đang chạy trên port ${PORT}`);
});
```

### Server nâng cao với Room/Chat

```javascript
const WebSocket = require('ws');
const http = require('http');
const url = require('url');

const server = http.createServer();
const wss = new WebSocket.Server({ server });

// Lưu trữ các room
const rooms = new Map();

wss.on('connection', function connection(ws, request) {
    const urlParts = url.parse(request.url, true);
    const roomId = urlParts.query.room || 'default';
    const username = urlParts.query.username || 'Anonymous';
    
    console.log(`${username} tham gia room ${roomId}`);
    
    // Thêm client vào room
    if (!rooms.has(roomId)) {
        rooms.set(roomId, new Set());
    }
    rooms.get(roomId).add(ws);
    
    // Gửi thông tin room
    ws.send(JSON.stringify({
        type: 'join',
        room: roomId,
        username: username,
        message: `Chào mừng ${username} đến với room ${roomId}`
    }));
    
    // Broadcast đến các client khác trong room
    broadcastToRoom(roomId, {
        type: 'user_joined',
        username: username,
        message: `${username} đã tham gia room`
    }, ws);
    
    ws.on('message', function incoming(data) {
        try {
            const message = JSON.parse(data);
            
            switch(message.type) {
                case 'chat':
                    // Broadcast tin nhắn chat
                    broadcastToRoom(roomId, {
                        type: 'chat',
                        username: username,
                        message: message.text,
                        timestamp: new Date().toISOString()
                    });
                    break;
                    
                case 'typing':
                    // Thông báo đang gõ
                    broadcastToRoom(roomId, {
                        type: 'typing',
                        username: username
                    }, ws);
                    break;
            }
        } catch (error) {
            console.error('Lỗi parse message:', error);
        }
    });
    
    ws.on('close', function close() {
        console.log(`${username} rời khỏi room ${roomId}`);
        
        // Xóa client khỏi room
        const room = rooms.get(roomId);
        if (room) {
            room.delete(ws);
            if (room.size === 0) {
                rooms.delete(roomId);
            }
        }
        
        // Thông báo user rời room
        broadcastToRoom(roomId, {
            type: 'user_left',
            username: username,
            message: `${username} đã rời room`
        });
    });
});

function broadcastToRoom(roomId, message, excludeWs = null) {
    const room = rooms.get(roomId);
    if (room) {
        room.forEach(function each(client) {
            if (client !== excludeWs && client.readyState === WebSocket.OPEN) {
                client.send(JSON.stringify(message));
            }
        });
    }
}

const PORT = 8080;
server.listen(PORT, function() {
    console.log(`Chat server đang chạy trên port ${PORT}`);
});
```

## Xây dựng Chat Application hoàn chỉnh

### HTML Interface

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebSocket Chat</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .chat-container {
            border: 1px solid #ddd;
            border-radius: 8px;
            height: 400px;
            overflow-y: auto;
            padding: 10px;
            margin-bottom: 10px;
            background-color: #f9f9f9;
        }
        
        .message {
            margin-bottom: 10px;
            padding: 8px;
            border-radius: 4px;
        }
        
        .message.own {
            background-color: #007bff;
            color: white;
            margin-left: 20%;
        }
        
        .message.other {
            background-color: #e9ecef;
            margin-right: 20%;
        }
        
        .message.system {
            background-color: #28a745;
            color: white;
            text-align: center;
            font-style: italic;
        }
        
        .input-group {
            display: flex;
            gap: 10px;
        }
        
        .input-group input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .input-group button {
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .input-group button:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }
        
        .status {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 4px;
            text-align: center;
        }
        
        .status.connected {
            background-color: #d4edda;
            color: #155724;
        }
        
        .status.disconnected {
            background-color: #f8d7da;
            color: #721c24;
        }
    </style>
</head>
<body>
    <h1>WebSocket Chat Application</h1>
    
    <div id="status" class="status disconnected">
        Đang kết nối...
    </div>
    
    <div id="chatContainer" class="chat-container">
        <!-- Messages sẽ được thêm vào đây -->
    </div>
    
    <div class="input-group">
        <input type="text" id="messageInput" placeholder="Nhập tin nhắn..." disabled>
        <button id="sendButton" disabled>Gửi</button>
    </div>
    
    <script>
        class ChatApp {
            constructor() {
                this.socket = null;
                this.username = null;
                this.room = null;
                
                this.initializeElements();
                this.setupEventListeners();
                this.connect();
            }
            
            initializeElements() {
                this.statusEl = document.getElementById('status');
                this.chatContainer = document.getElementById('chatContainer');
                this.messageInput = document.getElementById('messageInput');
                this.sendButton = document.getElementById('sendButton');
            }
            
            setupEventListeners() {
                this.sendButton.addEventListener('click', () => this.sendMessage());
                
                this.messageInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') {
                        this.sendMessage();
                    }
                });
                
                this.messageInput.addEventListener('input', () => {
                    this.sendTyping();
                });
            }
            
            connect() {
                // Lấy username và room từ URL hoặc prompt
                this.username = prompt('Nhập tên của bạn:') || 'Anonymous';
                this.room = prompt('Nhập tên room:') || 'default';
                
                // Tạo WebSocket connection
                this.socket = new WebSocket(`ws://localhost:8080?username=${this.username}&room=${this.room}`);
                
                this.socket.addEventListener('open', () => {
                    this.updateStatus('Đã kết nối', 'connected');
                    this.messageInput.disabled = false;
                    this.sendButton.disabled = false;
                });
                
                this.socket.addEventListener('message', (event) => {
                    this.handleMessage(JSON.parse(event.data));
                });
                
                this.socket.addEventListener('close', () => {
                    this.updateStatus('Đã mất kết nối', 'disconnected');
                    this.messageInput.disabled = true;
                    this.sendButton.disabled = true;
                });
                
                this.socket.addEventListener('error', (error) => {
                    console.error('WebSocket error:', error);
                    this.updateStatus('Lỗi kết nối', 'disconnected');
                });
            }
            
            updateStatus(message, className) {
                this.statusEl.textContent = message;
                this.statusEl.className = `status ${className}`;
            }
            
            handleMessage(data) {
                switch(data.type) {
                    case 'join':
                        this.addSystemMessage(data.message);
                        break;
                        
                    case 'chat':
                        this.addChatMessage(data.username, data.message, data.timestamp);
                        break;
                        
                    case 'user_joined':
                    case 'user_left':
                        this.addSystemMessage(data.message);
                        break;
                        
                    case 'typing':
                        this.showTyping(data.username);
                        break;
                }
            }
            
            addSystemMessage(message) {
                const messageEl = document.createElement('div');
                messageEl.className = 'message system';
                messageEl.textContent = message;
                this.chatContainer.appendChild(messageEl);
                this.scrollToBottom();
            }
            
            addChatMessage(username, message, timestamp) {
                const messageEl = document.createElement('div');
                const isOwn = username === this.username;
                messageEl.className = `message ${isOwn ? 'own' : 'other'}`;
                
                const time = new Date(timestamp).toLocaleTimeString();
                messageEl.innerHTML = `
                    <strong>${username}</strong> <small>${time}</small><br>
                    ${message}
                `;
                
                this.chatContainer.appendChild(messageEl);
                this.scrollToBottom();
            }
            
            showTyping(username) {
                // Hiển thị "đang gõ..." (có thể implement typing indicator)
                console.log(`${username} đang gõ...`);
            }
            
            sendMessage() {
                const message = this.messageInput.value.trim();
                if (message && this.socket.readyState === WebSocket.OPEN) {
                    this.socket.send(JSON.stringify({
                        type: 'chat',
                        text: message
                    }));
                    this.messageInput.value = '';
                }
            }
            
            sendTyping() {
                if (this.socket.readyState === WebSocket.OPEN) {
                    this.socket.send(JSON.stringify({
                        type: 'typing'
                    }));
                }
            }
            
            scrollToBottom() {
                this.chatContainer.scrollTop = this.chatContainer.scrollHeight;
            }
        }
        
        // Khởi tạo chat app
        new ChatApp();
    </script>
</body>
</html>
```

## So sánh WebSocket với HTTP Long Polling

### HTTP Long Polling

```javascript
// Client-side long polling
class LongPollingClient {
    constructor(url) {
        this.url = url;
        this.polling = false;
    }
    
    startPolling() {
        this.polling = true;
        this.poll();
    }
    
    async poll() {
        if (!this.polling) return;
        
        try {
            const response = await fetch(this.url, {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            });
            
            if (response.ok) {
                const data = await response.json();
                this.handleMessage(data);
            }
        } catch (error) {
            console.error('Polling error:', error);
        }
        
        // Tiếp tục polling sau 1 giây
        setTimeout(() => this.poll(), 1000);
    }
    
    stopPolling() {
        this.polling = false;
    }
    
    handleMessage(data) {
        console.log('Nhận được:', data);
    }
}

// Server-side long polling
app.get('/poll', (req, res) => {
    // Giữ connection mở trong 30 giây
    res.setTimeout(30000);
    
    // Kiểm tra có dữ liệu mới không
    const checkForNewData = () => {
        const newData = getNewData();
        if (newData) {
            res.json(newData);
        } else {
            // Tiếp tục chờ
            setTimeout(checkForNewData, 1000);
        }
    };
    
    checkForNewData();
});
```

### Bảng so sánh

| Đặc điểm | WebSocket | HTTP Long Polling |
|----------|-----------|-------------------|
| **Latency** | Thấp (real-time) | Cao (1-30 giây) |
| **Bandwidth** | Thấp | Cao (nhiều request) |
| **Server Resources** | Thấp | Cao (nhiều connection) |
| **Complexity** | Trung bình | Thấp |
| **Browser Support** | Modern browsers | Tất cả browsers |
| **Firewall** | Có thể bị chặn | Không bị chặn |
| **Scalability** | Tốt | Kém |

## Xử lý lỗi và Reconnection

### Auto-reconnect WebSocket

```javascript
class ReconnectingWebSocket {
    constructor(url, options = {}) {
        this.url = url;
        this.options = {
            maxReconnectAttempts: 5,
            reconnectInterval: 1000,
            maxReconnectInterval: 30000,
            ...options
        };
        
        this.reconnectAttempts = 0;
        this.reconnectTimeout = null;
        this.isManualClose = false;
        
        this.connect();
    }
    
    connect() {
        try {
            this.socket = new WebSocket(this.url);
            this.setupEventListeners();
        } catch (error) {
            console.error('WebSocket connection error:', error);
            this.scheduleReconnect();
        }
    }
    
    setupEventListeners() {
        this.socket.addEventListener('open', () => {
            console.log('WebSocket connected');
            this.reconnectAttempts = 0;
            this.onopen && this.onopen();
        });
        
        this.socket.addEventListener('message', (event) => {
            this.onmessage && this.onmessage(event);
        });
        
        this.socket.addEventListener('close', (event) => {
            console.log('WebSocket closed:', event.code, event.reason);
            this.onclose && this.onclose(event);
            
            if (!this.isManualClose) {
                this.scheduleReconnect();
            }
        });
        
        this.socket.addEventListener('error', (error) => {
            console.error('WebSocket error:', error);
            this.onerror && this.onerror(error);
        });
    }
    
    scheduleReconnect() {
        if (this.reconnectAttempts >= this.options.maxReconnectAttempts) {
            console.error('Max reconnection attempts reached');
            return;
        }
        
        const delay = Math.min(
            this.options.reconnectInterval * Math.pow(2, this.reconnectAttempts),
            this.options.maxReconnectInterval
        );
        
        console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts + 1})`);
        
        this.reconnectTimeout = setTimeout(() => {
            this.reconnectAttempts++;
            this.connect();
        }, delay);
    }
    
    send(data) {
        if (this.socket && this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(data);
        } else {
            console.warn('WebSocket not connected, message queued');
            // Có thể implement message queue ở đây
        }
    }
    
    close() {
        this.isManualClose = true;
        if (this.reconnectTimeout) {
            clearTimeout(this.reconnectTimeout);
        }
        if (this.socket) {
            this.socket.close();
        }
    }
}

// Sử dụng
const ws = new ReconnectingWebSocket('ws://localhost:8080');
ws.onopen = () => console.log('Connected!');
ws.onmessage = (event) => console.log('Message:', event.data);
ws.onclose = () => console.log('Disconnected');
ws.onerror = (error) => console.error('Error:', error);
```

## Best Practices cho WebSocket

### 1. Message Format

```javascript
// Sử dụng JSON format cho messages
const messageTypes = {
    JOIN: 'join',
    LEAVE: 'leave',
    MESSAGE: 'message',
    TYPING: 'typing',
    PING: 'ping',
    PONG: 'pong'
};

function createMessage(type, data) {
    return JSON.stringify({
        type: type,
        data: data,
        timestamp: Date.now(),
        id: generateId()
    });
}

function parseMessage(message) {
    try {
        return JSON.parse(message);
    } catch (error) {
        console.error('Invalid message format:', error);
        return null;
    }
}
```

### 2. Heartbeat/Ping-Pong

```javascript
class WebSocketWithHeartbeat {
    constructor(url) {
        this.url = url;
        this.pingInterval = 30000; // 30 giây
        this.pongTimeout = 5000;   // 5 giây
        this.heartbeatTimer = null;
        this.pongTimer = null;
        
        this.connect();
    }
    
    connect() {
        this.socket = new WebSocket(this.url);
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        this.socket.addEventListener('open', () => {
            this.startHeartbeat();
        });
        
        this.socket.addEventListener('message', (event) => {
            const message = JSON.parse(event.data);
            
            if (message.type === 'pong') {
                this.handlePong();
            } else {
                this.onmessage && this.onmessage(event);
            }
        });
        
        this.socket.addEventListener('close', () => {
            this.stopHeartbeat();
        });
    }
    
    startHeartbeat() {
        this.heartbeatTimer = setInterval(() => {
            this.sendPing();
        }, this.pingInterval);
    }
    
    stopHeartbeat() {
        if (this.heartbeatTimer) {
            clearInterval(this.heartbeatTimer);
            this.heartbeatTimer = null;
        }
        if (this.pongTimer) {
            clearTimeout(this.pongTimer);
            this.pongTimer = null;
        }
    }
    
    sendPing() {
        if (this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify({ type: 'ping' }));
            
            // Đặt timeout cho pong
            this.pongTimer = setTimeout(() => {
                console.warn('Pong timeout, reconnecting...');
                this.socket.close();
            }, this.pongTimeout);
        }
    }
    
    handlePong() {
        if (this.pongTimer) {
            clearTimeout(this.pongTimer);
            this.pongTimer = null;
        }
    }
}
```

### 3. Error Handling

```javascript
class RobustWebSocket {
    constructor(url) {
        this.url = url;
        this.maxRetries = 5;
        this.retryCount = 0;
        this.retryDelay = 1000;
        
        this.connect();
    }
    
    connect() {
        try {
            this.socket = new WebSocket(this.url);
            this.setupEventListeners();
        } catch (error) {
            this.handleError(error);
        }
    }
    
    setupEventListeners() {
        this.socket.addEventListener('open', () => {
            this.retryCount = 0;
            this.onopen && this.onopen();
        });
        
        this.socket.addEventListener('error', (error) => {
            this.handleError(error);
        });
        
        this.socket.addEventListener('close', (event) => {
            this.onclose && this.onclose(event);
            
            // Tự động reconnect nếu không phải manual close
            if (event.code !== 1000) {
                this.scheduleReconnect();
            }
        });
    }
    
    handleError(error) {
        console.error('WebSocket error:', error);
        this.onerror && this.onerror(error);
        
        if (this.retryCount < this.maxRetries) {
            this.scheduleReconnect();
        } else {
            console.error('Max retries reached, giving up');
        }
    }
    
    scheduleReconnect() {
        const delay = this.retryDelay * Math.pow(2, this.retryCount);
        this.retryCount++;
        
        setTimeout(() => {
            console.log(`Reconnecting... (attempt ${this.retryCount})`);
            this.connect();
        }, delay);
    }
}
```

## Kết luận

WebSocket là một công nghệ mạnh mẽ cho việc xây dựng các ứng dụng real-time. Với khả năng giao tiếp hai chiều, độ trễ thấp và hiệu suất cao, WebSocket đã trở thành lựa chọn hàng đầu cho:

- **Chat applications**
- **Live notifications** 
- **Online gaming**
- **Real-time collaboration**
- **Live streaming**

Khi sử dụng WebSocket, hãy nhớ:
- Implement reconnection logic
- Sử dụng heartbeat để duy trì kết nối
- Xử lý lỗi một cách đúng đắn
- Chọn message format phù hợp
- Test kỹ các trường hợp edge cases

Với những kiến thức này, bạn đã có thể xây dựng các ứng dụng real-time mạnh mẽ và ổn định!
