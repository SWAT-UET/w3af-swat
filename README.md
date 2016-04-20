# w3af 
ghi chép về w3af

## 1. w3af là gì ? 
**w3af** (web application attack and audit framework) là một công cụ hỗ trợ kiểm thử bảo mật cho các ứng dụng web. 

## 2. Cài đặt (ví dụ trên Ubuntu 14.04 LTS)
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

## 3. Cấu trúc 
Framework này chia làm 3 nhóm plugin chính : `crawl`, `audit` và `attack`.
### a. Các plugin chính
#### Crawl plugins
Chúng chỉ có một nhiệm vụ là tìm những URL, form hoặc những điểm tiêm nhiễm khác.

Ví dụ một discovery plugin là `web spider` sẽ nhận đầu vào là các URL và trả về một hay nhiều các điểm tiêm nhiễm.

Khi một người dùng cho phép nhiều hơn một plugin loại này, chúng sẽ chạy trong vòng lặp:

Nếu `plugin A` tìm thấy URL mới trong lần chạy đầu tiên, w3af sẽ gửi cho `plugin B`. Nếu `plugin B` sau đó tìm được URL mới, nó sẽ gửi cho `plugin A`. Tiến trình này sẽ tiếp diễn cho đến khi tất cả các tiến trình đều chạy và không tìm thêm được thông tin về các ứng dụng khác. 

#### Audit plugins

Nhận các điểm tiêm nhiễm được tìm thấy bởi `crawl plugins` và nhận diện các lỗ hổng.

Một ví dụ điển hình của `audit plugin` là khi tìm kiếm lỗ hổng SQL Injection, nó sẽ gửi `a'b''c` tới tất cả các điểm tiêm nhiễm.

#### Attack plugins
Đối tượng của chúng là những lỗ hổng được tìm thấy bởi các `audit plugin`. Chúng thường trả về một `shell` trên remote server hoặc một `dump` của `remote tables` trong trường hợp khai thác SQL Injection.

### b. Các plugin khác
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


### c. Cấu hình quét
Sau khi cấu hình những plugin `crawl` và `audit` và cài URL đích, bắt đầu quét và đợi cho các lỗ hổng xuất hiện trên giao diện người dung.

Một vài lỗ hổng được tìm thấy trong suốt quá trình quét được lưu lại và được dùng làm input cho các plugin `attack`. Một khi quá trình quét kết thúc, người dùng sẽ có thể thực thi các plugin `attack` trên các lỗ hổng được xác định.

### d. Khuyến nghị cấu hình
Chú ý: Thời gian quét phụ thuộc nhiều vào số lượng plugin được bật lên.

Trong hầu hết các trường hợp, `w3af` khuyến nghị nên sử dụng các plugin sau:

- crawl : web_spider
- audit : tất cả
- grep : tất cả 


## 4. Chạy w3af 
**w3af** có 2 giao diện người dùng : console và đồ họa. 

Bài viết này sẽ trình bày về giao diện console để dễ dàng giải thích các đặc trưng của framework.

Trên cửa sổ dòng lệnh (terminal), thư mục hiện tại là w3af: 

	$ ./w3af_console
 
Dấu nhắc lệnh của chương trình : 

	w3af>>> 
	
Gõ `help` để tìm hiểu các tùy chọn của chương trình

```
w3af>>> help
|----------------------------------------------------------------|
| start         | Start the scan.                                |
| plugins       | Enable and configure plugins.                  |
| exploit       | Exploit the vulnerability.                     |
| profiles      | List and use scan profiles.                    |
| cleanup       | Cleanup before starting a new scan.            |
|----------------------------------------------------------------|
| help          | Display help. Issuing: help [command] , prints |
|               | more specific help about "command"             |
| version       | Show w3af version information.                 |
| keys          | Display key shortcuts.                         |
|----------------------------------------------------------------|
| http-settings | Configure the HTTP settings of the framework.  |
| misc-settings | Configure w3af misc settings.                  |
| target        | Configure the target URL.                      |
|----------------------------------------------------------------|
| back          | Go to the previous menu.                       |
| exit          | Exit w3af.                                     |
|----------------------------------------------------------------|
| kb            | Browse the vulnerabilities stored in the       |
|               | Knowledge Base                                 |
|----------------------------------------------------------------|
w3af>>>
w3af>>> help target
Configure the target URL.
w3af>>>
```

