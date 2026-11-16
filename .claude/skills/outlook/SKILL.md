---
name: outlook
description: W5 – Báo cáo triển vọng kinh tế quý/năm cho Việt Nam trong bối cảnh Mỹ và quốc tế: kịch bản cơ sở/tích cực/tiêu cực, lộ trình Fed và NHNN, tỷ giá, lãi suất, tăng trưởng, lạm phát, hàm ý cho thị trường tài chính và ngành. Dùng khi người dùng gõ /outlook.
---

# W5 – Outlook (quý)

Tuân theo `WORKFLOW.md`. Bạn là orchestrator. Đây là workflow sâu nhất.

1. **S1:** xác định quý/năm dự báo; đọc tất cả W4 và W3 của quý trước, `data/history.csv`, `data/policy_log.csv`, báo cáo W5 trước (để đánh giá dự báo cũ).
2. **S2 song song (cả 4 collector), mỗi agent thêm nhiệm vụ:** thu thập dự báo của tổ chức (IMF WEO, WB, ADB, AMRO, Fed SEP, các NH đầu tư lớn, CTCK VN) cho tăng trưởng, lạm phát, lãi suất, tỷ giá – ghi tên, ngày, con số.
3. **S3:** `fact-checker`.
4. **S4:** `macro-analyst` – đủ 11 mục, với:
   - Bảng đồng thuận dự báo các tổ chức (min / trung vị / max).
   - Ma trận Fed × NHNN (knowledge/transmission.md mục 5).
   - 3 kịch bản có xác suất, điều kiện kích hoạt, và chỉ báo sớm để nhận biết kịch bản nào đang xảy ra.
   - Đánh giá độ chính xác của Outlook kỳ trước.
5. **S5:** báo cáo đầy đủ (mục C template) + phụ lục số liệu.
6. **S6–S7:** lưu; ghi các ngưỡng kích hoạt kịch bản vào `data/watchlist.md` để W1/W3 theo dõi.
