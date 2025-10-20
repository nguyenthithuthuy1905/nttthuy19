---
title: "Xử lý đa luồng trong ứng dụng mạng Java"
date: 2024-01-15T11:00:00+07:00
draft: false
description: "Hướng dẫn chi tiết cách sử dụng đa luồng để xử lý nhiều kết nối đồng thời trong ứng dụng mạng Java"
tags: ["java", "multithreading", "networking", "concurrency", "server"]
categories: ["java"]
---

## Giới thiệu về đa luồng trong mạng

Khi xây dựng các ứng dụng mạng, việc xử lý nhiều kết nối đồng thời là một thách thức lớn. Nếu server chỉ xử lý một kết nối tại một thời điểm, các client khác sẽ phải chờ đợi, dẫn đến trải nghiệm người dùng kém. Đa luồng (multithreading) là giải pháp hiệu quả để giải quyết vấn đề này.

### Tại sao cần đa luồng trong mạng?

1. **Xử lý đồng thời**: Cho phép server xử lý nhiều client cùng lúc
2. **Tăng hiệu suất**: Tận dụng tối đa tài nguyên CPU
3. **Cải thiện trải nghiệm**: Client không phải chờ đợi
4. **Khả năng mở rộng**: Hỗ trợ nhiều người dùng đồng thời

### Các mô hình đa luồng phổ biến:

- **Thread-per-connection**: Mỗi kết nối một thread
- **Thread Pool**: Sử dụng pool thread có sẵn
- **NIO (Non-blocking I/O)**: Sử dụng Selector và Channel
- **Reactive Programming**: Sử dụng CompletableFuture, RxJava

## Mô hình Thread-per-Connection

Đây là mô hình đơn giản nhất, mỗi kết nối client sẽ được xử lý bởi một thread riêng biệt.

### TCP Server với đa luồng

```java
import java.io.*;
import java.net.*;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MultiThreadedTCPServer {
    private static final int PORT = 8080;
    private static final int MAX_CLIENTS = 10;
    private ExecutorService threadPool;
    
    public MultiThreadedTCPServer() {
        // Tạo thread pool với số lượng thread cố định
        this.threadPool = Executors.newFixedThreadPool(MAX_CLIENTS);
    }
    
    public void start() {
        try {
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("Multi-threaded TCP Server đang chạy trên port " + PORT);
            System.out.println("Tối đa " + MAX_CLIENTS + " client đồng thời");
            
            while (true) {
                // Chờ kết nối từ client
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client mới kết nối: " + clientSocket.getInetAddress());
                
                // Tạo task mới cho client này
                ClientHandler clientHandler = new ClientHandler(clientSocket);
                
                // Gửi task vào thread pool để xử lý
                threadPool.submit(clientHandler);
            }
            
        } catch (IOException e) {
            System.err.println("Lỗi server: " + e.getMessage());
        }
    }
    
    // Inner class để xử lý từng client
    private static class ClientHandler implements Runnable {
        private Socket clientSocket;
        
        public ClientHandler(Socket socket) {
            this.clientSocket = socket;
        }
        
        @Override
        public void run() {
            try {
                // Lấy thông tin client
                String clientInfo = clientSocket.getInetAddress() + ":" + clientSocket.getPort();
                System.out.println("Thread " + Thread.currentThread().getName() + 
                                 " đang xử lý client: " + clientInfo);
                
                // Tạo stream để giao tiếp
                BufferedReader in = new BufferedReader(
                    new InputStreamReader(clientSocket.getInputStream())
                );
                PrintWriter out = new PrintWriter(
                    clientSocket.getOutputStream(), true
                );
                
                // Gửi lời chào
                out.println("Chào mừng bạn đến với server! Gõ 'exit' để thoát.");
                
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    System.out.println("Client " + clientInfo + " gửi: " + inputLine);
                    
                    if ("exit".equalsIgnoreCase(inputLine)) {
                        out.println("Tạm biệt!");
                        break;
                    }
                    
                    // Xử lý dữ liệu (ví dụ: chuyển thành chữ hoa)
                    String response = "Server phản hồi: " + inputLine.toUpperCase();
                    out.println(response);
                }
                
            } catch (IOException e) {
                System.err.println("Lỗi xử lý client: " + e.getMessage());
            } finally {
                try {
                    clientSocket.close();
                    System.out.println("Đã đóng kết nối với client");
                } catch (IOException e) {
                    System.err.println("Lỗi đóng socket: " + e.getMessage());
                }
            }
        }
    }
    
    public static void main(String[] args) {
        new MultiThreadedTCPServer().start();
    }
}
```

