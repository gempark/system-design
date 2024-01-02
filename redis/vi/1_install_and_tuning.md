[Back to home](../../README.md)

# Phần 1: Cài đặt redis cơ bản + Tuning redis.

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
./install_server.sh
```

Tắt bật redis:

```bash
/etc/init.d/redis_6379 status
/etc/init.d/redis_6379 stop
/etc/init.d/redis_6379 start
# Bạn nào muốn dùng systemd, Đọc. Phần 3: Bảo mật cho redis. (redis security) nhé.
```

Lệnh chạy cơ bản vào redis bằng lệnh

```bash
redis-cli ping                           # (Dùng để kiểm tra redis alive)
redis-cli set hello-world awesome1       # (đặt key hello-world có data value = awesome)
redis-cli get hello-world                # (lấy giá trị value của key ra)
redis-cli info                           # (xem thông tin status của redis)
redis-cli config get                     # (Xem toàn bộ config của redis đang chạy)
redis-cli monitor                        # (GIÁM SÁT các câu lệnh đang chạy real-time - RẤT hay)
redis-cli client list                    # (Kiểm tra xem có bao nhiêu client, ip client đang kết nối, client đang dùng lệnh gì.)
redis-cli info clients                   # (Xem số client đang kết nối tới redis server)
```

Ngoài việc gõ trực tiếp command trên unix. Ta có thể vào redis-cli để gõ các lệnh như sau:

```bash
[root@master-node ~]# redis-cli 
127.0.0.1:6379> PING
PONG
127.0.0.1:6379> SET hello-world awmsome2
OK
127.0.0.1:6379> GET hello-world
"awmsome2"
```

## 1.2 Tuning Redis:

### P1: Enable vm.overcommit_memory

```bash
vim /etc/sysctl.conf
# vm.overcommit_memory=1
# net.core.somaxconn=65535
# sudo sysctl --system
```

chạy lệnh sysctl vm.overcommit_memory=1 Khi ghi dữ liệu từ RAM xuống disk, Redis sử dụng các tiến trình con. Theo mặc định, tiến trình con này cần memory tương đương tiến trình mẹ, do nó có chứa dữ liệu tương đương. Điều này dẫn tới OS không đủ RAM cho việc ghi DB xuống Disk này > lỗi. Tuy nhiên Linux có hỗ trợ Copy-on-write, nghĩa là chỉ cần cấp memory cho những gì tiến trình con khác với tiến trình mẹ, phần giống nhau thì dùng chung memory cho tiết kiệm. Để bật nó thì cần set overcommit_memory = 1

### P2: net.core.somaxconn (tăng hàng đợi tối đa của linux)

```bash
sysctl -w net.core.somaxconn=65535
cat /proc/sys/net/core/somaxconn
65535
```

### P3: Set never transparent_hugepage

Giải thích: https://kipalog.com/posts/Tinh-nang-Transparent-HugePage-trong-RHEL6-va-anh-huong

```bash
# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
# echo never > /sys/kernel/mm/transparent_hugepage/enabled


# vim /etc/rc.local (thêm vào cuối file rc.local để mỗi lần khởi động lại Linux, hugpage lại chuyển thành never)
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

### P4: Tăng connection kết nối

Mặc định redis seting maxclients kết nối đồng thời là 10000, nếu coder tốt thì khi mở kết nối xong sẽ gọi lệnh close (cả hệ thống lớn chắc ko đến 100 kết nối đồng thời). Vẫn để phòng trường hợp quá connection khi bị miss code ở đoạn nào đó. Ta có thể tăng Maxclients lên.

Cách 1: Thay trong redis-cli

```bash
127.0.0.1:6379> CONFIG GET MAXCLIENTS
"10000"
127.0.0.1:6379> CONFIG SET MAXCLIENTS 100000
OK # (thực hiện tăng maxclient tạm thời)
127.0.0.1:6379> CONFIG REWRITE
OK # (Thực hiện ghi đè config hiện tại / vĩnh viễn)
```

Cách 2: Thay trong config

```bash
vim /opt/redis/conf/6379.conf
maxclients 100000
```

[Back to home](../../README.md)