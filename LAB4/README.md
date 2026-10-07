# BÁO CÁO THỰC HÀNH AN TOÀN HỆ THỐNG THÔNG TIN
## BÀI LAB 4: KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

---

### THÔNG TIN SINH VIÊN
* **Họ và tên sinh viên:** HUỲNH MINH KHOA
* **Mã số sinh viên:** 1150080139
* **Lớp:** 11_CNPM2
* **Link Video Demo YouTube:** [https://www.youtube.com/watch?v=abkUv1z0y6A](https://www.youtube.com/watch?v=abkUv1z0y6A)
* **Link GitHub Repository:** [https://github.com/KhoaMinh8386/LAB_AT_BMHTTT/tree/main/LAB4](https://github.com/KhoaMinh8386/LAB_AT_BMHTTT/tree/main/LAB4)

---

## 1. MỤC TIÊU VÀ PHẠM VI BÀI THỰC HÀNH
* Thiết lập mô hình mạng ảo cô lập an toàn Host-Only trên môi trường VMware Workstation.
* Thực hiện rà soát, thu thập thông tin mạng (Host Discovery) và xác định đúng địa chỉ IP của máy quét và máy mục tiêu.
* Nắm vững cơ chế, thực thi và so sánh các kỹ thuật quét cổng TCP/UDP (Connect, SYN Stealth, FIN, Xmas, NULL, ACK scan).
* Khảo sát dịch vụ chuyên sâu: nhận diện phiên bản phần mềm (-sV), fingerprinting hệ điều hành (-O, -A) và kiểm tra lỗ hổng bằng Nmap Scripting Engine (NSE).
* Xây dựng hồ sơ bằng chứng đánh giá (Evidence Files) dưới các định dạng Text, Grepable, XML và HTML.
* Đánh giá và đối chiếu trạng thái cổng trước và sau khi thực hiện các giải pháp làm vững chắc hệ thống (Hardening).

---

## 2. THIẾT LẬP MÔI TRƯỜNG THỰC HÀNH

### Bảng 4.2: Thông tin thiết bị trong mô hình mạng Host-Only (VMnet1)
| Thiết bị | Địa chỉ IP thực tế | Subnet Mask | Vai trò trong bài Lab |
| :--- | :--- | :--- | :--- |
| **Windows Host (Máy thật)** | `192.168.157.1` | `255.255.255.0` | Máy trạm quản trị / Card mạng ảo VMnet1 |
| **Kali Linux VM** | `192.168.157.133` | `255.255.255.0` | Máy kiểm thử / Máy quét chính (Attacker) |
| **Metasploitable 2 VM** | `192.168.157.134` | `255.255.255.0` | Máy chủ mục tiêu có lỗ hổng (Target) |

---

## 3. KẾT QUẢ THỰC HIỆN VÀ PHÂN TÍCH

### 3.1. Nhiệm vụ 1: Phát hiện các host đang hoạt động (Mục 5.1)
* **Câu lệnh thực thi:**
  ```bash
  sudo nmap -sn 192.168.157.0/24