### Giải thích code:

1. **ExecutorService**: Quản lý pool các thread
2. **ClientHandler**: Class xử lý logic cho từng client
3. **Runnable**: Interface cho thread, method `run()` chứa logic xử lý
4. **submit()**: Gửi task vào thread pool để xử lý

## Mô hình Thread Pool nâng cao

Sử dụng `ThreadPoolExecutor` để kiểm soát chi tiết hơn về thread pool.

```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class AdvancedThreadPoolServer {
    private static final int PORT = 8081;
    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 20;
    private static final int KEEP_ALIVE_TIME = 60;
    private static final int QUEUE_CAPACITY = 100;
    
    private ThreadPoolExecutor executor;
    private AtomicInteger clientCounter = new AtomicInteger(0);
    
    public AdvancedThreadPoolServer() {
        // Tạo thread pool với cấu hình chi tiết
        this.executor = new ThreadPoolExecutor(
            CORE_POOL_SIZE,           // Số thread cố định
            MAX_POOL_SIZE,            // Số thread tối đa
            KEEP_ALIVE_TIME,          // Thời gian chờ thread không hoạt động
            TimeUnit.SECONDS,         // Đơn vị thời gian
            new LinkedBlockingQueue<>(QUEUE_CAPACITY), // Queue chứa task
            new ThreadFactory() {     // Factory tạo thread
                private AtomicInteger threadNumber = new AtomicInteger(1);
                @Override
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(r, "ServerThread-" + threadNumber.getAndIncrement());
                    t.setDaemon(false);
                    return t;
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy() // Rejection policy
        );
    }
    
    public void start() {
        try {
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("Advanced Thread Pool Server đang chạy trên port " + PORT);
            System.out.println("Core threads: " + CORE_POOL_SIZE + 
                             ", Max threads: " + MAX_POOL_SIZE);
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                int clientId = clientCounter.incrementAndGet();
                
                System.out.println("Client #" + clientId + " kết nối từ: " + 
                                 clientSocket.getInetAddress());
                
                // Tạo task với client ID
                AdvancedClientHandler handler = new AdvancedClientHandler(clientSocket, clientId);
                
                try {
                    executor.submit(handler);
                } catch (RejectedExecutionException e) {
                    System.err.println("Server quá tải! Từ chối client #" + clientId);
                    clientSocket.close();
                }
            }
            
        } catch (IOException e) {
            System.err.println("Lỗi server: " + e.getMessage());
        }
    }
    
    private class AdvancedClientHandler implements Runnable {
        private Socket clientSocket;
        private int clientId;
        
        public AdvancedClientHandler(Socket socket, int id) {
            this.clientSocket = socket;
            this.clientId = id;
        }
        
        @Override
        public void run() {
            try {
                String threadName = Thread.currentThread().getName();
                System.out.println("Thread " + threadName + " xử lý client #" + clientId);
                
                BufferedReader in = new BufferedReader(
                    new InputStreamReader(clientSocket.getInputStream())
                );
                PrintWriter out = new PrintWriter(
                    clientSocket.getOutputStream(), true
                );
                
                out.println("Chào mừng client #" + clientId + "!");
                out.println("Thread pool status: " + 
                           "Active: " + executor.getActiveCount() + 
                           ", Pool size: " + executor.getPoolSize());
                
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    if ("exit".equalsIgnoreCase(inputLine)) {
                        out.println("Tạm biệt client #" + clientId + "!");
                        break;
                    }
                    
                    if ("status".equalsIgnoreCase(inputLine)) {
                        out.println("Thread pool status:");
                        out.println("- Active threads: " + executor.getActiveCount());
                        out.println("- Pool size: " + executor.getPoolSize());
                        out.println("- Completed tasks: " + executor.getCompletedTaskCount());
                        out.println("- Queue size: " + executor.getQueue().size());
                    } else {
                        // Mô phỏng xử lý phức tạp
                        Thread.sleep(1000); // Giả lập xử lý 1 giây
                        String response = "Client #" + clientId + " - " + inputLine.toUpperCase();
                        out.println(response);
                    }
                }
                
            } catch (IOException | InterruptedException e) {
                System.err.println("Lỗi xử lý client #" + clientId + ": " + e.getMessage());
            } finally {
                try {
                    clientSocket.close();
                    System.out.println("Đã đóng kết nối client #" + clientId);
                } catch (IOException e) {
                    System.err.println("Lỗi đóng socket client #" + clientId);
                }
            }
        }
    }
    
    public void shutdown() {
        System.out.println("Đang tắt server...");
        executor.shutdown();
        try {
            if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
        }
    }
    
    public static void main(String[] args) {
        AdvancedThreadPoolServer server = new AdvancedThreadPoolServer();
        
        // Thêm shutdown hook để tắt server gracefully
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Nhận tín hiệu tắt server...");
            server.shutdown();
        }));
        
        server.start();
    }
}
```

