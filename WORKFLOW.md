# Chuẩn Workflow – Theo dõi & Phân tích Kinh tế Vĩ mô

Phiên bản 1.0 · Áp dụng cho: Việt Nam, Mỹ, quốc tế (TQ, Eurozone, OPEC, IMF)

## 1. Mục tiêu đầu ra
1. **Theo dõi:** cập nhật số liệu và động thái chính sách đúng, đủ, kịp thời, có nguồn.
2. **Giải thích xu hướng:** vì sao số liệu như vậy, liên kết giữa các chỉ số.
3. **Đánh giá triển vọng:** kịch bản cơ sở / tích cực / tiêu cực, có điều kiện kích hoạt.
4. **Đo tác động lan truyền** tới Việt Nam và thị trường tài chính (tỷ giá, lãi suất, chứng khoán, ngành).

## 2. Kiến trúc: 1 điều phối + 6 agent chuyên trách

```
                         ┌──────────────────────┐
     Lịch / Sự kiện ───▶ │  ORCHESTRATOR (skill) │  ← phiên Claude chính, chạy workflow W1–W5
                         └──────────┬───────────┘
          ┌───────────────┬─────────┼──────────────┬────────────────┐   (chạy SONG SONG)
          ▼               ▼         ▼              ▼                │
   vn-data-collector  global-data-  policy-tracker  market-monitor  │
   (7 nhóm chỉ số VN) collector     (NHNN, BTC, Fed, (UST, DXY, VND, │
                      (Mỹ + QT)     ECB, PBoC, OPEC) liên NH, VNI…)  │
          └───────────────┴─────────┬──────────────┘                │
                                    ▼                               │
                              fact-checker   ── Gate G2: kiểm chứng │
                                    ▼                               │
                              macro-analyst  ── Gate G3: tương quan, lan truyền, kịch bản
                                    ▼
                         ORCHESTRATOR viết báo cáo ── Gate G4 → reports/ + data/history.csv
```

**Vì sao thiết kế như vậy**
- *Thu thập tách theo vùng/chủ đề*: mỗi agent có context riêng, chạy song song → nhanh, không lẫn số liệu.
- *Kiểm chứng tách riêng*: agent kiểm chứng không phải là agent đã thu thập → khách quan, bắt lỗi "bịa số".
- *Phân tích tập trung 1 agent*: tương quan cần nhìn toàn bộ bức tranh cùng lúc, không nên chia nhỏ.
- *Workflow cố định, agent linh hoạt bên trong*: quy trình, đầu vào/đầu ra, cổng chất lượng là cố định; cách tìm kiếm trong mỗi bước để agent tự quyết.

| Agent | Phạm vi | Model khuyến nghị | Đầu ra |
|---|---|---|---|
| `vn-data-collector` | GDP, CPI, IIP, bán lẻ, PMI VN, tín dụng, M2, XNK, FDI, đầu tư công, NSNN | sonnet | `02_raw_vn.md` |
| `global-data-collector` | Lạm phát, việc làm, tăng trưởng, nhà ở Mỹ; PMI TQ/EU, Ifo, EIA, IMF | sonnet | `02_raw_global.md` |
| `policy-tracker` | Công cụ CSTT NHNN, CSTK, FOMC/dot plot/phát biểu Fed, ECB, PBoC, OPEC+ | sonnet | `02_raw_policy.md` |
| `market-monitor` | UST 2Y/10Y, DXY, FedWatch, vàng, dầu, USD/VND, liên NH, TPCP VN, VN-Index, khối ngoại | sonnet | `02_raw_market.md` |
| `fact-checker` | Kiểm chứng mọi con số trong 02_raw_* | sonnet | `03_verification.md` |
| `macro-analyst` | Xu hướng, tương quan, lan truyền, kịch bản, hàm ý ngành | opus | `04_analysis.md` |

## 3. Năm workflow chuẩn

| Mã | Tên | Kích hoạt | Agent dùng | Lệnh | Thời hạn |
|---|---|---|---|---|---|
| **W1** | Flash – báo cáo nhanh theo sự kiện | Sau công bố chỉ số quan trọng (CPI, NFP, PCE, GDP, FOMC, CPI VN, NHNN điều chỉnh) | 1–2 collector liên quan + market-monitor + fact-checker + analyst (chế độ ngắn) | `/flash <sự kiện>` | ≤ 2 giờ sau công bố |
| **W2** | Week Ahead – lịch tuần tới | Chủ nhật 19:00 hoặc thứ Hai 07:00 (GMT+7) | global + vn collector (chế độ lịch), policy-tracker, analyst | `/week-ahead` | Trước phiên thứ Hai |
| **W3** | Week Review – tổng kết tuần | Thứ Bảy 09:00 | cả 6 agent | `/week-review` | Cuối tuần |
| **W4** | Monthly VN – vĩ mô VN tháng | Sau họp báo Cục Thống kê (≈ ngày 6) | vn-collector, policy-tracker, market-monitor, global (tóm tắt), fact-checker, analyst | `/monthly-vn` | ≤ 2 ngày sau công bố |
| **W5** | Outlook – triển vọng quý | Đầu quý / sau GDP quý VN, sau FOMC có SEP | cả 6 agent, chế độ sâu | `/outlook` | 1 lần/quý |

## 4. Các bước chuẩn của mọi workflow (S1 → S7)

