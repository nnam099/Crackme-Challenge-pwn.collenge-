# Substitution Sorcery
## 1. Tổng quan bài toán
Bài toán yêu cầu chúng ta tìm một license key dài 16 bytes. Tuy nhiên, input người dùng nhập vào sẽ bị biến đổi qua một cơ chế "sorcery" (ảo thuật) trước khi được so sánh với key đúng được lưu trong chương trình.

## 2. Phân tích tĩnh (Static Analysis)
Sử dụng lệnh strings để kiểm tra các chuỗi ký tự trong file binary:

Chương trình yêu cầu 16-byte key.

Sử dụng hàm memcmp để so sánh kết quả sau khi biến đổi.

Có một hàm win sẽ in ra flag nếu key chính xác.

## 3. Phân tích động (Dynamic Analysis)
Sử dụng GDB để xem mã máy của hàm main:

Đoạn mã
(gdb) disassemble main
Tại đây, chúng ta phát hiện logic biến đổi nằm trong một vòng lặp:

Lấy input: Chương trình đọc 16 bytes từ người dùng (fread).

Biến đổi (Substitution):

Từng byte input được lấy ra.

Sử dụng byte đó làm index (chỉ số) để tra cứu trong một bảng dữ liệu (lea 0xbd7(%rip), %rdx).

Giá trị tại vị trí index đó trong bảng sẽ là byte đã được biến đổi.

So sánh: Dùng memcmp để so sánh chuỗi sau biến đổi với một chuỗi đích (Target Key).

Sơ đồ cơ chế biến đổi:
## 4. Khai thác (Exploitation)
Bước 1: Tìm chuỗi đích (Target Key)
Đặt breakpoint tại hàm memcmp để xem chương trình mong muốn nhận được kết quả gì sau biến đổi.

Đoạn mã
b *main+434
run
*Nhập đại 16 chữ A*
(gdb) x/16xb $rsi
Kết quả thu được (Target bytes): 0x77 0x6c 0x64 0x77 0x6f 0x77 0x58 0x65 0x43 0x7a 0x75 0x6a 0x52 0x78 0x59 0x4d

Bước 2: Trích xuất bảng tra cứu (Substitution Table)
Bảng này nằm ở offset 0x2140 (nhãn <d> trong GDB). Chúng ta dump 128 bytes của bảng này:

Đoạn mã
(gdb) x/128xb &d
Bước 3: Giải mã ngược
Để tìm key gốc, ta tìm vị trí (index) của từng byte Target trong bảng tra cứu.

Ví dụ: - Byte đầu tiên của Target là 0x77.

Trong bảng tra cứu, giá trị 0x77 nằm ở vị trí thứ 0x50 (tương ứng ký tự 'P').

Vậy byte đầu tiên của key cần nhập là 0x50.

Bảng ánh xạ kết quả:

0x77 -> 0x50 ('P')

0x6c -> 0x5e ('^')

0x64 -> 0x26 ('&') ... (tiếp tục cho đến hết 16 bytes)

Bước 4: Nhập Key
Vì key chứa một số ký tự không in ấn được (non-printable), ta dùng Python để pipe dữ liệu vào chương trình:

Bash
python3 -c 'import sys; sys.stdout.buffer.write(bytes([0x50, 0x5e, 0x26, 0x50, 0x51, 0x50, 0x35, 0x6b, 0x01, 0x41, 0x03, 0x17, 0x58, 0x24, 0x53, 0x37]))' | ./substitution-sorcery
5. Kết quả
Chương trình báo Correct! và in ra flag: 
Correct!
You win! Here is your flag:
pwn.college{UcfOYLNMQzgyucF0s1gO5VNQ1NO.0VO5ATOxwyN1czM1EzW}

### Kỹ thuật rút ra: Đối với các bài toán Substitution Cipher trong Reverse Engineering, thay vì cố gắng hiểu hàm mã hóa, hãy tìm bảng tra cứu (Lookup Table) và thực hiện tra cứu ngược từ kết quả mong muốn về input ban đầu.