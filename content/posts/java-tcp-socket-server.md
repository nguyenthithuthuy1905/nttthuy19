---
title: "Java Networking – Viết TCP Socket Server tối giản"
date: 2025-10-13T20:10:00+07:00
tags: ["java", "networking", "tcp", "socket", "server", "multithreading"]
categories: ["Java"]
description: "Hướng dẫn chi tiết xây dựng TCP Socket Server: từ cơ bản đến nâng cao, xử lý đa luồng, error handling và best practices"
draft: false
featured_image: "/images/java.svg"
---

# Java Networking – Viết TCP Socket Server tối giản

## 🎯 **Mục tiêu học tập**
- Hiểu cách xây dựng TCP server bằng ServerSocket
- Nắm vững xử lý đa luồng với ExecutorService
- Thực hành error handling và resource management
- Áp dụng best practices cho production server

## 🏗️ **1. Tổng quan về TCP Server**

### **TCP Server hoạt động như thế nào?**
1. **Bind**: Server lắng nghe trên một port cụ thể
2. **Accept**: Chấp nhận kết nối từ client
3. **Handle**: Xử lý request từ client
4. **Response**: Gửi response về client
5. **Close**: Đóng kết nối (hoặc giữ để tái sử dụng)

### **Tại sao cần TCP Server?**
- **Real-time communication**: Chat, gaming, live streaming
- **File transfer**: Upload/download files
- **API services**: Custom protocols, legacy systems
- **Microservices**: Internal service communication

## 🚀 **2. TCP Server cơ bản**

### **Echo Server đơn giản**
```java
import java.io.*;
import java.net.*;

public class SimpleEchoServer {
    public static void main(String[] args) {
        int port = 9000;
        
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Server listening on port " + port);
            
            while (true) {
                // Chờ client kết nối
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client connected: " + clientSocket.getInetAddress());
                
                // Xử lý client (blocking - chỉ 1 client tại 1 thời điểm)
                handleClient(clientSocket);
            }
        } catch (IOException e) {
            System.err.println("Server error: " + e.getMessage());
        }
    }
    
    private static void handleClient(Socket clientSocket) {
        try (clientSocket;
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(
                 clientSocket.getOutputStream(), true)) {
            
            // Gửi welcome message
            out.println("Welcome to Echo Server! Type 'quit' to exit.");
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                if ("quit".equalsIgnoreCase(inputLine)) {
                    out.println("Goodbye!");
                    break;
                }
                // Echo lại message
                out.println("Echo: " + inputLine);
            }
        } catch (IOException e) {
            System.err.println("Error handling client: " + e.getMessage());
        }
    }
}
```

**Vấn đề với server cơ bản:**
- ❌ Chỉ xử lý được 1 client tại 1 thời điểm
- ❌ Client khác phải chờ client hiện tại disconnect
- ❌ Không scalable

## 🧵 **3. Multi-threaded TCP Server**

### **Sử dụng ExecutorService**
```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;

public class MultiThreadedEchoServer {
    private static final int MAX_THREADS = 10;
    private static final int PORT = 9000;
    
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(MAX_THREADS);
        
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Multi-threaded server listening on port " + PORT);
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("New client connected: " + clientSocket.getInetAddress());
                
                // Submit task vào thread pool
                threadPool.submit(new ClientHandler(clientSocket));
            }
        } catch (IOException e) {
            System.err.println("Server error: " + e.getMessage());
        } finally {
            threadPool.shutdown();
        }
    }
}

class ClientHandler implements Runnable {
    private final Socket clientSocket;
    
    public ClientHandler(Socket socket) {
        this.clientSocket = socket;
    }
    
    @Override
    public void run() {
        try (clientSocket;
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(
                 clientSocket.getOutputStream(), true)) {
            
            out.println("Welcome! You are client #" + Thread.currentThread().getId());
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                if ("quit".equalsIgnoreCase(inputLine)) {
                    out.println("Goodbye!");
                    break;
                }
                
                // Xử lý command
                String response = processCommand(inputLine);
                out.println(response);
            }
        } catch (IOException e) {
            System.err.println("Error handling client: " + e.getMessage());
        }
    }
    
    private String processCommand(String command) {
        // Xử lý các command khác nhau
        switch (command.toLowerCase()) {
            case "time":
                return "Current time: " + java.time.LocalDateTime.now();
            case "info":
                return "Server: Java TCP Echo Server v1.0";
            case "help":
                return "Available commands: time, info, help, quit";
            default:
                return "Echo: " + command;
        }
    }
}
```

## ⚙️ **4. Server với cấu hình nâng cao**

