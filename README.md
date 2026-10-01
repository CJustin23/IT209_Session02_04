# Báo cáo Bài 4: Cấu hình Tường Lửa Bảo Vệ Máy Chủ (UFW & Cloud Firewall Integration)

## 1. Mục tiêu và Yêu cầu

**Mục tiêu:**
- Thiết lập **UFW (Uncomplicated Firewall)** trên Ubuntu để lọc gói tin ở tầng hệ điều hành.
- Kết hợp **Cloud Firewall** ở tầng hạ tầng để bảo vệ máy chủ từ biên mạng.
- Kết quả: chỉ **SSH (cổng 22)** và **HTTP (cổng 80)** được phép đi vào; tất cả cổng khác bị chặn hoàn toàn.

**Ràng buộc:**

| Cổng | Giao thức | Hành động |
|------|-----------|-----------|
| 22   | TCP       | ALLOW IN (SSH) |
| 80   | TCP       | ALLOW IN (HTTP) |
| Khác | Tất cả    | DENY (chặn hoàn toàn) |

---

## 2. Các bước thực hiện

### Bước 1: Kiểm tra trạng thái UFW ban đầu

```bash
sudo ufw status
```

Output ban đầu (UFW chưa được kích hoạt):

```
Status: inactive
```

### Bước 2: Thiết lập chính sách mặc định

```bash
# Chặn toàn bộ lưu lượng đi vào theo mặc định
sudo ufw default deny incoming

# Cho phép toàn bộ lưu lượng đi ra theo mặc định
sudo ufw default allow outgoing
```

### Bước 3: Mở cổng cần thiết

```bash
# Mở cổng SSH (phải làm TRƯỚC khi enable để không bị mất kết nối)
sudo ufw allow 22/tcp

# Mở cổng HTTP
sudo ufw allow 80/tcp
```

### Bước 4: Kích hoạt UFW

```bash
sudo ufw enable
# Nhập 'y' để xác nhận
```

---

## 3. Kiểm tra kết quả (Verification)

### 3.1. Kiểm tra trạng thái UFW

```bash
sudo ufw status verbose
```

Kết quả đầu ra thực tế:

```
root@azvps-tm7f1n6r4:~# sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)
```

**Nhận xét:**
- UFW đang hoạt động (`Status: active`).
- Chính sách mặc định: `deny (incoming)` – chặn tất cả traffic vào.
- Chỉ cho phép đúng 2 cổng: `22/tcp` (SSH) và `80/tcp` (HTTP).

### 3.2. Kiểm tra kết nối SSH thành công

```bash
ssh root@160.250.246.219
```

Kết quả đầu ra thực tế:

```
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-198-generic x86_64)
...
root@azvps-tm7f1n6r4:~#
```

**Nhận xét:** Kết nối SSH thành công – cổng 22 hoạt động bình thường sau khi bật UFW.

### 3.3. Kiểm tra cổng khác bị chặn

Kiểm tra cổng 3306 (MySQL) từ máy cá nhân:

```powershell
Test-NetConnection -ComputerName 160.250.246.219 -Port 3306
```

Kết quả đầu ra thực tế:

```
ComputerName     : 160.250.246.219
RemoteAddress    : 160.250.246.219
RemotePort       : 3306
TcpTestSucceeded : False
```

**Kết luận:**
- UFW đã kích hoạt thành công với chính sách `default deny incoming`.
- Cổng 22 (SSH) và 80 (HTTP) được phép đi vào, tất cả cổng khác bị chặn hoàn toàn.
- Bài thực hành hoàn thành đúng yêu cầu đề ra.
