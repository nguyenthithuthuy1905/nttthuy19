---
title: "Java RMI – Hướng dẫn toàn tập: từ khái niệm đến triển khai và khắc phục lỗi"
date: 2025-10-13T12:00:00+07:00
draft: false
description: "Hướng dẫn chi tiết về Java RMI từ cơ bản đến nâng cao, bao gồm cách triển khai, xử lý lỗi và best practices"
tags: ["java", "rmi", "distributed", "remote", "networking"]
categories: ["java"]
---

## Giới thiệu về Java RMI

Java RMI (Remote Method Invocation) là một cơ chế cho phép một ứng dụng Java chạy trên một máy ảo (JVM) gọi phương thức của một đối tượng Java chạy trên một JVM khác. RMI là một phần của Java API và cung cấp khả năng phân tán cho các ứng dụng Java.

### RMI hoạt động như thế nào?

RMI sử dụng kiến trúc client-server với các thành phần chính:

1. **Remote Interface**: Định nghĩa các phương thức có thể gọi từ xa
2. **Remote Object**: Triển khai interface, chạy trên server
3. **RMI Registry**: Dịch vụ đăng ký và tra cứu đối tượng remote
4. **Stub**: Proxy object trên client, đại diện cho remote object
5. **Skeleton**: Object trên server, nhận request từ client

### Use case thực tế của RMI:

- **Distributed Computing**: Tính toán phân tán trên nhiều máy
- **Microservices**: Giao tiếp giữa các service
- **Load Balancing**: Phân tải xử lý
- **Remote Database Access**: Truy cập database từ xa
- **File Sharing**: Chia sẻ file giữa các máy

## Tạo RMI Server cơ bản

### Bước 1: Tạo Remote Interface

Đầu tiên, chúng ta cần tạo một interface kế thừa từ `java.rmi.Remote`:

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

// Remote interface định nghĩa các phương thức có thể gọi từ xa
public interface CalculatorService extends Remote {
    
    // Tất cả phương thức trong remote interface phải throw RemoteException
    double add(double a, double b) throws RemoteException;
    double subtract(double a, double b) throws RemoteException;
    double multiply(double a, double b) throws RemoteException;
    double divide(double a, double b) throws RemoteException;
    
    // Phương thức lấy thông tin server
    String getServerInfo() throws RemoteException;
}
```

### Bước 2: Triển khai Remote Object

Tiếp theo, chúng ta tạo class triển khai interface:

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.util.Date;

// Class triển khai remote interface
public class CalculatorServiceImpl extends UnicastRemoteObject implements CalculatorService {
    
    // Constructor phải throw RemoteException
    public CalculatorServiceImpl() throws RemoteException {
        super(); // Gọi constructor của UnicastRemoteObject
    }
    
    @Override
    public double add(double a, double b) throws RemoteException {
        System.out.println("Server: Thực hiện phép cộng " + a + " + " + b);
        return a + b;
    }
    
    @Override
    public double subtract(double a, double b) throws RemoteException {
        System.out.println("Server: Thực hiện phép trừ " + a + " - " + b);
        return a - b;
    }
    
    @Override
    public double multiply(double a, double b) throws RemoteException {
        System.out.println("Server: Thực hiện phép nhân " + a + " * " + b);
        return a * b;
    }
    
    @Override
    public double divide(double a, double b) throws RemoteException {
        if (b == 0) {
            throw new RemoteException("Không thể chia cho 0!");
        }
        System.out.println("Server: Thực hiện phép chia " + a + " / " + b);
        return a / b;
    }
    
    @Override
    public String getServerInfo() throws RemoteException {
        return "Calculator Server - " + new Date().toString();
    }
}
```

### Bước 3: Tạo RMI Server

Bây giờ chúng ta tạo server để đăng ký remote object:

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.AlreadyBoundException;

public class CalculatorServer {
    private static final int REGISTRY_PORT = 1099;
    private static final String SERVICE_NAME = "CalculatorService";
    
