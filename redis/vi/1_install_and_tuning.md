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

Khi make install, câu lệnh sẽ thực hiện copy các file chạy redis gồm (redis-server , redis-benchmark , redis-cli ) vào thư mục /usr/local/bin/

Bước 2: Cài đặt các thư mục riêng cho redis, để quản lý về sau ( như: data , conf, log)

```bash
mkdir -p /opt/redis/conf

mkdir -p /opt/redis/data

mkdir -p /opt/redis/log

cd /opt/setup/redis-7.0.8/utils/

vim install_server.sh (ta comment lại dòng 82 vì có exit 1 không tương thích với systemd - lỗi ko cần thiết)
```

Thực hiện chạy file, và khai báo như sau

```bash
$ ./install_server.sh
```