## Xử lý đồng bộ và Thread Safety

Khi sử dụng đa luồng, cần đảm bảo thread safety để tránh race condition và data corruption.

### Ví dụ về Thread Safety

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.ReentrantLock;

public class ThreadSafeServer {
    private static final int PORT = 8082;
    private ExecutorService executor;
    
    // Sử dụng AtomicInteger để thread-safe counter
    private AtomicInteger totalConnections = new AtomicInteger(0);
    private AtomicInteger activeConnections = new AtomicInteger(0);
    
    // Sử dụng ConcurrentHashMap để thread-safe map
    private ConcurrentHashMap<String, Integer> clientStats = new ConcurrentHashMap<>();
    
    // Sử dụng ReentrantLock cho critical section
    private ReentrantLock statsLock = new ReentrantLock();
    
    public ThreadSafeServer() {
        this.executor = Executors.newCachedThreadPool();
    }
    
    public void start() {
        try {
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("Thread-Safe Server đang chạy trên port " + PORT);
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                
                // Tăng counter một cách thread-safe
                int connectionId = totalConnections.incrementAndGet();
                activeConnections.incrementAndGet();
                
                System.out.println("Kết nối #" + connectionId + " từ " + 
                                 clientSocket.getInetAddress());
                
                ThreadSafeClientHandler handler = 
                    new ThreadSafeClientHandler(clientSocket, connectionId);
                executor.submit(handler);
            }
            
        } catch (IOException e) {
            System.err.println("Lỗi server: " + e.getMessage());
        }
    }
    
    private class ThreadSafeClientHandler implements Runnable {
        private Socket clientSocket;
        private int connectionId;
        
        public ThreadSafeClientHandler(Socket socket, int id) {
            this.clientSocket = socket;
            this.connectionId = id;
        }
        
        @Override
        public void run() {
            try {
                String clientAddress = clientSocket.getInetAddress().getHostAddress();
                
                // Cập nhật thống kê client một cách thread-safe
                clientStats.merge(clientAddress, 1, Integer::sum);
                
                BufferedReader in = new BufferedReader(
                    new InputStreamReader(clientSocket.getInputStream())
                );
                PrintWriter out = new PrintWriter(
                    clientSocket.getOutputStream(), true
                );
                
                out.println("Chào mừng! Bạn là kết nối #" + connectionId);
                out.println("Tổng kết nối: " + totalConnections.get());
                out.println("Kết nối đang hoạt động: " + activeConnections.get());
                
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    if ("exit".equalsIgnoreCase(inputLine)) {
                        break;
                    }
                    
                    if ("stats".equalsIgnoreCase(inputLine)) {
                        // Sử dụng lock để đảm bảo thread safety
                        statsLock.lock();
                        try {
                            out.println("=== THỐNG KÊ SERVER ===");
                            out.println("Tổng kết nối: " + totalConnections.get());
                            out.println("Kết nối đang hoạt động: " + activeConnections.get());
                            out.println("Thống kê theo IP:");
                            clientStats.forEach((ip, count) -> 
                                out.println("  " + ip + ": " + count + " kết nối"));
                        } finally {
                            statsLock.unlock();
                        }
                    } else {
                        out.println("Echo: " + inputLine);
                    }
                }
                
            } catch (IOException e) {
                System.err.println("Lỗi xử lý client: " + e.getMessage());
            } finally {
                // Giảm số kết nối đang hoạt động
                activeConnections.decrementAndGet();
                
                try {
                    clientSocket.close();
                    System.out.println("Đã đóng kết nối #" + connectionId);
                } catch (IOException e) {
                    System.err.println("Lỗi đóng socket: " + e.getMessage());
                }
            }
        }
    }
    
    public static void main(String[] args) {
        new ThreadSafeServer().start();
    }
}
```

## Sử dụng NIO cho hiệu suất cao

Java NIO (New I/O) cung cấp cách xử lý I/O không blocking, hiệu quả hơn cho các ứng dụng cần xử lý nhiều kết nối.

### NIO Server với Selector

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class NIOServer {
    private static final int PORT = 8083;
    private static final int BUFFER_SIZE = 1024;
    private Selector selector;
    private ExecutorService workerPool;
    
    public NIOServer() {
        this.workerPool = Executors.newFixedThreadPool(10);
    }
    
    public void start() {
        try {
            // Tạo Selector để quản lý các channel
            selector = Selector.open();
            
            // Tạo ServerSocketChannel
            ServerSocketChannel serverChannel = ServerSocketChannel.open();
            serverChannel.configureBlocking(false); // Non-blocking mode
            serverChannel.bind(new InetSocketAddress(PORT));
            
            // Đăng ký channel với selector
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);
            
            System.out.println("NIO Server đang chạy trên port " + PORT);
            
            while (true) {
                // Chờ các channel sẵn sàng
                int readyChannels = selector.select();
                
                if (readyChannels == 0) continue;
                
                // Lấy các key đã sẵn sàng
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
                
                while (keyIterator.hasNext()) {
                    SelectionKey key = keyIterator.next();
                    keyIterator.remove();
                    
                    if (key.isAcceptable()) {
                        handleAccept(key);
                    } else if (key.isReadable()) {
                        handleRead(key);
                    }
                }
            }
            
        } catch (IOException e) {
            System.err.println("Lỗi NIO server: " + e.getMessage());
        }
    }
    
    private void handleAccept(SelectionKey key) throws IOException {
        ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
        SocketChannel clientChannel = serverChannel.accept();
        
        if (clientChannel != null) {
            clientChannel.configureBlocking(false);
            clientChannel.register(selector, SelectionKey.OP_READ);
            
            System.out.println("Client mới kết nối: " + clientChannel.getRemoteAddress());
        }
    }
    
    private void handleRead(SelectionKey key) throws IOException {
        SocketChannel clientChannel = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);
        
        int bytesRead = clientChannel.read(buffer);
        
        if (bytesRead > 0) {
            buffer.flip();
            byte[] data = new byte[buffer.remaining()];
            buffer.get(data);
            
            String message = new String(data);
            System.out.println("Nhận được: " + message + " từ " + clientChannel.getRemoteAddress());
            
            // Xử lý dữ liệu trong worker thread
            workerPool.submit(() -> {
                try {
                    // Mô phỏng xử lý phức tạp
                    Thread.sleep(1000);
                    
                    String response = "NIO Server phản hồi: " + message.toUpperCase();
                    ByteBuffer responseBuffer = ByteBuffer.wrap(response.getBytes());
                    clientChannel.write(responseBuffer);
                    
                } catch (IOException | InterruptedException e) {
                    System.err.println("Lỗi xử lý client: " + e.getMessage());
                }
            });
            
        } else if (bytesRead == -1) {
            // Client đã đóng kết nối
            System.out.println("Client đã ngắt kết nối: " + clientChannel.getRemoteAddress());
            clientChannel.close();
        }
    }
    
    public static void main(String[] args) {
        new NIOServer().start();
    }
}
```

