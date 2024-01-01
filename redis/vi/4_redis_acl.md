[Back to home](../../README.md)

# Phần 4: Access List Redis (ACL tính năng mới ở bản 6 + 7)

ACL trong Redis giống với gán Role quyền trong Database vậy. Đôi khi 1 client kết nối mà chọc được toàn bộ dữ liệu Redis vẫn là quá rủi ro về bảo mật (nhẹ thì bị xóa trắng, mà nặng thì public data ngoài internet). Để enable ACL, nếu hệ thống mới ta sẽ extend quyền dần, nhưng hệ thống cũ đã chạy - bạn cần nắm rõ code đang chạy loại dữ liệu gì để giới hạn quyền thật chuẩn, tránh việc làm lỗi hệ thống. (sử dụng lệnh MONITOR đến giám sát code đang gọi redis như nào và nên trao đổi với DEV Team).

Mặc định Redis luôn có user là default và không có mật khẩu:

```bash
127.0.0.1:6379> ACL LIST
1) "user default on nopass ~* &* +@all"
```

Để đặt password cho user default này ta làm như sau:

```bash
127.0.0.1:6379> ACL SETUSER default on >matkhau_default
OK
127.0.0.1:6379> ACL LIST
1) "user default on #231d539d073fe5d91a24a56df3196aab2ace130b905d39011840923f9a893ddb ~* &* +@all"
127.0.0.1:6379> CONFIG REWRITE
OK
```

[Back to home](../../README.md)