| Bước | Ai làm | Việc | File đầu ra | Cổng |
|---|---|---|---|---|
| S1 Scope | Orchestrator | Xác định sự kiện, kỳ, phạm vi, danh sách "Tìm thêm" (knowledge/indicators.md); đọc báo cáo kỳ trước + data/history.csv; **gán ưu tiên P1/P2/P3 theo agent chủ sở hữu**; ghi mốc tham chiếu (tỷ giá, VN-Index, 10Y tuần trước) để kiểm tra biên hợp lý | `01_scope.md` | G1 |
| S2 Collect | 4 collector song song | Tìm số liệu theo **Data Card** (mục 5) | `02_raw_*.md` | |
| S2.5 Cross-check | Orchestrator | Đọc nhanh 4 file, ghi các số mâu thuẫn giữa file vào mục "Cần kiểm" của `01_scope.md` | | |
| S3 Verify | fact-checker | Đối chiếu nguồn A hoặc ≥ 2 nguồn; gắn nhãn ✅ / ⚠️ / ❌ | `03_verification.md` | G2 |
| S4 Analyze | macro-analyst | Xu hướng → tương quan → lan truyền → kịch bản → hàm ý | `04_analysis.md` | G3 |
| S5 Write | Orchestrator | Viết theo knowledge/report-template.md | `report.md` | G4 |
| S6 Archive | Orchestrator | Ghi số liệu ✅ vào data/history.csv; copy report sang reports/ | | |
| S7 Follow-up | Orchestrator | Ghi sự kiện cần theo dõi tiếp vào data/watchlist.md | | |

Thư mục mỗi lần chạy: `runs/YYYY-MM-DD_W#_<slug>/`

## 5. Data Card – định dạng chuẩn cho mọi số liệu
Định nghĩa đầy đủ và quy tắc kiểm tra năm/thứ/biên hợp lý: `knowledge/collection-rules.md`.

```
| Quốc gia | Chỉ số | Kỳ | Loại kỳ | Phạm vi | Đơn vị | Actual | Consensus | Previous | Revised prev. | Surprise | Nguồn 1 | Nguồn 2 | Số nguồn khớp | Tier | Ngày công bố |
```
- **Đơn vị** bắt buộc ghi: %YoY, %MoM, %QoQ SAAR, nghìn việc làm, tỷ USD, điểm…
- **Surprise** = Actual − Consensus; ghi kèm "cao hơn / đúng / thấp hơn dự báo".
- Thiếu trường nào ghi `N/A` + lý do. **Tuyệt đối không ước đoán.**
- Động thái chính sách ghi theo bảng: `Công cụ | Mức hiện tại | Mức kỳ trước | Thay đổi | Ngày | Nguồn`.

## 6. Cổng chất lượng (Quality Gates)

| Cổng | Tiêu chí đạt | Nếu không đạt |
|---|---|---|
| **G1 Scope** | Có kỳ số liệu, danh sách chỉ số chính + liên quan, ngày hôm nay | Hỏi lại người dùng hoặc tra lịch kinh tế |
| **G2 Verify** | *G2-chặn:* không còn ❌ trong số chính. *G2-cảnh báo:* số chính ⚠️ được dùng nhưng gắn ⚠️; báo cáo ghi tỉ lệ ✅ | Gửi lại collector tìm bổ sung 1 lần; vẫn không được → chuyển vào "Chưa xác minh được" |
| **G3 Analysis** | Mỗi chuỗi lan truyền có: cơ chế, bằng chứng số liệu, mức độ, độ trễ; có ít nhất 1 quan điểm ngược (bear case); phân biệt sự kiện/nhận định | Analyst bổ sung |
| **G4 Report** | Đúng template; kết luận 1 câu ở đầu; có nguồn & mục chưa xác minh; giờ GMT+7 | Orchestrator sửa |

## 7. Lưu trữ & bộ nhớ dài hạn
- `data/history.csv` – lịch sử số liệu đã kiểm chứng (dùng cho xu hướng, so sánh kỳ trước).
- `data/policy_log.csv` – nhật ký thay đổi công cụ chính sách NHNN/Fed/BTC.
- `data/watchlist.md` – các sự kiện/giả thuyết cần theo dõi; đầu mỗi workflow phải đọc file này.
- `reports/` – báo cáo cuối cùng, đặt tên `YYYY-MM-DD_W#_<slug>.md`.

## 8. Lịch vận hành mẫu (GMT+7)

| Thời điểm | Việc |
|---|---|
| Thứ Hai 07:00 | W2 Week Ahead |
| Ngày làm việc 1 của tháng, 08:00 | W1 PMI VN + China PMI |
| ≈ Ngày 6, 15:00 | W4 Monthly VN |
| Tối ngày CPI Mỹ (≈ 20:30–21:30 tùy giờ mùa hè/đông) | W1 Flash CPI |
| Tối thứ Sáu đầu tháng | W1 Flash NFP |
| Sáng sau FOMC (02:00–03:00 GMT+7) | W1 Flash FOMC |
| Thứ Bảy 14:00 (hoặc thứ Hai 10:00) | W3 Week Review – chạy muộn để công cụ tìm kiếm kịp lập chỉ mục số đóng cửa thứ Sáu |
| Đầu quý | W5 Outlook |

## 9. Điều kiện vận hành
- **Mạng:** môi trường phải cho phép WebFetch tới nguồn chính thức (`knowledge/sources.md`); thiếu thì agent chỉ có tóm tắt tìm kiếm → phần lớn số liệu chỉ đạt ⚠️.
- **Hạn mức:** 4 collector song song tiêu tốn nhiều hạn mức gói; ngân sách 25 lượt/collector, 60 lượt cho fact-checker.
- **Model:** collector & fact-checker dùng Sonnet; analyst dùng Opus.

## 10. Cải tiến liên tục
Mỗi báo cáo cuối có mục "Bài học" (1–3 dòng): nguồn nào hỏng, consensus nào khó tìm, nhận định kỳ trước đúng/sai. Hằng tháng rà lại và cập nhật knowledge/sources.md, indicators.md.
