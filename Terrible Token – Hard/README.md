## 🧩 Terrible Token – Hard (Reverse Engineering)
Reverse engineer this challenge to find the correct       license key.



## 📌 Thông tin chung

Tên bài: Terrible Token (Hard)

Nền tảng: pwn.college

Thể loại: Reverse Engineering

Độ khó: Hard

Mục tiêu: Tìm license key hợp lệ để đọc flag

## 🔍 1. Recon ban đầu
Kiểm tra loại file
file terrible-token-hard

-Kết quả:
setuid ELF 64-bit LSB pie executable, x86-64, stripped

-Nhận xét:
+Binary bị stripped → không có symbol
+Có PIE → địa chỉ runtime thay đổi
+Có SUID bit → phải chạy trực tiếp binary để đọc /flag
+Không thể disassemble main trực tiếp

➡️ Reverse sẽ phải dựa vào runtime analysis (GDB).

## 🧵 2. Phân tích strings
strings terrible-token-hard
-Các chuỗi quan trọng:
Ready to receive your license key!
Checking the received license key!
Wrong! No flag for you!
You win! Here is your flag:
puobq
mgUa

-Nhận xét:
+puobq, mgUa là các chuỗi không có nghĩa
+Rất giống license key hoặc dữ liệu so sánh
+Ngoài ra thấy các hàm: read, memcmp


➡️ Gợi ý : chương trình đọc input rồi so sánh bằng memcmp.

## 🧠 3. Chiến lược reverse

Dump binary:

stripped

PIE

không có symbol main

➡️ Không thể reverse tĩnh hiệu quả.

Chiến lược đúng

Bám theo các hàm libc:

read

memcmp

Dùng GDB breakpoint để xem dữ liệu so sánh tại runtime

🐞 4. Debug với GDB
gdb ./terrible-token-hard

Đặt breakpoint quan trọng
break read
break memcmp
run

## ⚠️ 5. Lỗi dễ mắc: nhập input trong GDB

Khi chương trình dừng tại:

Breakpoint, __GI___libc_read


❌ Không được nhập input tại prompt (gdb).

Cách đúng
continue


Sau đó terminal sẽ chờ stdin, lúc này mới nhập:

AAAA


➡️ Đây là lỗi rất phổ biến khi debug chương trình đọc stdin.

## 🔬 6. Phân tích tại breakpoint memcmp

Chương trình dừng tại:

Breakpoint, __memcmp_sse4_1

ABI x86-64 của memcmp
memcmp(rdi, rsi, rdx)


Dump các thanh ghi:

p $rdi
p $rsi
p $rdx


Kết quả:

$rdx = 5


➡️ So sánh 5 byte.

Dump dữ liệu raw
x/16bx $rdi
x/16bx $rsi


Kết quả:

$rdi: 41 41 41 41 0a    → "AAAA\n"
$rsi: 70 75 6f 62 71    → "puobq"


Dump dạng chuỗi:

x/s $rdi
x/s $rsi

"AAAA\n"
"puobq"

## h2🧠 7. Kết luận

Chương trình không biến đổi input

Input được so sánh trực tiếp với chuỗi "puobq" bằng memcmp

Không có XOR, Caesar hay hashing

Pseudo-code
char buf[5];
read(0, buf, 5);

if (memcmp(buf, "puobq", 5) == 0)
    win();
else
    lose();

=> License key hợp lệ: puobq


## 🚀 9. Lấy flag

Thoát GDB và chạy trực tiếp binary SUID:

./terrible-token-hard


Nhập license key: puobq

Kết quả:
You win! Here is your flag:
pwn.college{gQ4jpmUfAPezgSateZustht_47Q.dJTNywyN1czM1EzW}

## ✅ Tổng kết

Bài “hard” về mặt kỹ thuật debug, không hard về thuật toán

Trọng tâm:

Hiểu stdin trong GDB

Bắt đúng breakpoint

Reverse hoàn toàn bằng dynamic analysis