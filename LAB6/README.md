# LAB 6 – pfSense: Active Directory, Captive Portal, Alias, Schedule & Firewall Rule

**Môn:** An toàn hệ thống thông tin

| | |
|---|---|
| Họ và tên | HUỲNH MINH KHOA |
| MSSV | 1150080139 |
| Lớp | 11_CNPM2 |
| Video demo | https://youtu.be/MAjqAeBWHVg |
| GitHub | https://github.com/KhoaMinh8386/LAB_AT_BMHTTT/tree/main/LAB6 |

## 1. Mục tiêu

- Dựng pfSense với 4 interface: **WAN, MGMT, LAN, DMZ** trên VMware Workstation.
- Chuẩn bị Domain Controller **DC01** (Windows Server 2022, AD DS + DNS).
- Tích hợp xác thực **LDAP** (Active Directory) và **Captive Portal** trên LAN.
- Tạo **Alias**, **Schedule** và viết **Firewall Rule** cho các kịch bản truy cập web.
- Kiểm tra bằng Client-A / Client-B, đọc **Firewall Log** và **State Table**.

## 2. Sơ đồ địa chỉ

| Thành phần | Địa chỉ | Vai trò |
|---|---|---|
| WAN | DHCP (NAT) | Ra ngoài |
| MGMT | 192.168.56.254/24 | Quản trị WebGUI (Host-only) |
| LAN | 10.6.0.1/24 | Gateway người dùng, Captive Portal |
| DC01 | 10.6.0.10/24 | AD DS, DNS |
| Client-A / Client-B | 10.6.0.21 / 10.6.0.22 | Máy kiểm thử |
| DMZ | 172.16.6.1/24 | Gateway DMZ |
| DMZ-WEB | 172.16.6.10 / .20 / .30 / .40 | General / Job / Social / Restricted |

## 3. Ánh xạ adapter máy ảo pfSense

| Adapter | Mạng VMware | Interface |
|---|---|---|
| 1 | NAT (VMnet8) | WAN |
| 2 | Host-only (VMnet2) | MGMT (OPT1) |
| 3 | LAN Segment `lab6-lan` | LAN |
| 4 | LAN Segment (DMZ) | DMZ (OPT2) |

## 4. Cấu hình chính

**LDAP (System > User Manager > Authentication Servers):** `LAB6-AD`, LDAP, `10.6.0.10:389`, TCP Standard, v3, Entire Subtree, Base DN `DC=lab6,DC=test`, container `OU=LAB6-Users,...`, bind bằng `svc_pfsense_ldap`, thuộc tính `samAccountName`.

**Captive Portal:** zone `LAB6_USERS` trên LAN, xác thực backend `LAB6-AD`.

**Alias:** `CLIENT_A`, `CLIENT_B`, `POLICY_CLIENTS`, `GENERAL_SITE`, `JOB_SITE`, `SOCIAL_SITE`, `RESTRICTED_SITE`, `WEB_PORTS (80, 443)`.

**Schedule:** `LUNCH_WINDOW` (11:30–13:00, T2–T6), `WORK_HOURS` (08:00–17:00, T2–T6), `TEST_NOW` (thử nhanh, xóa sau khi dùng).

## 5. Bộ rule trên tab LAN (first-match, từ trên xuống)

| # | Action | Source → Destination | Port | Schedule |
|---|---|---|---|---|
| 1 | Block + Log | CLIENT_A → JOB_SITE | TCP/80 | – |
| 2 | Block + Log | POLICY_CLIENTS → RESTRICTED_SITE | TCP/80 | – |
| 3 | Pass | CLIENT_B → SOCIAL_SITE | TCP/80 | LUNCH_WINDOW |
| 4 | Block | CLIENT_B → SOCIAL_SITE | TCP/80 | – |
| 5 | Pass | POLICY_CLIENTS → GENERAL_SITE | TCP/80 | – |
| 6 | Pass | POLICY_CLIENTS → Any | WEB_PORTS | WORK_HOURS |
| 7 | Block + Log | POLICY_CLIENTS → Any | Any | – |

Tab DMZ: `Block DMZ net → LAN net`.

> Quy tắc: Block đặc hiệu → Pass ngoại lệ → Pass chung → Block mặc định. Rule Pass có Schedule hết hiệu lực **không** tự thành Block, nên cần Block ngay bên dưới.

## 6. Cách kiểm thử

1. Client-A mở `http://172.16.6.20` → **bị chặn**; `http://172.16.6.10` → vào được.
2. Client-A/B mở `http://172.16.6.40` → **bị chặn**.
3. Client-B mở `http://172.16.6.30` trong giờ nghỉ → được; ngoài giờ → bị chặn.
4. Trong giờ hành chính: web 80/443 được, dịch vụ khác bị chặn.
5. Máy trong DMZ không truy cập được LAN.
6. So sánh **Block** (chờ timeout) và **Reject** (báo lỗi nhanh).
7. Xem `Status > System Logs > Firewall` và `Diagnostics > States`; **Apply Changes + Reset States** sau mỗi lần đổi rule.

## 7. Trạng thái minh chứng

| Hạng mục | Trạng thái |
|---|---|
| Mạng ảo, 4 adapter, IP LAN/MGMT/DMZ | Có ảnh |
| Ping 192.168.56.254 (4/4, 0% loss), WebGUI | Có ảnh |
| DC01 `ipconfig`, ping gateway, Server Manager | Có ảnh |
| 7 rule LAN và 2 rule DMZ | Có ảnh cấu hình (States 0/0 B, chưa có lưu lượng khớp) |
| LDAP Test, Captive Portal, Alias, Schedule | **Chưa có ảnh** |
| Kiểm thử kịch bản 1–7 | **Chưa có ảnh** |

## 8. Điểm cần xác minh

- Server Manager hiển thị miền **`vietnam.local`**, đề yêu cầu **`lab6.test`** → Base DN trên pfSense phải khớp miền thực tế.
- Rule `Allow_MGMT_WebGUI` xuất hiện trên tab tên “DMZ”; kiểm tra tên interface MGMT/DMZ.
- Dải mạng ảo VMware (192.168.252.0, 192.168.130.0) khác dải mẫu của đề.
- Đổi mật khẩu admin mặc định; LDAP 389 không mã hóa chỉ dùng trong lab, thực tế dùng LDAPS/StartTLS.

## 9. Tệp

- `Lab6_11CNPM2_1150080139_Huynh_MinhKhoa.docx` – báo cáo đầy đủ (kèm ảnh, 12 câu phân tích).
- `README.md` – tóm tắt này.
