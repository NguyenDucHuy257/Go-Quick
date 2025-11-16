# Tool Xuất Hóa Đơn Điện Tử - Hướng Dẫn Sử Dụng



##  Mục Lục

1. [Yêu Cầu Hệ Thống](#yêu-cầu-hệ-thống)
2. [Cài Đặt](#cài-đặt)
3. [Cấu Trúc Thư Mục](#cấu-trúc-thư-mục)
4. [Hướng Dẫn Sử Dụng](#hướng-dẫn-sử-dụng)
5. [Các Hàm Chính](#các-hàm-chính)
6. [Cấu Trúc Return Data](#cấu-trúc-return-data)
7. [Ví Dụ Thực Tế](#ví-dụ-thực-tế)
8. [Xử Lý Lỗi](#xử-lý-lỗi)
9. [Các Thư Mục Output](#các-thư-mục-output)

---


##  Cài Đặt

### 1. Cài Đặt Python Dependencies

```bash
# Từ thư mục project
pip install -r requirements.txt
```

### 2. Các Package Chính

```
requests >= 2.28.0          # HTTP client
openpyxl >= 3.8.0          # Excel generation
playwright >= 1.40.0       # HTML → PDF conversion
PyQt5 >= 5.15.0            # Image processing (CAPTCHA)
```

### 3. Cài Đặt Playwright Browsers

```bash
# Run once để cài Chromium, Firefox, WebKit
playwright install chromium
```

---

## Cấu Trúc Thư Mục

```
Tool Mia Pro 2025 SV2 Vip version 3.6.9 CHUAN/
├── main.py                          # Entry point chính
├── README.md                        # File hướng dẫn này
├── requirements.txt                 # Dependencies
│
├── backend_/                        # Thư mục backend services
│   ├── base_service.py             # Base class (SSL, session)
│   ├── auth_service.py             # Authentication & CAPTCHA
│   └── backend_service.py          # Core business logic
│
├── InvoiceBackend.py               # Wrapper layer (task-based interface)
│
├── output/                         # Output files (tự động tạo)
│   ├── excel/                      # Excel files
│   ├── xmlhtml/                    # XML/HTML ZIP files
│   └── pdf/                        # PDF ZIP files
│
├── captcha/                        # CAPTCHA images (tự động tạo)
│   └── captcha.svg                # SVG CAPTCHA
│
├── __pycache__/
│   ├── template/                  # Excel templates
│   │   ├── Thống kê tổng quát.xlsx
│   │   └── Chi tiết hóa đơn.xlsx
│   ├── cache_/                    # Cache data
│   └── ...
│
└── temp/                          # Temporary files (auto cleanup)
```

---

## Hướng Dẫn Sử Dụng

### Bước 1: 

1. Nhìn chung, sẽ lấy raw data từ tải tổng quát trước , sau đó tái chế để tải chi tiết , .... 
2. Chạy theo các mục như trong main.py

### Bước 2: Nhập CAPTCHA

Chương trình sẽ:
1. Lấy CAPTCHA từ API
2.  Lưu vào `captcha/captcha.png`
3. Hiển thị ảnh (hoặc mở folder `captcha/`)
4. Chờ nhập giá trị CAPTCHA

```
Nhập giá trị captcha: [VUI LÒNG NHẬP GIÁ TRỊ CAPTCHA]
```

### Bước 3: Đăng Nhập

```python
username = "0106374735" #Tài khoản đăng nhập web https://hoadondientu.gdt.gov.vn/ ,nhận từ client
password = "SLAT1aA@" 
```

### Bước 4: Chọn Loại Dữ Liệu & Ngày
```python
task = {
    "headers": headers,
    "type_invoice": 1,        # 1: Hóa đơn bán ra, 2: Hóa đơn mua vào
    "start_date": "01/01/2025",
    "end_date": "05/01/2025",
}
```

### Bước 5: Output Files
- Xuất Excel (tổng quát + chi tiết)
- Nén XML/HTML thành ZIP
- Convert HTML → PDF, nén thành ZIP
- Lưu tất cả vào thư mục `output/`

---

## Các Hàm Chính

### 1. **call_tongquat()** - Tổng Quát Hóa Đơn

**Mục đích:** Xuất báo cáo tổng quát các hóa đơn

```python
result = begoinv.call_tongquat(task)
```

**Return:**
```json
{
  "status": "success",
  "message": "Hoàn tất tải thống kê tổng quát 10/10 hóa đơn",
  "data": {
    "excel_bytes": "<base64>",
    "filename": "Thong_ke_tong_quat.xlsx",
    "total_records": 10
  },
  "datas": [...]  // Chi tiết từng hóa đơn
}
```

---

### 2. **call_chitiet()** - Chi Tiết Hóa Đơn

**Mục đích:** Xuất danh sách chi tiết từng dòng hóa đơn

```python
result = begoinv.call_chitiet(result_tongquat) #tái chế results từ tổng quát đã crawl
```

**Return:**
```json
{
  "status": "success",
  "message": "Hoàn tất tải chi tiết 10 hóa đơn",
  "data": {
    "excel_bytes": "<base64>",
    "filename": "Chi_tiet_hoa_don.xlsx",
    "total_records": 10
  },
  "raw_data": [...]
}
```

---

### 3. **call_xmlahtml()** - Xuất XML/HTML

**Mục đích:** Tải XML/HTML của hóa đơn, nén vào ZIP

```python
options = {
    "xml": True,   # Tải XML
    "html": True   # Tải HTML
}
result = begoinv.call_xmlahtml(result_tongquat, options)
```

**Return:**
```json
{
  "status": "success",
  "message": "Hoàn tất tải xml/html 10 hóa đơn",
  "xml_list": [...],
  "html_list": [...],
  "data": {
    "zip_bytes": "<base64>",
    "filename": "invoices_xmlhtml.zip",
    "total_xml": 10,
    "total_html": 10
  }
}
```

**ZIP Structure:**
```
invoices_xmlhtml.zip/
├── xml/
│   ├── C25TMD_1.xml
│   ├── C25TMD_2.xml
│   └── ...
└── html/
    ├── C25TMD_1.html
    ├── C25TMD_2.html
    └── ...
```

---

### 4. **getpdf()** - Xuất PDF

**Mục đích:** Convert HTML → PDF, nén thành ZIP

```python
result = begoinv.getpdf(result_tongquat)
```

**Process:**
1. Gọi `call_xmlahtml()` để lấy HTML
2. Dùng Playwright (headless Chrome) convert HTML → PDF
3. Nén tất cả PDFs thành ZIP
4. Return base64-encoded ZIP

**Return:**
```json
{
  "status": "success",
  "message": "Hoàn tất chuyển 10/10 PDF",
  "data": {
    "zip_bytes": "<base64>",
    "filename": "invoices_pdf.zip",
    "total_pdf": 10
  },
  "pdf_list": [
    {
      "khhdon": "C25TMD",
      "shdon": 1,
      "khmshdon": 1,
      "filename": "C25TMD_1.pdf"
    },
    ...
  ]
}
```

**ZIP Structure:**
```
invoices_pdf.zip/
├── C25TMD_1.pdf
├── C25TMD_2.pdf
└── ...
```

---

## Cấu Trúc Return Data

### Hóa Đơn Object (từ `datas`):

```json
{
  "nbmst": "0106374735",           // Mã số thuế người mua
  "khhdon": "C25TMD",               // Ký hiệu hóa đơn
  "shdon": 1,                       // Số thứ tự hóa đơn
  "khmshdon": 1,                    // Ký hiệu mẫu số hóa đơn
  "ttxly": 1,                       // Trạng thái xử lý (1=sold, 8=sco-sold)
  "tdlap": "2025-01-05T10:30:00",   // Thời điểm lập
  "tbao": "2025-01-05T10:35:00",    // Thời điểm báo cáo
  "ghi_chu": "Ghi chú...",          // Ghi chú
  "tong_tien": 1000000,             // Tổng tiền
  "tong_thue": 100000,              // Tổng thuế
  ...
}
```

### XML/HTML Item (từ `xml_list` / `html_list`):

```json
{
  "khhdon": "C25TMD",
  "shdon": 1,
  "khmshdon": 1,
  "xml_content": "<xml>...</xml>" hoặc "<html>...</html>"
}
```

### PDF Item (từ `pdf_list`):

```json
{
  "khhdon": "C25TMD",
  "shdon": 1,
  "khmshdon": 1,
  "index": 1,
  "filename": "C25TMD_1.pdf",
  "pdf_bytes": <binary>
}
```

---

##  Ví Dụ Thực Tế

### Ví dụ 1: Xuất Excel Tổng Quát

```python
from InvoiceBackend import InvoiceBackend

begoinv = InvoiceBackend()

# 1. CAPTCHA & Login
ckey, captcha_path = begoinv.get_and_save_captcha()
cvalue = input("Nhập CAPTCHA: ")
headers = begoinv.login(username, password, ckey, cvalue)

# 2. Xuất Excel tổng quát
task = {
    "headers": headers,
    "type_invoice": 1,
    "start_date": "01/01/2025",
    "end_date": "31/01/2025"
}
result = begoinv.call_tongquat(task)

# 3. Lưu file
if result['status'] == 'success':
    import base64
    excel_bytes = base64.b64decode(result["data"]["excel_bytes"])
    with open("output/tong_quat.xlsx", 'wb') as f:
        f.write(excel_bytes)
    print(f"✓ Đã lưu: {result['data']['filename']}")
```

### Ví dụ 2: Xuất PDF từ HTML ( Chức năng tải PDF THƯỜNG TRÊN WEB)

```python
# 1. Lấy tổng quát (để có dữ liệu)
result_tongquat = begoinv.call_tongquat(task)

# 2. Convert HTML → PDF
result_pdf = begoinv.getpdf(result_tongquat)

# 3. Lưu ZIP PDF
if result_pdf['status'] == 'success':
    import base64
    pdf_zip_bytes = base64.b64decode(result_pdf["data"]["zip_bytes"])
    with open("invoices_pdf.zip", 'wb') as f:
        f.write(pdf_zip_bytes)
    print(f"✓ Đã lưu: {result_pdf['data']['filename']}")
```

### Ví dụ 3: Xuất XML/HTML + Lưu

```python
result_xmlhtml = begoinv.call_xmlahtml(result_tongquat, {
    "xml": True,
    "html": True
})

if result_xmlhtml['status'] == 'success':
    import base64
    import os
    
    zip_bytes = base64.b64decode(result_xmlhtml["data"]["zip_bytes"])
    with open("invoices_xmlhtml.zip", 'wb') as f:
        f.write(zip_bytes)
    
    # In ra thông tin
    print(f"✓ Total XML: {result_xmlhtml['data']['total_xml']}")
    print(f"✓ Total HTML: {result_xmlhtml['data']['total_html']}")
```

---

## ⚠️ Xử Lý Lỗi

### Lỗi CAPTCHA Sai

- Tool trả về message Phiên đăng nhập hết hạn
### Lỗi Timeout

- **Nguyên nhân:** API chậm (thường xảy ra khi lấy dữ liệu lớn)
- **Giải pháp:**
  Tool tự tăng time delay lên sau N lần fail 

### Lỗi Playwright / PDF Conversion

- **Nguyên nhân:** HTML không hợp lệ hoặc Chromium lỗi
- **Giải pháp:**
  ```bash
  playwright install chromium
  ```

### Lỗi Login Fail

- **Nguyên nhân:** ckey hết hạn, ...
- **Giải pháp:**
  Đăng nhập lại

---

## Workflow Tổng Quan

```
┌─────────────────────────────────────────────────────┐
│ 1. Chạy main.py                                     │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│ 2. Lấy CAPTCHA → Nhập giá trị → Login              │
└──────────────────┬──────────────────────────────────┘
                   │
       ┌───────────┴───────Tổng quát-raw data─┐
       │                       │              │
┌──────▼──────┐      ┌────────▼─────┐  ┌────▼──────┐
│ Tổng Quát   │      │ Chi Tiết     │  │ XML/HTML  │
│ (Excel)     │      │ (Excel)      │  │ + PDF     │
└──────┬──────┘      └────────┬─────┘  └────┬──────┘
       │                      │             │
       └──────────────────────┼─────────────┘
                              │
                    ┌─────────▼────────┐
                    │ Lưu output/      │
                    │ ✓ excel/         │
                    │ ✓ xmlhtml/       │
                    │ ✓ pdf/           │
                    └──────────────────┘
```

---

