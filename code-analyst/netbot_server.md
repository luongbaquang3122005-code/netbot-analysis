# Phân tích netbot_server.py – Command & Control Server (Tiếng Việt)

> **File:** `netbot_server.py`  
> **Vai trò:** Command & Control (C2) server – điều khiển bot  
> **Dự án:** NetBot v1.0  
> **Mục đích:** Phân tích mã nguồn phục vụ học tập & nghiên cứu botnet PoC

---

## 1. Tổng quan

`netbot_server.py` là trung tâm điều khiển của hệ thống NetBot (C2 – Command & Control).  
Server này:

- Lắng nghe kết nối từ các bot.
- Nhận heartbeat từ bot.
- Reload `netbot_config.py` mỗi lần bot gửi heartbeat.
- Gửi lệnh điều khiển: `LAUNCH`, `HALT`, `HOLD`, `UPDATE`.
- Quản lý số bot đang online bằng biến global `connected`.

Thiết kế đơn giản phục vụ nghiên cứu.

---

## 2. Imports và banner

```python
import socket
import threading
from termcolor import colored
from importlib import reload
```

### Ý nghĩa:
- `socket`: dùng TCP server.  
- `threading`: mỗi bot dùng một thread riêng.  
- `reload`: reload file config mỗi lần bot hỏi lệnh.  
- `colored`: gần như không dùng, thay vào đó dùng mã màu ANSI.

Banner ASCII là phần hiển thị, không ảnh hưởng logic.

---

## 3. Hàm config() – Reload cấu hình

```python
def config():
    import netbot_config
    netbot_config = reload(netbot_config)
    return netbot_config.ATTACK_STATUS
```

### Chức năng:
- Import module `netbot_config`.
- `reload()` → đọc lại file từ ổ đĩa.
- Trả về biến `ATTACK_STATUS`.

→ Đây là **cơ chế hot reload**: chỉnh file `netbot_config.py` là bot nhận lệnh mới ngay lập tức, không cần restart server.

---

## 4. Xử lý từng bot – threaded(c)

```python
def threaded(c):
    while True:
        data = c.recv(1024)
        if not data:
            global connected
            connected -= 1
            print("Bot Offline")
            break
        c.send(config().encode())
```

### Giải thích:
- Nhận dữ liệu từ bot (heartbeat).
- Nếu không có dữ liệu → bot offline.
- Ngược lại, C2 gửi lại `ATTACK_STATUS`.

### Điểm quan trọng:
- Không phân tích nội dung gói tin.
- `connected` không có lock → race condition.
- Socket không được đóng thủ công.
- Không xử lý exception.

---

## 5. Hàm Main() – TCP Server Loop

```python
def Main():
    host = "0.0.0.0"
    port = 5555
    global connected
    connected = 0

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind((host, port))
    s.listen(50)

    while True:
        c, addr = s.accept()
        connected += 1
        print("Bot Online:", addr, "Total:", connected)
        threading.Thread(target=threaded, args=(c,)).start()
```

C2 server chạy trên port 5555, cho phép tối đa **50** kết nối chờ, và tạo thread cho mỗi bot.

---

## 6. Nhận xét bảo mật

| Vấn đề | Ý nghĩa |
|-------|---------|
| Không mã hóa | Lệnh ATTACK_STATUS gửi plaintext |
| Không xác thực | Bất kỳ client nào kết nối đều được xem là bot |
| Không handle lỗi | Dễ crash nếu bot gửi dữ liệu bất thường |
| shared global không lock | Dẫn đến race condition |
| Không đóng socket | Có thể gây leak tài nguyên |

Đây là các hạn chế cố ý để giữ code đơn giản, phù hợp mục đích PoC.

---

## 7. Kết luận

`netbot_server.py` là một **C2 server tối giản**, dùng TCP + threading + reload config để điều khiển bot trong demo DDoS.  
Dễ đọc, dễ phân tích, rất phù hợp làm tài liệu nghiên cứu C2/Botnet cấu trúc đơn giản.
