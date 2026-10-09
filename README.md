# DBM301 – Assignment 1 – Đề 04 | Nhóm 3

> **Chủ đề:** Khai phá luật kết hợp mô tả phản hồi chiến dịch tiếp thị ngân hàng  
> **Dataset:** UCI Bank Marketing — `bank-full.csv` (45.211 dòng, 17 thuộc tính)  
> **Seed cố định:** `42` (xuyên suốt toàn bộ dự án)

## 🧱 Cấu trúc và nguồn tham khảo

Dự án sử dụng cấu trúc thư mục theo định hướng của **Cookiecutter Data Science (CCDS) phiên bản 2**,
một mẫu tổ chức dự án khoa học dữ liệu có cấu trúc rõ ràng và linh hoạt. Cấu trúc này giúp tách
dữ liệu gốc, dữ liệu trung gian, dữ liệu đã xử lý, notebook, mã nguồn, tài liệu và kết quả phân tích.

Repo này điều chỉnh mẫu CCDS cho yêu cầu bài tập DBM301: dữ liệu được tổ chức trong `data/`,
phân tích và tiền xử lý nằm trong `notebooks/`, mã nguồn dự án trong `dbm301/`, tài liệu trong
`docs/` và hình/báo cáo đầu ra trong `reports/`.

**Nguồn tham khảo:**

- [Cookiecutter Data Science — trang dự án chính thức](https://cookiecutter-data-science.drivendata.org/)
- [Cookiecutter Data Science — mã nguồn và mẫu dự án](https://github.com/drivendataorg/cookiecutter-data-science)
- [Cookiecutter Data Science — hướng dẫn sử dụng mẫu](https://cookiecutter-data-science.drivendata.org/using-the-template/)

---

## ⚡ THÀNH VIÊN MỚI — ĐỌC PHẦN NÀY TRƯỚC (3 phút)

### Bước 1: Clone repo và vào thư mục
```bash
git clone <URL_REPO>
cd dbm301_asm_group3_de4
```

### Bước 2: Tạo môi trường Python và cài thư viện
```powershell
# Windows PowerShell — chạy 1 trong 2 cách:

# Cách A (uv — nhanh hơn, khuyến nghị):
uv sync

# Cách B (pip truyền thống):
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -e .
```

### Bước 3: Đăng ký Kernel Jupyter (chỉ cần làm 1 lần)
```powershell
.venv\Scripts\python.exe -m ipykernel install --user --name=dbm301-venv --display-name "DBM301 (.venv)"
```

### Bước 4: Mở JupyterLab để bắt đầu làm việc
```powershell
.venv\Scripts\jupyter.exe lab
```
> ⚠️ Khi mở notebook, nhớ chọn Kernel là **"DBM301 (.venv)"**, không dùng kernel Python hệ thống!

---

## 📦 DỮ LIỆU — TÔI CẦN TẢI GÌ KHÔNG?

| Bạn cần làm gì? | Câu trả lời |
|---|---|
| Làm **OLAP / Khai phá luật kết hợp** (Task 3.x trở đi) | ✅ **KHÔNG cần tải gì thêm.** Dùng thẳng `data/processed/processed_exploration.csv` đã có sẵn trên repo. |
| Muốn **chạy lại toàn bộ từ đầu** (để học hiểu) | ⬇️ Tải `bank-full.csv` theo hướng dẫn bên dưới. |

### Cách tải dữ liệu gốc (chỉ cần nếu muốn chạy lại từ đầu):
1.  Vào trang: **https://archive.ics.uci.edu/static/public/222/bank+marketing.zip**
2.  Giải nén, lấy file **`bank-full.csv`**
3.  Đặt vào thư mục: **`data/raw/bank-full.csv`**
4.  Xác minh file đúng bằng mã SHA-256 (xem [`data/raw/data_manifest.txt`](data/raw/data_manifest.txt)):
```powershell
Get-FileHash data\raw\bank-full.csv -Algorithm SHA256
# Kết quả phải là: D1513EC63B385506F7CFCE9F2C5CAA9FE99E7BA4E8C3FA264B3AAF0F849ED32D
```

---

## 📁 Cấu trúc thư mục

```
data/
├── raw/
│   ├── bank-full.csv          ← KHÔNG trên Git (tải theo hướng dẫn trên)
│   ├── data_manifest.txt      ← ✅ Có trên Git (nguồn, SHA-256, license)
│   └── manifest.json          ← ✅ Có trên Git
├── interim/
│   ├── exploration_80.csv     ← KHÔNG trên Git (chạy notebook 2.4 để tái tạo)
│   └── test_20.csv            ← KHÔNG trên Git
└── processed/                 ← ✅ TOÀN BỘ CÓ TRÊN GIT (dùng ngay!)
    ├── processed_exploration.csv       (36.168 dòng × 13 cột — cho Mining & OLAP)
    ├── processed_test.csv              (9.043 dòng × 13 cột — cho Validation)
    ├── binning_config.json             (cấu hình ranh giới age/balance/campaign)
    └── preprocessing_decision_table.csv

notebooks/
├── 2.2_2.3_task.ipynb   ← EDA + Bảng quyết định tiền xử lý
├── 2.4_2.5_task.ipynb   ← Chia tập 80/20 + Kiểm tra phân tầng
└── 2.6_2.7_task.ipynb   ← Rời rạc hóa + Xuất dữ liệu sạch

docs/
└── BAO_CAO_CHUNG_MINH_TASK_2.3.md  ← Chứng minh khoa học cho quyết định tiền xử lý

reports/figures/         ← Biểu đồ tự động sinh ra
WBS_De4_ASM_GROUP3_DBM301.xlsx      ← Bảng phân công công việc nhóm
```

---

## 📚 Thư viện chính

| Thư viện | Mục đích |
|---|---|
| `pandas`, `numpy` | Xử lý và phân tích dữ liệu |
| `mlxtend` | Khai phá luật kết hợp (Apriori, FP-Growth) |
| `scikit-learn` | Chia tập, tiền xử lý |
| `matplotlib`, `seaborn` | Trực quan hóa |
| `jupyterlab` | Môi trường notebook |

---

## 📖 Trích dẫn Dữ liệu

Moro, S., Rita, P., & Cortez, P. (2012). *Bank Marketing [Dataset]*. UCI Machine Learning Repository. https://doi.org/10.24432/C5K306

*Giấy phép: Creative Commons Attribution 4.0 International (CC BY 4.0)*
