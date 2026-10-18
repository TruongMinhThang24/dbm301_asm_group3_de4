# Phú — WBS 5.0–6.2: FP-Growth, holdout và Weka

## Dữ liệu và phạm vi

- Một dòng là **một quan sát chiến dịch**, không phải một khách hàng duy nhất.
- Khai phá trên 36.168 dòng `processed_exploration.csv`; 9.043 dòng `processed_test.csv` chỉ dùng để kiểm chứng các luật đã cố định.
- Giữ đúng 14 item dạng `attribute=value` của Task 4: bỏ `previous` chưa rời rạc hóa, giữ `contacted_before`, giữ `y`; `duration` đã bị loại từ Task 2.7. Không có `y` ở vế trái luật.
- Ba cấu hình từ `reports/apriori/configs.csv`: A/B/C có support tối thiểu 1%/2%/3%; confidence tối thiểu 60%; độ dài itemset tối đa 3. Ngưỡng số quan sát hỗ trợ tối thiểu 100 chỉ dùng trong chọn luật khám phá.

## 5.0 — FP-Growth với ba cấu hình

[`notebooks/5.0_6.2_phu_separate_tasks.ipynb`](../../notebooks/5.0_6.2_phu_separate_tasks.ipynb) chạy FP-Growth bằng chính ma trận one-hot và [`configs.csv`](configs.csv) tương ứng Apriori. Ba bộ itemset và chỉ số luật nằm trong `frequent_itemsets_A/B/C.csv` và `rule_metrics_A/B/C.csv`.

## 5.1 — So sánh FP-Growth và Apriori

Notebook chạy lại Apriori trên cùng máy để ghi thời gian có thể so sánh trong một lần chạy; đồng thời đối chiếu từng itemset, luật hướng đến `y` và các chỉ số.

| Cấu hình | Support | Itemset Apriori = FP | Luật RHS=y Apriori = FP | Chênh lệch itemset / luật / chỉ số | Apriori (s) | FP-Growth (s) |
|---|---:|---:|---:|---:|---:|---:|
| A | 1% | 6.696 | 796 | 0 / 0 / 0 | 0,368 | 2,255 |
| B | 2% | 4.483 | 619 | 0 / 0 / 0 | 0,525 | 1,392 |
| C | 3% | 3.418 | 505 | 0 / 0 / 0 | 0,304 | 1,263 |

Các giá trị thời gian trên là **một lần đo trên máy đang chạy**, chỉ minh họa chi phí thực thi của hai triển khai và tùy chọn `low_memory=True` của Apriori. Không kết luận thuật toán nào nhanh hơn nói chung. Bảng chi tiết và thời gian chính xác nằm ở [`comparison.csv`](comparison.csv). Notebook dừng bằng lỗi nếu thiếu/thừa itemset, thiếu/thừa luật hoặc lệch số đếm/chỉ số.

## 5.2 — Kiểm tra chỉ số và chọn luật cuối

Notebook gộp luật từ ba cấu hình, loại trùng, giữ các luật có lift > 1 và ít nhất 100 quan sát hỗ trợ, rồi xếp theo lift, confidence và số quan sát. [`final_top_10_rules.csv`](final_top_10_rules.csv) chứa tối đa 10 luật cuối cùng của FP-Growth. Cả danh sách và số đếm, support, confidence, lift, leverage đã được kiểm tra bằng giá trị tương ứng của Apriori; cả 10 luật đều khớp.

## 6.0 — Kiểm chứng 5 luật trên tập test

Chốt trước khi đọc test các luật **R01, R02, R04, R08, R10** từ `top_10_rules.csv`. Bốn luật `y=yes` đại diện các dạng điều kiện khác nhau; R10 là luật `y=no`. Không chọn lại ngưỡng hoặc xếp hạng bằng test.

| Luật | Điều kiện → phản hồi | n(A∩B) test | Support test | Confidence exploration → test | Lift exploration → test |
|---|---|---:|---:|---:|---:|
| R01 | housing=no & poutcome=success → y=yes | 147 | 1,626% | 69,95% → 70,67% | 5,98 → 6,04 |
| R02 | campaign_group=1_call & poutcome=success → y=yes | 99 | 1,095% | 67,55% → 62,26% | 5,77 → 5,32 |
| R04 | loan=no & poutcome=success → y=yes | 191 | 2,112% | 65,12% → 66,32% | 5,57 → 5,67 |
| R08 | poutcome=success → y=yes | 199 | 2,201% | 64,65% → 65,03% | 5,53 → 5,56 |
| R10 | contact=unknown & marital=married → y=no | 1.565 | 17,306% | 96,84% → 96,07% | 1,097 → 1,088 |

Các phép tính test dùng `n(A)`, `n(B)`, `n(A∩B)` đếm trực tiếp trên 9.043 dòng: support = `n(A∩B)/N`, confidence = `n(A∩B)/n(A)`, lift = `confidence/(n(B)/N)`, leverage = `support - n(A)n(B)/N²`. [`holdout_5_rules.csv`](../validation/holdout_5_rules.csv) lưu mọi số đếm và chỉ số ở cả hai tập. Tỷ lệ nền test là 11,70% với `y=yes` và 88,30% với `y=no`.

