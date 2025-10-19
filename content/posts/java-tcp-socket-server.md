---
title: "Java Networking – Viết TCP Socket Server tối giản"
date: 2025-10-13T20:10:00+07:00
tags: ["java", "networking", "tcp", "socket"]
categories: ["Java"]
description: "Tạo TCP server tối giản bằng ServerSocket, xử lý kết nối đồng thời bằng thread pool."
draft: false
featured_image: "/images/java.svg"
---

Bài này hướng dẫn tạo một TCP server đơn giản nhận chuỗi từ client và echo lại. Ta sẽ dùng `ServerSocket` và `ExecutorService` để phục vụ nhiều client.

## Mã nguồn mẫu

```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;

public class EchoServer {
    public static void main(String[] args) throws Exception {
        int port = 9000;
        var pool = Executors.newFixedThreadPool(8);
        try (var server = new ServerSocket(port)) {
            System.out.println("EchoServer listening on " + port);
            while (true) {
                Socket socket = server.accept();
                pool.submit(() -> handle(socket));
            }
        }
    }

    static void handle(Socket socket) {
        try (socket;
             var in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             var out = new PrintWriter(socket.getOutputStream(), true)) {
            out.println("Hello from EchoServer");
            String line;
            while ((line = in.readLine()) != null) {
                out.println("echo: " + line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Lưu ý vận hành

- Đặt timeout (`setSoTimeout`, read timeouts ở tầng app)
- Giới hạn kích thước message, sanitize input
- Dùng pool/selector (NIO) cho số lượng kết nối lớn


