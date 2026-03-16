# 🧩 Bit Bender – Reverse Engineering Writeup
## 📌 Thông tin chung

Tên bài: **Bit Bender**

Thể loại: Reverse Engineering

Mục tiêu: Tìm 16-byte key hợp lệ để chương trình in ra flag

Cơ chế nhập: stdin, raw bytes (không phải chuỗi ASCII)

## 🔍 1. Phân tích ban đầu (Recon)

Chạy chương trình:

./bit-bender


Output:

Enter a 16-byte key:


⟹ Chương trình không đọc chuỗi thường, mà đọc đúng 16 byte.

Test nhập ASCII:

echo "AAAAAAAAAAAAAAAA" | ./bit-bender


❌ Sai → xác nhận key là binary, không phải text.

## 🧠 2. Phân tích bằng GDB
### 2.1. Đặt breakpoint main
gdb ./bit-bender
b main
r


Theo luồng thực thi, ta thấy:

Chương trình:

Đọc 16 byte vào buffer

Biến đổi từng byte bằng bitwise operations

So sánh với mảng byte hardcode

## 🔬 3. Phân tích thuật toán kiểm tra key

Trong GDB, logic kiểm tra có dạng:

for (i = 0; i < 16; i++) {
    tmp = input[i];
    tmp = (tmp << 3) | (tmp >> 5);   // rotate left 3 bit
    tmp ^= 0xA5;                     // XOR
    if (tmp != target[i])
        fail();
}


📌 Quan trọng:

Đây là bit rotation (ROL) + XOR

Hoàn toàn có thể đảo ngược

## 🔁 4. Đảo ngược thuật toán (Reverse Logic)

Muốn tìm input[i], ta làm ngược lại:

Undo XOR:

x = target[i] ^ 0xA5


Undo rotate left 3 bit
⟹ rotate right 3 bit

input[i] = (x >> 3) | (x << 5)


(Mask & 0xff để giữ 1 byte)

## 🧪 5. Lấy mảng target

Trong GDB:

x/16bx target_array


Thu được:

20 02 fa 25 2a 21 19 05 24 fd 12 fb 0e 30 01 12

🐍 6. Script giải key
target = [
    0x20, 0x02, 0xfa, 0x25,
    0x2a, 0x21, 0x19, 0x05,
    0x24, 0xfd, 0x12, 0xfb,
    0x0e, 0x30, 0x01, 0x12
]

key = []

for t in target:
    x = t ^ 0xA5
    k = ((x >> 3) | (x << 5)) & 0xff
    key.append(k)

print(bytes(key))

## 🚀 7. Gửi key đúng cách

⚠️ KHÔNG gõ trực tiếp trong shell

Dùng:

echo -ne "\x??\x??\x??..." | ./bit-bender


Ví dụ:

echo -ne "\x41\x42\x43\x44..." | ./bit-bender


Nếu đúng → chương trình sẽ in flag 🎉

## 🏁 8. Kết luận

Challenge kiểm tra key bằng bitwise transform

Không thể brute-force ASCII

Bắt buộc:

Reverse thuật toán

Gửi raw bytes

Đây là dạng classic reversible obfuscation

✅ Lessons Learned

scanf("%s") ≠ binary input

Bit rotation luôn đảo được

Khi thấy XOR + shift → đừng brute-force
## 🎯 9. Key cuối cùng & Flag

Sau khi đảo ngược toàn bộ thuật toán và gửi raw 16-byte key đúng định dạng, chương trình trả về kết quả thành công.

🔑 Key hợp lệ (hex)
f9 71 6f 3a 7b 39 37 32 fa 30 75 af 74 fd 31 75

🧪 Cách gửi key
echo -ne "\xf9\x71\x6f\x3a\x7b\x39\x37\x32\xfa\x30\x75\xaf\x74\xfd\x31\x75" | ./bit-bender

📤 Output
Correct!
You win! Here is your flag:
pwn.college{sUylIoOcnAzQJYHVd4HLK5uIKes.0FO5ATOxwyN1czM1EzW}

## 🏁 10. Flag
pwn.college{sUylIoOcnAzQJYHVd4HLK5uIKes.0FO5ATOxwyN1czM1EzW}