Để nhập một menu cấu hình, bạn chỉ cần gõ tên nó và Enter.

Một điều thú vị là chức năng autocomplete. Khi bạn viết một lệnh, bạn chỉ cần nhập một vài ký tự rồi ấn TAB, chương trình sẽ tự động hoàn thiện lệnh đó.

Tất cả các menu cấu hình cung cấp theo bởi các lệnh: 
- help : đưa ra các thông tin chi tiết
- view : liệt kê các thông số cấu hình 
- set : thay đổi các giá trị (hoặc ấn Ctrl+C)
- back : trở lại menu trước 

Ví dụ các lệnh trong menu `http-settings`:

```
w3af/config:http-settings>>> help
|-----------------------------------------------------------------|
| view  | List the available options and their values.            |
| set   | Set a parameter value.                                  |
| save  | Save the configured settings.                           |
|-----------------------------------------------------------------|
| back  | Go to the previous menu.                                |
| exit  | Exit w3af.                                              |
|-----------------------------------------------------------------|
w3af/config:http-settings>>> view
|-----------------------------------------------------------------------------------------------|
| Setting                | Value    | Description                                               |
|-----------------------------------------------------------------------------------------------|
| url_parameter          |          | Append the given URL parameter to every accessed URL.     |
|                        |          | Example: http://www.foobar.com/index.jsp;<parameter>?id=2 |
| timeout                | 15       | The timeout for connections to the HTTP server            |
| headers_file           |          | Set the headers filename. This file has additional headers|
|                        |          | which are added to each request.                          |
|-----------------------------------------------------------------------------------------------|
...
|-----------------------------------------------------------------------------------------------|
| basic_auth_user        |          | Set the basic authentication username for HTTP requests   |
| basic_auth_passwd      |          | Set the basic authentication password for HTTP requests   |
| basic_auth_domain      |          | Set the basic authentication domain for HTTP requests     |
|-----------------------------------------------------------------------------------------------|
w3af/config:http-settings>>> set timeout 5
w3af/config:http-settings>>> save
w3af/config:http-settings>>> back
w3af>>>
```

### Chạy w3af giao diện đồ họa GTK

Trên cửa sổ dòng lệnh (terminal), thư mục hiện tại là w3af: 

	$ ./w3af_gui

<img src="http://i.imgur.com/3hCUXRn.png">

 
GUI có phụ thuộc vào bên thứ ba nên phải cài thêm một số gói cài đặt khác.

### Cấu hình các plugin

Sử dụng `plugin` để cấu hình các plugins:

```
w3af>>> plugins
w3af/plugins>>> help
|-----------------------------------------------------------------------------|
| list             | List available plugins.                                  |
|-----------------------------------------------------------------------------|
| back             | Go to the previous menu.                                 |
| exit             | Exit w3af.                                               |
|-----------------------------------------------------------------------------|
| output           | View, configure and enable output plugins                |
| audit            | View, configure and enable audit plugins                 |
| crawl            | View, configure and enable crawl plugins                 |
| bruteforce       | View, configure and enable bruteforce plugins            |
| grep             | View, configure and enable grep plugins                  |
| evasion          | View, configure and enable evasion plugins               |
| infrastructure   | View, configure and enable infrastructure plugins        |
| auth             | View, configure and enable auth plugins                  |
| mangle           | View, configure and enable mangle plugins                |
|-----------------------------------------------------------------------------|
w3af/plugins>>>
```

