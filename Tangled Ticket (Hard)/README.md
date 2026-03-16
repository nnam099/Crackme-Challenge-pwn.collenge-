# Writeup – Tangled Ticket (Hard)
## Thông tin challenge
Tên: **Tangled Ticket (Hard)**

Loại: Reverse Engineering (ELF x86-64)

Đặc điểm: Binary setuid, không strip, cơ chế mangling không công khai.

## Mục tiêu
Reverse engineer chương trình để tìm license key đúng. Ở cấp độ Hard, chương trình không thông báo cụ thể phép biến đổi (mangling) nào được áp dụng, buộc ta phải phân tích động để tìm quy luật.

## Phân tích sơ bộ
Kiểm tra chuỗi ký tự bằng strings: Bash strings tangled-ticket-hard
Kết quả xuất hiện chuỗi lạ:vojov
Chuỗi này có độ dài 5 ký tự và nằm gần các thông báo về flag, khả năng cao đây là EXPECTED_RESULT sau khi input bị biến đổi.Phân tích động bằng GDBDo không biết phép mangling là gì, ta sử dụng gdb để quan sát sự thay đổi của input ngay trước hàm so sánh memcmp.Các bước thực hiện:Đặt breakpoint tại hàm so sánh:
Đoạn mãbreak memcmp
Chạy với input mẫu bcdef: Đoạn mã run(nhập bcdef)
Kiểm tra bộ nhớ tại điểm dừng:
Sử dụng thanh ghi $rdi (chứa input đã bị mangling) và $rsi (chứa chuỗi mục tiêu):
Đoạn mãx/s $rdi
# Kết quả: 0x...: "cbdef"

x/s $rsi
# Kết quả: 0x...: "vojov"
Nhận xét: Input gốc bcdef đã bị biến thành cbdef.=>EXPECTED_RESULT = "vojov" với độ dài 5 byte.Suy ngược license keyChương trình thực hiện:C++swap(input[0], input[1])
Để sau khi swap thu được vojov, ta đảo ngược vị trí 0 và 1 của chuỗi đích:input[0] = expected[1] = oinput[1] = expected[0] = vinput[2] = expected[2] = jinput[3] = expected[3] = oinput[4] = expected[4] = v=> License key đúng:ovjov
Chạy chương trình và lấy flag⚠️ Vì binary là setuid, phải chạy trực tiếp trong shell để có quyền đọc /flag. GDB sẽ làm mất quyền này.Thực hiện:Bash./tangled-ticket-hard
(nhập ovjov)
Kết quả:PlaintextChecking the received license key!

You win! Here is your flag:
pwn.college{UwLol6YE2NmvyEi5jT7DO0pi66j.dRTNywyN1czM1EzW}
## Kết luận:
Swap byte 0 và 1.Quy luật mangling được xác định thông qua việc quan sát thanh ghi tại hàm memcmp.License key: ovjov. Thử thách này yêu cầu kỹ năng debug cơ bản để nhận diện thuật toán biến đổi dữ liệu thay vì chỉ đọc chuỗi tĩnh. flag là pwn.college{UwLol6YE2NmvyEi5jT7DO0pi66j.dRTNywyN1czM1EzW}