# 🧩 Terrible Token – Easy (Reverse Engineering)

[TOC]

---

## 📌 Thông tin chung

- **Tên bài**: terrible-token-easy  
- **Nền tảng**: pwn.college  
- **Thể loại**: Reverse Engineering  
- **Độ khó**: Easy  
- **Mục tiêu**: Tìm license key hợp lệ để đọc flag  

---

## 🔍 1. Recon ban đầu

Kiểm tra loại file:

```bash
file terrible-token-easy


Kết quả:

setuid ELF 64-bit LSB pie executable, x86-64, dynamically linked, not stripped


Nhận xét:

Binary không bị strip → reverse dễ

Có SUID bit → phải chạy trực tiếp để đọc /flag

Không có anti-debug

🧵 2. Phân tích strings
strings terrible-token-easy


Các chuỗi quan trọng:

Initial input:
The mangling is done!
Final result of mangling input:
Expected result:
Checking the received license key!
Wrong! No flag for you!
You win! Here is your flag:
/flag


Đặc biệt chú ý symbol:

EXPECTED_RESULT


→ Gợi ý rõ ràng rằng chương trình sẽ so sánh input với một giá trị cố định bằng memcmp.

🧠 3. Xác định EXPECTED_RESULT

Tìm symbol:

nm -C terrible-token-easy | grep EXPECTED

0000000000004010 D EXPECTED_RESULT


Dump section .data:

objdump -s -j .data terrible-token-easy


Kết quả:

Contents of section .data:
 4010  69 61 74 78 62 00


Giải mã:

Hex	ASCII
69	i
61	a
74	t
78	x
62	b

➡️ EXPECTED_RESULT = "iatxb"

⚠️ 4. Hiểu đúng cơ chế input (điểm dễ sai)

Ban đầu có thể nghĩ chương trình đọc input dạng hex vì có %02x.

Thử nhập:

69 61 74 78 62


Chương trình in ra:

Initial input:
36 39 20 36 31


Giải thích:

Hex	ASCII
36	'6'
39	'9'
20	space
36	'6'
31	'1'

➡️ Chương trình không parse hex
➡️ Nó dùng read() để đọc raw bytes, rồi in ra bằng %02x

%02x chỉ dùng để hiển thị, không phải để đọc input.

🔬 5. Kiểm tra “mangling”

Output cho thấy:

Initial input:
69 61 74 78 62

Final result of mangling input:
69 61 74 78 62


➡️ Không có mangling thực sự
➡️ Input được so sánh trực tiếp với EXPECTED_RESULT bằng memcmp

🔑 6. License key hợp lệ

EXPECTED_RESULT (raw bytes):

69 61 74 78 62


Tương ứng ASCII:

iatxb


⚠️ License key là chuỗi raw, không phải hex text.

🚀 7. Cách nhập đúng để lấy flag

Không được gõ tay vì sẽ có newline (0x0a).

Cách đúng:

echo -ne "iatxb" | /challenge/terrible-token-easy


Hoặc:

printf "iatxb" | /challenge/terrible-token-easy

🏁 8. Kết quả
You win! Here is your flag:
pwn.college{gq0nPcnPz_wPUmpd2Pc41_89a8d.dFTNywyN1czM1EzW}

🧠 9. Lessons Learned

Không nên vội tin rằng %02x nghĩa là input hex

Cần phân biệt rõ hex text và raw byte input

Với binary không strip, nm + objdump thường là đủ

Khi so sánh bằng memcmp, newline có thể làm sai kết quả

✅ Tổng kết

Bài reverse mức rất cơ bản

Trọng tâm là hiểu đúng input

Không có mã hóa hay obfuscation phức tạp