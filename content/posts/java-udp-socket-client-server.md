---
title: "Xây dựng mô hình Client-Server với UDP trong Java"
date: 2025-10-15T10:00:00+07:00
draft: false
description: "Hướng dẫn chi tiết cách xây dựng ứng dụng client-server sử dụng giao thức UDP trong Java với ví dụ thực tế"
tags: ["java", "udp", "networking", "socket", "client-server"]
categories: ["java"]
---

## Giới thiệu về UDP (User Datagram Protocol)

UDP là một giao thức truyền tải không kết nối (connectionless) trong mạng Internet. Khác với TCP, UDP không đảm bảo việc gửi dữ liệu thành công và không có cơ chế kiểm soát lỗi tự động. Tuy nhiên, UDP có ưu điểm là nhanh hơn và ít overhead hơn TCP, phù hợp cho các ứng dụng yêu cầu tốc độ cao như streaming video, game online, hoặc các ứng dụng real-time.

### Đặc điểm chính của UDP:
- **Không kết nối**: Không cần thiết lập kết nối trước khi gửi dữ liệu
- **Không đảm bảo**: Dữ liệu có thể bị mất hoặc đến không đúng thứ tự
- **Nhanh**: Ít overhead, phù hợp cho dữ liệu real-time
- **Đơn giản**: Cấu trúc header đơn giản hơn TCP

## Tạo UDP Server trong Java

Để tạo một UDP server, chúng ta sử dụng class `DatagramSocket` và `DatagramPacket`. Server sẽ lắng nghe trên một port cụ thể và xử lý các gói dữ liệu đến.

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;

public class UDPServer {
    private static final int PORT = 8888;
    private static final int BUFFER_SIZE = 1024;
    
