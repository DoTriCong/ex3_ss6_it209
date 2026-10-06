# Bài 3: Cấu hình tường lửa UFW và chẩn đoán cổng mạng

## 1. Mục tiêu

- Dùng UFW (Uncomplicated Firewall) để bảo vệ máy chủ Cloud VPS.
- Chỉ mở các cổng thực sự cần thiết: **22/tcp (SSH)** và **8080/tcp (ứng dụng Web)**.
- Dùng `ss`, `curl`, `ufw status` để chẩn đoán cổng mạng.

## 2. Các bước thực hiện

### Bước 1: Đặt chính sách mặc định

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

- `deny incoming`: chặn mọi kết nối đi vào mà chưa có quy tắc cho phép.
- `allow outgoing`: cho phép máy chủ chủ động kết nối ra ngoài (cập nhật gói, gọi API...).

### Bước 2: Mở cổng SSH và cổng ứng dụng

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8080/tcp
```

> ⚠️ **Phải mở cổng 22 TRƯỚC khi bật tường lửa**, nếu không phiên SSH hiện tại sẽ bị ngắt và mất quyền quản trị máy chủ.

### Bước 3: Kích hoạt tường lửa

```bash
sudo ufw enable
```

Nhấn `y` khi có cảnh báo _"Command may disrupt existing ssh connections. Proceed with operation (y|n)?"_.

### Bước 4: Kiểm tra trạng thái

```bash
sudo ufw status verbose
```

## 3. Kết quả

### 3.1. `sudo ufw status verbose`

> 📌 **Dán kết quả thực tế từ VPS của bạn vào đây** (hoặc chèn ảnh chụp màn hình). Kết quả đúng sẽ có dạng:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
8080/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
8080/tcp (v6)              ALLOW IN    Anywhere (v6)
```

### 3.2. `ss -tlnp`

```bash
ss -tlnp
```

> 📌 **Dán kết quả thực tế vào đây.** Ví dụ khi ứng dụng đã chạy ở cổng 8080:

```text
State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22           0.0.0.0:*          users:(("sshd",pid=...,fd=3))
LISTEN  0       511     0.0.0.0:8080         0.0.0.0:*          users:(("<app>",pid=...,fd=...))
```

## 4. Phân tích

| Tiêu chí             | Kết quả mong đợi                    | Đạt |
| -------------------- | ----------------------------------- | --- |
| Trạng thái tường lửa | `Status: active`                    | ☐   |
| Chính sách mặc định  | `deny (incoming), allow (outgoing)` | ☐   |
| Cổng 22              | `22/tcp ALLOW IN Anywhere`          | ☐   |
| Cổng 8080            | `8080/tcp ALLOW IN Anywhere`        | ☐   |

**Lưu ý quan trọng:** UFW chỉ quyết định cổng nào được **phép** truy cập từ bên ngoài, còn `ss -tlnp` cho biết cổng nào **thực sự có tiến trình đang lắng nghe**. Một cổng cần cả hai điều kiện (tường lửa mở + ứng dụng lắng nghe) thì mới kết nối được. Ứng dụng nên bind vào `0.0.0.0:8080` (không phải `127.0.0.1`) nếu muốn truy cập từ bên ngoài.

## 5. Kiểm tra kết nối bằng curl (tuỳ chọn)

```bash
# Trên chính VPS
curl -I http://localhost:8080

# Từ máy khác (thay <IP_VPS> bằng IP của máy chủ)
curl -I http://<IP_VPS>:8080
```

## 6. Một số lệnh hữu ích khác

```bash
sudo ufw status numbered      # xem quy tắc kèm số thứ tự
sudo ufw delete <số>          # xoá quy tắc theo số
sudo ufw disable              # tắt tường lửa
sudo ufw reset                # xoá toàn bộ quy tắc
```

## 7. Kết luận

Tường lửa UFW đã được cấu hình theo nguyên tắc **"chặn tất cả, chỉ mở cái cần thiết"**: chặn toàn bộ kết nối đi vào, cho phép kết nối đi ra, chỉ mở SSH (22/tcp) để quản trị và 8080/tcp để phục vụ ứng dụng web, giúp giảm bề mặt tấn công của máy chủ.
