# Bài 1: Khảo sát FHS và Phân quyền File/Folder nâng cao

## 1. Mục tiêu
- Tạo cấu trúc thư mục web theo chuẩn FHS tại `/var/www/my-app`.
- Áp dụng phân quyền octal (750, 770) và đổi sở hữu `chown`.
- Kiểm tra bằng `ls -la` và `stat`.

## 2. Môi trường thực hiện
- OS: Ubuntu 26.04 LTS (WSL - JACKPOT)
- User non-root: `duong`
- Group web: `www-data`

## 3. Nhật ký lệnh đã thực hiện

```bash
# Kiểm tra môi trường
pwd
lsb_release -a
whoami
# /home/duong
# Ubuntu 26.04 LTS
# duong

# 3.1 Tạo cấu trúc thư mục (cần sudo vì /var là của root)
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs

# 3.2 Kiểm tra nhóm www-data (tạo nếu chưa có)
getent group www-data || sudo groupadd www-data

# 3.3 Gán quyền octal
# public: Owner rwx, Group r-x, Other --- => 750
sudo chmod 750 /var/www/my-app/public
# logs: Owner rwx, Group rwx, Other --- => 770
sudo chmod 770 /var/www/my-app/logs

# Cách tương đương bằng symbolic (tham khảo):
# sudo chmod u=rwx,g=rx,o= /var/www/my-app/public
# sudo chmod u=rwx,g=rwx,o= /var/www/my-app/logs

# 3.4 Đổi chủ sở hữu về user thường + nhóm web
sudo chown -R duong:www-data /var/www/my-app
```

## 4. Kiểm tra

Lệnh:
```bash
ls -la /var/www/my-app
stat -c '%a %A %U %G %n' /var/www/my-app/public /var/www/my-app/logs
```

Đầu ra thực tế:
```text
duong@JACKPOT:~$ ls -la /var/www/my-app
total 16
drwxr-xr-x 4 duong www-data 4096 Oct  5 05:42 .
drwxr-xr-x 5 root  root     4096 Oct  5 05:41 ..
drwxrwx--- 2 duong www-data 4096 Oct  5 05:42 logs
drwxr-x--- 2 duong www-data 4096 Oct  5 05:41 public

duong@JACKPOT:~$ stat -c '%a %A %U %G %n' /var/www/my-app/public /var/www/my-app/logs
750 drwxr-x--- duong www-data /var/www/my-app/public
770 drwxrwx--- duong www-data /var/www/my-app/logs
```

## 5. Đối chiếu yêu cầu

| Thư mục | Yêu cầu | Thực tế | Đạt |
|---------|---------|---------|-----|
| `public` | `drwxr-x---` (750), owner `duong`, group `www-data` | `750 drwxr-x--- duong www-data` | ✅ |
| `logs` | `drwxrwx---` (770), owner `duong`, group `www-data` | `770 drwxrwx--- duong www-data` | ✅ |

> Ghi chú: đề ghi `drwxrwxr--` cho logs là typo, chuẩn phải là `drwxrwx---` tương đương 770.

## 6. Giải thích

- `/var/www` là vị trí chuẩn FHS chứa dữ liệu web (DocumentRoot).
- `750 = rwxr-x---`: Owner `rwx` (đọc/ghi/cd), Group `r-x` (đọc + cd vào, không ghi), Other `---` (không quyền).
- `770 = rwxrwx---`: Owner + Group `rwx` (toàn quyền), Other `---`.
- Bit `x` trên thư mục = quyền điều hướng vào (`cd`) và liệt kê khi kết hợp với `r`.
- `chown -R duong:www-data`: gán chủ sở hữu là tài khoản non-root `duong`, nhóm là `www-data` để web server (nginx/apache) cùng nhóm có thể đọc/ghi logs.
- `chmod` đổi quyền, `chown` đổi sở hữu, cần `sudo` vì `/var/www` thuộc root.