## So sánh các mô hình đa luồng

| Mô hình | Ưu điểm | Nhược điểm | Phù hợp |
|---------|---------|------------|---------|
| **Thread-per-Connection** | Đơn giản, dễ hiểu | Tốn tài nguyên, không scale | Ứng dụng nhỏ |
| **Thread Pool** | Cân bằng tài nguyên, scale tốt | Phức tạp hơn | Ứng dụng vừa và lớn |
| **NIO** | Hiệu suất cao, ít thread | Phức tạp, khó debug | Ứng dụng cần hiệu suất cao |
| **Reactive** | Xử lý bất đồng bộ tốt | Học curve cao | Ứng dụng real-time |

## Best Practices cho đa luồng mạng

### 1. Quản lý Thread Pool
```java
// Sử dụng ThreadPoolExecutor với cấu hình phù hợp
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    Runtime.getRuntime().availableProcessors(), // Core threads
    Runtime.getRuntime().availableProcessors() * 2, // Max threads
    60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(1000),
    new ThreadFactory() {
        private AtomicInteger counter = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "NetworkThread-" + counter.getAndIncrement());
            t.setDaemon(true);
            return t;
        }
    }
);
```

### 2. Xử lý Exception
```java
try {
    // Network operations
} catch (IOException e) {
    logger.error("Network error: " + e.getMessage(), e);
    // Cleanup resources
} catch (Exception e) {
    logger.error("Unexpected error: " + e.getMessage(), e);
    // Handle unexpected errors
} finally {
    // Always cleanup
    if (socket != null && !socket.isClosed()) {
        try {
            socket.close();
        } catch (IOException e) {
            logger.warn("Error closing socket: " + e.getMessage());
        }
    }
}
```

