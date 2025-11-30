# Phân tích netbot_config.py – File cấu hình điều khiển bot (Tiếng Việt)

> **File:** `netbot_config.py`  
> **Vai trò:** Lệnh điều khiển bot dưới dạng chuỗi ATTACK_STATUS  
> **Dự án:** NetBot v1.0

---

## 1. Mục đích của file

File này đóng vai trò như **“lệnh từ xa”** do C2 gửi cho bot.  
Mỗi lần bot gửi HEARTBEAT, C2 sẽ:

1. Reload file này  
2. Lấy biến `ATTACK_STATUS`  
3. Gửi cho bot  

Vì vậy, thay đổi file = thay đổi behavior của TẤT CẢ bot ngay lập tức.

Đây là dạng **remote control bằng text command** đơn giản.

---

## 2. Cấu trúc ATTACK_STATUS

Ví dụ:

```python
ATTACK_STATUS = "192.168.0.10_80_LAUNCH_HTTPFLOOD_1"
```

Sau khi bot `split('_')`, ta được:

| Vị trí | Giá trị        | Ý nghĩa                                  |
|-------|----------------|-------------------------------------------|
| `[0]` | 192.168.0.10   | Target IP                                |
| `[1]` | 80             | Target Port                              |
| `[2]` | LAUNCH         | Command (LAUNCH / HALT / HOLD / UPDATE)  |
| `[3]` | HTTPFLOOD      | Loại tấn công                            |
| `[4]` | 1              | Delay (giây) trong HTTPFLOOD             |

---

## 3. Các chế độ điển hình

### 🔹 Khởi chạy HTTP Flood

```python
ATTACK_STATUS = "10.0.0.5_80_LAUNCH_HTTPFLOOD_1"
```

### 🔹 Khởi chạy Ping Flood

```python
ATTACK_STATUS = "10.0.0.5_443_LAUNCH_PINGFLOOD_0"
```

### 🔹 Dừng tấn công (HALT)

```python
ATTACK_STATUS = "0_0_HALT_NONE_0"
```

### 🔹 Tạm dừng (HOLD)

```python
ATTACK_STATUS = "0_0_HOLD_NONE_0"
```

### 🔹 Yêu cầu tự cập nhật (UPDATE)

```python
ATTACK_STATUS = "0_0_UPDATE_NONE_0"
```

---

## 4. Nhận xét

- Format chuỗi đơn giản, dễ sửa bằng editor.  
- Không có validate → sai format có thể làm bot crash.  
- Là ví dụ rõ ràng về command & control bằng chuỗi text.  

---

## 5. Kết luận

`netbot_config.py` là **trái tim điều khiển** của NetBot PoC:  
C2 chỉ cần chỉnh một dòng `ATTACK_STATUS` là có thể:

- Chọn target  
- Chọn port  
- Chọn kiểu tấn công  
- Ra lệnh cho toàn bộ bot LAUNCH / HALT / HOLD / UPDATE.
