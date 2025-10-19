---
title: "Java Networking – Best Practices với HttpClient"
date: 2025-10-13T20:20:00+07:00
tags: ["java", "networking", "http", "best-practices"]
categories: ["Java"]
description: "Kinh nghiệm cấu hình HttpClient: timeouts, connection pooling, retry, JSON."
draft: false
featured_image: "/images/java.svg"
---

`HttpClient` (Java 11+) là lựa chọn hiện đại để gọi HTTP. Dưới đây là một số thiết lập khuyến nghị.

## Cấu hình khởi tạo

```java
var client = java.net.http.HttpClient.newBuilder()
        .connectTimeout(java.time.Duration.ofSeconds(5))
        .executor(java.util.concurrent.Executors.newFixedThreadPool(8))
        .followRedirects(java.net.http.HttpClient.Redirect.NORMAL)
        .version(java.net.http.HttpClient.Version.HTTP_2)
        .build();
```

## Timeout cho từng request

```java
var req = java.net.http.HttpRequest.newBuilder()
        .uri(java.net.URI.create("https://api.example.com/data"))
        .timeout(java.time.Duration.ofSeconds(10))
        .header("Accept", "application/json")
        .build();
```

## Retry đơn giản (ví dụ minh họa)

```java
java.net.http.HttpResponse<String> res = null;
for (int i = 0; i < 3; i++) {
    try {
        res = client.send(req, java.net.http.HttpResponse.BodyHandlers.ofString());
        if (res.statusCode() >= 200 && res.statusCode() < 500) break;
    } catch (Exception ignore) {}
    Thread.sleep(200L * (i + 1));
}
```

## Lưu ý bảo mật

- Validate URL/headers, không log secrets
- Giới hạn kích thước body, đặt timeouts
- Dùng HTTPS và xác thực chứng chỉ đúng chuẩn


