# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

- **MSSV:** 2A202603563
- **Họ và tên:** Nguyễn Đức Minh
- **SEED:** 7403

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | ☑ |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | ☑ |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | ☑ |

## 2. Kết quả chính

| Phương án | F1 test | F1 SUB 40 | p95 ms/tin | USD/1k tin |
| --- | ---: | ---: | ---: | ---: |
| 1. BoW + LogReg | 0.907 | 0.857 | 0.09 | 0 |
| 2. BoW + MLP | 0.953 | 0.895 | 0.46 | 0 |
| 3a. Encoder + kNN | 0.895 | 0.900 | 16.67 | 0 |
| 3b. Encoder + LogReg | 0.977 | 0.974 | 7.24 | 0 |
| 3c. Encoder + MLP head | 0.966 | 0.950 | 7.31 | 0 |
| 4a. LLM nhỏ zero-shot (Gemini) | — | 0.974 | 1401 | 0.028 |
| 4c. LLM nhỏ few-shot | — | 1.000 | 800 | 0.072 |

4b (LLM lớn) không chạy xong do hết quota free tier (lỗi 429).

## 3. Dự đoán và thực tế

1. **hard_eval:** đoán LLM đúng 75–85% trên 50 case khó, thực tế 100%. Tôi đánh giá thấp khả năng hiểu ngữ cảnh của LLM.
2. **emb_tricky:** đoán Encoder 5/6, BoW 3/6, thực tế hoà 5/6. Tin lừa đảo kiểu tài khoản không giống spam cũ nên cả hai cùng sai.
3. **ml_lr:** đoán lr=50 làm loss dao động mãi, thực tế loss vọt lên ≈2.6 rồi tụt về 0.007. Bài toán lồi, dữ liệu gần tách tuyến tính nên vẫn hội tụ.

## 4. Phân tích 50 case khó

BoW + LogReg tụt nhiều nhất: F1 0.857 → 0.286. LLM đúng 100%; Encoder + LogReg 0.627.

- **#29 (né từ khoá):** "C0ngr4tulat10ns! Y0u h4ve w0n…" — các token viết bằng số không có trong vocab nên BoW không thấy dấu hiệu spam.
- **#1 (ham trông như spam):** "Congrats!!! You WON the bet lol… Call me later" — BoW cộng trọng số của `congrats`, `won`, `call` mà không hiểu ngữ cảnh bạn bè.

Test thường cùng phân phối với train nên điểm quá lạc quan.

## 5. Thử thách cá nhân

**Đề #6:** LLM trả JSON `{"label", "reason"}`, so với bản trả 1 từ trên 10 tin SUB.

Cách làm: viết `build_json_prompt`, `parse_json_label` (regex lấy `{...}`, fallback `parse_label`), chạy hai bản trên cùng 10 tin.

Kết quả: F1 đều 1.00. Token out 13 → 157 (≈12×), chi phí 26.3 → 67.5 USD/1M tin (≈2.57×). Chỉ đáng dùng JSON khi cần lý do để người duyệt kiểm tra lại.

## 6. Phương án cho sản phẩm thật

Chọn **3b. Encoder + LogReg**: F1 0.977, p95 7.24 ms, đạt SLA 50 ms. LLM có p95 ≈1.4 s và tốn ≈8.400 USD/tháng nên không đạt yêu cầu. Đề xuất router: 3b xử lý mọi tin, chỉ tin lưỡng lự (p ∈ [0.3, 0.7]) mới gửi LLM. Đánh đổi: 3b yếu với spam né từ khoá, cần thêm bước chuẩn hoá đầu vào.

## 7. AI log và điều đã học

| Phần | Hỏi AI gì | AI trả lời | Tôi sửa / kiểm chứng |
| --- | --- | --- | --- |
| 1.1 | Ý tưởng `vectorize` | Ví dụ tương tự với ký tự | Tự viết cho từ; ô kiểm tra ✅ |
| 4 | Lấy và nạp API key | Gemini free + Colab Secrets | Kiểm tra notebook không chứa key |
| 4.4, thử thách | Lỗi 429 | Thêm `sleep` + tăng `base_delay` | Chạy hết 40 tin |
| 4b | Lỗi 400 `minimal` | Đổi sang `low` | Vẫn hết quota → bỏ 4b |
| Tính tay | Cách hiểu đề | Công thức và các bước | Tự tính lại; 5 hash ✅ |
| Báo cáo | Sắp xếp báo cáo | Bản nháp | Đối chiếu số với notebook |

**Điều đã học:** điểm test cao chưa chắc model bền ngoài đời (BoW 0.86 → 0.29 trên case khó). Chọn model phải cân F1, độ trễ và chi phí, không chỉ nhìn F1.