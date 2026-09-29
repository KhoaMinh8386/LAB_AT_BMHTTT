# BÁO CÁO THỰC HÀNH LAB 3: NHẬN DIỆN VÀ ỨNG PHÓ CÁC MỐI ĐE DỌA ĐẾN AN TOÀN THÔNG TIN

## 1. Thông tin sinh viên
* **Họ và tên:** Huỳnh Minh Khoa
* **Mã số sinh viên (MSSV):** *1150080139*
* **Lớp:** *K11-CNPM2*
* **Môn học:** Thực hành An toàn Hệ thống Thông tin
* **Học kỳ / Năm học:** Học kỳ 1 – Năm học 2026 - 2027

---

## 2. Thông tin môi trường thực hành
* **Phần mềm ảo hóa:** VMware Workstation Pro 26H1 (Chế độ card mạng: Host-only)
* **Hệ điều hành máy ảo (VM):** Windows 11 25H2 x64, OS Build 26200.9445 (Bản vá KB5124008)
* **Shell thực thi:** Windows PowerShell 5.1 (Run as administrator)
* **Endpoint Protection:** Microsoft Defender Antivirus tích hợp sẵn (Real-time protection & Tamper Protection: Enabled)
* **Bộ công cụ chuẩn hóa:**
  * Microsoft Sysmon: v15.22 (Schema 4.90)
  * Microsoft Sysinternals Autoruns: v14.3
  * Microsoft Sysinternals Process Explorer: v17.14
  * Wireshark: v4.6.8 Stable + Npcap
  * Python: v3.14.7

---

## 3. Cách dựng môi trường (Environment Setup)
1. **Khởi tạo máy ảo:** Thiết lập VM Windows 11 với 2 vCPU, 6 GB RAM, 64 GB Disk; gán Network Adapter ở chế độ `Host-only`; cập nhật KB5124008 và tạo snapshot sạch `LAB3_CLEAN_20260914`.
2. **Khởi tạo thư mục làm việc:** Sử dụng PowerShell tạo cây thư mục chuẩn `C:\LAB3\Evidence`, `C:\LAB3\Tools`, `C:\LAB3\Downloads`, `C:\LAB3\Assets` và ghi nhận mốc thời gian bắt đầu vào `start_time.txt`.
3. **Giải nén tài nguyên:** Chép gói `LAB3_Threats_Assets.zip` vào `C:\LAB3\Downloads`, đối chiếu khớp mã SHA-256 (`96236f95ce59d0cc37f9b7ab8fbb04522f21870cd0b53aad6e52da5a23655439`), sau đó giải nén vào `C:\LAB3`.
4. **Cài đặt công cụ:**
   * Cài đặt Python 3.14.7 và Wireshark 4.6.8 qua công cụ `winget`.
   * Tải trực tiếp các gói zip từ `download.sysinternals.com` và giải nén tương ứng vào `C:\LAB3\Tools`.
5. **Thu thập Baseline ban đầu:** Xuất thông tin hệ điều hành, Defender, Firewall, IP cấu hình mạng và tiến trình đang chạy vào các file text trong `Evidence/`.

---

## 4. Các tình huống đã thực hiện & Kết quả đánh giá

| Tình huống | Nội dung kỹ thuật | Kết quả | Bằng chứng & Ghi chú |
| :--- | :--- | :---: | :--- |
| **TH1** | Xác định chuỗi Asset - Vulnerability - Threat - Risk - Control & Phân loại 5 nhóm nguồn đe dọa | **PASS** | Lập đủ 5 hàng tài sản/nguy cơ trong Risk Register; phân loại đúng 5 tình huống lý thuyết có kèm giải thích. |
| **TH2** | Kiểm chứng chu trình phát hiện mã độc bằng tệp chuẩn EICAR | **PASS** | Defender phát hiện và cô lập tệp `eicar.com.txt` theo thời gian thực; log ghi nhận trong Protection History (`H4_ProtectionHistory_EICAR.png`). |
| **TH3** | Phân tích xác thực Event ID 4624, 4625, 4648 & Xoay vòng mật khẩu (Credential Rotation) | **PASS** | Bật Audit Logon; sinh sự kiện đăng nhập đúng và sai của user `lab3user`; mật khẩu cũ mất hoàn toàn hiệu lực sau khi đổi (`H5_Event4625.png`). |
| **TH4** | Nhận diện kỹ thuật duy trì (Persistence) & Dịch vụ lắng nghe (Listener) qua Sysmon, Autoruns, Process Explorer | **PASS** | Cài Sysmon 15.22; phát hiện khóa Run (`LAB3_Run_Demo`) trên Autoruns tab Logon; ánh xạ listener port 8080 (127.0.0.1) về đúng PID `python.exe` (`H6`, `H7`, `H8`). |
| **TH5** | Sniffing lưu lượng mạng: So sánh bản rõ HTTP và mã hóa HTTPS (TLS/443) trên Wireshark | **PASS** | Bắt gói tin loopback đọc được tham số `TRAINING_ONLY` ở HTTP GET; lưu lượng HTTPS chỉ hiển thị metadata và payload đã mã hóa (`H9`, `H10`). |
| **TH6** | Phân tích tấn công từ chối dịch vụ DoS/DDoS và Mail Bombing | **PASS** | Chạy script đo tải nội bộ trên `127.0.0.1:8080` (50 request/5 worker); phân tích dataset offline TEST-NET nhận diện nguồn phân tán; thống kê dung lượng và tần suất email bất thường. |
| **TH7** | Phân tích kỹ thuật Kỹ nghệ xã hội (Social Engineering) và Phishing Email offline | **PASS** | Xác định đủ 5 chỉ dấu lừa đảo trong `phishing_email.txt`; phân loại chính xác 6 trường hợp xã hội kỹ thuật trong `social_engineering_cases.csv`. |
| **Cleanup** | Cô lập, dọn dẹp artefact, kiểm tra phục hồi hệ thống và băm bằng chứng | **PASS** | Xóa sạch Run key, Task, User lab, dừng web listener; xuất mã băm `evidence_sha256.csv` đầy đủ (`H11_Recovery_Verification.png`). |