    public static void main(String[] args) {
        try {
            // Tạo RMI Registry trên port 1099
            Registry registry = LocateRegistry.createRegistry(REGISTRY_PORT);
            System.out.println("RMI Registry đã được tạo trên port " + REGISTRY_PORT);
            
            // Tạo instance của remote object
            CalculatorService calculatorService = new CalculatorServiceImpl();
            
            // Đăng ký remote object với registry
            registry.bind(SERVICE_NAME, calculatorService);
            System.out.println("CalculatorService đã được đăng ký với tên: " + SERVICE_NAME);
            
            // Giữ server chạy
            System.out.println("Server đang chạy... Nhấn Ctrl+C để dừng.");
            Thread.currentThread().join();
            
        } catch (RemoteException e) {
            System.err.println("Lỗi RMI: " + e.getMessage());
        } catch (AlreadyBoundException e) {
            System.err.println("Service đã được đăng ký: " + e.getMessage());
        } catch (InterruptedException e) {
            System.out.println("Server đã dừng.");
        }
    }
}
```

## Tạo RMI Client

Client sẽ kết nối đến RMI Registry và sử dụng remote object:

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.NotBoundException;
import java.util.Scanner;

public class CalculatorClient {
    private static final String SERVER_HOST = "localhost";
    private static final int REGISTRY_PORT = 1099;
    private static final String SERVICE_NAME = "CalculatorService";
    
    public static void main(String[] args) {
        try {
            // Kết nối đến RMI Registry
            Registry registry = LocateRegistry.getRegistry(SERVER_HOST, REGISTRY_PORT);
            System.out.println("Đã kết nối đến RMI Registry tại " + SERVER_HOST + ":" + REGISTRY_PORT);
            
            // Lấy remote object từ registry
            CalculatorService calculator = (CalculatorService) registry.lookup(SERVICE_NAME);
            System.out.println("Đã lấy CalculatorService từ registry");
            
            // Lấy thông tin server
            String serverInfo = calculator.getServerInfo();
            System.out.println("Thông tin server: " + serverInfo);
            
            // Tương tác với user
            Scanner scanner = new Scanner(System.in);
            boolean running = true;
            
            while (running) {
                System.out.println("\n=== CALCULATOR CLIENT ===");
                System.out.println("1. Cộng");
                System.out.println("2. Trừ");
                System.out.println("3. Nhân");
                System.out.println("4. Chia");
                System.out.println("5. Thoát");
                System.out.print("Chọn phép tính (1-5): ");
                
                int choice = scanner.nextInt();
                
                if (choice == 5) {
                    running = false;
                    continue;
                }
                
                if (choice < 1 || choice > 4) {
                    System.out.println("Lựa chọn không hợp lệ!");
                    continue;
                }
                
                System.out.print("Nhập số thứ nhất: ");
                double a = scanner.nextDouble();
                System.out.print("Nhập số thứ hai: ");
                double b = scanner.nextDouble();
                
                double result = 0;
                String operation = "";
                
                try {
                    switch (choice) {
                        case 1:
                            result = calculator.add(a, b);
                            operation = "cộng";
                            break;
                        case 2:
                            result = calculator.subtract(a, b);
                            operation = "trừ";
                            break;
                        case 3:
                            result = calculator.multiply(a, b);
                            operation = "nhân";
                            break;
                        case 4:
                            result = calculator.divide(a, b);
                            operation = "chia";
                            break;
                    }
                    
                    System.out.println("Kết quả phép " + operation + ": " + result);
                    
                } catch (RemoteException e) {
                    System.err.println("Lỗi gọi remote method: " + e.getMessage());
                }
            }
            
            scanner.close();
            System.out.println("Client đã thoát.");
            
        } catch (RemoteException e) {
            System.err.println("Lỗi kết nối RMI: " + e.getMessage());
        } catch (NotBoundException e) {
            System.err.println("Không tìm thấy service: " + e.getMessage());
        }
    }
}
```

## Xử lý lỗi thường gặp trong RMI

### 1. Lỗi "Connection refused"

**Nguyên nhân**: RMI Registry chưa chạy hoặc không thể kết nối.

