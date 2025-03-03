## Support

1. Nmap scan:
``nmap -sC -sV -Pn 10.129.230.181``
2. Enumeration host:
``netexec smb 10.129.230.181``

![image](https://github.com/user-attachments/assets/43548b7b-c927-43d3-9879-02958d20161e)

3. List Shares on SMB:
``smbclient -N -L 10.129.230.181``

![image](https://github.com/user-attachments/assets/e5eeb11c-ff81-4549-a22e-fac0fb58643e)

4. Enumeration file:
``smbclient //[ip]/[share] -N``

![image](https://github.com/user-attachments/assets/59a4e76e-cd7d-4174-a364-aae698c8ec1d)

5. Folder contains a few application installers such as Putty, Wireshark but one file stands out. `UserInfo.exe.zip`. Isolate this file to investigate further.

![image](https://github.com/user-attachments/assets/77ee05f1-9ae1-428b-bd05-a373452af546)

6. 
