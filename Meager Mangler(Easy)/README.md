# Meager Mangler(Easy)
## 1. Phân tích ban đầu
Khi chạy file binary ./meager-mangler-easy, chương trình yêu cầu chúng ta nhập vào một "license key". Sau đó, nó sẽ thực hiện một chuỗi các thao tác biến đổi (mangling) trên đầu vào đó và so sánh kết quả cuối cùng với một giá trị mục tiêu (Expected Result).

Dựa vào kết quả lệnh strings và chạy thử trong gdb, chúng ta xác định được 3 bước biến đổi chính:

Sort Mangler: Sắp xếp các byte nhập vào theo thứ tự tăng dần.

XOR Mangler: XOR các byte với mã khóa 0x6644 (byte chẵn XOR với 0x66, byte lẻ XOR với 0x44).

Swap Mangler: Hoán đổi vị trí của byte tại index 6 và index 9.

## 2. Thu thập dữ liệu từ GDB
Khi chạy chương trình với đầu vào giả định là 16 chữ A và 1 ký tự xuống dòng (tổng 17 bytes), chúng ta lấy được giá trị mục tiêu mà chương trình mong đợi:

Expected Result (Hex): 07 26 05 22 0d 2f 2b 2a 08 0a 16 34 17 37 13 33 11

## 3. Giải mã ngược (Reverse Engineering)
    
    Để tìm ra đầu vào đúng, chúng ta cần thực hiện ngược lại các bước biến đổi từ dưới lên trên.
    
    Bước A: Đảo ngược Swap (6 và 9)Hoán đổi lại byte ở vị trí 6 (2b) và vị trí 9 (0a).Dãy byte sau khi đảo swap: 07 26 05 22 0d 2f 0a 2a 08 2b 16 34 17 37 13 33 11
    
    Bước B: Đảo ngược XOR (với 0x6644)Phép XOR có tính chất đảo ngược: nếu $A \oplus B = C$ thì $C \oplus B = A$. Ta thực hiện XOR lại dãy trên với chu kỳ 0x66 cho index chẵn và 0x44 cho index lẻ.
    
    0x07 ^ 0x66 = 0x61 ('a')
    
    0x26 ^ 0x44 = 0x62 ('b')
    
    0x05 ^ 0x66 = 0x63 ('c')... 
    
    (tiếp tục cho đến hết 17 bytes)
    
    Kết quả thu được: 61 62 63 66 6b 6b 6c 6e 6e 6f 70 70 71 73 75 77 77
    
    Bước C: Xử lý Sort ManglerVì bước đầu tiên của chương trình là sắp xếp (Sort), nên thứ tự ban đầu của các ký tự không quan trọng, miễn là sau khi sắp xếp chúng ta có dãy byte như ở Bước B. May mắn thay, dãy byte chúng ta vừa tìm được đã ở sẵn dạng tăng dần.
    
## 4. Kết quả
Chuyển đổi dãy Hex ở Bước B sang ASCII, ta được chuỗi: abcfkklnnoppqsuww

Thực thi lệnh:

Bash
echo -n "abcfkklnnoppqsuww" | ./meager-mangler-easy
Flag: pwn.college{MMHfRWH2rHnR3o2TneYWSAv5G56.dFjNywyN1czM1EzW}

## 5. Bài học rút ra
Khi gặp chuỗi biến đổi, hãy luôn tư duy theo hướng LIFO (Last In, First Out): Đảo ngược bước cuối cùng trước.

Lệnh strings rất hữu ích để đoán nhanh các hằng số (như 0x6644) và các hàm mangling.

Sử dụng gdb để quan sát dữ liệu thực tế tại thời điểm so sánh (memcmp) giúp tránh các sai sót khi tính toán lý thuyết.