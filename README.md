# w3af 
ghi chép về w3af

## w3af là gì ? 
w3af (web application attack and audit framework) là một công cụ hỗ trợ kiểm thử bảo mật cho các ứng dụng web. 

## Cài đặt (ví dụ trên Ubuntu 14.04 LTS)
Cài đặt trên các nền tảng khác có thể xem [tại đây](http://docs.w3af.org/en/latest/install.html)

#### Các công cụ cần có :
- Git client : 
	
	$ apt-get install git

- Python 2.7 có sẵn trong hệ thống Linux
- Pip version 1.1 (công cụ để quản lý các cài cắm lib của Python)
	
	$ apt-get install python-pip

#### Cài đặt
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

 