**Giải pháp**:
```bash
# Khởi động RMI Registry
rmiregistry 1099

# Hoặc sử dụng code để tạo registry
Registry registry = LocateRegistry.createRegistry(1099);
```

### 2. Lỗi "ClassNotFoundException"

**Nguyên nhân**: Không tìm thấy class trong classpath.

**Giải pháp**:
```bash
# Thêm current directory vào classpath
java -cp . CalculatorServer

# Hoặc sử dụng -Djava.rmi.server.codebase
java -Djava.rmi.server.codebase=file:/path/to/classes/ CalculatorServer
```

### 3. Lỗi "AccessException"

**Nguyên nhân**: Không có quyền truy cập remote object.

**Giải pháp**: Tạo security policy file:

```java
// policy.policy
grant {
    permission java.security.AllPermission;
};
```

Chạy với policy:
```bash
java -Djava.security.policy=policy.policy CalculatorServer
```

### 4. Lỗi "AlreadyBoundException"

**Nguyên nhân**: Service đã được đăng ký với tên đó.

**Giải pháp**:
```java
try {
    registry.bind(SERVICE_NAME, calculatorService);
} catch (AlreadyBoundException e) {
    // Unbind service cũ và bind lại
    registry.unbind(SERVICE_NAME);
    registry.bind(SERVICE_NAME, calculatorService);
}
```

## Ví dụ nâng cao: File Sharing Service

Dưới đây là ví dụ về một service chia sẻ file sử dụng RMI:

### Remote Interface cho File Service

```java
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.List;

public interface FileService extends Remote {
    
    // Lấy danh sách file
    List<String> listFiles() throws RemoteException;
    
    // Đọc nội dung file
    String readFile(String fileName) throws RemoteException;
    
    // Ghi nội dung vào file
    boolean writeFile(String fileName, String content) throws RemoteException;
    
    // Xóa file
    boolean deleteFile(String fileName) throws RemoteException;
    
    // Kiểm tra file có tồn tại không
    boolean fileExists(String fileName) throws RemoteException;
}
```

### Triển khai File Service

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class FileServiceImpl extends UnicastRemoteObject implements FileService {
    
    private static final String FILE_DIRECTORY = "shared_files/";
    
    public FileServiceImpl() throws RemoteException {
        super();
        // Tạo thư mục nếu chưa có
        File dir = new File(FILE_DIRECTORY);
        if (!dir.exists()) {
            dir.mkdirs();
        }
    }
    
    @Override
    public List<String> listFiles() throws RemoteException {
        List<String> files = new ArrayList<>();
        File directory = new File(FILE_DIRECTORY);
        
        if (directory.exists() && directory.isDirectory()) {
            File[] fileList = directory.listFiles();
            if (fileList != null) {
                for (File file : fileList) {
                    if (file.isFile()) {
                        files.add(file.getName());
                    }
                }
            }
        }
        
        return files;
    }
    
    @Override
    public String readFile(String fileName) throws RemoteException {
        try {
            File file = new File(FILE_DIRECTORY + fileName);
            if (!file.exists()) {
                throw new RemoteException("File không tồn tại: " + fileName);
            }
            
            StringBuilder content = new StringBuilder();
            try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    content.append(line).append("\n");
                }
            }
            
            return content.toString();
            
        } catch (IOException e) {
            throw new RemoteException("Lỗi đọc file: " + e.getMessage());
        }
    }
    
    @Override
    public boolean writeFile(String fileName, String content) throws RemoteException {
        try {
            File file = new File(FILE_DIRECTORY + fileName);
            
            try (PrintWriter writer = new PrintWriter(new FileWriter(file))) {
                writer.print(content);
            }
            
            return true;
            
        } catch (IOException e) {
            System.err.println("Lỗi ghi file: " + e.getMessage());
            return false;
        }
    }
    
    @Override
    public boolean deleteFile(String fileName) throws RemoteException {
        try {
            File file = new File(FILE_DIRECTORY + fileName);
            return file.delete();
        } catch (Exception e) {
            System.err.println("Lỗi xóa file: " + e.getMessage());
            return false;
        }
    }
    
    @Override
    public boolean fileExists(String fileName) throws RemoteException {
        File file = new File(FILE_DIRECTORY + fileName);
        return file.exists() && file.isFile();
    }
}
```

### File Service Server

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.AlreadyBoundException;

public class FileServiceServer {
    private static final int REGISTRY_PORT = 1099;
    private static final String SERVICE_NAME = "FileService";
    
    public static void main(String[] args) {
        try {
            // Tạo RMI Registry
            Registry registry = LocateRegistry.createRegistry(REGISTRY_PORT);
            System.out.println("RMI Registry đã được tạo trên port " + REGISTRY_PORT);
            
            // Tạo và đăng ký File Service
            FileService fileService = new FileServiceImpl();
            registry.bind(SERVICE_NAME, fileService);
            System.out.println("FileService đã được đăng ký với tên: " + SERVICE_NAME);
            
            System.out.println("File Service Server đang chạy...");
            Thread.currentThread().join();
            
        } catch (RemoteException e) {
            System.err.println("Lỗi RMI: " + e.getMessage());
        } catch (AlreadyBoundException e) {
            System.err.println("Service đã được đăng ký: " + e.getMessage());
        } catch (InterruptedException e) {
            System.out.println("Server đã dừng.");
        }
    }
}
```

