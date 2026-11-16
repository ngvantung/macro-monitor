# Quy tắc thu thập dùng chung (bắt buộc cho mọi collector)

Rút ra từ lần chạy thử W3 ngày 16/11/2026. Mọi collector đọc file này trước khi tìm kiếm.

## 1. Phân công – mỗi số liệu chỉ một chủ sở hữu
| Nhóm | Chủ sở hữu duy nhất |
|---|---|
| Mọi **giá thị trường**: UST, DXY, FX (USD/VND trung tâm, NHTM, tự do), lãi suất liên NH, lợi suất TPCP VN, chứng khoán, vàng, dầu, FedWatch | `market-monitor` |
| **Công cụ & quyết định chính sách**: lãi suất điều hành, DTBB, OMO/tín phiếu (khối lượng, lãi suất trúng), room tín dụng, văn bản; Fed/ECB/PBoC/BoJ/OPEC+; thu chi NSNN, đầu tư công, thuế, đấu thầu TPCP | `policy-tracker` |
| **Kinh tế thực VN**: GDP, CPI, lạm phát cơ bản, IIP, bán lẻ, PMI, XNK, FDI, tăng trưởng tín dụng & huy động (số liệu thống kê) | `vn-data-collector` |
| **Kinh tế thực Mỹ & quốc tế**: lạm phát, việc làm, tăng trưởng, nhà ở, PMI/Ifo/TQ, EIA | `global-data-collector` |
Không thu thập số thuộc agent khác. Nếu cần làm bối cảnh, ghi "xem file của <agent>".

## 2. Thứ tự nguồn & công cụ
1. **Nguồn có cấu trúc trước** (khi WebFetch hoạt động): FRED CSV `https://fred.stlouisfed.org/graph/fredgraph.csv?id=<SERIES>&cosd=YYYY-MM-DD&coed=YYYY-MM-DD` (DGS2, DGS10, DTWEXBGS, DCOILBRENTEU, SP500, ICSA, CPIAUCSL, PAYEMS…); trang gốc BLS/DOL/Fed/NHNN/NSO.
2. WebSearch sau, chỉ khi bước 1 thất bại. Ghi rõ "chỉ có tóm tắt tìm kiếm" → mặc định Tier C.
3. Ghi kết quả WebFetch theo từng host (được / bị chặn) trong mục ghi chú.

## 3. Kiểm tra bắt buộc trước khi ghi một con số
- **Năm:** nguồn phải nêu rõ năm hiện tại (trong URL, tiêu đề hoặc nội dung). Không rõ năm → không dùng.
- **Thứ trong tuần:** ngày giao dịch phải là thứ Hai–thứ Sáu của đúng năm (ví dụ "5–9/11 là thứ Hai–thứ Sáu" chỉ đúng với năm 2018, không đúng 2026).
- **Biên hợp lý:** so với mức gần nhất đã biết (history.csv hoặc số đã xác nhận trong run). Lệch bất thường (tỷ giá lệch > 3%, VN-Index lệch > 10%, lợi suất lệch > 50 bp so với tuần trước) → coi là nghi sai năm, cần nguồn thứ hai.
- **Bài nhận định ≠ số thực tế:** bài "nhận định/preview/dự báo" không dùng làm số đóng cửa.
- **Kiến thức nền có thể lỗi thời:** tin số liệu tìm được có năm rõ ràng hơn trí nhớ; không "sửa" số theo trí nhớ.

## 4. Data Card chuẩn (thay thế bản cũ)
```
| Quốc gia | Chỉ số | Kỳ | Loại kỳ | Phạm vi | Đơn vị | Actual | Consensus | Previous | Revised prev. | Surprise | Nguồn 1 | Nguồn 2 | Số nguồn khớp | Tier | Ngày công bố |
```
- **Loại kỳ:** Tháng / Lũy kế (n tháng) / Quý / Tuần / Ngày. Với lũy kế, Previous = lũy kế cùng kỳ năm trước hoặc lũy kế tháng trước (ghi rõ).
- **Phạm vi:** `Trong tuần` (công bố trong cửa sổ) / `Bối cảnh` (công bố trước) / `Tuần sau` (chưa công bố – không tìm số).
- **Previous:** khi history.csv trống, được lấy từ nguồn của kỳ trước (ghi nguồn).
- **Consensus:** Mỹ – ưu tiên Reuters/Dow Jones/Bloomberg poll qua tin tức; lệch giữa nguồn thì ghi khoảng (ví dụ 0,2–0,3%). **Việt Nam: mặc định N/A, không tốn truy vấn.**
- **Số tự tính:** cho phép, gắn `[tự tính: công thức]`.

## 5. Ngân sách & ưu tiên
- Ngân sách mặc định: **25 lượt tool** mỗi collector (giới hạn hạn mức gói khi chạy 4 agent song song).
- Làm theo thứ tự **P1 → P2 → P3** do orchestrator ghi trong `01_scope.md`. Hết ngân sách thì dừng và liệt kê mục chưa làm.
- Một ngôn ngữ trước (tiếng Việt cho nguồn VN, tiếng Anh cho nguồn quốc tế); chỉ đổi ngôn ngữ khi truy vấn đầu thất bại.
- Ghi file đầu ra **sớm và cập nhật dần** (không đợi cuối) để không mất dữ liệu khi bị ngắt.