### **Configurable Server**
```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class AdvancedEchoServer {
    private static final int DEFAULT_PORT = 9000;
    private static final int DEFAULT_MAX_THREADS = 20;
    private static final int DEFAULT_TIMEOUT = 30000; // 30 seconds
    private static final int DEFAULT_MAX_MESSAGE_SIZE = 1024;
    
    private final int port;
    private final int maxThreads;
    private final int timeout;
    private final int maxMessageSize;
    private final AtomicInteger clientCount = new AtomicInteger(0);
    
    public AdvancedEchoServer(int port, int maxThreads, int timeout, int maxMessageSize) {
        this.port = port;
        this.maxThreads = maxThreads;
        this.timeout = timeout;
        this.maxMessageSize = maxMessageSize;
    }
    
    public void start() {
        ExecutorService threadPool = Executors.newFixedThreadPool(maxThreads);
        
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Advanced Echo Server started:");
            System.out.println("- Port: " + port);
            System.out.println("- Max threads: " + maxThreads);
            System.out.println("- Timeout: " + timeout + "ms");
            System.out.println("- Max message size: " + maxMessageSize + " bytes");
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                clientSocket.setSoTimeout(timeout);
                
                int clientId = clientCount.incrementAndGet();
                System.out.println("Client #" + clientId + " connected: " + 
                    clientSocket.getInetAddress());
                
                threadPool.submit(new AdvancedClientHandler(
                    clientSocket, clientId, maxMessageSize));
            }
        } catch (IOException e) {
            System.err.println("Server error: " + e.getMessage());
        } finally {
            threadPool.shutdown();
        }
    }
    
    public static void main(String[] args) {
        int port = args.length > 0 ? Integer.parseInt(args[0]) : DEFAULT_PORT;
        int maxThreads = args.length > 1 ? Integer.parseInt(args[1]) : DEFAULT_MAX_THREADS;
        int timeout = args.length > 2 ? Integer.parseInt(args[2]) : DEFAULT_TIMEOUT;
        int maxMessageSize = args.length > 3 ? Integer.parseInt(args[3]) : DEFAULT_MAX_MESSAGE_SIZE;
        
        AdvancedEchoServer server = new AdvancedEchoServer(port, maxThreads, timeout, maxMessageSize);
        server.start();
    }
}

class AdvancedClientHandler implements Runnable {
    private final Socket clientSocket;
    private final int clientId;
    private final int maxMessageSize;
    
    public AdvancedClientHandler(Socket socket, int clientId, int maxMessageSize) {
        this.clientSocket = socket;
        this.clientId = clientId;
        this.maxMessageSize = maxMessageSize;
    }
    
    @Override
    public void run() {
        try (clientSocket;
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(
                 clientSocket.getOutputStream(), true)) {
            
            out.println("Welcome! You are client #" + clientId);
            out.println("Type 'help' for available commands");
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                // Validate message size
                if (inputLine.length() > maxMessageSize) {
                    out.println("ERROR: Message too long (max " + maxMessageSize + " chars)");
                    continue;
                }
                
                // Sanitize input
                String sanitizedInput = sanitizeInput(inputLine);
                
                if ("quit".equalsIgnoreCase(sanitizedInput)) {
                    out.println("Goodbye, client #" + clientId + "!");
                    break;
                }
                
                String response = processCommand(sanitizedInput);
                out.println(response);
            }
        } catch (java.net.SocketTimeoutException e) {
            System.out.println("Client #" + clientId + " timed out");
        } catch (IOException e) {
            System.err.println("Error handling client #" + clientId + ": " + e.getMessage());
        }
    }
    
    private String sanitizeInput(String input) {
        // Loại bỏ các ký tự nguy hiểm
        return input.replaceAll("[<>\"'&]", "");
    }
    
    private String processCommand(String command) {
        String[] parts = command.split(" ", 2);
        String cmd = parts[0].toLowerCase();
        String args = parts.length > 1 ? parts[1] : "";
        
        switch (cmd) {
            case "time":
                return "Current time: " + java.time.LocalDateTime.now();
            case "info":
                return "Server: Advanced Echo Server v2.0 | Client ID: " + clientId;
            case "echo":
                return "Echo: " + args;
            case "reverse":
                return "Reversed: " + new StringBuilder(args).reverse().toString();
            case "uppercase":
                return "Uppercase: " + args.toUpperCase();
            case "help":
                return "Commands: time, info, echo <text>, reverse <text>, uppercase <text>, help, quit";
            default:
                return "Unknown command: " + cmd + ". Type 'help' for available commands.";
        }
    }
}
```

## 🔒 **5. Error Handling và Security**