### File Service Client

```java
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.NotBoundException;
import java.util.List;
import java.util.Scanner;

public class FileServiceClient {
    private static final String SERVER_HOST = "localhost";
    private static final int REGISTRY_PORT = 1099;
    private static final String SERVICE_NAME = "FileService";
    
    public static void main(String[] args) {
        try {
            // Kết nối đến RMI Registry
            Registry registry = LocateRegistry.getRegistry(SERVER_HOST, REGISTRY_PORT);
            FileService fileService = (FileService) registry.lookup(SERVICE_NAME);
            
            System.out.println("Đã kết nối đến File Service");
            
            Scanner scanner = new Scanner(System.in);
            boolean running = true;
            
            while (running) {
                System.out.println("\n=== FILE SERVICE CLIENT ===");
                System.out.println("1. Liệt kê file");
                System.out.println("2. Đọc file");
                System.out.println("3. Ghi file");
                System.out.println("4. Xóa file");
                System.out.println("5. Kiểm tra file tồn tại");
                System.out.println("6. Thoát");
                System.out.print("Chọn chức năng (1-6): ");
                
                int choice = scanner.nextInt();
                scanner.nextLine(); // Consume newline
                
                switch (choice) {
                    case 1:
                        listFiles(fileService);
                        break;
                    case 2:
                        readFile(fileService, scanner);
                        break;
                    case 3:
                        writeFile(fileService, scanner);
                        break;
                    case 4:
                        deleteFile(fileService, scanner);
                        break;
                    case 5:
                        checkFileExists(fileService, scanner);
                        break;
                    case 6:
                        running = false;
                        break;
                    default:
                        System.out.println("Lựa chọn không hợp lệ!");
                }
            }
            
            scanner.close();
            System.out.println("Client đã thoát.");
            
        } catch (RemoteException e) {
            System.err.println("Lỗi RMI: " + e.getMessage());
        } catch (NotBoundException e) {
            System.err.println("Không tìm thấy service: " + e.getMessage());
        }
    }
    
    private static void listFiles(FileService fileService) {
        try {
            List<String> files = fileService.listFiles();
            if (files.isEmpty()) {
                System.out.println("Không có file nào.");
            } else {
                System.out.println("Danh sách file:");
                for (String fileName : files) {
                    System.out.println("- " + fileName);
                }
            }
        } catch (RemoteException e) {
            System.err.println("Lỗi liệt kê file: " + e.getMessage());
        }
    }
    
    private static void readFile(FileService fileService, Scanner scanner) {
        try {
            System.out.print("Nhập tên file: ");
            String fileName = scanner.nextLine();
            
            String content = fileService.readFile(fileName);
            System.out.println("Nội dung file:");
            System.out.println(content);
            
        } catch (RemoteException e) {
            System.err.println("Lỗi đọc file: " + e.getMessage());
        }
    }
    
    private static void writeFile(FileService fileService, Scanner scanner) {
        try {
            System.out.print("Nhập tên file: ");
            String fileName = scanner.nextLine();
            System.out.print("Nhập nội dung: ");
            String content = scanner.nextLine();
            
            boolean success = fileService.writeFile(fileName, content);
            if (success) {
                System.out.println("Đã ghi file thành công!");
            } else {
                System.out.println("Lỗi ghi file!");
            }
            
        } catch (RemoteException e) {
            System.err.println("Lỗi ghi file: " + e.getMessage());
        }
    }
    
    private static void deleteFile(FileService fileService, Scanner scanner) {
        try {
            System.out.print("Nhập tên file cần xóa: ");
            String fileName = scanner.nextLine();
            
            boolean success = fileService.deleteFile(fileName);
            if (success) {
                System.out.println("Đã xóa file thành công!");
            } else {
                System.out.println("Lỗi xóa file!");
            }
            
        } catch (RemoteException e) {
            System.err.println("Lỗi xóa file: " + e.getMessage());
        }
    }
    
    private static void checkFileExists(FileService fileService, Scanner scanner) {
        try {
            System.out.print("Nhập tên file: ");
            String fileName = scanner.nextLine();
            
            boolean exists = fileService.fileExists(fileName);
            System.out.println("File " + fileName + " " + (exists ? "tồn tại" : "không tồn tại"));
            
        } catch (RemoteException e) {
            System.err.println("Lỗi kiểm tra file: " + e.getMessage());
        }
    }
}
```

