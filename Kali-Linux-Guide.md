# Hướng dẫn Kali Linux — Cơ bản đến nâng cao

Tài liệu này dành cho mục đích học tập, luyện tập trong môi trường lab/CTF do bạn tự dựng hoặc được cấp phép (ví dụ TryHackMe, HackTheBox, VM tự tạo). Không dùng các công cụ bên dưới để truy cập hệ thống, mạng hoặc tài khoản mà bạn không có quyền kiểm thử.

## Mục lục

1. Cài đặt và thiết lập ban đầu
2. Làm quen với shell và quản lý gói
3. Quản lý người dùng, quyền, tiến trình
4. Mạng cơ bản
5. Thu thập thông tin (reconnaissance)
6. Quét mạng và dò quét dịch vụ
7. Phân tích lỗ hổng
8. Khai thác (trong phạm vi lab được phép)
9. Sau khai thác và báo cáo
10. Viết script và tự động hóa
11. Thực hành an toàn và pháp lý

---

## 1. Cài đặt và thiết lập ban đầu

**Cách chạy Kali:**
- Máy ảo (VMware/VirtualBox): dùng ảnh `.ova` chính thức từ kali.org, dễ snapshot/rollback.
- WSL2 (Windows): phù hợp học command-line, hạn chế với tool cần driver mạng đặc biệt.
- Bare-metal hoặc USB live: dùng khi cần phần cứng thật (ví dụ card Wi-Fi hỗ trợ monitor mode).

