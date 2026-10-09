# BÁO CÁO CHỨNG MINH CƠ SỞ KHOA HỌC CHO CÁC QUYẾT ĐỊNH TIỀN XỬ LÝ (TASK 2.3)
**Học phần:** DBM301 — Khai phá dữ liệu & Kho dữ liệu  
**Đề bài:** Đề 04 — Khai phá luật kết hợp phản hồi chiến dịch ngân hàng  
**Tập dữ liệu:** UCI Bank Marketing (`bank-full.csv`, 45.211 dòng, 17 thuộc tính)

---

## TỔNG QUAN
Báo cáo này cung cấp **minh chứng thực tế**, **nguồn trích dẫn chính thức**, và **số liệu tính toán trực tiếp từ 45.211 dòng dữ liệu thật** để chứng minh tính đúng đắn cho từng quyết định xử lý biến trong Task 2.3.

---

## 1. NHÓM QUYẾT ĐỊNH BẮT BUỘC THEO ĐỀ BÀI VÀ TÀI LIỆU GỐC

### 1.1. Biến `duration` (Thời lượng cuộc gọi) — QUYẾT ĐỊNH: LOẠI BỎ HOÀN TOÀN
*   **Trích dẫn Đề bài (*Assignment_1_De_04.pdf*, Trang 1, Mục 2, dòng 14-15):**  
    > *"Loại duration khỏi tập item vì chỉ biết sau cuộc gọi."*
