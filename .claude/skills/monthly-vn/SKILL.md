---
name: monthly-vn
description: W4 – Báo cáo vĩ mô Việt Nam hằng tháng sau họp báo Cục Thống kê: 7 nhóm chỉ số (tăng trưởng, lạm phát, tín dụng, lãi suất, tỷ giá, thương mại, đầu tư), bảng công cụ điều hành NHNN, chính sách tài khóa, phối hợp chính sách, bối cảnh quốc tế và lan truyền. Dùng khi người dùng gõ /monthly-vn.
---

# W4 – Monthly VN

Tuân theo `WORKFLOW.md`. Bạn là orchestrator.

1. **S1:** xác định tháng; đọc báo cáo W4 tháng trước, `data/history.csv`, `data/policy_log.csv`, `data/watchlist.md`.
2. **S2 song song:**
   - `vn-data-collector`: đủ 7 nhóm, cả số tháng và lũy kế, so cùng kỳ; mục tiêu năm.
   - `policy-tracker`: bảng công cụ NHNN (so tháng trước & 3 tháng trước); tài khóa (thu chi NSNN, giải ngân đầu tư công, thuế, TPCP).
   - `market-monitor`: tỷ giá, liên NH, TPCP, VN-Index, khối ngoại trong tháng.
   - `global-data-collector`: tóm tắt bối cảnh (Fed, CPI/NFP Mỹ, China PMI, Brent) của tháng.
3. **S3:** `fact-checker` – đặc biệt kiểm tra lũy kế vs riêng tháng.
4. **S4:** `macro-analyst` – nhấn mạnh: tiến độ so mục tiêu năm, NHNN ưu tiên mục tiêu nào, đánh đổi tỷ giá–lãi suất, phối hợp CSTT–CSTK, triển vọng quý.
5. **S5:** báo cáo theo mục C của template, thêm "Bảng điều khiển 7 nhóm chỉ số" (giá trị, so tháng trước, so cùng kỳ, so mục tiêu, tín hiệu ↑↓→).
6. **S6–S7:** lưu và cập nhật dữ liệu.
