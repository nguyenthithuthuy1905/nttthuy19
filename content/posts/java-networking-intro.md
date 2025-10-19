---
title: "Java Networking – Tổng quan và các giao thức phổ biến"
date: 2025-10-13T20:00:00+07:00
tags: ["java", "networking"]
categories: ["Java"]
description: "Giới thiệu tổng quan về Java Networking, mô hình TCP/IP và cách Java hỗ trợ làm việc với mạng."
draft: false
featured_image: "/images/java.svg"
---

Trong bài viết này, chúng ta điểm qua tổng quan Java Networking: mô hình OSI/TCP-IP, sự khác nhau giữa TCP và UDP, và các API mạng cốt lõi của Java.

## Nội dung chính

- Kiến trúc TCP/IP, vai trò của IP, TCP, UDP
- DNS, cổng (port), socket là gì
- Thư viện mạng trong Java: `java.net` (Socket, ServerSocket, DatagramSocket, URL/HttpURLConnection) và `java.net.http.HttpClient`
- Quy tắc bảo mật khi lập trình mạng: timeouts, validation, giới hạn tài nguyên

## Khi nào dùng TCP, khi nào dùng UDP?

- TCP bảo đảm thứ tự, độ tin cậy. Phù hợp truyền dữ liệu quan trọng (giao dịch, API, file).
- UDP độ trễ thấp, không bảo đảm. Phù hợp streaming/real-time, game, telemetry.

## Ví dụ nhỏ

Ví dụ request HTTP đơn giản bằng `HttpClient` (Java 11+):

```java
var client = java.net.http.HttpClient.newHttpClient();
var req = java.net.http.HttpRequest.newBuilder()
        .uri(java.net.URI.create("https://example.com"))
        .GET()
        .build();
var res = client.send(req, java.net.http.HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());
```

Trong các bài tiếp theo, ta sẽ xây dựng socket server TCP cơ bản và thực hành best practices với `HttpClient`.