Tất cả các plugin (trừ attack plugin) đều có thể cấu hình trong menu này 

Liệt kê các plugin của `audit`: 

```
w3af>>> plugins
w3af/plugins>>> list audit
|-----------------------------------------------------------------------------|
| Plugin name        | Status | Conf | Description                            |
|-----------------------------------------------------------------------------|
| blind_sqli         |        | Yes  | Identify blind SQL injection           |
|                    |        |      | vulnerabilities.                       |
| buffer_overflow    |        |      | Find buffer overflow vulnerabilities.  |
...
```

Bật plugin `xss` và `sqli` :

```
w3af/plugins>>> audit xss, sqli
w3af/plugins>>> audit
|----------------------------------------------------------------------------|
| Plugin name        | Status  | Conf | Description                          |
|----------------------------------------------------------------------------|
| sqli               | Enabled |      | Find SQL injection bugs.             |
| ssi                |         |      | Find server side inclusion           |
|                    |         |      | vulnerabilities.                     |
| ssl_certificate    |         | Yes  | Check the SSL certificate validity   |
|                    |         |      | (if https is being used).            |
| un_ssl             |         |      | Find out if secure content can also  |
|                    |         |      | be fetched using http.               |
| xpath              |         |      | Find XPATH injection                 |
|                    |         |      | vulnerabilities.                     |
| xss                | Enabled | Yes  | Identify cross site scripting        |
|                    |         |      | vulnerabilities.                     |
| xst                |         |      | Find Cross Site Tracing              |
|                    |         |      | vulnerabilities.                     |
|----------------------------------------------------------------------------|
w3af/plugins>>>
```

Nếu muốn biết plugin đó làm gì thì sử dụng lệnh `desc`:

```
w3af/plugins>>> audit desc xss

This plugin finds Cross Site Scripting (XSS) vulnerabilities.

One configurable parameters exists:
    - persistent_xss

To find XSS bugs the plugin will send a set of javascript strings to
every parameter, and search for that input in the response.

The "persistent_xss" parameter makes the plugin store all data
sent to the web application and at the end, request all URLs again
searching for those specially crafted strings.

w3af/plugins>>>
```

Kiểm tra các thông số chi tiết cấu hình sử dụng : `config`

```
w3af/plugins>>> audit config xss
w3af/plugins/audit/config:xss>>> view
|-----------------------------------------------------------------------------|
| Setting        | Value | Description                                        |
|-----------------------------------------------------------------------------|
| persistent_xss | True  | Identify persistent cross site scripting           |
|                |       | vulnerabilities                                    |
|-----------------------------------------------------------------------------|
w3af/plugins/audit/config:xss>>> set persistent_xss False
w3af/plugins/audit/config:xss>>> back
The configuration has been saved.
w3af/plugins>>>
```

### Lưu cấu hình 
Mỗi cấu hình được lưu vào profiles, sử dụng lệnh `save_as`:

```
w3af>>> profiles
w3af/profiles>>> save_as tutorial
Profile saved.
```

Đường dẫn đến profiles là `~/.w3af/profiles/` và được load lên bằng lệnh `use`  : 

```
w3af>>> profiles
w3af/profiles>>> use fast_scan
The plugins configured by the scan profile have been enabled, and their options configured.
Please set the target URL(s) and start the scan.
w3af/profiles>>>
```

Để chia sẻ profile với những người dùng khác, sử dụng cờ `self-contained`:

```
w3af>>> profiles
w3af/profiles>>> save_as tutorial self-contained
Profile saved.
```

### Bắt đầu quét 
Sau khi cấu hình xong, phải chọn URL làm mục tiêu quét :

```
w3af>>> target
w3af/config:target>>> set target http://localhost/
w3af/config:target>>> back
w3af>>>
```

Cuối cùng, `start` để bắt đầu :

```
w3af>>> start
```

## 5. Tự động hóa việc sử dụng script