Cả năm luật vẫn có lift > 1 và confidence ≥ 60% trên test. R02 giảm confidence 5,29 điểm phần trăm; joint count trên test là 99, vì tập test nhỏ hơn exploration. Ngưỡng 100 của giai đoạn chọn luật không dùng để loại luật sau khi nhìn test. R10 có confidence cao nhưng nền `y=no` đã 88,30%, nên lift 1,088 chỉ cho thấy mức tăng tương đối nhỏ. Các luật `y=yes` đều có `poutcome=success`, vì vậy chúng không phải bốn phát hiện độc lập.

## 6.1–6.2 — Weka Apriori và đối chiếu 3 luật

- `exploration_14_items.arff` chứa đủ 14 thuộc tính phân loại dùng trong Python; mỗi giá trị biểu diễn cùng item `attribute=value`; `y` là thuộc tính cuối. Weka Apriori đã chạy thật bằng Java 21 và Weka 3.8.7. Tệp `weka_apriori_A_full.txt` giữ nguyên stdout/stderr; `run_metadata_full.json` ghi cấu hình, phiên bản, thời gian và checksum.
- Weka mặc định cho phép luật dài hơn giới hạn 3 item của Python và chỉ in tối đa `-N 10000` luật. Vì vậy lượt chạy đầy đủ dùng làm bằng chứng thực thi trên toàn bộ biến, còn phép đối chiếu chính xác dùng thêm `exploration_4_items.arff` gồm `housing`, `campaign_group`, `poutcome`, `y` trên **cùng toàn bộ 36.168 dòng**. Đây là phép chiếu thuộc tính, không phải lấy mẫu dòng; các số đếm cho luật chỉ dùng các thuộc tính này không đổi.
- Cấu hình lượt chiếu: `-A -c last -N 10000 -T 0 -C 0.6 -U 1 -M 0.01 -D 0.99`. `-A` yêu cầu class association rule với RHS là `y`, `-T 0` xếp theo confidence, `-C 0.6` là ngưỡng confidence, `-M 0.01` là cận dưới support. Weka có thể hiển thị khác ở header lượt đầy đủ khi chạm giới hạn số luật; không dùng header đó để suy ra phép đối chiếu. Cấu hình này theo [tài liệu chính thức của Weka Apriori](https://weka.sourceforge.io/doc.stable-3-8/weka/associations/Apriori.html).

| Luật | Hạng Weka chiếu | n(A) Weka = Python | n(A∩B) Weka = Python | Confidence Weka in → Python đầy đủ |
|---|---:|---:|---:|---:|
| R01: housing=no & poutcome=success → y=yes | 39 | 832 | 582 | 0,70 → 0,699519 |
| R02: campaign_group=1_call & poutcome=success → y=yes | 40 | 678 | 458 | 0,68 → 0,675516 |
| R08: poutcome=success → y=yes | 41 | 1.205 | 779 | 0,65 → 0,646473 |

Weka in confidence với hai chữ số; số đếm trùng tuyệt đối và confidence trùng khi làm tròn. [`three_rule_comparison.csv`](../weka/three_rule_comparison.csv) ghi nguyên dòng Weka, hạng, số đếm và chỉ số Python. [`weka_apriori_A_projection.txt`](../weka/weka_apriori_A_projection.txt) là log Weka của lượt đối chiếu. Weka chỉ in confidence trong chế độ `-A`; support, lift và leverage của Python được tính từ các số đếm gốc.

## Giới hạn diễn giải

Tách ngẫu nhiên phân tầng giữ tỷ lệ `y` nhưng dữ liệu gốc có thứ tự thời gian và không có mã khách hàng; tập test không thay cho kiểm chứng theo thời gian hoặc trên khách hàng độc lập. Month thiếu năm; `poutcome=success` phản ánh phản hồi chiến dịch trước, không chứng minh quan hệ nhân quả hay hiệu quả can thiệp. Tránh dùng tình trạng hôn nhân hoặc thuộc tính nhạy cảm để loại trừ khách hàng. Các kết quả là mô tả và cơ sở để đề xuất thử nghiệm có kiểm soát.

## Khai báo hỗ trợ AI

OpenAI Codex hỗ trợ viết mã trong notebook và bản thảo phần WBS 5.0–6.2. Các bảng số liệu và log trong thư mục này được tạo bằng việc chạy mã Python và Weka trên dữ liệu repo; nhóm cần tự rà soát nội dung và chịu trách nhiệm về bài nộp cuối.

## Chạy lại

Từ thư mục repo, chạy notebook từ đầu đến cuối. Trên Windows, notebook tự tìm `weka.jar` trong `Program Files/Weka-*`. Nếu Weka cài ở nơi khác, đặt đường dẫn JAR trước khi mở Jupyter:

```powershell
$env:WEKA_JAR = "C:\Program Files\Weka-3-8-7\weka.jar"
jupyter lab notebooks/5.0_6.2_phu_separate_tasks.ipynb
```

Thay giá trị `WEKA_JAR` bằng đường dẫn tới `weka.jar` của máy đó. Notebook chạy lại FP-Growth, holdout, xuất hai ARFF, chạy Weka trên cả hai ARFF và kiểm tra ba luật. Cần Java và các thư viện trong `pyproject.toml`. Notebook không thay đổi đầu vào Apriori hay tập dữ liệu đã xử lý.
