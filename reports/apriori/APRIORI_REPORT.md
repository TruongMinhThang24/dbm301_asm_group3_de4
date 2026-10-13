# Báo cáo Task 4.0–4.3 Apriori

Nguồn: exploration; 36,168 dòng, 15 cột gốc; mã hóa 14 thuộc tính thành 58 item. Bỏ previous; giữ contacted_before; giữ y trong transaction. Test không được dùng.

## Thí nghiệm
Support 1%, 2%, 3%; confidence 60%; max_len=3; low_memory=True. Xem experiment_logs.csv: số itemset, số luật trước/sau lọc và thời gian thực tế. Không coi thời gian một lần chạy là benchmark ổn định.

## Chọn luật
RHS đúng y=yes/no; LHS không có y. 796 luật duy nhất; 469 luật lift>1 và count>=100; bảng cuối 10 luật. Loại trùng giữa cấu hình; xếp lift, confidence, count giảm dần, tên luật tăng dần khi hòa. Tỷ lệ nền yes 11.70%; no 88.30%. Không chọn chỉ theo confidence.

## Low frequency
Đánh dấu dưới 100 supporting observations: 0 luật. Không có luật bị đánh dấu không có nghĩa tiêu chí chưa được thực hiện; minimum support thấp nhất yêu cầu 362 quan sát. Mốc 100 là lựa chọn thực dụng, không chứng minh ý nghĩa thống kê.

### Phép tính tay: poutcome=success → y=yes
- N = 36,168; n(A) = 1,205; n(B) = 4,231; n(A∩B) = 779.
- Support = 779/36168 = 0.02153838 (2.15%).
- Confidence = 779/1205 = 0.64647303 (64.65%).
- Lift = (779/1205) / (4231/36168) = 5.52626720.
- Leverage = 779/36168 − (1205/36168) × (4231/36168) = 0.01764092.

Trong nhóm có poutcome=success, tỷ lệ yes là 64.65%, so với nền 11.70%; supporting observations là 779. Mối liên hệ không chứng minh nguyên nhân. Luật đạt A và B, không đạt support 3% của C.

## Giới hạn và việc tiếp theo
LHS tối đa 2 điều kiện; bỏ previous làm mất thông tin số lần liên hệ; luật có thể trùng nội dung; lớp không cân bằng; month gộp nhiều năm; dữ liệu không có mã khách hàng. Confidence 60% hạn chế luật yes. Cần đánh giá các luật đã cố định trên processed_test.csv ở bước validation, không dùng test điều chỉnh lựa chọn.

## Tệp đầu ra
configs.csv, experiment_logs.csv, transactions.csv, onehot_exploration.csv, item_dictionary.csv, frequent_itemsets_A/B/C.csv, rule_metrics_A/B/C.csv, rule_metrics_all_configs.csv, unique_target_rules.csv, top_10_rules.csv, supporting_observation_ids.csv, hand_calculation.csv/md, low_count_rules.csv, run_metadata.json.

## Tài liệu
https://rasbt.github.io/mlxtend/user_guide/preprocessing/TransactionEncoder/
https://rasbt.github.io/mlxtend/user_guide/frequent_patterns/apriori/
https://rasbt.github.io/mlxtend/user_guide/frequent_patterns/association_rules/
