---
name: week-ahead
description: W2 – Lịch kinh tế tuần tới (Mỹ, Việt Nam, Trung Quốc, Eurozone, Nhật) kèm consensus, mức quan trọng, vì sao quan trọng với Việt Nam, và kịch bản phản ứng cho 3 sự kiện lớn nhất. Dùng khi người dùng gõ /week-ahead hoặc hỏi "tuần tới có gì".
---

# W2 – Week Ahead

Tuân theo `WORKFLOW.md`. Bạn là orchestrator.

1. **S1:** xác định tuần (thứ Hai–Chủ nhật), đọc `data/watchlist.md` và báo cáo W3 gần nhất. Tạo `runs/<date>_W2_<tuần>/01_scope.md`.
2. **S2 song song:**
   - `global-data-collector` chế độ **calendar** (toàn cầu + VN).
   - `policy-tracker`: lịch FOMC/phát biểu Fed, đấu thầu Treasury lớn, NHNN/BTC, OPEC+, đấu thầu TPCP VN trong tuần.
   - `market-monitor`: mức hiện tại của các tài sản chính (làm mốc).
3. **S3:** `fact-checker` kiểm tra ngày giờ và consensus của các sự kiện mức "cao".
4. **S4:** `macro-analyst` – chọn 3 sự kiện quan trọng nhất; với mỗi sự kiện: kịch bản cao hơn / đúng / thấp hơn dự báo → phản ứng UST, DXY, USD/VND, VN-Index.
5. **S5:** viết theo `knowledge/report-template.md` mục B.
6. **S6–S7:** lưu `reports/`, cập nhật `data/watchlist.md` với các sự kiện trong tuần (để kích hoạt W1).