### **Robust Error Handling**
```java
public class RobustEchoServer {
    private static final int MAX_RETRIES = 3;
    private static final long RETRY_DELAY = 1000; // 1 second
    
    public static void main(String[] args) {
        int port = 9000;
        int retryCount = 0;
        
        while (retryCount < MAX_RETRIES) {
            try (ServerSocket serverSocket = new ServerSocket(port)) {
                System.out.println("Server started successfully on port " + port);
                runServer(serverSocket);
                break; // Thoát khỏi retry loop nếu thành công
            } catch (BindException e) {
                System.err.println("Port " + port + " is already in use");
                port++; // Thử port khác
                retryCount++;
            } catch (IOException e) {
                System.err.println("Server error: " + e.getMessage());
                retryCount++;
                
                if (retryCount < MAX_RETRIES) {
                    try {
                        Thread.sleep(RETRY_DELAY);
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
        
        if (retryCount >= MAX_RETRIES) {
            System.err.println("Failed to start server after " + MAX_RETRIES + " attempts");
        }
    }
    
    private static void runServer(ServerSocket serverSocket) throws IOException {
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        
        try {
            while (true) {
                Socket clientSocket = serverSocket.accept();
                threadPool.submit(new SafeClientHandler(clientSocket));
            }
        } finally {
            threadPool.shutdown();
        }
    }
}

class SafeClientHandler implements Runnable {
    private final Socket clientSocket;
    
    public SafeClientHandler(Socket socket) {
        this.clientSocket = socket;
    }
    
    @Override
    public void run() {
        try (clientSocket;
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(
                 clientSocket.getOutputStream(), true)) {
            
            // Set timeouts
            clientSocket.setSoTimeout(30000); // 30 seconds
            
            out.println("Welcome to Safe Echo Server!");
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                try {
                    String response = processSafely(inputLine);
                    out.println(response);
                } catch (Exception e) {
                    out.println("ERROR: " + e.getMessage());
                }
            }
        } catch (java.net.SocketTimeoutException e) {
            System.out.println("Client timeout: " + clientSocket.getInetAddress());
        } catch (IOException e) {
            System.err.println("IO Error: " + e.getMessage());
        } catch (Exception e) {
            System.err.println("Unexpected error: " + e.getMessage());
        }
    }
    
    private String processSafely(String input) {
        // Validate input length
        if (input.length() > 1000) {
            throw new IllegalArgumentException("Input too long");
        }
        
        // Check for dangerous patterns
        if (input.contains("DROP") || input.contains("DELETE")) {
            throw new SecurityException("Potentially dangerous input detected");
        }
        
        return "Echo: " + input;
    }
}
```

## 📊 **6. Monitoring và Logging**

### **Server với Logging**
```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class LoggedEchoServer {
    private static final DateTimeFormatter TIMESTAMP_FORMAT = 
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    
    public static void main(String[] args) {
        int port = 9000;
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            log("Server started on port " + port);
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                log("Client connected: " + clientSocket.getInetAddress());
                
                threadPool.submit(new LoggedClientHandler(clientSocket));
            }
        } catch (IOException e) {
            log("Server error: " + e.getMessage());
        } finally {
            threadPool.shutdown();
        }
    }
    
    private static void log(String message) {
        String timestamp = LocalDateTime.now().format(TIMESTAMP_FORMAT);
        System.out.println("[" + timestamp + "] " + message);
    }
}

class LoggedClientHandler implements Runnable {
    private final Socket clientSocket;
    
    public LoggedClientHandler(Socket socket) {
        this.clientSocket = socket;
    }
    
    @Override
    public void run() {
        String clientInfo = clientSocket.getInetAddress().toString();
        
        try (clientSocket;
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(
                 clientSocket.getOutputStream(), true)) {
            
            out.println("Welcome to Logged Echo Server!");
            
            String inputLine;
            int messageCount = 0;
            
            while ((inputLine = in.readLine()) != null) {
                messageCount++;
                log("Client " + clientInfo + " sent: " + inputLine);
                
                String response = "Echo #" + messageCount + ": " + inputLine;
                out.println(response);
                
                log("Server responded: " + response);
            }
        } catch (IOException e) {
            log("Error with client " + clientInfo + ": " + e.getMessage());
        }
    }
    
    private static void log(String message) {
        String timestamp = LocalDateTime.now().format(
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        System.out.println("[" + timestamp + "] " + message);
    }
}
```

## 📋 **7. Best Practices**

### **Do's (Nên làm)**
- ✅ Sử dụng try-with-resources để tự động đóng connections
- ✅ Set timeout cho socket để tránh hang
- ✅ Validate và sanitize input từ client
- ✅ Sử dụng thread pool thay vì tạo thread mới cho mỗi client
- ✅ Log các events quan trọng để debug
- ✅ Handle exceptions gracefully
- ✅ Giới hạn kích thước message và số lượng connections

### **Don'ts (Không nên)**
- ❌ Tạo thread mới cho mỗi client (không scalable)
- ❌ Ignore exceptions từ network operations
- ❌ Trust input từ client (luôn validate)
- ❌ Hardcode port numbers và timeouts
- ❌ Block main thread với blocking operations
- ❌ Không giới hạn tài nguyên (memory, connections)

## 🎯 **Kết luận**

Xây dựng TCP server trong Java cần chú ý:

1. **Scalability**: Sử dụng thread pool thay vì tạo thread mới
2. **Reliability**: Error handling và timeout management
3. **Security**: Input validation và sanitization
4. **Monitoring**: Logging và metrics để debug
5. **Resource Management**: Proper cleanup và resource limits

Với những kiến thức này, bạn có thể xây dựng TCP server robust và scalable cho các ứng dụng thực tế!

**Bước tiếp theo:**
- NIO (Non-blocking I/O) cho high-performance servers
- WebSocket implementation
- Load balancing và clustering
- Security enhancements (SSL/TLS)