### 3. Monitoring và Metrics
```java
// Sử dụng metrics để monitor
public class ServerMetrics {
    private AtomicLong totalRequests = new AtomicLong(0);
    private AtomicLong activeConnections = new AtomicLong(0);
    private AtomicLong errorCount = new AtomicLong(0);
    
    public void incrementRequests() {
        totalRequests.incrementAndGet();
    }
    
    public void incrementActiveConnections() {
        activeConnections.incrementAndGet();
    }
    
    public void decrementActiveConnections() {
        activeConnections.decrementAndGet();
    }
    
    public void incrementErrors() {
        errorCount.incrementAndGet();
    }
    
    // Getter methods...
}
```

## Kết luận

Đa luồng trong mạng là một kỹ thuật quan trọng để xây dựng các ứng dụng server hiệu quả. Việc chọn mô hình phù hợp phụ thuộc vào:

- **Quy mô ứng dụng**: Số lượng client đồng thời
- **Yêu cầu hiệu suất**: Throughput và latency
- **Độ phức tạp**: Khả năng maintain và debug
- **Tài nguyên**: CPU, memory, network

Hãy bắt đầu với mô hình đơn giản (Thread-per-Connection) và nâng cấp dần lên các mô hình phức tạp hơn khi cần thiết. Luôn nhớ đảm bảo thread safety và xử lý exception đúng cách để tránh memory leak và crash server.