    public static void main(String[] args) {
        try {
            // Tạo DatagramSocket để lắng nghe trên port 8888
            DatagramSocket serverSocket = new DatagramSocket(PORT);
            System.out.println("UDP Server đang chạy trên port " + PORT);
            
            while (true) {
                // Tạo buffer để nhận dữ liệu
                byte[] buffer = new byte[BUFFER_SIZE];
                DatagramPacket receivePacket = new DatagramPacket(buffer, buffer.length);
                
                // Chờ nhận dữ liệu từ client
                System.out.println("Đang chờ dữ liệu từ client...");
                serverSocket.receive(receivePacket);
                
                // Lấy thông tin client
                InetAddress clientAddress = receivePacket.getAddress();
                int clientPort = receivePacket.getPort();
                
                // Chuyển đổi dữ liệu nhận được thành String
                String receivedData = new String(receivePacket.getData(), 0, receivePacket.getLength());
                System.out.println("Nhận được từ " + clientAddress + ":" + clientPort + " - " + receivedData);
                
                // Xử lý dữ liệu và tạo phản hồi
                String response = "Server nhận được: " + receivedData.toUpperCase();
                byte[] responseData = response.getBytes();
                
                // Gửi phản hồi về client
                DatagramPacket sendPacket = new DatagramPacket(
                    responseData, 
                    responseData.length, 
                    clientAddress, 
                    clientPort
                );
                serverSocket.send(sendPacket);
                System.out.println("Đã gửi phản hồi: " + response);
            }
            
        } catch (SocketException e) {
            System.err.println("Lỗi tạo socket: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("Lỗi I/O: " + e.getMessage());
        }
    }
}
```

### Giải thích code server:

1. **DatagramSocket**: Tạo socket UDP để lắng nghe trên port 8888
2. **DatagramPacket**: Đại diện cho một gói dữ liệu UDP
3. **receive()**: Phương thức blocking chờ nhận dữ liệu từ client
4. **send()**: Gửi dữ liệu phản hồi về client

## Tạo UDP Client trong Java

Client sẽ kết nối đến server và gửi/nhận dữ liệu. Khác với TCP, UDP client không cần thiết lập kết nối trước.

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;
import java.net.UnknownHostException;

public class UDPClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 8888;
    private static final int BUFFER_SIZE = 1024;
    
    public static void main(String[] args) {
        try {
            // Tạo DatagramSocket cho client
            DatagramSocket clientSocket = new DatagramSocket();
            
            // Lấy địa chỉ IP của server
            InetAddress serverAddress = InetAddress.getByName(SERVER_HOST);
            
            BufferedReader userInput = new BufferedReader(new InputStreamReader(System.in));
            
            System.out.println("UDP Client đã kết nối đến server " + SERVER_HOST + ":" + SERVER_PORT);
            System.out.println("Nhập tin nhắn (gõ 'exit' để thoát):");
            
            while (true) {
                // Đọc input từ người dùng
                System.out.print("Client: ");
                String message = userInput.readLine();
                
                if ("exit".equalsIgnoreCase(message)) {
                    break;
                }
                
                // Chuyển đổi message thành byte array
                byte[] sendData = message.getBytes();
                
                // Tạo packet để gửi
                DatagramPacket sendPacket = new DatagramPacket(
                    sendData, 
                    sendData.length, 
                    serverAddress, 
                    SERVER_PORT
                );
                
                // Gửi dữ liệu đến server
                clientSocket.send(sendPacket);
                System.out.println("Đã gửi: " + message);
                
                // Tạo buffer để nhận phản hồi
                byte[] receiveBuffer = new byte[BUFFER_SIZE];
                DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);
                
                // Nhận phản hồi từ server
                clientSocket.receive(receivePacket);
                
                // Chuyển đổi phản hồi thành String
                String response = new String(receivePacket.getData(), 0, receivePacket.getLength());
                System.out.println("Server phản hồi: " + response);
                System.out.println("---");
            }
            
            // Đóng socket
            clientSocket.close();
            System.out.println("Client đã ngắt kết nối");
            
        } catch (SocketException e) {
            System.err.println("Lỗi socket: " + e.getMessage());
        } catch (UnknownHostException e) {
            System.err.println("Không tìm thấy host: " + e.getMessage());
        } catch (IOException e) {
            System.err.println("Lỗi I/O: " + e.getMessage());
        }
    }
}
```

### Giải thích code client:

1. **DatagramSocket()**: Tạo socket UDP cho client (không cần chỉ định port)
2. **InetAddress.getByName()**: Lấy địa chỉ IP của server
3. **send()**: Gửi dữ liệu đến server
4. **receive()**: Nhận phản hồi từ server

## Ví dụ nâng cao: Chat Room với UDP

Dưới đây là ví dụ về một chat room đơn giản sử dụng UDP, cho phép nhiều client kết nối và gửi tin nhắn.

### UDP Chat Server

```java
import java.io.IOException;
import java.net.*;
import java.util.ArrayList;
import java.util.List;

public class UDPChatServer {
    private static final int PORT = 9999;
    private static final int BUFFER_SIZE = 1024;
    private List<SocketAddress> clients = new ArrayList<>();
    
    public static void main(String[] args) {
        new UDPChatServer().start();
    }
    
    public void start() {
        try {
            DatagramSocket serverSocket = new DatagramSocket(PORT);
            System.out.println("Chat Server đang chạy trên port " + PORT);
            
            while (true) {
                byte[] buffer = new byte[BUFFER_SIZE];
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                
                // Nhận dữ liệu
                serverSocket.receive(packet);
                
                String message = new String(packet.getData(), 0, packet.getLength());
                SocketAddress clientAddress = packet.getSocketAddress();
                
                // Thêm client mới nếu chưa có
                if (!clients.contains(clientAddress)) {
                    clients.add(clientAddress);
                    System.out.println("Client mới tham gia: " + clientAddress);
                }
                
                System.out.println("Nhận được: " + message + " từ " + clientAddress);
                
                // Gửi tin nhắn đến tất cả client khác
                broadcast(message, clientAddress, serverSocket);
                
            }
        } catch (IOException e) {
            System.err.println("Lỗi server: " + e.getMessage());
        }
    }
    
    private void broadcast(String message, SocketAddress sender, DatagramSocket socket) {
        byte[] data = message.getBytes();
        
        for (SocketAddress client : clients) {
            if (!client.equals(sender)) {
                try {
                    DatagramPacket packet = new DatagramPacket(data, data.length, client);
                    socket.send(packet);
                } catch (IOException e) {
                    System.err.println("Lỗi gửi đến " + client + ": " + e.getMessage());
                }
            }
        }
    }
}
```

### UDP Chat Client

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.*;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class UDPChatClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 9999;
    private static final int BUFFER_SIZE = 1024;
    
