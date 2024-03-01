## Codify
``capy3ra``

1. Add ip vào file /etc/hosts

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/a76576bc-97cb-4e20-a5f4-dbd351339701)

2. Dùng nmap scan các cổng dịch vụ nào đang mở.

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/b868737a-7bd8-4648-98b0-e007d8092c41)

3. Truy cập trang web ở port 80. Nhận thấy đây là một code editor cho phép test code nodejs. Thử một vài payload để exec code trên nodejs dùng module như child_process, ... thì thấy bị filter và chặn

![image](https://github.com/capy3ra/HackTheBox/assets/80744099/dd40e654-a95b-4127-ab75-b283dac12ddc)

4. Xem những path khác, ở trang `About us` admin cho biết trang editor đang sử dụng vm2 để tạo ra một môi trường js sandboxing.
5. Search thì nhận thấy vm2 có lỗ hổng Sandbox Bypass  [CVE-2023-32314](https://security.snyk.io/vuln/SNYK-JS-VM2-5537100) cho phép RCE.
6. Chèn poc vào editor.
