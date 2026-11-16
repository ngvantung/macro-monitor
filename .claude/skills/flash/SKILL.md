---
name: flash
description: W1 – Báo cáo nhanh sau một sự kiện kinh tế quan trọng (CPI/PCE/NFP/GDP/FOMC Mỹ, CPI/GDP VN, NHNN điều chỉnh công cụ, PMI, OPEC+...). Dùng khi người dùng gõ /flash <sự kiện> hoặc hỏi về một kỳ công bố số liệu cụ thể.
---

# W1 – Flash

Tuân theo `WORKFLOW.md` (S1→S7, G1→G4). Bạn là orchestrator.

## S1 Scope
- Xác định: sự kiện, quốc gia, kỳ số liệu, ngày giờ công bố (GMT+7), ngày hôm nay.
- Đọc `data/watchlist.md`, báo cáo W1 gần nhất cùng sự kiện trong `reports/`, mục "Tìm thêm" của chỉ số trong `knowledge/indicators.md`.
- Tạo `runs/<YYYY-MM-DD>_W1_<slug>/01_scope.md`.

## S2 Collect – gọi SONG SONG trong một lượt
- Collector chính theo sự kiện: `global-data-collector` (Mỹ/QT) **hoặc** `vn-data-collector` (VN) – số liệu chính + các mục "Tìm thêm".
- `market-monitor` – phản ứng thị trường trước/sau sự kiện.
- `policy-tracker` – chỉ khi sự kiện là quyết định chính sách hoặc cần bối cảnh Fed/NHNN.

## S3 Verify
`fact-checker`. Nếu G2 không đạt: gửi lại collector đúng các mục cần tìm (tối đa 1 vòng), rồi kiểm chứng lại phần đó.

## S4 Analyze
`macro-analyst`, chế độ ngắn: mục 1, 2, 4, 5, 6, 7, 9, 10 của khung phân tích.

## S5 Write
Theo `knowledge/report-template.md` mục A. Tối đa ~1 trang. Ghi vào `runs/.../report.md`.

## S6–S7 Archive
- Thêm dòng ✅ vào `data/history.csv`; thay đổi chính sách vào `data/policy_log.csv`.
- Copy báo cáo sang `reports/<YYYY-MM-DD>_W1_<slug>.md`.
- Cập nhật `data/watchlist.md`.
- Trả lời người dùng: kết luận 1 câu + 5 ý chính + đường dẫn báo cáo.
