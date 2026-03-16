# Meager Mangler (Hard)
## 1. Thông tin thử thách
Tên bài: meager-mangler-hard

Mục tiêu: Tìm License Key đúng để đọc file flag.

Đặc điểm: File binary bị stripped (mất symbol), đầu vào bị xáo trộn (mangling) qua nhiều giai đoạn phức tạp trước khi so sánh.
## 2. Phân tích (Reversing)
Tìm kiếm điểm bắt đầu
Vì binary bị stripped, tôi sử dụng gdb và đặt breakpoint tại hàm memcmp (nơi chương trình so sánh input đã qua xử lý với key đúng).

Bash
gdb ./meager-mangler-hard
(gdb) break memcmp
(gdb) run
Thu thập dữ liệu mục tiêu
Khi chương trình dừng lại tại memcmp, tôi kiểm tra hai thanh ghi $rdi (input đã mangled) và $rsi (địa chỉ key đúng trong bộ nhớ).

Target Key (tại 0x4010): 0x71 0x73 0x4b 0xeb 0xe5 0xd8 0x7f 0x6c 0x49 0xed 0xfa 0xd6 0x60 0x77 0x47 0x72 0x69 0xc0 (18 bytes).

Phân tích mã máy (Disassembly)
Sử dụng lệnh disas tại hàm gọi memcmp, tôi phát hiện logic xáo trộn gồm 3 giai đoạn:

### Giai đoạn 1 (Modulo 3 XOR): Duyệt qua chuỗi, XOR từng byte với các key khác nhau dựa trên index % 3.

index % 3 == 0: XOR 0xc7

index % 3 == 1: XOR 0x53

index % 3 == 2: XOR 0xfb

### Giai đoạn 2 (Swap): Hoán đổi vị trí hai byte tại index 15 và 16.

### Giai đoạn 3 (Modulo 2 XOR): Duyệt lại chuỗi, XOR dựa trên index % 2.

index % 2 == 0: XOR 0xde

index % 2 == 1: XOR 0x48
## 3. Khôi phục License Key
Vì phép XOR có tính chất đảo ngược ($A \oplus B \oplus B = A$), ta chỉ cần thực hiện ngược lại các bước trên với Target Key thu được.

Script giải mã (Python))
goal = [
    0x71, 0x73, 0x4b, 0xeb, 0xe5, 0xd8, 0x7f, 0x6c, 
    0x49, 0xed, 0xfa, 0xd6, 0x60, 0x77, 0x47, 0x72,
    0x69, 0xc0
]
for i in range(18):
  goal[i] ^= 0xde if i % 2 == 0 else 0x48
  goal[15], goal[16] = goal[16], goal[15]
keys = [0xc7, 0x53, 0xfb]
for i in range(18):
    goal[i] ^= keys[i % 3]
key = "".join(chr(b & 0xFF) for b in goal)
print(f"License Key: {key}")

## 4. Kết quả
Nhập Key tìm được vào chương trình: Key: hhndhkfwlbweylbpis

Flag: pwn.college{IRfUrk_BaYlcnmD6ob2103yRaX7.dJjNywyN1czM1EzW}

## 5. Bài học rút ra
Luôn kiểm tra độ dài thực tế mà chương trình đọc vào (read syscall).

Khi gặp binary stripped, bt (backtrace) và xem thanh ghi tại các hàm thư viện (memcmp, strcmp) là chìa khóa để tìm logic xử lý.

Logic xáo trộn phức tạp đến đâu cũng có thể giải ngược nếu nó là các phép toán thuận nghịch (XOR, Swap, Add/Sub).