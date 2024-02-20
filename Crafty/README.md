## Crafty
`capy3ra`

1. Kết nối vpn.
2. Scan các port trên target IP bằng nmap nhận được kết quả sau:

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/4c888eb3-89a1-4da6-9ccc-d77a6c808de7)

3. Truy cập dịch vụ http trên port 80 tới được trang web. Nhưng có vẻ như không có gì đặc biệt.

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/ebd680f3-a508-47ef-ab0a-294b81941ce7)

4. Còn port 25565 có ghi là đang chạy dịch vụ Minecraft Server khá là khả nghi
5. Search lỗ hổng liên quan đến dịch vụ chạy trên port này với key word như: `vulnerable in minecraft server 1.16.5` -> các kết quả dẫn đến: Lỗ hổng Log4j trên Minecraft: Java edition với mã CVE-2021-44228
6. Đọc tài liệu của VCS-TI phân tích mã CVE này [Ref](https://blog.viettelcybersecurity.com/cve-2021-44228-lo-hong-nghiem-trong-tren-thu-vien-apache-log4j-va-cac-van-de-lien-quan/)
7. Kiếm [POC](https://github.com/kozmer/log4j-shell-poc) của CVE
8. Đọc readme để poc có thể chạy như tải jdk8u20...
9. Đọc thêm một vài blog sau để hiểu thêm về lỗ hổng log4shell:
- [VCS_TI](https://blog.viettelcybersecurity.com/cve-2021-44228-lo-hong-nghiem-trong-tren-thu-vien-apache-log4j-va-cac-van-de-lien-quan/)
- [CyRadar](https://cyradar.com/2022/01/05/phan-tich-ky-thuat-cve-2021-44228-log4shell/)

10. Tải phiên bản minecraft trên server 1.16.5 tại [trang](https://tlauncher.org/en/download_1/minecraft-1-16-5_12582.html#download)
11. Sau đó connect đến server

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/206ebd34-7c81-422f-9947-c99774d45386)

12. Direct connection đến server nhưng không thành công nó báo timeout... Có thể là do VPN yếu 🤨
