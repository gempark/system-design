# Phần 2: Làm quen lệnh quản trị Redis cơ bản

## MONITOR

Đây là 1 lệnh rất hay, dùng để GIÁM SÁT realtime toàn bộ hoạt động của Redis (VD trong trường hợp bạn đang không biết code đang bung lệnh gì vào Redis)

```bash
redis-cli
127.0.0.1:6379> MONITOR
1645808925.291724 [0 127.0.0.1:54180] "SET" "TUANDA" "123"
1645808928.157719 [0 127.0.0.1:54180] "GET" "TUANDA"
1645808939.043673 [0 127.0.0.1:54180] "EXPIRE" "TUANDA" "5"

# hoặc
[root@master-node ~]# redis-cli MONITOR
```

## CLIENT LIST

Giúp chúng ta liệt kê có bao nhiêu client đang kết nối vào redis-server, đang chạy command gì

```bash
127.0.0.1:6379> CLIENT LIST
id=3086 addr=192.168.88.13:35764 laddr=192.168.88.12:6379 fd=10 name= age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=40954 argv-mem=0 obl=0 oll=0 omem=0 tot-mem=61464 events=r cmd=set user=default redir=-1
```

Như trên ta thấy client có IP là 192.168.88.13:35764 đang kết nối đến server 192.168.88.12:6379 và sử dụng command cmd=SET để ghi key vào Redis.

Lệnh này rất thuận tiện để cho ta debug rằng tiến trình / code tại server nào đang gọi + giữ kết nối nhiều đến Redis server, làm tràn connection đến Redis > để báo DEV kiểm tra và fix close connection.

## INFO

Show toàn bộ status của Redis, bao gồm CPU/RAM/KEY. Các công cụ giám sát như prometheus, zabbix, nagios... đều dùng lệnh này để lấy giá trị ra giám sát:

```bash
# Clients
connected_clients:2
cluster_connections:0
maxclients:20000
# Memory
used_memory:1435904
# Replication
role:master
connected_slaves:0
# Keyspace
db0:keys=2,expires=0,avg_ttl=0
```

## SCAN thay vì dùng KEYS

Lệnh KEYS * tốt khi dùng redis với ít key-value. Nhưng khi có vài triệu key thì KEYS * không phải là lựa chọn tốt (gây ra cao tải I/O, lệnh keys bị cấm ở Prod). Thay vào đó ta có các lựa chọn khác

```bash
# Search 5000 key ngẫu nhiên
"scan" "0" "MATCH" "*" "COUNT" "500"

# Search 1 key ngẫu nhiên
RANDOMKEY

# HOẶC dùng regex
KEYS ABC*
```

## CONFIG GET / CONFIG SET

Chức Năng: Lấy thông tin cấu hình của redis đang chạy, phù hợp với sysadmin mới tiếp nhận redis, sẽ hiểu được toàn bộ cấu hình đang chạy. (Ví dụ: log nằm ở đâu, file rdb dump (dữ liệu) nằm ở đâu, thời gian bgsave như nào )

```bash
127.0.0.1:6379> CONFIG GET maxclients
"20000"
127.0.0.1:6379> CONFIG SET maxclients 100000
OK
127.0.0.1:6379> CONFIG GET *
# hoặc
[root@master-node ~]# redis-cli  CONFIG GET "*"
```

## SLOWLOG GET

Hiển thị các command chiếm I/O của redis

```bash
127.0.0.1:6379> SLOWLOG help
127.0.0.1:6379> SLOWLOG GET
```

## SELECT

Chắc ít bạn sẽ biết là redis có tận 16 vùng chứa db khác nhau. Mặc dịnh tất cả các thao tác nằm ở db0. Lệnh select giúp chúng ta chuyển Namespace (giống với k8s). Dữ liệu của mỗi namespace sẽ không liên quan tới nhau, ta có thể đặt trùng tên. Dùng để tách biệt môi trường ví dụ 2 key giống tên nhau mà có value khác nhau.

## BGSAVE / SAVE

Cập nhập mới toàn bộ data trên ram của redis vào dump.rdb file. (Phục vụ cho công việc upgrade redis version, restart redis node... để tránh mất dữ liệu trên ram)

```
- BGSAVE: 
BackGround save. Lưu toàn bộ dữ liệu thay đổi của redis được ghi đồng thời, không ảnh hưởng đến hoạt động của redis-server.
- SAVE: 
Redis sẽ LOCK toàn bộ command tác động đến dữ liệu và tiến hành lưu dữ liệu vào rdb.
```

## Backup/Clone dữ liệu redis:

Redis lưu toàn bộ dữ liệu trên file dump.rdb, để clone dữ liệu sang 1 redis khác, có 2 cách:

- Để redis mới thành slave của con đang chạy, sau đó ngắt slave và cho chạy làm Master.
- Copy offline file dump.rdb, để vào thư mục data của redis mới, start redis mới là ta đã clone thành công.

## Other, Sử dụng pipeline để export/import toàn bộ Key/Value (String) của redis.

(Chỉ sử dụng cho redis đang lưu String, dành cho ai cần đến, Hash/List/Sort/ZSET.. không áp dụng đc) Để import ta có thể chuẩn bị file mẫu như sau:

```bash
vi import.txt
SET TUANDA1 1231
SET TUANDA2 1232
EXPIRE TUANDA1 1001
EXPIRE TUANDA2 1002
[root@master-node ~]# redis-cli < import.txt
[root@master-node ~]# redis-cli GET TUANDA1
"1231"
[root@master-node ~]# redis-cli TTL TUANDA1
(integer) 971
```

Để export toàn bộ key String ta có thể làm cách sau:

```bash
[root@master-node ~]# redis-cli KEYS "*" > key_export.txt
[root@master-node ~]# cat key_export.txt 
TUANDA1
TUANDA
TUANDA2
[root@master-node ~]# sed -i -e 's/^/GET /g' key_export.txt 
[root@master-node ~]# cat key_export.txt 
GET TUANDA1
GET TUANDA
GET TUANDA2
[root@master-node ~]# redis-cli < key_export.txt > value_export.txt
[root@master-node ~]# cat value_export.txt 
1231
123
1232
[root@master-node ~]# paste key_export.txt value_export.txt 
GET TUANDA1	1231
GET TUANDA	123
GET TUANDA2	1232
```