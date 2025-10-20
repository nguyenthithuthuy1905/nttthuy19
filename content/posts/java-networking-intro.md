---
title: "Java Networking – Tổng quan và các giao thức phổ biến"
date: 2025-10-13T20:00:00+07:00
tags: ["java", "networking", "tcp", "udp", "http"]
categories: ["Java"]
description: "Hướng dẫn chi tiết về Java Networking: mô hình TCP/IP, giao thức HTTP, socket programming và các API mạng cốt lõi"
draft: false
featured_image: "/images/java.svg"
---

# Java Networking – Tổng quan và các giao thức phổ biến

## 🎯 **Mục tiêu học tập**
- Hiểu mô hình TCP/IP và các giao thức mạng cơ bản
- Phân biệt TCP vs UDP và ứng dụng thực tế
- Nắm vững các API mạng trong Java
- Thực hành lập trình socket và HTTP client

## 🌐 **1. Tổng quan về mạng máy tính**

### **Mô hình OSI vs TCP/IP**
Mạng máy tính được tổ chức theo các tầng (layers) để dễ quản lý và phát triển:

**Mô hình OSI (7 tầng):**
- Application (7) - HTTP, FTP, SMTP
- Presentation (6) - Encryption, Compression
- Session (5) - Session management
- Transport (4) - TCP, UDP
- Network (3) - IP, Routing
- Data Link (2) - Ethernet, WiFi
- Physical (1) - Cables, Wireless

**Mô hình TCP/IP (4 tầng):**
- Application - HTTP, FTP, SMTP, DNS
- Transport - TCP, UDP
- Internet - IP, ICMP
- Network Access - Ethernet, WiFi

### **Tại sao cần hiểu mô hình mạng?**
- **Debugging**: Khi lỗi xảy ra, biết tầng nào để tìm nguyên nhân
- **Performance**: Tối ưu hóa đúng tầng để cải thiện hiệu suất
- **Security**: Áp dụng bảo mật phù hợp cho từng tầng
- **Development**: Chọn công cụ và API phù hợp

## 🔌 **2. Giao thức TCP vs UDP**

### **TCP (Transmission Control Protocol)**
**Đặc điểm:**
- **Reliable**: Đảm bảo dữ liệu được truyền đúng và đầy đủ
- **Ordered**: Dữ liệu đến đúng thứ tự
- **Connection-oriented**: Cần thiết lập kết nối trước khi truyền
- **Flow control**: Điều chỉnh tốc độ truyền phù hợp

**Khi nào dùng TCP:**
```java
// Ví dụ: Web browsing, file transfer, email
// Dữ liệu quan trọng, cần đảm bảo tính toàn vẹn
var socket = new Socket("example.com", 80);
```

**Ưu điểm:**
- Đáng tin cậy, dữ liệu không bị mất
- Phù hợp cho hầu hết ứng dụng

**Nhược điểm:**
- Overhead cao (header lớn, ACK packets)
- Độ trễ cao hơn UDP

### **UDP (User Datagram Protocol)**
**Đặc điểm:**
- **Unreliable**: Không đảm bảo dữ liệu đến đích
- **Unordered**: Dữ liệu có thể đến không đúng thứ tự
- **Connectionless**: Không cần thiết lập kết nối
- **Lightweight**: Header nhỏ, overhead thấp

**Khi nào dùng UDP:**
```java
// Ví dụ: Video streaming, online gaming, DNS queries
// Dữ liệu real-time, chấp nhận mất một số packet
var socket = new DatagramSocket();
```

**Ưu điểm:**
- Tốc độ cao, độ trễ thấp
- Phù hợp cho real-time applications

**Nhược điểm:**
- Không đảm bảo tính toàn vẹn dữ liệu
- Cần xử lý lỗi ở tầng ứng dụng

## 🌍 **3. Các khái niệm cơ bản**

### **IP Address (Địa chỉ IP)**
```java
// IPv4: 192.168.1.1 (32-bit)
// IPv6: 2001:0db8:85a3::8a2e:0370:7334 (128-bit)

// Java hỗ trợ cả IPv4 và IPv6
InetAddress address = InetAddress.getByName("google.com");
System.out.println("IP: " + address.getHostAddress());
```

### **Port (Cổng)**
- **Port**: Số hiệu để phân biệt các dịch vụ trên cùng một máy
- **Range**: 0-65535 (16-bit)
- **Well-known ports**: 0-1023 (HTTP: 80, HTTPS: 443, SSH: 22)
- **Registered ports**: 1024-49151
- **Dynamic ports**: 49152-65535

### **DNS (Domain Name System)**
```java
// Chuyển đổi domain name thành IP address
InetAddress address = InetAddress.getByName("google.com");
System.out.println("IP: " + address.getHostAddress());
System.out.println("Hostname: " + address.getHostName());
```

### **Socket**
**Socket** là endpoint của kết nối mạng, cho phép hai chương trình giao tiếp qua mạng:

```java
// TCP Socket
Socket clientSocket = new Socket("server.com", 8080);

// UDP Socket  
DatagramSocket udpSocket = new DatagramSocket();
```

## ☕ **4. Java Networking APIs**

### **java.net Package - Các class chính**

