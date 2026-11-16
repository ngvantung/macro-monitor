# Macro Monitor

Hệ thống agent (Claude Code) theo dõi và phân tích kinh tế vĩ mô **Việt Nam – Mỹ – quốc tế**: giải thích xu hướng, đánh giá triển vọng và đo tác động lan truyền tới Việt Nam và thị trường tài chính (tỷ giá, lãi suất, chứng khoán, ngành).

## Mục tiêu
1. **Theo dõi** số liệu và động thái chính sách đúng, đủ, kịp thời, có nguồn.
   - Việt Nam: tăng trưởng, lạm phát, tín dụng, lãi suất, tỷ giá, thương mại, đầu tư; công cụ CSTT của NHNN và CSTK.
   - Mỹ: lạm phát, việc làm, tăng trưởng, nhà ở, Fed.
   - Quốc tế: Trung Quốc, Eurozone, OPEC+, IMF.
2. **Giải thích xu hướng** và tương quan giữa các chỉ số (actual / consensus / revision, phản ứng thị trường).
3. **Đánh giá triển vọng** với kịch bản cơ sở / diều hâu / dịu và điều kiện kích hoạt.
4. **Đo tác động lan truyền** Mỹ/quốc tế → Việt Nam theo từng kênh.

## Kiến trúc
1 orchestrator (các skill workflow) + 6 agent:

| Agent | Vai trò | Model |
|---|---|---|
| `vn-data-collector` | Kinh tế thực Việt Nam | Sonnet |
| `global-data-collector` | Mỹ & quốc tế; lịch kinh tế | Sonnet |
| `policy-tracker` | NHNN, Bộ Tài chính, Fed, ECB, PBoC, OPEC+ | Sonnet |
| `market-monitor` | Mọi giá thị trường (UST, DXY, USD/VND, liên NH, VN-Index…) | Sonnet |
| `fact-checker` | Kiểm chứng độc lập (cổng G2) | Sonnet |
| `macro-analyst` | Tương quan, lan truyền, kịch bản (cổng G3) | Opus |

Quy trình chuẩn S1→S7 với 4 cổng chất lượng G1→G4: xem **`WORKFLOW.md`**.

## Cách chạy
Mở thư mục repo bằng Claude Code (máy local, app desktop hoặc claude.ai/code), rồi gõ:

| Lệnh | Workflow | Khi nào |
|---|---|---|
| `/flash <sự kiện>` | W1 – báo cáo nhanh | ≤ 2 giờ sau CPI, NFP, PCE, FOMC, CPI VN, NHNN đổi công cụ |
| `/week-ahead` | W2 – lịch tuần tới | Sáng thứ Hai |
| `/week-review` | W3 – tổng kết tuần | Chiều thứ Bảy |
| `/monthly-vn` | W4 – vĩ mô VN tháng | Sau họp báo Cục Thống kê (~ngày 3–6) |
| `/outlook` | W5 – triển vọng quý | Đầu quý |

**Điều kiện:** môi trường cần cho phép WebFetch tới các nguồn chính thức trong `knowledge/sources.md`. Nếu không, agent chỉ dùng được tóm tắt tìm kiếm và phần lớn số liệu chỉ đạt ⚠️.

## Cấu trúc thư mục
```
.claude/agents/     6 agent
.claude/skills/     5 workflow (flash, week-ahead, week-review, monthly-vn, outlook)
knowledge/          sources.md, indicators.md ("Tìm thêm"), transmission.md,
                    report-template.md, collection-rules.md (quy tắc thu thập chung)
data/               history.csv (số liệu đã kiểm chứng), policy_log.csv, watchlist.md
reports/            báo cáo cuối cùng
runs/               file trung gian mỗi lần chạy (không đưa lên repo – .gitignore)
WORKFLOW.md         quy chuẩn workflow
CLAUDE.md           quy tắc dự án cho Claude
```

## Trạng thái hiện tại (16/11/2026)
- Đã chạy thử **W3 tuần 09–15/11/2026**. Báo cáo: `reports/2026-11-16_W3_tuan-9-15-11.md`.
  - G2 lần đầu không đạt (✅ 33 · ⚠️ 25 · ❌ 5), sau đó các số ❌ bị loại.
  - G3 đạt.
- `data/` đã có số liệu nền: 23 dòng trong history, 5 thay đổi chính sách, watchlist đến giữa tháng 12/2026.
- Sau lần chạy thử, các agent đã được chỉnh:
  - Thêm `collection-rules.md`: phân công chủ sở hữu số liệu; kiểm tra năm, thứ trong tuần và biên hợp lý; ưu tiên FRED CSV.
  - Data Card có thêm `Loại kỳ` / `Phạm vi` / `Số nguồn khớp`.
  - G2 tách hai mức (chặn / cảnh báo).
  - Thêm bước S2.5 đối chiếu chéo.

## Việc còn dở
1. **Chưa kiểm tra quyền truy cập mạng mới.** Môi trường đã được mở cho 20 host nguồn (BLS, Fed, FRED, NHNN, Cục Thống kê, Bộ Tài chính, HNX, CafeF, VnEconomy…). Cần xác nhận WebFetch tới các host này đã hoạt động; lần chạy thử bị chặn (ENOTFOUND).
2. **Chạy kiểm tra với bộ quy tắc mới:** `/flash Retail Sales Mỹ T10` (công bố 17/11/2026). Mục tiêu là tỉ lệ ✅ trên số chính > 80%.
3. **Lấp khoảng trống dữ liệu Việt Nam:** OMO, tín phiếu, lãi suất liên NH hằng tuần chưa có nguồn ổn định. Cần thử sbv.gov.vn và bản tin tuần của các công ty chứng khoán.
4. **Hạn mức gói:** 4 collector song song làm chạm giới hạn 5 giờ. Cân nhắc chạy lần lượt hoặc giảm ngân sách truy vấn.
5. **Quyền chế độ auto:** `curl` và việc khởi chạy agent có truy cập mạng từng bị bộ phân loại chặn. Cần thêm quy tắc cho phép nếu muốn chạy tự động.
6. **Lịch tự động:** đặt scheduled task cho W2 (thứ Hai), W3 (thứ Bảy), W4 (~ngày 6).
7. **Chấm điểm dự báo:** ở W3 sau 13/12/2026, đối chiếu các giả thuyết trong `data/watchlist.md` (Fed +25 bp ngày 09/12; tỷ giá trung tâm 25.600–25.700; CPI VN T11 4,9–5,2%; UST 10Y 4,85–5,10%).

## Lưu ý
Repo này công khai. Không commit khóa API, token hay dữ liệu nhạy cảm. Thư mục `runs/` chứa bản nháp có lẫn số liệu bị loại, nên được để ngoài repo.
