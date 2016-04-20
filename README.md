# w3af 
ghi chép về w3af

## w3af là gì ? 
w3af (web application attack and audit framework) là một công cụ hỗ trợ kiểm thử bảo mật cho các ứng dụng web. 

## Cài đặt (ví dụ trên Ubuntu 14.04 LTS)
Cài đặt trên các nền tảng khác có thể xem [tại đây](http://docs.w3af.org/en/latest/install.html)

### Các công cụ cần có :
- Git client : 
	
	$ apt-get install git

- Python 2.7 có sẵn trong hệ thống Linux
- Pip version 1.1 (công cụ để quản lý các cài cắm lib của Python)
	
	$ apt-get install python-pip

### Cài đặt
```
git clone https://github.com/andresriancho/w3af.git
cd w3af/
./w3af_console
. /tmp/w3af_dependency_install.sh
```

Giải thích : 
- Đầu tiên clone project của w3af từ github về local.
- cd vào thư mục w3af
- Chạy w3af_console
- Trong khi chạy, chương trình báo thiếu một số gói để chạy    
- Chạy w3af_dependency_install.sh để cài các gói thiếu đó

Sau khi quá trình hoàn tất
- ./w3af_console để sử dụng giao diện console.
- ./w3af_gui để sử dụng giao diện đồ họa (lưu ý khi chạy có thể yêu cầu thêm một số gói, chỉ cần làm theo hướng dẫn hiện ra)

## Cấu trúc 
Framework này chia làm 3 nhóm plugin chính : `crawl`, `audit` và `attack`.
### Các plugin chính
#### Crawl plugins
Chúng chỉ có một nhiệm vụ là tìm những URL, form hoặc những điểm tiêm nhiễm khác.

Ví dụ một discovery plugin là `web spider` sẽ nhận đầu vào là các URL và trả về một hay nhiều các điểm tiêm nhiễm.

Khi một người dùng cho phép nhiều hơn một plugin loại này, chúng sẽ chạy trong vòng lặp:

Nếu `plugin A` tìm thấy URL mới trong lần chạy đầu tiên, w3af sẽ gửi cho `plugin B`. Nếu `plugin B` sau đó tìm được URL mới, nó sẽ gửi cho `plugin A`. Tiến trình này sẽ tiếp diễn cho đến khi tất cả các tiến trình đều chạy và không tìm thêm được thông tin về các ứng dụng khác. 

#### Audit plugins

Nhận các điểm tiêm được tìm thấy bởi `crawl plugins` và nhận diện các lỗ hổng.

Một ví dụ điển hình của `audit plugin` là khi tìm kiếm lỗ hổng SQL Injection, nó sẽ gửi `a'b''c` tới tất cả các điểm tiêm nhiễm.

#### Attack plugins
Đối tượng của chúng là những lỗ hổng được tìm thấy bởi các `audit plugin`. Chúng thường trả về một `shell` trên remote server hoặc một `dump` của `remote tables` trong trường hợp khai thác SQL Injection.

### Các plugin khác
#### Infrastructure
Xác định các thông tin về mục tiêu hệ thống như WAF (web application firewalls), hệ điều hành, HTTP daemon.

#### Grep 
Phân tích HTTP request và HTTP response được gửi từ những plugin khác và xác định các lỗ hổng.
Ví dụ, một grep plugin sẽ tìm comment trong HTML body có chứa "password" và phát ra một lỗ hổng.

<img src="http://i.imgur.com/IIl8tv9.png">

#### Output 
Cách giao tiếp giữa framework và plugin với người dùng. 

Output plugin sẽ lưu dữ liệu  dưới dạng file text, xml, html. Những thông tin gỡ rối cũng được xuất ra và lưu lại để phân tích.

Thông điệp được gửi ra output sẽ được gửi tới các plugin được bật, nên nếu bạn cho phép ra 2 output plugin là `text_file` và `xml_file` thì cả hai sẽ log các lỗ hổng được tìm thấy bởi audit plugin.

#### Mangle
Cho phép thay đổi các request và response trên cơ sở các biểu thức thông thường.

#### Bruteforce
Bruteforce logins sẽ được tìm thấy trong suốt giai đoạn `crawl`.

#### Evasion 
Những quy tắc tránh phát hiện xâm nhập đơn giản bằng cách thay đổi giao thức HTTP được tạo ra bởi các plugin khác. 


### Quét câu hình
Sau khi cấu hình những plugin `crawl` và `audit` và cài URL đích, bắt đầu quét và đợi cho các lỗ hổng xuất hiện trên giao diện người dung.

Một vài lỗ hổng được tìm thấy trong suốt quá trình quét được lưu lại và được dùng làm input cho các plugin `attack`. Một khi quá trình quét kết thúc, người dùng sẽ có thể thực thi các plugin `attack` trên các lỗ hổng được xác định.

### Khuyến nghị cấu hình
Chú ý: Thời gian quét phụ thuộc nhiều vào số lượng plugin được bật lên.
Trong hầu hết các trường hợp, `w3af` khuyến nghị nên sử dụng các plugin sau:
- crawl : web_spider
- audit : tất cả
- grep : tất cả 


## Chạy w3af 


 





 