# myspace7.6_linux
eploying DSpace 7.0 on Linux (Ubuntu) using Docker Compose. Includes configurations for PostgreSQL, Solr, and automated CSV metadata import workflow.
DSpace 7.0 Manual Deployment on Linux (From Source)

  Tổng quan dự án (Overview)
Kho lưu trữ này cung cấp tài liệu và file cấu hình cho quá trình biên dịch và triển khai thủ công (Manual Build) hệ thống thư viện số DSpace 7.6 trực tiếp trên môi trường hệ điều hành Linux (Ubuntu). 

Thay vì sử dụng Docker đóng gói sẵn, dự án này hướng dẫn cách build mã nguồn gốc (Source Code) thành các bản phân phối thực thi, giúp lập trình viên kiểm soát hoàn toàn cấu trúc hệ thống, dễ dàng can thiệp vào giao diện Angular và tùy biến API backend.

Yêu cầu Hệ thống (
Để biên dịch hệ thống, máy chủ Linux cần cài đặt sẵn các thành phần lõi sau:
Backend: Java (JDK 11), Apache Maven (3.3+), Apache Ant (1.10+).
Frontend: Node.js (v14 hoặc v16), Yarn.
Lưu trữ & Tìm kiếm PostgreSQL (cùng thư viện `pgcrypto`), Apache Solr (8.x).
Web Server: Apache Tomcat (9.x).

Quy trình Biên dịch & Triển khai (Build & Deploy Flow)

 Khởi tạo Cơ sở dữ liệu (PostgreSQL)
Tạo database và cấp quyền, đồng thời kích hoạt extension bắt buộc `pgcrypto` cho DSpace:
```sql
CREATE ROLE dspace WITH LOGIN PASSWORD 'dspace';S
CREATE DATABASE dspace WITH OWNER dspace;
\c dspace
CREATE EXTENSION pgcrypto;

Dự án này đã thực hiện các tùy biến quan trọng sau:
### 1. Tùy biến giao diện (Frontend)
- **Thư mục Theme:** `dspace-angular/src/themes/custom/`
- **Chỉnh sửa CSS:** `src/themes/custom/styles/`
- **Cấu hình Sidebar:** Đã chỉnh sửa nhãn bộ lọc tìm kiếm (ví dụ: đổi "Thời gian xuất bản" thành "Năm xuất bản") trong tệp ngôn ngữ `vi.json5`.
### 2. Xử lý Video & Media
- Đã cấu hình hỗ trợ phát video trực tuyến (Streaming) cho định dạng MP4.
- **Lưu ý:** Cần kiểm tra MIME Type trong Backend để đảm bảo trình duyệt nhận diện đúng định dạng video.
---
##  Các Script hỗ trợ (Maintenance Scripts)
Để đơn giản hóa việc quản lý, hệ thống sử dụng các script nằm trong thư mục `/scripts`:
- `start-dspace.sh`: Khởi động nhanh cả Tomcat và Frontend (SSR).
- `stop-dspace.sh`: Dừng toàn bộ các dịch vụ một cách an toàn.
- `rebuild-frontend.sh`: Xóa cache và build lại giao diện Angular khi có thay đổi.

Xử lý sự cố thường gặp (Troubleshooting)
Lỗi | Nguyên nhân | Cách khắc phục  **No video with supported format** | MIME type chưa đúng hoặc lỗi SSL | Kiểm tra tệp cấu hình `dspace.cfg` và đảm bảo Bitstream có định dạng `video/mp4`. |
Solr Core không nhận diện** | Lỗi phân quyền hoặc Solr chưa chạy | Chạy `sudo chown -R solr:solr [dspace]/solr/` và khởi động lại dịch vụ Solr. |
Lỗi Java Heap Space** | Thiếu bộ nhớ RAM cho Tomcat | Tăng tham số `-Xmx` trong tệp `setenv.sh` của Tomcat. |
Giao diện không cập nhật** | Browser/Server Cache | Chạy lệnh xóa thư mục `dist` của Angular và khởi động lại PM2. |
---
## Sao lưu và Khôi phục (Backup & Restore)
### 1. Sao lưu Database
```bash
pg_dump -U dspace -h localhost dspace > dspace_backup_$(date +%F).sql