---

## 5. Lỗi kỹ thuật gặp phải và cách khắc phục

1. **Lỗi không nhận diện cmdlet `Invoke-WebRequest` và `Expand-Archive`:**
   * *Hiện tượng:* Cửa sổ dòng lệnh báo lỗi `'Invoke-WebRequest' is not recognized as an internal or external command...`
   * *Nguyên nhân:* Mở nhầm cửa sổ Command Prompt (cmd.exe) thay vì PowerShell.
   * *Khắc phục:* Gõ lệnh `powershell` trực tiếp tại cửa sổ hiện tại để chuyển sang môi trường PowerShell rồi mới thực thi các lệnh tải và giải nén.

2. **Lỗi `OpenError: DirectoryNotFoundException` khi lưu file Baseline:**
   * *Hiện tượng:* Lệnh xuất log báo lỗi không tìm thấy đường dẫn `C:\LAB3\Evidence\baseline_*.txt`.
   * *Nguyên nhân:* Thư mục `C:\LAB3\Evidence` chưa được tạo thực tế trên phân vùng ổ đĩa.
   * *Khắc phục:* Khởi tạo thư mục tường minh bằng lệnh: `New-Item -ItemType Directory -Force -Path 'C:\LAB3\Evidence' | Out-Null`.

3. **Lỗi dịch vụ đã tồn tại khi cài Sysmon (`Sysmon64 is already registered`):**
   * *Hiện tượng:* Lệnh cài đặt Sysmon với cờ `-i` báo lỗi do service Sysmon64 đã được đăng ký từ trước.
   * *Nguyên nhân:* Môi trường máy ảo đã có service Sysmon đang chạy ngầm.
   * *Khắc phục:* Chuyển sang dùng cờ `-c` để nạp trực tiếp file cấu hình quy tắc của bài lab: `Sysmon64.exe -c C:\LAB3\lab3_assets\sysmon-lab.xml`.

---

## 6. Cấu trúc thư mục nộp bài trên GitHub (`LAB3/`)

```text
LAB3/
├── README.md                          # Tài liệu tổng quan và báo cáo tóm tắt
├── [MãLớp]-LAB3_MSSV-HoTen.docx        # Báo cáo chi tiết và 20 câu hỏi lý thuyết
├── evidence_sha256.csv                # Bảng mã băm toàn vẹn SHA-256 của toàn bộ bằng chứng
└── Evidence/                          # Thư mục lưu trữ log và ảnh chụp minh chứng
    ├── H1_VM_WindowsVersion.png
    ├── H2_ToolVersions.png
    ├── H3_Baseline_Defender_Firewall.png
    ├── H4_ProtectionHistory_EICAR.png
    ├── H5_Event4625.png
    ├── H6_Sysmon_Event1.png
    ├── H7_Autoruns_LAB3_Run_Demo.png
    ├── H8_ProcessExplorer_Python.png
    ├── H9_HTTP_Plaintext.png
    ├── H10_TLS_443.png
    ├── H10_Load_and_Log_Analysis.png
    ├── H10_Phishing_Offline.png
    ├── H11_Recovery_Verification.png
    ├── baseline_os.txt
    ├── baseline_defender.txt
    ├── baseline_firewall.txt
    ├── baseline_network.txt
    ├── baseline_processes.txt
    ├── defender_eicar.txt
    ├── auth_events_before_rotation.txt
    ├── sysmon_persistence.txt
    ├── local_load_test.txt
    ├── ddos_sources.txt
    ├── mail_sender_counts.txt
    ├── mail_volume.txt
    ├── autoruns_diff.txt
    └── start_time.txt
