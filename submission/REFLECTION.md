# Reflection — Lab 20 (Personal Report)

> **Báo cáo thực hành cá nhân.** Đây là kết quả ghi nhận trên hệ thống máy tính của học viên Lương Hoàng Anh. Các chỉ số hiệu năng được dùng để so sánh hiệu quả tối ưu hóa (before vs after) trên chính thiết bị này.

---

**Học viên:** Lương Hoàng Anh  
**Lớp:** A20-K1  
**Ngày thực hiện:** 06/05/2026

---

## 1. Thông số kỹ thuật hệ thống

Dưới đây là thông tin phần cứng ghi nhận từ script `detect-hardware.py`:

- **Hệ điều hành:** Windows 10 (môi trường MINGW64)
- **CPU:** Intel(R) Core(TM) i5-8250U @ 1.60GHz (4 nhân vật lý / 8 luồng)
- **Tập lệnh hỗ trợ:** AVX, AVX2, FMA, F16C
- **Bộ nhớ RAM:** 15.9 GB
- **Card đồ họa (GPU):** NVIDIA GeForce GTX 1050 (2GB VRAM)
- **Cấu hình llama.cpp:** Sử dụng backend CPU (tối ưu hóa AVX/NEON)
- **Model khuyến nghị:** TinyLlama-1.1B (phiên bản Q4_K_M)

**Nhật ký thiết lập:** Để triển khai lab mượt mà trên MINGW, tôi đã tùy chỉnh Makefile để tương thích tốt hơn với Windows. Đồng thời, tôi cài thêm `prometheus-client` và thực hiện patch `app.py` của `llama_cpp.server` để theo dõi các chỉ số qua endpoint `/metrics`. Một vấn đề kỹ thuật nhỏ về giới hạn đường dẫn dài (Long Path) trên Windows cũng đã được xử lý bằng cách sử dụng thư mục cài đặt `tmp_pip` ở root.

---

## 2. Track 01 — Đánh giá hiệu năng ban đầu

Bảng kết quả benchmark từ `01-llama-cpp-quickstart/benchmark.py`:

| Phiên bản Quant | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Tốc độ (tok/s) |
| :-------------- | --------: | ----------------: | ----------------: | -------------------: | -------------: |
| TinyLlama (Q4)  |       992 |         191 / 247 |       40.3 / 70.3 |   2712 / 2791 / 2793 |           24.8 |
| TinyLlama (Q2)  |       204 |         324 / 414 |       37.6 / 52.8 |   2585 / 2929 / 3024 |           26.6 |

**Phân tích:** Mặc dù phiên bản Q2 cho tốc độ tải cực nhanh và throughput nhỉnh hơn một chút, nhưng tôi vẫn ưu tiên sử dụng Q4_K_M. Với tốc độ giải mã ổn định trên 20 tokens/giây, phiên bản này vẫn đảm bảo sự mượt mà cần thiết cho ứng dụng chat mà không phải đánh đổi quá nhiều về độ chính xác của mô hình.

---

## 3. Track 02 — Thử nghiệm chịu tải llama-server

Kết quả đo đạc bằng Locust ở các mức concurrency khác nhau:

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Tỉ lệ lỗi |
| ----------: | --------: | ------------: | -----------: | -----------: | --------: |
|          10 |      0.23 |         21000 |        45000 |        45000 |        0% |
|          50 |      0.22 |         19000 |        45000 |        45000 |        0% |

- **Giám sát KV-cache:** Peak `llamacpp:kv_cache_usage_ratio` đạt mức **0.0**, cho thấy bộ nhớ cache hoàn toàn đáp ứng tốt cho các mô hình nhỏ.
- **Đánh giá:** Với TinyLlama 1.1B, bộ nhớ không phải là vấn đề. Tuy nhiên, việc độ trễ P95 lên tới 45 giây cho thấy CPU i5 thế hệ cũ đã đạt giới hạn khi phải xử lý đa người dùng đồng thời. Đây chính là điểm nghẽn hiệu năng chính của hệ thống.

---

## 4. Track 03 — Tích hợp Milestone

- **Hệ hệ thống:** IaC (localhost), Pipeline (in-memory), Lakehouse (SQLite), Vector Store (TOY_DOCS).
- **Phân tích thời gian phản hồi:**
    - Embedding: ~0.0 ms
    - Retrieval: 0.1 ms
    - llama-server (Inference): 12116 ms

**Kết luận:** Điểm nghẽn nằm hoàn toàn ở khâu suy luận của LLM (llama-server). Thời gian xử lý hơn 12 giây mỗi yêu cầu chủ yếu do CPU phải thực hiện các phép tính prefill/decode phức tạp mà không có sự hỗ trợ từ GPU. Các bước xử lý dữ liệu khác gần như không tốn thời gian.

---

## 5. Bonus — Cải tiến đáng giá nhất

> **Phần quan trọng nhất.** Chọn một thay đổi từ bonus track đã mang lại hiệu quả speedup cao nhất.

**Thay đổi:** _<Ví dụ: Tối ưu build flags hoặc GPU offload>_

**Kết quả (Trước vs Sau):**

```
Trước: <số liệu>
Sau:  <số liệu>
Tốc độ tăng: ~<X.Y>×
```

**Nguyên lý hoạt động:**

_Mô tả ngắn gọn về mặt kỹ thuật tại đây._

---

## 6. Điều ngạc nhiên nhất

_Ghi lại quan sát thú vị nhất của bạn trong quá trình làm lab._

---

## 7. Danh mục tự kiểm tra (Checklist)

- [ ] `hardware.json` đã commit
- [ ] `models/active.json` đã commit
- [ ] `benchmarks/01-quickstart-results.md` đã commit
- [ ] `benchmarks/02-server-results.md` đã commit
- [ ] `benchmarks/bonus-*.md` đã commit
- [ ] Ít nhất 6 ảnh chụp màn hình trong `submission/screenshots/`
- [ ] Lệnh `make verify` trả về 0
- [ ] Repo GitHub ở chế độ **Public**
- [ ] Đã nộp URL repo lên hệ thống LMS

---

**Lưu ý:** Vui lòng giữ repo ở chế độ **Public** cho đến khi có điểm chính thức.