    public static void main(String[] args) {
        try {
            DatagramSocket clientSocket = new DatagramSocket();
            InetAddress serverAddress = InetAddress.getByName(SERVER_HOST);
            
            // Gửi tin nhắn chào mừng
            String joinMessage = "Client đã tham gia chat";
            byte[] joinData = joinMessage.getBytes();
            DatagramPacket joinPacket = new DatagramPacket(joinData, joinData.length, serverAddress, SERVER_PORT);
            clientSocket.send(joinPacket);
            
            // Tạo thread để nhận tin nhắn
            ExecutorService executor = Executors.newSingleThreadExecutor();
            executor.submit(() -> {
                while (true) {
                    try {
                        byte[] buffer = new byte[BUFFER_SIZE];
                        DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                        clientSocket.receive(packet);
                        
                        String message = new String(packet.getData(), 0, packet.getLength());
                        System.out.println("Nhận được: " + message);
                    } catch (IOException e) {
                        System.err.println("Lỗi nhận tin nhắn: " + e.getMessage());
                        break;
                    }
                }
            });
            
            // Thread chính để gửi tin nhắn
            BufferedReader userInput = new BufferedReader(new InputStreamReader(System.in));
            System.out.println("Chat Client - Nhập tin nhắn (gõ 'exit' để thoát):");
            
            while (true) {
                System.out.print("Bạn: ");
                String message = userInput.readLine();
                
                if ("exit".equalsIgnoreCase(message)) {
                    break;
                }
                
                byte[] data = message.getBytes();
                DatagramPacket packet = new DatagramPacket(data, data.length, serverAddress, SERVER_PORT);
                clientSocket.send(packet);
            }
            
            clientSocket.close();
            executor.shutdown();
            
        } catch (IOException e) {
            System.err.println("Lỗi client: " + e.getMessage());
        }
    }
}
```

## So sánh UDP và TCP

| Đặc điểm | UDP | TCP |
|----------|-----|-----|
| **Kết nối** | Không kết nối | Có kết nối |
| **Đảm bảo gửi** | Không | Có |
| **Thứ tự dữ liệu** | Không đảm bảo | Đảm bảo |
| **Tốc độ** | Nhanh | Chậm hơn |
| **Overhead** | Thấp | Cao |
| **Phù hợp** | Real-time, streaming | Web, email, file transfer |

## Khi nào nên sử dụng UDP?

UDP phù hợp cho các ứng dụng:
- **Game online**: Cần tốc độ cao, có thể chấp nhận mất một số gói dữ liệu
- **Streaming video/audio**: Real-time, mất một số frame không ảnh hưởng nhiều
- **DNS queries**: Yêu cầu nhanh, đơn giản
- **IoT sensors**: Gửi dữ liệu cảm biến liên tục
- **Live chat**: Tin nhắn nhanh, có thể mất một số tin nhắn

## Kết luận

UDP là một giao thức mạnh mẽ cho các ứng dụng yêu cầu tốc độ cao và có thể chấp nhận mất dữ liệu. Trong Java, việc sử dụng `DatagramSocket` và `DatagramPacket` giúp chúng ta dễ dàng xây dựng các ứng dụng UDP. Tuy nhiên, cần lưu ý rằng UDP không đảm bảo việc gửi dữ liệu thành công, vì vậy ứng dụng cần có cơ chế xử lý lỗi phù hợp.

Khi phát triển ứng dụng thực tế, hãy cân nhắc kỹ giữa tốc độ và độ tin cậy để chọn giao thức phù hợp nhất cho nhu cầu của bạn.