## Best Practices cho RMI

### 1. Security
```java
// Tạo security policy
grant {
    permission java.security.AllPermission;
};

// Chạy với security manager
java -Djava.security.manager -Djava.security.policy=policy.policy CalculatorServer
```

### 2. Error Handling
```java
try {
    double result = calculator.divide(a, b);
} catch (RemoteException e) {
    // Xử lý lỗi network
    System.err.println("Lỗi kết nối: " + e.getMessage());
} catch (Exception e) {
    // Xử lý lỗi business logic
    System.err.println("Lỗi tính toán: " + e.getMessage());
}
```

### 3. Connection Pooling
```java
public class RMIConnectionPool {
    private static final int MAX_CONNECTIONS = 10;
    private Queue<CalculatorService> connections = new LinkedList<>();
    
    public CalculatorService getConnection() throws RemoteException {
        if (connections.isEmpty()) {
            Registry registry = LocateRegistry.getRegistry("localhost", 1099);
            return (CalculatorService) registry.lookup("CalculatorService");
        }
        return connections.poll();
    }
    
    public void returnConnection(CalculatorService service) {
        if (connections.size() < MAX_CONNECTIONS) {
            connections.offer(service);
        }
    }
}
```

## So sánh RMI với các công nghệ khác

| Công nghệ | Ưu điểm | Nhược điểm | Phù hợp |
|-----------|----------|------------|---------|
| **RMI** | Đơn giản, tích hợp Java | Chỉ hỗ trợ Java | Ứng dụng Java thuần |
| **REST** | Đa ngôn ngữ, HTTP | Chậm hơn, stateless | Web services |
| **gRPC** | Nhanh, đa ngôn ngữ | Phức tạp setup | Microservices |
| **WebSocket** | Real-time, 2-way | Phức tạp quản lý | Real-time apps |

## Kết luận

Java RMI là một công nghệ mạnh mẽ cho việc xây dựng các ứng dụng phân tán trong Java. Mặc dù có một số hạn chế về bảo mật và hiệu suất, RMI vẫn là lựa chọn tốt cho các ứng dụng Java thuần cần giao tiếp giữa các JVM.

Khi sử dụng RMI, hãy nhớ:
- Luôn xử lý `RemoteException`
- Cấu hình security policy phù hợp
- Sử dụng connection pooling cho hiệu suất cao
- Test kỹ các trường hợp lỗi network

Với những kiến thức này, bạn đã có thể xây dựng các ứng dụng phân tán mạnh mẽ sử dụng Java RMI!
