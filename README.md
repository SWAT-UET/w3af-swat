<a name="top"></a> 
# w3af 
ghi chép về w3af
***
## Mục lục
- [1. Khái niệm](#contents)
- [2. Cài đặt](#install)
  - [2.1. Chuẩn bị](#prepare)
  - [2.2. Cài đặt](#setup)
- [3. Cấu trúc](#structure)
  - [3.1 Plugins](#plugins)
    - [3.1.1 Crawl plugins](#crawl)
    - [3.1.2 Audit plugins](#audit)
    - [3.1.3 Attack plugins](#attack)
  - [3.2 Plugins khác](#otherplugins)
    - [3.2.1 Infrastructure](#infra)
    - [3.2.2 Grep](#grep)
    - [3.2.3 Output](#output)
    - [3.2.4 Mangle](#mangle)
    - [3.2.5 Bruteforce](#brute)
    - [3.2.6 Evasion](#eva)
  - [3.3 Cấu hình quét](#scan)
  - [3.4 Khuyến nghị cấu hình](#reconfig)
- [4. Thực hiện](#running)
- [5. Tự động hóa việc sử dụng script ](#auto)
- [6. Xác thực](#auth)
- [7. Các trường hợp sử dụng thông thường](#regular)
- [8. Các trường hợp sử dụng nâng cao](#specially)
- [9. W3AF bên trong docker](#docker)
  - [9.1 Các port và service](#port)
  - [9.2 Chia sẻ dữ liệu với container](#container)
  - [9.3 Sửa lỗi container](#fix)
- [10. Khai thác lỗ hổng ứng dụng web ](#exploit)
- [11. Thanh tóan ứng dụng web](#payload)
- [12. Báo lỗi](#warning)
  - [12.1 Thực hành báo lỗi](#pracwarn)
- [13. Giao diện đồ họa (GUI)](#gui)
  - [13.1 Cấu trúc chung](#guistr)
  - [13.2 Quét lỗ hổng](#scanvul)
  - [13.3 Phân tích kết quả](#analysis)
  - [13.4 Khai thác](#discovery)
  - [13.5 Các công cụ](#tools)
  - [13.6 Cấu hình](#config)
- [14. Tham khảo](#refer)

===
<a name="contents"></a>
## 1. w3af là gì ? 
**w3af** (web application attack and audit framework) là một công cụ hỗ trợ kiểm thử bảo mật cho các ứng dụng web. 

<a name="install"></a>
## 2. Cài đặt (ví dụ trên Ubuntu 14.04 LTS)

Cài đặt trên các nền tảng khác có thể xem [tại đây](http://docs.w3af.org/en/latest/install.html)

<a name="prepare"></a>
### Các công cụ cần có :
- Git client : 
	
		$ apt-get install git

- Python 2.7 có sẵn trong hệ thống Linux
- Pip version 1.1 (công cụ để quản lý các cài cắm lib của Python)
	
		$ apt-get install python-pip
<a name="setup"></a>
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

<a name="structure"></a>
## 3. Cấu trúc 

Framework này chia làm 3 nhóm plugin chính : `crawl`, `audit` và `attack`.
<a name="plugins"></a>
### 3.1. Các plugin chính 
<a name="crawl"></a>
#### Crawl plugins
Chúng chỉ có một nhiệm vụ là tìm những URL, form hoặc những điểm tiêm nhiễm khác.

Ví dụ một discovery plugin là `web spider` sẽ nhận đầu vào là các URL và trả về một hay nhiều các điểm tiêm nhiễm.

Khi một người dùng cho phép nhiều hơn một plugin loại này, chúng sẽ chạy trong vòng lặp:

Nếu `plugin A` tìm thấy URL mới trong lần chạy đầu tiên, w3af sẽ gửi cho `plugin B`. Nếu `plugin B` sau đó tìm được URL mới, nó sẽ gửi cho `plugin A`. Tiến trình này sẽ tiếp diễn cho đến khi tất cả các tiến trình đều chạy và không tìm thêm được thông tin về các ứng dụng khác. 

<a name="audit"></a>
#### Audit plugins

Nhận các điểm tiêm nhiễm được tìm thấy bởi `crawl plugins` và nhận diện các lỗ hổng.

Một ví dụ điển hình của `audit plugin` là khi tìm kiếm lỗ hổng SQL Injection, nó sẽ gửi `a'b''c` tới tất cả các điểm tiêm nhiễm.

<a name="attack"></a>
#### Attack plugins
Đối tượng của chúng là những lỗ hổng được tìm thấy bởi các `audit plugin`. Chúng thường trả về một `shell` trên remote server hoặc một `dump` của `remote tables` trong trường hợp khai thác SQL Injection.

<a name="otherplugins"></a>
### b. Các plugin khác
<a name="infra"></a>
#### Infrastructure
Xác định các thông tin về mục tiêu hệ thống như WAF (web application firewalls), hệ điều hành, HTTP daemon.
<a name="grep"></a>
#### Grep 
Phân tích HTTP request và HTTP response được gửi từ những plugin khác và xác định các lỗ hổng.
Ví dụ, một grep plugin sẽ tìm comment trong HTML body có chứa "password" và phát ra một lỗ hổng.

<img src="http://i.imgur.com/IIl8tv9.png">

<a name="output"></a>
#### Output 
Cách giao tiếp giữa framework và plugin với người dùng. 

Output plugin sẽ lưu dữ liệu  dưới dạng file text, xml, html. Những thông tin gỡ rối cũng được xuất ra và lưu lại để phân tích.

Thông điệp được gửi ra output sẽ được gửi tới các plugin được bật, nên nếu bạn cho phép ra 2 output plugin là `text_file` và `xml_file` thì cả hai sẽ log các lỗ hổng được tìm thấy bởi audit plugin.

<a name="mangle"></a>
#### Mangle
Cho phép thay đổi các request và response trên cơ sở các biểu thức thông thường.

<a name="brute"></a>
#### Bruteforce
Bruteforce logins sẽ được tìm thấy trong suốt giai đoạn `crawl`.

<a name="eva"></a>
#### Evasion 
Những quy tắc tránh phát hiện xâm nhập đơn giản bằng cách thay đổi giao thức HTTP được tạo ra bởi các plugin khác. 

<a name="scan"></a>
### c. Cấu hình quét
Sau khi cấu hình những plugin `crawl` và `audit` và cài URL đích, bắt đầu quét và đợi cho các lỗ hổng xuất hiện trên giao diện người dung.

Một vài lỗ hổng được tìm thấy trong suốt quá trình quét được lưu lại và được dùng làm input cho các plugin `attack`. Một khi quá trình quét kết thúc, người dùng sẽ có thể thực thi các plugin `attack` trên các lỗ hổng được xác định.

<a name="reconfig"></a>
### d. Khuyến nghị cấu hình
Chú ý: Thời gian quét phụ thuộc nhiều vào số lượng plugin được bật lên.

Trong hầu hết các trường hợp, `w3af` khuyến nghị nên sử dụng các plugin sau:

- crawl : web_spider
- audit : tất cả
- grep : tất cả 

<a name="running"></a>
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

<a name="auto"></a>
## 5. Tự động hóa việc sử dụng script

Khi phát triển w3af, chúng ta cần thực thi một cách nhanh chóng và dễ dàng theo từng bước, vì thế script được sinh ra để làm điều này. Sử dụng option `-s` để chạy script.

File script là một file text các lệnh trong `w3af_console`. Ví dụ một file script : 

```
plugins
output text_file
output config text_file
set output_file output-w3af.txt
set verbose True
back
```

Những file script được lưu trong thư mục `scripts`

<a name="auth"></a>
## 6. Xác thực (Authentication)

Các kiểu xác thực mà w3af hỗ trợ :
- Xác thực HTTP Basic 
- Xác thực NTLM
- Xác thực form
- Cài đặt một HTTP cookie

HTTP Basic và NTLM là hai loại xác thực HTTP thường được cung cấp bởi các máy chủ web, còn form và cookie thì được cung cấp bởi những ứng dụng của chính nó.

### Xác thực HTTP Basic và NTLM
Sử dụng `http-settings`, cấu hình cài đặt sẽ làm ảnh hưởng tới tất cả các plugin và các thư viện khác. 

```
w3af>>> http-settings
w3af/config:http-settings>>> view
|--------------------------------------------------------------------------------------|
| Setting                | Description                                                 |
|--------------------------------------------------------------------------------------|
...
|--------------------------------------------------------------------------------------|
| ntlm_auth_url          | Set the NTLM authentication domain for HTTP requests        |
| ntlm_auth_user         | Set the NTLM authentication username for HTTP requests      |
| ntlm_auth_passwd       | Set the NTLM authentication password for HTTP requests      |
| ntlm_auth_domain       | Set the NTLM authentication domain (the windows domain name)|
|                        | requests. Please note that only NTLM v1 is supported.       |
|--------------------------------------------------------------------------------------|
...
|--------------------------------------------------------------------------------------|
| basic_auth_user        | Set the basic authentication username for HTTP requests     |
| basic_auth_passwd      | Set the basic authentication password for HTTP requests     |
| basic_auth_domain      | Set the basic authentication domain for HTTP requests       |
|--------------------------------------------------------------------------------------|
w3af/config:http-settings>>>
```

Hãy lưu lại 2 cấu hình khác nhau cho HTTP Basic và NTLM. Nhập các cài đặt bạn ưa thích sau đó `save` lại. Tiếp theo là kích hoạt các plugin cụ thể và bắt đầu quét.

Xác thực NTLM và basic thường yêu cầu username với ký tự `\`, cần nhập \\ trong `w3af_console`. 
Ví dụ người dùng là domain\user : 
 
	set basic_auth_user domain\\user
	
### Xác thực Form 
Sử dụng nhóm plugin `auth` gồm 2 plugin : 
- detailed
- generic 

Những plugin này là những plugin đặc biệt, lưu giữ một phiên trong suốt quá trình quét, được làm mới 5 giây một lần trong khi quét.

##### Cấu hình `generic` 
- username : username của ứng dụng web 
- password : password của ứng dụng web
- username_field : tên form username (có thể tìm thấy trong mã nguồn)
- password_field : tên form password (có thể tìm thấy trong mã nguồn)
- auth_url : URL mà username và password POST tới.
- check_url : URL được sử dụng để kiểm tra phiên vẫn hoạt động, thường được đặt bởi trang cài đặt ứng dụng web.
- check_string : một chuỗi tìm thấy trong response HTTP của check_url vẫn hoạt động.

Một khi đã thiết lập được tất cả các cấu hình, khuyến khích bắt đầu với `crawl.web_spider` và `auth.generic` 
Để mắt tới phần log của w3af, các plugin sẽ tạo ra các bản ghi như : 
	
    `Login success for admin/password` `User "admin" is currently logged into the application`

Đó là những gì bạn mong đợi khi login vào ứng dụng web. Nhưng cũng có thể là một thông báo lỗi : 

	
    Can't login into web application as admin/password

Một trong hai cấu hình không chính xác, hoặc ứng dụng cần thêm các thông số gửi tới `auth_url` và một trong số đó là sử dụng plugin `detailed`.

### Cài đặt HTTP Cookie
Trong trường hợp xác thực Form không hoạt động, có thể liên quan tới đăng nhập chứa thẻ chống tấn công CSRF hoặc 2 nhân tố xác thức. `w3af` cung cấp cho người dùng một phương thức để cài đặt một hoặc nhiều HTTP cookie để sử dụng trong suốt quá trình quét.

Bạn có thể capture lại cookie theo các cách mà bạn thích : trực tiếp từ trình duyệt, sử dụng web proxy, wireshark, ...

Sử dụng file cookie với định dạng Netscape, thay thế các giá trị trong ví dụ : 

```
# Netscape HTTP Cookie File
.netscape.com     TRUE   /  FALSE  946684799   NETSCAPE_ID  100103
```

Mỗi file tạo ra được thiết lập cài đặt `cookie_jar_file` trong `http-settings`.

Chắc chắn rằng các file tạo ra theo đặc tả, bộ phận phân tích cookie của Python thực sự nghiêm ngặt, nó sẽ không load cookie nếu có lỗi được tìm thấy.

### Cài đặt HTTP Header 
Một vài ứng dụng web sử dụng HTTP Header cho việc xác thực, w3af cũng hỗ trợ điều này.

Phương thức này sẽ đặt một HTTP request header vào mỗi HTTP request, lưu ý là không có xác minh tình trạng phiên, nếu phiên bị vô hiệu hóa thì quá trình quét vẫn sử dụng một phiên không hợp lệ.

Để sử dụng phương pháp này, trước hết cần : 
- Tạo một file text với nội dụng (không có dấu ngoặc kép và chèn các cookie mong muốn) : 
	Cookie: <insert-cookie-here>
- Sau đó, trong `http-settings`, cấu hình các thông số của `headers_file` trỏ đến tập tin vừa tạo.
- `save`	 

<a name="regular"></a>
## 7. Các trường hợp sử dụng thông thường
### Quét một thư mục
Thực hiện các bước sau : 
- Đặt URL đích :  http://domain/directory/
- Bật `audit` 
- Bật `crawl.web_spider`
- Trong `crawl.web_spider`, đặt `True` cho `only_forward`

### Lưu các URL làm input cho các lần quét khác
Thu thập dữ liệu là một quá trình khá tốn kém, trong một số trường hợp phải can thiệp bằng tay (spider man plugin).

Để tối ưu việc này, các URL trong quá trình quét sẽ được lưu vào một file thông qua `output.export_requests`

Để load dữ liệu đã lưu, sử dụng plugin ` import_results`.

<a name="specially"></a>
## 8. Các trường hợp sử dụng nâng cao
### Ứng dụng web phức tạp 
Một vài ứng dụng sử dụng ngôn ngữ phía trình duyệt như Javascript, Flash, Java applets nhưng w3af thì không hiểu.

Một plugin gọi là `spider_man` được tạo ra để giải quyết vấn đề này, cho phép người dùng phân tích một ứng dụng web phức tạp. 

Plugin bắt đầu với một HTTP proxy được sử dụng bởi người dùng để điều hướng các trang web mục tiêu. Trong quá trình này, các plugin sẽ trích xuất thông tin từ các yêu cầu và gửi tới các plugin `audit`.

Một ví dụ đơn giản, giả sử w3af đang thực hiện `audit` mà không thấy bất kỳ liên kết nào trên trang chính. Sau khi người sử dụng ra soát lại thấy có một menu Java applet, nơi các phần khác được liên kết đến. Người dùng chạy w3af một lần nữa và kích hoạt plugin `crawl.spider_man`, điều hướng các trang web bằng tay bằng cách sử dụng trình duyệt và proxy spiderman.

Ví dụ chạy plugin `spider_man` :

```
w3af>>> plugins
w3af/plugins>>> crawl spider_man
w3af/plugins>>> audit sqli
w3af/plugins>>> back
w3af>>> target
w3af/target>>> set target http://localhost/
w3af/target>>> back
w3af>>> start
spider_man proxy is running on 127.0.0.1:44444 .
Please configure your browser to use these proxy settings and navigate the target site.
To exit spider_man plugin please navigate to http://127.7.7.7/spider_man?terminate .
```

Trình duyệt được cấu hình sử dụng địa chỉ 127.0.0.1:4444 như một HTTP proxy và điều hướng các trang đích. 

### REST APIs   
//TODO phần này còn sơ sài 

`w3af` có thể được sử dụng để xác định và khai thác lỗ hổng trong REST API. Hai cách phổ biến nhất là: 
- Javascript như một phần của ứng dụng web
- Một chương trình chạy phía bên ngoài trình duyệt

Các bước thực hiện để xác định các lỗ hổng trong một REST API sử dụng một ứng dụng `non-browser`:
- Sử dụng `spider_man` theo các bước phía trên 
- Cấu hình REST API client gửi HTTP request qua `127.0.0.1:44444`
- Chạy REST APT client
- Dừng sử dụng proxy `spider_man`

	curl -X GET http://127.7.7.7/spider_man?terminate --proxy http://127.0.0.1:44444
	
<a name="docker"></a>
## 9. w3af bên trong docker
//TODO chưa hoàn thành 

<a name="port"></a>
### a. Các port và service
Một vài plugin như `crawl.spider_man` hay `audit.rfi` bắt đầu dịch vụ HTTP. 

<a name="container"></a>
### b. Chia sẻ dữ liệu với container
<a name="fix"></a>
### c. Sửa lỗi container
Container chạy một SSH daemon, có thể chạy bằng `w3af_console` hoặc `w3af_gui`. Để kết nối một container đang chạy, sử dụng username `root` và password `w3af`. Bạn không cần quan tâm đến điều này, các script trợ giúp sẽ kết nối container cho bạn.

Một cách khác để debug container là chạy script với cờ `-d`:

```
$ sudo ./w3af_console_docker -d
root@a01aa9631945:~#
```

<a name="exploit"></a>
## 10. Khai thác lỗ hổng ứng dụng web 
`w3af` cho phép người dùng khai thác những lỗ hổng ứng dụng web một cách tự động. Những lỗ hổng được khai thác có thể sử dụng plugin `audit` hoặc vận dụng bằng tay.

Trong suốt quá trình quét, các lỗ hổng được tìm thấy và lưu tại một địa chỉ xác định, từ đó các plugin có thể đọc và sử dụng các thông tin để khai thác các lỗ hổng.

Khai thác một lỗ hổng bằng việc sử dụng một plugin `audit` một cách dễ dàng: 

```
w3af>>> plugins
w3af/plugins>>> audit os_commanding
w3af/plugins>>> back
w3af>>> target
w3af/config:target>>> set target http://localhost/w3af/os_commanding/v.php?command=f0as9
w3af/config:target>>> back
w3af>>> start
Found 1 URLs and 1 different points of injection.
The list of URLs is:
- http://localhost/w3af/os_commanding/v.php
The list of fuzzable requests is:
- http://localhost/w3af/os_commanding/v.php | Method: GET | Parameters: (command)
Starting os_commanding plugin execution.
OS Commanding was found at: "http://localhost/w3af/os_commanding/v.php", using HTTP method GET.
The sent data was: "command=+ping+-c+9+localhost". The vulnerability was found in the request with id 5.
Finished scanning process.
w3af>>> exploit
w3af/exploit>>> exploit os_commanding
os_commanding exploit plugin is starting.
Vulnerability successfully exploited. This is a list of available shells:
- [0] <os_commanding_shell object (ruser: "www-data" | rsystem: "Linux brick 2.6.24-19")>
Please use the interact command to interact with the shell objects.
w3af/exploit>>> interact 0
Execute "end_interaction" to get out of the remote shell.
Commands typed in this menu will run on the remote web server.
w3af/exploit/os_commanding-0>>> ls
v.php
v2.php
v3.php
w3af/exploit/os_commanding-0>>> end_interaction
w3af/exploit>>> back
w3af>>>

Exploiting one you’ve found manually, requires you to provide some input:

w3af>>> kb
w3af/kb>>> help
| list            | List the items in the knowledge base.
| add             | Add a vulnerability to the KB
w3af/kb>>> add os_commanding
w3af/kb/config:os_commanding>>> view
| operating_system         | Remote operating system (linux or windows).
| name                     | Vulnerability name (eg. SQL Injection)
| url                      | URL (without query string parameters)
| vulnerable_parameter     | Vulnerable parameter
| separator                | Command separator used for injecting commands.
| data                     | Query string or postdata parameters in url-encoded form
| method                   | HTTP method
w3af/kb/config:os_commanding>>>
```

Bạn chỉ đơn giản là `set` tất cả những cấu hình sau đó `save` và `back` để lưu những lỗ hổng trong Knowledge Base. Bạn có thể theo các bước sau:

```
w3af>>> exploit
w3af/exploit>>> exploit os_commanding
os_commanding exploit plugin is starting.
Vulnerability successfully exploited. This is a list of available shells:
- [0] <os_commanding_shell object (ruser: "www-data" | rsystem: "Linux brick 2.6.24-19")>
Please use the interact command to interact with the shell objects.
```

<a name="payload"></a>
## 11. Thanh toán ứng dụng web
//TODO chưa hoàn thành


<a name="warning"></a>
## 12. Báo lỗi 
<a name="pracwarn">
### a. Thực hành báo lỗi 
Nếu bạn sử dụng framework bản mới nhất và tìm thấy một lỗi, hãy báo cáo lại theo các thông tin sau:
- Chi tiết các bước tạo ra lỗi
- Kết quả kỳ vọng và đạt được
- Python traceback (nếu có)
- Output của lệnh `./w3af_console --version`
- File log

<a name="gui"></a>
## 13. Giao diện đồ họa (GUI)
//TODO chưa hoàn thành 
<a name="guistr">
### a. Cấu trúc chung  
// TODO @k54hungyb	
<img src="http://docs.w3af.org/en/latest/_images/general-structure.png">

1. Thanh menu
2. Toolbar
3. Các tab chức năng
4. 
#### Toolbar
<a name="scanvul"></a>
### b. Quét lỗ hổng
// TODO @k54hungyb
<a name="analysis"></a>
### c. Phân tích kết quả 
// TODO @ngtuanthanh
<a name="discovery"></a>
### d. Khai thác 
// TODO @congoccho
<a name="tools"></a>
### e. Các công cụ 
// TODO @toioccho
<a name="config"></a>
### f. Cấu hình
Các bảng điều khiển cấu hình khác nhau trên mọi hệ thống w3af.Ở đây tất cả bảng đều được giải thích

#### HTTP configuration
Phần này thường sử dụng cài đặt cấu hình `URL`,nó sẽ ảnh hưởng lên core và tất cả plugins.
<img src="http://docs.w3af.org/en/latest/_images/http-settings.png">

####Miscellaneous configuration
DÙng để cài đặt cấu hình `misc`, nó ảnh hưởng lên core và tất cả plugins.
<img src="">

####Advanced Target configuration
Thường được dùng để cung cấp thông tin chi tiết về hệ thống `mục tiêu`.
<img src="http://docs.w3af.org/en/latest/_images/target-conf.png">

----
<a name="refer"></a>
## 14. Tham khảo

http://docs.w3af.org/en/latest/
