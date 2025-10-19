# Blog Cá Nhân - Nguyễn Thị Thu Thủy

## 🎯 Giới thiệu
Blog cá nhân được xây dựng bằng Hugo với giao diện hiện đại, chuyên nghiệp và responsive.

## 🚀 Tính năng
- ✅ **Giao diện hiện đại**: Glass morphism với gradient xanh dương
- ✅ **Responsive design**: Hoạt động tốt trên mọi thiết bị
- ✅ **Code highlighting**: Syntax highlighting cho code blocks
- ✅ **Profile page**: Trang giới thiệu với layout đẹp
- ✅ **Blog posts**: Bài viết về JavaScript và lập trình
- ✅ **Navigation**: Menu điều hướng dễ sử dụng

## 📁 Cấu trúc dự án
```
blog/
├── content/
│   ├── posts/           # Bài viết blog
│   └── profile.md       # Trang giới thiệu
├── layouts/             # Templates Hugo
├── static/
│   ├── css/
│   │   └── custom.css   # CSS chính
│   └── images/          # Hình ảnh
├── themes/              # Theme Hugo
└── config.toml          # Cấu hình Hugo
```

## 🎨 Giao diện
- **Màu chủ đạo**: Gradient xanh dương (#1e3a8a → #60a5fa)
- **Font**: Inter (Google Fonts)
- **Effect**: Glass morphism với backdrop-filter
- **Code blocks**: Nền đen với syntax highlighting
- **Responsive**: Mobile-first design

## 📝 Nội dung
### Bài viết hiện có:
1. **JavaScript Networking Cơ Bản** - Fetch API và WebSocket
2. **JavaScript Essentials - Advanced Functions** - Closures và Functional Programming
3. **JavaScript Essentials - Objects & Prototypes** - OOP trong JavaScript
4. **JavaScript Essentials - Async Programming** - Promises và async/await

### Trang giới thiệu:
- Thông tin cá nhân với hình ảnh
- Kỹ năng và kinh nghiệm
- Dự án và thành tích
- Thông tin liên hệ

## 🛠️ Cài đặt và chạy
1. **Cài đặt Hugo**:
   ```bash
   # Windows (với Chocolatey)
   choco install hugo
   
   # Hoặc tải từ: https://github.com/gohugoio/hugo/releases
   ```

2. **Chạy development server**:
   ```bash
   cd blog
   hugo server --buildDrafts
   ```

3. **Build production**:
   ```bash
   hugo --minify
   ```

## 📸 Thêm hình ảnh profile
1. Đặt file hình vào `static/images/`
2. Đổi tên thành `profile_image.jpg`
3. Refresh trang web

## 🎯 Cấu hình
- **Site title**: Được cấu hình trong `config.toml`
- **Theme**: Sử dụng theme Ananke với custom CSS
- **Deployment**: Có thể deploy lên Netlify, Vercel, GitHub Pages

## 📱 Responsive Breakpoints
- **Desktop**: > 768px
- **Tablet**: 768px - 480px  
- **Mobile**: < 480px

## 🔧 Customization
- **Màu sắc**: Thay đổi trong `:root` variables của CSS
- **Font**: Thay đổi font-family trong CSS
- **Layout**: Điều chỉnh spacing và sizing variables

## 📄 License
MIT License - Tự do sử dụng và chỉnh sửa.

---
**Tác giả**: Nguyễn Thị Thu Thủy  
**Email**: nguyenthithuthuyy1905@gmail.com  
**Website**: https://byvn.net/vt50
