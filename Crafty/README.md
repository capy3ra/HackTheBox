## Crafty
`capy3ra`

1. Kết nối vpn.
2. Scan các port trên target IP bằng nmap nhận được kết quả sau:

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/4c888eb3-89a1-4da6-9ccc-d77a6c808de7)

3. Truy cập dịch vụ http trên port 80 tới được trang web. Nhưng có vẻ như không có gì đặc biệt.

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/ebd680f3-a508-47ef-ab0a-294b81941ce7)

4. Còn port 25565 có ghi là đang chạy dịch vụ Minecraft Server khá là khả nghi
5. Search lỗ hổng liên quan đến dịch vụ chạy trên port này với key word như: `vulnerable in minecraft server 1.16.5` -> các kết quả dẫn đến: Lỗ hổng Log4j trên Minecraft: Java edition với mã CVE-2021-44228
6. 
