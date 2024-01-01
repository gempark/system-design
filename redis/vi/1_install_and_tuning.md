# Phần 1: Cài đặt redis cơ bản + Turning redis.

## 1.1 Cài đặt:

(Sao không cài yum hoặc apt mà lại cài từ tar.gz: Vì dễ quản lý file và config về sau, thuận tiện cho việc backup data file/config/logs/quản lý phân vùng lưu trữ, upgrade version...)

Bước 1: Tải redis từ trang chủ: https://redis.io/download . các bạn cứ chọn bản mới nhất mà cài. Vì redis build rất ổn định (update 7.0.8 - 2023-01-20).

```bash
# Bản mới nhất (2023-01) 
yum install gcc wget systemd-devel -y
mkdir -p /opt/setup
cd /opt/setup
wget https://github.com/redis/redis/archive/7.0.8.tar.gz
tar -xvzf 7.0.8.tar.gz
cd redis-7.0.8
make
# Khi make install, các file chạy binary sẽ được copy sang thư mục /usr/local/bin/
make install
```