#### **Socket & ServerSocket (TCP)**
```java
// Server side
ServerSocket serverSocket = new ServerSocket(8080);
Socket clientSocket = serverSocket.accept();

// Client side
Socket socket = new Socket("localhost", 8080);
```

#### **DatagramSocket (UDP)**
```java
// UDP Server
DatagramSocket serverSocket = new DatagramSocket(8080);
byte[] buffer = new byte[1024];
DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
serverSocket.receive(packet);

// UDP Client
DatagramSocket clientSocket = new DatagramSocket();
String message = "Hello UDP";
byte[] data = message.getBytes();
InetAddress address = InetAddress.getByName("localhost");
DatagramPacket packet = new DatagramPacket(data, data.length, address, 8080);
clientSocket.send(packet);
```

#### **URL & HttpURLConnection (HTTP)**
```java
// HTTP Request (cách cũ)
URL url = new URL("https://api.example.com/data");
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
connection.setRequestMethod("GET");
connection.setRequestProperty("Accept", "application/json");

int responseCode = connection.getResponseCode();
if (responseCode == 200) {
    BufferedReader reader = new BufferedReader(
        new InputStreamReader(connection.getInputStream()));
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
```

### **java.net.http Package (Java 11+)**

#### **HttpClient - Cách hiện đại**
```java
// Tạo HTTP client
HttpClient client = HttpClient.newHttpClient();

// Tạo request
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .header("Accept", "application/json")
    .timeout(Duration.ofSeconds(10))
    .GET()
    .build();

// Gửi request và xử lý response
HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());

System.out.println("Status: " + response.statusCode());
System.out.println("Body: " + response.body());
```

## 🔒 **5. Bảo mật trong lập trình mạng**

### **Các nguyên tắc cơ bản**
```java
// 1. Validate input
public void processRequest(String input) {
    if (input == null || input.length() > MAX_LENGTH) {
        throw new IllegalArgumentException("Invalid input");
    }
    // Process...
}

// 2. Set timeouts
Socket socket = new Socket();
socket.setSoTimeout(5000); // 5 seconds timeout

// 3. Use HTTPS instead of HTTP
HttpClient client = HttpClient.newBuilder()
    .build();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://secure-api.com/data")) // HTTPS
    .build();

// 4. Limit resource usage
private static final int MAX_CONNECTIONS = 100;
private static final int MAX_REQUEST_SIZE = 1024 * 1024; // 1MB
```

### **Xử lý lỗi mạng**
```java
try {
    Socket socket = new Socket("example.com", 80);
    // Network operations...
} catch (ConnectException e) {
    System.err.println("Cannot connect to server: " + e.getMessage());
} catch (SocketTimeoutException e) {
    System.err.println("Connection timeout: " + e.getMessage());
} catch (IOException e) {
    System.err.println("IO Error: " + e.getMessage());
} finally {
    // Clean up resources
    if (socket != null) {
        try {
            socket.close();
        } catch (IOException e) {
            // Log error
        }
    }
}
```

## 🚀 **6. Ví dụ thực tế: HTTP Client**

### **Tạo HTTP Client đơn giản**
```java
public class SimpleHttpClient {
    private final HttpClient client;
    
    public SimpleHttpClient() {
        this.client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(5))
            .build();
    }
    
    public String get(String url) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .timeout(Duration.ofSeconds(10))
            .GET()
            .build();
            
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
            
        if (response.statusCode() >= 200 && response.statusCode() < 300) {
            return response.body();
        } else {
            throw new IOException("HTTP Error: " + response.statusCode());
        }
    }
}

// Sử dụng
SimpleHttpClient httpClient = new SimpleHttpClient();
try {
    String response = httpClient.get("https://api.github.com/users/octocat");
    System.out.println(response);
} catch (Exception e) {
    System.err.println("Error: " + e.getMessage());
}
```

## 📋 **7. Best Practices**

### **Do's (Nên làm)**
- ✅ Luôn set timeout cho network operations
- ✅ Sử dụng try-with-resources để tự động đóng connections
- ✅ Validate input trước khi gửi qua mạng
- ✅ Log errors để debug dễ dàng
- ✅ Sử dụng HTTPS cho dữ liệu nhạy cảm

### **Don'ts (Không nên)**
- ❌ Hardcode URLs và credentials
- ❌ Ignore exceptions từ network operations
- ❌ Tạo quá nhiều connections cùng lúc
- ❌ Gửi dữ liệu không được validate
- ❌ Sử dụng HTTP cho dữ liệu nhạy cảm

## 🎯 **Kết luận**

Java cung cấp một bộ API mạng mạnh mẽ và linh hoạt. Hiểu rõ các giao thức TCP/UDP và cách sử dụng các API này sẽ giúp bạn:

1. **Xây dựng ứng dụng mạng hiệu quả** với performance tốt
2. **Debug dễ dàng** khi có vấn đề về mạng
3. **Bảo mật tốt hơn** với các best practices
4. **Chọn đúng công cụ** cho từng use case

Trong các bài tiếp theo, chúng ta sẽ đi sâu vào:
- **Socket Programming**: Xây dựng TCP server/client
- **HTTP Best Practices**: Cấu hình HttpClient tối ưu
- **Async Networking**: Sử dụng CompletableFuture cho hiệu suất cao

Hãy bắt đầu với những kiến thức cơ bản này và từ từ nâng cao kỹ năng lập trình mạng!