**Việc đầu tiên sau khi cài:**

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y kali-linux-headless   # hoặc kali-linux-default nếu muốn đủ GUI tool
```

Đổi mật khẩu mặc định ngay nếu dùng bản live/VM tải sẵn:

```bash
passwd
```

## 2. Làm quen với shell và quản lý gói

**Shell mặc định** của Kali hiện nay là `zsh`. Các lệnh cơ bản giống Bash:

```bash
pwd                 # thư mục hiện tại
ls -la               # liệt kê, gồm file ẩn
cd /path             # di chuyển
cp, mv, rm           # copy, di chuyển, xóa
man <lệnh>           # đọc tài liệu lệnh
```

**Quản lý gói (APT):**

```bash
apt search <tên>          # tìm gói
apt show <tên>             # xem thông tin gói
sudo apt install <tên>     # cài
sudo apt remove <tên>      # gỡ
```

**Metapackage theo chủ đề**, cài nhanh một nhóm công cụ:

```bash
sudo apt install kali-tools-web        # công cụ web pentest
sudo apt install kali-tools-wireless   # công cụ không dây
```

## 3. Quản lý người dùng, quyền, tiến trình

```bash
whoami               # user hiện tại
sudo -i              # vào shell root
chmod 750 file        # đổi quyền (rwx theo owner/group/other)
chown user:group file # đổi chủ sở hữu
ps aux                # liệt kê tiến trình
top / htop            # theo dõi tài nguyên
kill -9 <pid>          # buộc dừng tiến trình
```

**Hiểu quyền file:** `rwx` ứng với đọc (4), ghi (2), thực thi (1). `chmod 750` nghĩa là chủ sở hữu toàn quyền, nhóm đọc+thực thi, người khác không quyền.

## 4. Mạng cơ bản

```bash
ip a                   # xem địa chỉ IP, thay cho ifconfig cũ
ip route               # bảng định tuyến
ping <host>            # kiểm tra kết nối
curl -I <url>          # xem header HTTP
ss -tulnp              # cổng đang mở, thay cho netstat
```

**Đổi địa chỉ MAC** (hữu ích khi học về anonymization, chỉ trên máy/lab của bạn):

```bash
sudo ip link set dev eth0 down
sudo macchanger -r eth0
sudo ip link set dev eth0 up
```

## 5. Thu thập thông tin (reconnaissance)

Giai đoạn này **chỉ dùng thông tin công khai hoặc trong phạm vi được cấp phép kiểm thử**.

- `whois <domain>`: thông tin đăng ký tên miền.
- `dig <domain>` / `nslookup <domain>`: tra cứu DNS.
- `theHarvester -d <domain> -b all`: thu thập email, subdomain từ nguồn công khai.
- `recon-ng`: framework thu thập thông tin theo module, có thể lưu kết quả vào workspace.

## 6. Quét mạng và dò quét dịch vụ

**Nmap** là công cụ trung tâm:

```bash
nmap -sV -O 192.168.56.0/24        # quét dải mạng, nhận diện service + OS
nmap -p- -T4 192.168.56.10          # quét toàn bộ 65535 cổng
nmap --script vuln 192.168.56.10    # script có sẵn để gợi ý lỗ hổng
```

Dùng `-T4` để tăng tốc độ quét trong lab, giảm xuống `-T2`/`-T1` nếu cần quét "êm" hơn.

## 7. Phân tích lỗ hổng

- `nmap --script vuln`: bước đầu, dựa trên script NSE có sẵn.
- `nikto -h <url>`: quét nhanh lỗi cấu hình và lỗ hổng phổ biến trên web server.
- `OpenVAS`/`Greenbone`: quét lỗ hổng toàn diện hơn nhưng nặng tài nguyên, cần cài thêm.

Ở bước này, mục tiêu là lập danh sách lỗ hổng tiềm năng và mức độ nghiêm trọng (CVSS), chưa khai thác.

## 8. Khai thác (chỉ trong lab được phép)

**Metasploit Framework** là bộ khung phổ biến nhất:

```bash
msfconsole
search <tên lỗ hổng/CVE>
use <đường dẫn module>
show options
set RHOSTS <IP mục tiêu>
run
```

**Burp Suite** cho kiểm thử ứng dụng web: bắt request qua proxy, sửa tham số, kiểm tra input validation, kiểm thử SQLi/XSS bằng tay hoặc Intruder.

Lưu ý: nội dung hướng dẫn ở mức khái niệm và quy trình chung. Chi tiết khai thác một lỗ hổng cụ thể nên học qua các lab có thiết kế sẵn (HackTheBox, TryHackMe, VulnHub, PortSwigger Web Security Academy), nơi mục tiêu được dựng riêng để luyện tập hợp pháp.

## 9. Sau khai thác và báo cáo

- Ghi lại mọi bước: lệnh đã chạy, thời gian, kết quả, ảnh chụp màn hình.
- Đánh giá mức độ ảnh hưởng, không chỉ dừng ở "khai thác thành công".
- Viết báo cáo gồm: tóm tắt, phạm vi, phương pháp, danh sách lỗ hổng (mức độ, bằng chứng), khuyến nghị khắc phục.
- Dọn dẹp mọi thay đổi đã tạo ra trên hệ thống mục tiêu (tài khoản tạm, file test, backdoor nếu có trong bài tập).

## 10. Viết script và tự động hóa

- **Bash**: phù hợp để nối nhiều lệnh có sẵn (ví dụ: chạy nmap rồi lọc kết quả bằng `grep`/`awk`).
- **Python**: dùng khi cần xử lý dữ liệu phức tạp hơn, có thư viện như `requests`, `scapy`, `pwntools`.

Ví dụ script Bash đơn giản, quét nhanh một danh sách host:

```bash
#!/bin/bash
while read -r host; do
    echo "== $host =="
    nmap -sV -oG - "$host" | grep "open"
done < hosts.txt
```

## 11. Thực hành an toàn và pháp lý

- Chỉ kiểm thử hệ thống bạn sở hữu hoặc có văn bản/phạm vi cho phép (scope of engagement) rõ ràng.
- Dùng VM cô lập (NAT/Host-only network) khi luyện tập, tránh để công cụ quét lan ra mạng thật ngoài ý muốn.
- Snapshot VM trước khi thử nghiệm để dễ khôi phục.
- Tìm hiểu luật an ninh mạng tại Việt Nam (Luật An toàn thông tin mạng, Luật An ninh mạng) trước khi thực hành ngoài phạm vi lab cá nhân.

## Tài nguyên học thêm

- Tài liệu chính thức: kali.org/docs
- Lab thực hành: TryHackMe, HackTheBox, VulnHub, PortSwigger Web Security Academy
- Sách: "The Web Application Hacker's Handbook", "Penetration Testing: A Hands-On Introduction to Hacking"