*   **Trích dẫn tài liệu gốc UCI Machine Learning & Tác giả công bố (Moro et al., 2014):**  
    > *"Important note: this attribute highly affects the output target... Yet, the duration is not known before a call is performed. Also, after the end of the call y is obviously known. Thus, this input should only be included for benchmark purposes and should be discarded if the intention is to have a realistic model."*  
    *(Nguồn: [UCI Bank Marketing Documentation](https://archive.ics.uci.edu/dataset/222/bank+marketing))*
*   **Minh chứng thực tế:**  
    Nếu nhân viên ngân hàng cầm danh sách khách hàng lên để chuẩn bị gọi điện, họ **hoàn toàn chưa có con số thời lượng cuộc gọi**. Nếu đưa `duration` vào tập luật, mô hình sẽ sinh ra luật hiển nhiên kiểu: `duration > 600s => y = yes` (gian lận rò rỉ thông tin - Data Leakage), vô giá trị trong thực tế.

---

### 1.2. Biến `pdays = -1` và các giá trị `unknown` — QUYẾT ĐỊNH: KHÔNG XÓA DÒNG, KHÔNG TÍNH SỐ ĐO BÌNH THƯỜNG
*   **Trích dẫn Đề bài (*Assignment_1_De_04.pdf*, Trang 1, Mục 3, Yêu cầu 1, dòng 21-22):**  
    > *"Lập bảng xử lý cho từng biến; không coi unknown hoặc -1 là số đo bình thường."*
*   **Số liệu thực tế chạy từ file `bank-full.csv`:**
    *   `pdays = -1`: Có đúng **36.954 dòng (chiếm 81.73%)**.
    *   Tỷ lệ đồng ý gửi tiền (`y=yes`) của nhóm `pdays = -1` là **9.16%**.
    *   Tỷ lệ đồng ý gửi tiền của nhóm `pdays > 0` (đã từng gọi trước đây) là **23.07%** (cao gấp **2.52 lần**!).
*   **Kết luận xử lý:**  
    `-1` không phải là "âm 1 ngày", mà là mã quy ước "chưa từng liên hệ". Việc tách thành biến nhị phân `contacted_before` (`no` nếu -1, `yes` nếu >0) hoàn toàn khớp với bản chất dữ liệu và yêu cầu đề bài.

---

## 2. NHÓM RỜI RẠC HÓA CÁC BIẾN SỐ (DISCRETIZATION)
Thuật toán Apriori yêu cầu dữ liệu dạng các tập hạng mục rời rạc (Agrawal & Srikant, 1994). Số liệu thực nghiệm tính trực tiếp từ 45.211 dòng giải thích cho từng ranh giới cắt như sau:

### 2.1. Biến `age` (Độ tuổi) — QUYẾT ĐỊNH: Chia thành `<30`, `30–59`, `>=60`
*   **Bảng số liệu thực tế từ dữ liệu:**

| Nhóm tuổi | Nhãn đặt tên | Số lượng khách hàng (dòng) | Tỷ lệ trong dữ liệu | Tỷ lệ gửi tiền (`y = yes`) |
|:---|:---|:---:|:---:|:---:|
| Dưới 30 | `young` | 5.273 | 11.66% | **17.60%** |
| 30 đến 59 | `middle_age` | 38.154 | 84.39% | **9.86%** |
| Từ 60 trở lên | `senior` | 1.784 | 3.95% | **33.63%** |

*   **Minh chứng tính đúng đắn:**
    1.  Nhóm tuổi lao động chính (`30–59`) có tỷ lệ gửi tiền thấp nhất (**9.86%**) vì gánh nặng chi tiêu gia đình/vay nợ mua nhà.
    2.  Nhóm cao tuổi (`>=60`) có tỷ lệ gửi tiền vọt lên **33.63% (cao gấp 3.4 lần nhóm trung niên)** vì có tiền tiết kiệm hưu trí.
    3.  Ranh giới cắt tại 30 và 60 phản ánh chính xác sự phân hóa đột biến về hành vi tài chính trong dữ liệu.

---

### 2.2. Biến `balance` (Số dư tài khoản) — QUYẾT ĐỊNH: `<0`, `0–500`, `500–2000`, `>2000`
*   **Thống kê 5 số (Five-number summary) của `balance` trong dữ liệu:**
    *   Min: -8.019 Euro
    *   Phân vị 25% (Q1): **72 Euro**
    *   Trung vị 50% (Q2): **448 Euro** (~500 Euro)
    *   Phân vị 75% (Q3): **1.428 Euro** (~1.500–2.000 Euro)
    *   Max: 102.127 Euro
*   **Bảng số liệu thực tế phân theo nhóm:**

| Khoảng số dư | Nhãn đặt tên | Số khách hàng | Tỷ lệ trong dữ liệu | Tỷ lệ gửi tiền (`y = yes`) |
|:---|:---|:---:|:---:|:---:|
| Nhỏ hơn 0 Euro | `negative` | 3.766 | 8.33% | **5.58%** |
| 0 đến 500 Euro | `low` | 19.899 | 44.01% | **9.91%** |
| 501 đến 2000 Euro | `medium` | 13.045 | 28.85% | **13.02%** |
| Trên 2000 Euro | `high` | 8.501 | 18.80% | **16.57%** |

*   **Minh chứng tính đúng đắn:**
    1.  Khách hàng mang số dư âm (`< 0`) đang thấu chi nợ nần, tỷ lệ gửi tiền là thấp nhất toàn bộ tập dữ liệu (**5.58%**). Tách riêng nhóm này là bắt buộc.
    2.  Mốc 500 Euro bám sát giá trị trung vị của dữ liệu (448 Euro).
    3.  Tỷ lệ gửi tiền tăng tuyến tính rất đẹp theo số dư: `5.58% -> 9.91% -> 13.02% -> 16.57%`.

---

### 2.3. Biến `campaign` (Số lần liên hệ trong chiến dịch) — QUYẾT ĐỊNH: `1`, `2–3`, `>=4`
*   **Bảng số liệu thực tế từ dữ liệu:**

| Số lần gọi | Nhãn đặt tên | Số khách hàng | Tỷ lệ trong dữ liệu | Tỷ lệ gửi tiền (`y = yes`) |
|:---|:---|:---:|:---:|:---:|
| 1 lần | `1_call` | 17.544 | 38.80% | **14.60%** |
| 2 đến 3 lần | `2_3_calls` | 18.026 | 39.87% | **11.20%** |
| Từ 4 lần trở lên | `4plus_calls` | 9.641 | 21.32% | **7.35%** |

*   **Minh chứng tính đúng đắn:**
    *   Gọi 1 lần đạt hiệu quả cao nhất (**14.60%**).
    *   Càng gọi nhiều từ lần thứ 4 trở đi, hiệu quả giảm đi một nửa (**chỉ còn 7.35%**) do gây khó chịu/làm phiền khách hàng. Ranh giới `1`, `2–3`, `>=4` nắm bắt đúng ngưỡng suy giảm hiệu quả của chiến dịch telesales.

---

### 2.4. Biến `day` (Ngày trong tháng 1–31) — QUYẾT ĐỊNH: LOẠI BỎ KHỎI ITEMSET
*   **Cơ sở lý thuyết Data Mining:**
    *   Thuật toán Apriori có độ phức tạp lũy thừa theo số lượng hạng mục: $O(2^{|I|})$.
    *   Cột `day` có 31 giá trị phân tán ngẫu nhiên, không có tính chu kỳ kinh tế rõ ràng (khác với `month` phản ánh mùa vụ tài chính).
    *   Nếu đưa 31 giá trị `day` vào, không gian itemset bùng nổ, sinh ra hàng loạt luật rác vô nghĩa như `{job=admin, day=17} => y=no`. Do đó, loại bỏ `day` là chuẩn mực để tối ưu không gian khai phá luật.

---

## 3. NGUỒN TÀI LIỆU THAM KHẢO CHÍNH THỨC
1.  **Đề bài:** Bộ môn Phân tích Dữ liệu, ĐH FPT — *Đề cương Assignment 1 DBM301, Đề 04 (Assignment_1_De_04.pdf)*.
2.  **Bài báo công bố dataset gốc:** Moro, S., Cortez, P., & Rita, P. (2014). *A data-driven approach to predict the success of bank telemarketing.* Decision Support Systems, 62, 22-31. Elsevier. DOI: [10.1016/j.dss.2014.03.001](https://doi.org/10.1016/j.dss.2014.03.001).
3.  **Tài liệu Kho lưu trữ Dữ liệu UCI:** [UCI Machine Learning Repository: Bank Marketing Dataset](https://archive.ics.uci.edu/dataset/222/bank+marketing).
4.  **Lý thuyết Khai phá Luật kết hợp:** Agrawal, R., & Srikant, R. (1994). *Fast algorithms for mining association rules.* Proc. 20th Int. Conf. Very Large Data Bases (VLDB), 487-499.

