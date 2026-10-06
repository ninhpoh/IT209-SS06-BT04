# Bài 4: Quản lý tiến trình nền với nohup và tín hiệu Kill

## 1. Mục tiêu

* Khởi chạy shell script chạy nền bằng `nohup` và `&`.
* Đảm bảo tiến trình vẫn hoạt động sau khi ngắt kết nối SSH.
* Kiểm tra tiến trình bằng `ps` và `pgrep`.
* Kiểm tra file log bằng `tail`.
* Dừng tiến trình bằng tín hiệu `SIGTERM (15)`.
* Biết cách sử dụng `SIGKILL (9)` khi cần thiết.

---

## 2. Tạo file loop-monitor.sh

File `loop-monitor.sh` có nội dung:

```bash
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

Cấp quyền thực thi:

```bash
chmod +x loop-monitor.sh
```

---

## 3. Khởi chạy tiến trình bằng nohup

Lệnh sử dụng:

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

Lệnh trên giúp script chạy dưới nền và tiếp tục hoạt động ngay cả khi phiên SSH bị ngắt.

---

## 4. Kiểm tra PID

Sử dụng lệnh:

```bash
pgrep -af loop-monitor.sh
```

Kết quả thực tế:

```text
15479 /bin/bash ./loop-monitor.sh
15565 /bin/bash ./loop-monitor.sh
```

Có 2 tiến trình `loop-monitor.sh` đang chạy.

Nguyên nhân là script đã được khởi chạy 2 lần.

PID của hai tiến trình là:

```text
15479
15565
```

---

## 5. Kiểm tra file log

Sử dụng:

```bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
System time: Tue Oct  6 10:01:51 PM +07 2026
System time: Tue Oct  6 10:01:51 PM +07 2026
System time: Tue Oct  6 10:01:56 PM +07 2026
System time: Tue Oct  6 10:01:56 PM +07 2026
System time: Tue Oct  6 10:02:01 PM +07 2026
System time: Tue Oct  6 10:02:01 PM +07 2026
System time: Tue Oct  6 10:02:06 PM +07 2026
System time: Tue Oct  6 10:02:06 PM +07 2026
System time: Tue Oct  6 10:02:11 PM +07 2026
System time: Tue Oct  6 10:02:11 PM +07 2026
```

File `/tmp/monitor.log` được cập nhật đều đặn sau mỗi 5 giây.

Do có 2 tiến trình cùng chạy nên mỗi thời điểm xuất hiện 2 dòng log giống nhau.

---

## 6. Kiểm tra sau khi đăng nhập SSH lại

Sau khi đăng xuất SSH và đăng nhập lại VPS, sử dụng:

```bash
pgrep -af loop-monitor.sh
```

Kết quả:

```text
15479 /bin/bash ./loop-monitor.sh
15565 /bin/bash ./loop-monitor.sh
```

Tiến trình vẫn tồn tại sau khi đăng nhập SSH lại.

Điều này chứng minh `nohup` đã giúp tiến trình tiếp tục chạy độc lập với phiên Terminal/SSH.

---

## 7. Dừng tiến trình bằng SIGTERM

Sử dụng tín hiệu `SIGTERM (15)` để dừng hai tiến trình:

```bash
kill -15 15479
kill -15 15565
```

Sau đó kiểm tra:

```bash
pgrep -af loop-monitor.sh
```

Nếu không còn kết quả thì các tiến trình đã được dừng thành công.

---

## 8. SIGKILL

Trong trường hợp `SIGTERM (15)` không thể dừng tiến trình, có thể sử dụng:

```bash
kill -9 <PID>
```

Ví dụ:

```bash
kill -9 15479
```

`SIGKILL (9)` là tín hiệu buộc hệ thống kết thúc tiến trình.

Trong bài thực hành này ưu tiên sử dụng `SIGTERM (15)` trước.

---

## 9. Kết luận

Đã thực hiện các yêu cầu của bài:

* Tạo shell script `loop-monitor.sh`.
* Sử dụng vòng lặp `while true`.
* Ghi thời gian hệ thống vào `/tmp/monitor.log` mỗi 5 giây.
* Cấp quyền thực thi cho script.
* Chạy script dưới nền bằng `nohup`.
* Sử dụng `pgrep` để tìm PID.
* Sử dụng `tail` để kiểm tra file log.
* Kiểm tra tiến trình sau khi đăng nhập SSH lại.
* Sử dụng `kill -15` để dừng tiến trình.
* Biết cách sử dụng `kill -9` khi `SIGTERM` không hoạt động.

## 10. Đường dẫn nộp bài

```text
homework/session_06/ex4/
├── loop-monitor.sh
└── README.md
```
