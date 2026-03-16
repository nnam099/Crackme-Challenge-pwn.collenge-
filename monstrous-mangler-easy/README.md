# Writeup: monstrous-mangler-easy
## 1. Phân tích mục tiêu (Reconnaissance)
Thử thách cung cấp một file thực thi ELF 64-bit. Khi chạy chương trình, nó yêu cầu một "license key". Nếu nhập sai, chương trình sẽ thực hiện một chuỗi các thao tác biến đổi (mangling) trên chuỗi nhập vào và so sánh kết quả cuối cùng với một mảng byte mục tiêu (Expected result).

## 2. Tìm hiểu cơ chế biến đổi (Reverse Engineering)
Bằng cách quan sát kết quả từ gdb hoặc chạy thử với các chuỗi nháp (ví dụ: AAAAAAAAAAAAAAAA...), chúng ta xác định được chương trình thực hiện 7 bước biến đổi theo thứ tự sau:

1.Reverse: Đảo ngược chuỗi.

2.Swap (22, 35): Hoán đổi giá trị tại vị trí index 22 và 35.

3.XOR: XOR chuỗi với key 0x7997e425241f4a. Key này sẽ lặp lại sau mỗi 7 byte.

4.Reverse: Đảo ngược chuỗi một lần nữa.

5.Swap (2, 32): Hoán đổi giá trị tại index 2 và 32.

6.Swap (1, 5): Hoán đổi giá trị tại index 1 và 5.

7.Reverse: Đảo ngược chuỗi lần cuối.

Dữ liệu mục tiêu (Expected Result): 0a fa 87 41 2f 7b 21 1a e5 8b 43 40 68 3b 08 e6 86 47 4c 69 25 17 e0 81 55 41 69 3c 1e fc 81 1d 45 68 40 4e ef

## 3. Giải quyết vấn đề (The Solution)
Để tìm được License Key ban đầu, ta cần thực hiện các thao tác nghịch đảo (inverse) theo thứ tự ngược lại (từ bước 7 về bước 1).

Nghịch đảo của Reverse vẫn là Reverse.

Nghịch đảo của Swap vẫn là Swap chính nó.

Nghịch đảo của XOR vẫn là XOR với cùng một key.

Script giải mã (Python):
Python
target = [
    0x0a, 0xfa, 0x87, 0x41, 0x2f, 0x7b, 0x21, 0x1a, 0xe5, 0x8b, 0x43, 0x40, 0x68, 0x3b, 0x08, 0xe6, 
    0x86, 0x47, 0x4c, 0x69, 0x25, 0x17, 0xe0, 0x81, 0x55, 0x41, 0x69, 0x3c, 0x1e, 0xfc, 0x81, 0x1d, 
    0x45, 0x68, 0x40, 0x4e, 0xef
]

def swap(arr, i, j):
    arr[i], arr[j] = arr[j], arr[i]

key = [0x79, 0x97, 0xe4, 0x25, 0x24, 0x1f, 0x4a]

target.reverse()
swap(target, 1, 5)
swap(target, 2, 32)
target.reverse() 
for i in range(len(target)):
    target[i] ^= key[i % len(key)]
swap(target, 22, 35)           
target.reverse()             
print("License Key:", "".join([chr(b) for b in target]))

## 4. Kết quả
Chạy script trên ta thu được key: xwewakekgvvepednovhbbqqqwdforckdddcms

Nhập key này vào chương trình:

Bash
hacker@reverse-engineering:~$ ./monstrous-mangler-easy
...
Ready to receive your license key!
xwewakekgvvepednovhbbqqqwdforckdddcms
You win! Here is your flag:
pwn.college{cfR17Hig6D7oI00S7L-4Mn3MwFw.dVjNywyN1czM1EzW}