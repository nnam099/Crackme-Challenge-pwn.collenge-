# Writeup – Tangled Ticket (Easy)

## Thông tin challenge

* Tên: **Tangled Ticket (Easy)**
* Loại: Reverse Engineering (ELF x86-64)
* Đặc điểm: Binary **setuid**, không strip

---

## Mục tiêu

Reverse engineer chương trình để tìm **license key** đúng. Lưu ý: input sẽ bị **mangling** trước khi so sánh với giá trị mong đợi.

---

## Phân tích sơ bộ

Kiểm tra file:

```
file tangled-ticket-easy
```

Kết quả cho thấy đây là ELF 64-bit, PIE, dynamically linked, **not stripped**.

Dùng `strings` để xem thông báo trong chương trình:

* Chương trình đọc license key từ `stdin`
* In từng byte input dạng `%02x`
* Áp dụng mangler **swap** cho index **0** và **2**
* So sánh kết quả cuối cùng với **EXPECTED_RESULT**
* Nếu trùng → đọc `/flag`

Thông báo quan trọng:

```
This challenge is now mangling your input using the `swap` mangler for indexes `0` and `2`.
```

=> Phép biến đổi duy nhất là **hoán đổi byte 0 và byte 2**.

---

## Xác định EXPECTED_RESULT

Vì binary không strip, symbol `EXPECTED_RESULT` vẫn còn.

Trong `gdb`:

```
info variables EXPECTED_RESULT
```

Kết quả:

```
0x4010 EXPECTED_RESULT
```

Dump dữ liệu tại địa chỉ này:

```
x/32bx 0x4010
```

Kết quả:

```
0x4010 <EXPECTED_RESULT>: 74 64 63 78 67 00
```

Chuyển sang ASCII:

| Index | Hex  | ASCII |
| ----: | ---- | ----- |
|     0 | 0x74 | t     |
|     1 | 0x64 | d     |
|     2 | 0x63 | c     |
|     3 | 0x78 | x     |
|     4 | 0x67 | g     |

=> `EXPECTED_RESULT = "tdcxg"`

---

## Suy ngược license key

Chương trình thực hiện:

```
swap(input[0], input[2])
```

Do đó để sau swap thu được `EXPECTED_RESULT`, ta cần:

```
input[0] = expected[2]
input[1] = expected[1]
input[2] = expected[0]
input[3] = expected[3]
input[4] = expected[4]
```

Áp dụng cho `tdcxg`:

```
input = c d t x g
```

=> **License key đúng:**

```
cdtxg
```

---

## Chạy chương trình và lấy flag

⚠️ Vì binary là **setuid**, phải chạy **trực tiếp**, không chạy trong gdb.

Cách 1: nhập tay

```
./tangled-ticket-easy
cdtxg
```

Cách 2: dùng `printf`

```
printf "cdtxg" | ./tangled-ticket-easy
```

Kết quả:

```
You win! Here is your flag:
pwn.college{wcSe-_HExO0dI3VcB8Z9ug7CgRt.dNTNywyN1czM1EzW}
```

---

## Kết luận

* Mangler chỉ là **swap byte 0 và 2**
* `EXPECTED_RESULT` lưu sẵn trong `.data`
* Reverse phép swap để tìm input gốc
* License key: **cdtxg**

Đây là một bài reverse cơ bản, tập trung vào việc đọc hiểu logic và suy ngược phép biến đổi đơn giản.

---

## Flag

```
pwn.college{wcSe-_HExO0dI3VcB8Z9ug7CgRt.dNTNywyN1czM1EzW}
```
