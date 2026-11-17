# Macro Monitor – Tài liệu hệ thống & Hướng dẫn sử dụng

*Phiên bản 1.1 · Cập nhật 17/11/2026 · Áp dụng cho repo `ngvantung/macro-monitor`*

**Mục lục**
1. Macro Monitor là gì
2. Hệ sinh thái Claude và vị trí của từng thành phần
3. Kiến trúc chức năng
4. Thành phần chi tiết
5. Một lần chạy diễn ra như thế nào
6. Hướng dẫn sử dụng & thư viện prompt
7. Lộ trình triển khai
8. Xử lý sự cố
9. Phụ lục: thuật ngữ, cấu trúc thư mục, lịch vận hành

---

## 1. Macro Monitor là gì

Macro Monitor là một **hệ thống nghiên cứu vĩ mô chạy bằng Claude**. Nó làm thay một nhóm chuyên viên nhỏ với các vai trò: người thu thập số liệu, người theo dõi chính sách, người theo dõi thị trường, người kiểm chứng và chuyên gia phân tích.

### Hệ thống trả lời 4 câu hỏi
| # | Câu hỏi | Đầu ra |
|---|---|---|
| 1 | **Chuyện gì đã xảy ra?** | Số liệu: thực tế, dự báo, kỳ trước, điều chỉnh, mức bất ngờ. Kèm nguồn và mức độ tin cậy. |
| 2 | **Vì sao?** | Xu hướng 3–6 kỳ và tương quan giữa các chỉ số: chỉ số nào xác nhận, chỉ số nào mâu thuẫn. |
| 3 | **Ảnh hưởng gì tới Việt Nam?** | Chuỗi lan truyền theo 7 kênh: lãi suất – tỷ giá, dòng vốn, thương mại, thuế quan, dầu, Trung Quốc, tâm lý. Mỗi kênh có mức độ và độ trễ. |
| 4 | **Sắp tới thế nào?** | 3 kịch bản có xác suất, điều kiện kích hoạt và lịch các ngưỡng cần theo dõi. |

### Phạm vi theo dõi
- **Việt Nam:** 7 nhóm chỉ số là tăng trưởng, lạm phát, tín dụng, lãi suất, tỷ giá, thương mại, đầu tư. Thêm chính sách tiền tệ của NHNN (tái cấp vốn, tái chiết khấu, dự trữ bắt buộc, OMO, tín phiếu, room tín dụng, tỷ giá trung tâm) và chính sách tài khóa (thu chi ngân sách, đầu tư công, thuế, trái phiếu chính phủ).
- **Mỹ:** lạm phát (CPI, PCE, PPI), việc làm (NFP, Jobless Claims), tăng trưởng (GDP, ISM, Retail Sales, sản xuất công nghiệp), nhà ở, Fed (FOMC, dot plot, phát biểu), lợi suất, USD.
- **Quốc tế:** PMI Trung Quốc và Eurozone, Ifo, báo cáo năng lượng EIA, OPEC+, IMF.

### Ba nguyên tắc nền tảng
1. **Không bịa số.** Mỗi con số có nguồn và ngày công bố. Không tìm được thì ghi `N/A`.
2. **Kiểm chứng độc lập.** Agent kiểm chứng không phải là agent đã thu thập số liệu.
3. **Có bộ nhớ.** Mỗi lần chạy ghi lại số liệu, thay đổi chính sách và ngưỡng cần theo dõi. Lần sau so sánh với kỳ trước và chấm lại các dự báo cũ.

---

## 2. Hệ sinh thái Claude và vị trí của từng thành phần

### 2.1 Các khái niệm cần phân biệt

| Khái niệm | Là gì trong Claude | Trong Macro Monitor |
|---|---|---|
| **Claude Code** | Môi trường để Claude đọc/ghi file, chạy lệnh, gọi tool, tạo subagent. Có trên terminal, app desktop, claude.ai/code (cloud) | Nơi Macro Monitor chạy |
| **CLAUDE.md** | Hướng dẫn dự án, tự động nạp vào mọi phiên mở thư mục này | 7 quy tắc cứng: không bịa số, chạy song song, bắt buộc kiểm chứng, giờ GMT+7… |
| **Workflow** | *Quy trình* gồm các bước cố định, có đầu vào/đầu ra và cổng chất lượng. Đây là khái niệm thiết kế, không phải tính năng riêng của Claude | 5 workflow W1–W5, mô tả trong `WORKFLOW.md` |
| **Skill** | Gói hướng dẫn (`SKILL.md`) Claude nạp khi cần. Gọi bằng `/tên-skill` hoặc tự kích hoạt theo mô tả | Mỗi workflow là một skill: `/flash`, `/week-ahead`, `/week-review`, `/monthly-vn`, `/outlook` |
| **Orchestrator (điều phối)** | Phiên Claude chính: đang chạy skill, giao việc cho subagent, gộp kết quả | Phiên bạn đang trò chuyện |
| **Agent / Subagent** | Một Claude phụ có vai trò, công cụ và model riêng, định nghĩa trong `.claude/agents/*.md`. Chạy trong context riêng, trả kết quả về cho orchestrator | 6 agent chuyên trách |
| **Tool** | Khả năng Claude gọi được: WebSearch, WebFetch, Read, Write, Bash… | Collector dùng WebSearch/WebFetch; analyst chủ yếu đọc file |
| **Knowledge base** | Tài liệu tham chiếu agent đọc khi làm việc | Thư mục `knowledge/` |
| **Bộ nhớ dài hạn** | File dữ liệu được ghi lại sau mỗi lần chạy | Thư mục `data/` |
| **Môi trường cloud** | Container chạy phiên trên claude.ai/code, có danh sách host được phép truy cập | Môi trường Default đã mở 20 host nguồn chính thức |
| **Scheduled task** | Lịch tự động gửi prompt vào một phiên | Dùng cho W2, W3, W4 định kỳ |
| **Connector (MCP)** | Kết nối tới dịch vụ ngoài như Notion, Google Drive, Slack | Có thể dùng để lưu hoặc gửi báo cáo (giai đoạn mở rộng) |
| **GitHub repo** | Nơi lưu bộ công cụ và lịch sử | `ngvantung/macro-monitor` |

> **Workflow, skill, agent khác nhau ở đâu?**
> - **Workflow** = *làm gì, theo thứ tự nào, đạt chuẩn gì*. Đây là bản thiết kế.
> - **Skill** = *bản hướng dẫn để orchestrator thực hiện một workflow*. Đây là phần hiện thực hóa workflow.
> - **Agent (subagent)** = *người làm một bước cụ thể* trong workflow.
>
> Subagent không tự gọi được subagent khác. Vì vậy việc điều phối luôn nằm ở phiên chính, qua skill.

### 2.2 Hai phiên bản trên hai nền tảng

| | **Macro Monitor (đầy đủ)** | **macro-research (bản nhẹ)** |
|---|---|---|
| Nền tảng | Claude Code (máy local, desktop, claude.ai/code) | claude.ai chat (Project + Skill) |
| Kiến trúc | 1 orchestrator + 6 subagent, chạy song song | 1 luồng duy nhất |
| Kiểm chứng độc lập | Có (fact-checker) | Không (tự kiểm) |
| Bộ nhớ dữ liệu | `data/*.csv` | Lịch sử chat trong Project |
| Tự động định kỳ | Scheduled task | Hạn chế |
| Phù hợp khi | Báo cáo định kỳ, cần độ tin cậy cao | Hỏi nhanh một chỉ số, dùng trên điện thoại |

### 2.3 Sơ đồ hệ sinh thái

```
┌─────────────────────────────── NGƯỜI DÙNG ───────────────────────────────┐
│  Gõ lệnh /week-review, /flash …  ·  Hỏi tự do  ·  Duyệt quyền, mạng      │
└───────────────┬─────────────────────────────────────────────┬────────────┘
                │                                             │
   ┌────────────▼─────────────┐                  ┌────────────▼────────────┐
   │ claude.ai/code · Desktop │                  │ claude.ai chat (Project)│
   │ · Terminal (Claude Code) │                  │ + Skill macro-research  │
   └────────────┬─────────────┘                  └─────────────────────────┘
                │ mở repo ngvantung/macro-monitor
   ┌────────────▼─────────────────────────────────────────────────────────┐
   │ CLAUDE.md (quy tắc) ──► ORCHESTRATOR (phiên chính chạy SKILL W1–W5)  │
   │                              │                                        │
   │      ┌──────────┬───────────┼───────────┬───────────┐                 │
   │      ▼          ▼           ▼           ▼           │ SUBAGENTS       │
   │  vn-data   global-data   policy-     market-        │ (.claude/agents)│
   │  collector collector     tracker     monitor        │                 │
   │      └──────────┴─────┬─────┴───────────┘           │                 │
   │                       ▼                             │                 │
   │                 fact-checker ──► macro-analyst      │                 │
   │                                                     │                 │
   │ knowledge/ (tri thức)   data/ (bộ nhớ)   reports/ (đầu ra)            │
   └───────┬──────────────────────────────────────────────┬───────────────┘
           │ WebSearch / WebFetch (qua mạng môi trường)   │ git push
   ┌───────▼────────────────────────────┐        ┌────────▼────────┐
   │ Nguồn: BLS, BEA, Fed, FRED, NHNN,   │        │ GitHub repo     │
   │ Cục Thống kê, BTC, HNX, CafeF…      │        │ (lưu & đồng bộ) │
   └─────────────────────────────────────┘        └─────────────────┘
   Tùy chọn: Scheduled tasks (tự chạy) · Notion/Drive (lưu báo cáo)
```

---

## 3. Kiến trúc chức năng

Hệ thống chia thành **7 lớp chức năng**:

| Lớp | Chức năng | Thành phần |
|---|---|---|
| **1. Điều phối** | Nhận yêu cầu, xác định phạm vi, giao việc, gộp kết quả, viết báo cáo, lưu trữ | Phiên chính + 5 skill |
| **2. Thu thập** | Tìm số liệu và tin tức theo Data Card chuẩn, mỗi agent một vùng chủ đề riêng | 4 collector |
| **3. Kiểm chứng** | Đối chiếu nguồn, phát hiện sai năm, sai kỳ, sai đơn vị, mâu thuẫn giữa các file; gắn nhãn ✅ ⚠️ ❌ | `fact-checker` |
| **4. Phân tích** | Xu hướng, tương quan, chính sách, lan truyền, kịch bản, quan điểm ngược | `macro-analyst` |
| **5. Tri thức** | Danh sách nguồn, danh mục chỉ số kèm "Tìm thêm", bản đồ lan truyền, mẫu báo cáo, quy tắc thu thập | `knowledge/` |
| **6. Bộ nhớ** | Lịch sử số liệu, nhật ký chính sách, danh sách theo dõi | `data/` |
| **7. Vận hành** | Mạng, quyền, hạn mức, lịch tự động, lưu trữ GitHub | Môi trường cloud, scheduled task, repo |

### Phân công dữ liệu: mỗi số liệu chỉ một agent phụ trách
| Loại dữ liệu | Agent phụ trách |
|---|---|
| Giá thị trường: UST, DXY, tỷ giá USD/VND, lãi suất liên ngân hàng, lợi suất TPCP, VN-Index, khối ngoại, vàng, dầu, FedWatch | `market-monitor` |
| Công cụ và quyết định chính sách: NHNN, Fed, ECB, PBoC, OPEC+; ngân sách, đầu tư công, thuế, đấu thầu TPCP | `policy-tracker` |
| Kinh tế thực Việt Nam: GDP, CPI, IIP, bán lẻ, PMI, xuất nhập khẩu, FDI, tín dụng (số thống kê) | `vn-data-collector` |
| Kinh tế thực Mỹ và quốc tế | `global-data-collector` |

Quy tắc này ra đời sau lần chạy thử đầu tiên. Khi đó ba agent cùng lấy tỷ giá và giá dầu, rồi cho ra ba con số khác nhau.

---

## 4. Thành phần chi tiết

### 4.1 Sáu agent

| Agent | Vai trò | Đầu vào | Đầu ra | Model | Giới hạn |
|---|---|---|---|---|---|
| `vn-data-collector` | Thu thập kinh tế thực Việt Nam | `01_scope.md`, `knowledge/`, `history.csv` | `02_raw_vn.md` | Sonnet | 25 lượt tool |
| `global-data-collector` | Thu thập Mỹ và quốc tế. Có 2 chế độ: lấy số liệu và lấy lịch kinh tế | như trên | `02_raw_global.md` | Sonnet | 25 lượt |
| `policy-tracker` | Theo dõi công cụ và thông điệp chính sách | như trên + `policy_log.csv` | `02_raw_policy.md` | Sonnet | 25 lượt |
| `market-monitor` | Ghi mức giá và phản ứng thị trường | `01_scope.md` | `02_raw_market.md` | Sonnet | 25 lượt |
| `fact-checker` | Kiểm chứng độc lập, đối chiếu chéo giữa các file | Các file `02_raw_*` | `03_verification.md` | Sonnet | 60 lượt |
| `macro-analyst` | Phân tích theo khung 11 mục | Toàn bộ run + `knowledge/` + `data/` | `04_analysis.md` | Opus | Hầu như không tìm web |

**Vì sao chia model như vậy:** thu thập và kiểm chứng là việc nhiều thao tác nhưng ít suy luận, nên dùng Sonnet để tiết kiệm. Phân tích tương quan và dựng kịch bản cần suy luận sâu, nên dùng Opus.

### 4.2 Năm skill (workflow)

| Lệnh | Workflow | Agent tham gia | Đầu ra chính | Thời hạn |
|---|---|---|---|---|
| `/flash <sự kiện>` | W1 – Báo cáo nhanh | 1–2 collector + market-monitor (+ policy-tracker) + fact-checker + analyst (bản ngắn) | Báo cáo 1 trang | ≤ 2 giờ sau công bố |
| `/week-ahead` | W2 – Lịch tuần tới | global-data-collector (chế độ lịch) + policy-tracker + market-monitor + fact-checker + analyst | Lịch sự kiện kèm dự báo, 3 sự kiện chính và kịch bản phản ứng | Trước phiên thứ Hai |
| `/week-review` | W3 – Tổng kết tuần | Cả 6 agent | Báo cáo đầy đủ 9 phần | Cuối tuần |
| `/monthly-vn` | W4 – Vĩ mô Việt Nam tháng | Cả 6 agent, tập trung Việt Nam | Bảng điều khiển 7 nhóm chỉ số, bảng công cụ NHNN | ≤ 2 ngày sau họp báo Cục Thống kê |
| `/outlook` | W5 – Triển vọng quý | Cả 6 agent, chế độ sâu | Đồng thuận dự báo của các tổ chức, ma trận Fed × NHNN, 3 kịch bản | Mỗi quý |

### 4.3 Tri thức (`knowledge/`)
| File | Nội dung | Ai dùng |
|---|---|---|
| `sources.md` | Nguồn dữ liệu theo 3 mức tin cậy: A (chính thức), B (tổng hợp có uy tín), C (báo chí) | Collector, fact-checker |
| `indicators.md` | Danh mục chỉ số. Mỗi chỉ số có cột **"Tìm thêm"**: các chỉ số liên quan phải tìm cùng | Orchestrator (S1), collector |
| `collection-rules.md` | Quy tắc thu thập: phân công, kiểm tra năm, thứ trong tuần, biên hợp lý, Data Card chuẩn, giới hạn lượt | Mọi collector |
| `transmission.md` | Bản đồ tương quan, 7 kênh lan truyền, tác động theo ngành, ma trận Fed × NHNN | Analyst |
| `report-template.md` | Mẫu báo cáo nhanh, lịch tuần, báo cáo tuần/tháng | Orchestrator (S5) |

### 4.4 Bộ nhớ (`data/`)
| File | Nội dung | Ghi khi nào |
|---|---|---|
| `history.csv` | Số liệu đã được xác nhận ✅ (ngày công bố, quốc gia, chỉ số, kỳ, thực tế, dự báo, nguồn, nhãn…) | Bước S6 |
| `policy_log.csv` | Mỗi lần một công cụ chính sách thay đổi | Bước S6 |
| `watchlist.md` | Sự kiện sắp tới, ngưỡng làm thay đổi nhận định, giả thuyết cần chấm điểm | Bước S7. Đầu mỗi workflow phải đọc file này |

### 4.5 Cổng chất lượng & nhãn tin cậy
| Cổng | Đạt khi |
|---|---|
| **G1 Phạm vi** | Xác định được kỳ, danh sách chỉ số kèm mức ưu tiên P1/P2/P3, và các mốc tham chiếu |
| **G2 Kiểm chứng** | Mức chặn: không còn số ❌ trong các số chính. Mức cảnh báo: số ⚠️ vẫn được dùng nhưng phải gắn nhãn |
| **G3 Phân tích** | Mỗi chuỗi lan truyền có cơ chế, bằng chứng, mức độ và độ trễ. Có quan điểm ngược. Tách bạch số liệu, nhận định của tổ chức và đánh giá riêng |
| **G4 Báo cáo** | Đúng mẫu, có kết luận 1 câu ở đầu, có danh sách nguồn và mục "Chưa xác minh được" |

**Nhãn tin cậy:**
- ✅ = nguồn chính thức, hoặc ít nhất 2 nguồn độc lập khớp nhau.
- ⚠️ = chỉ 1 nguồn, hoặc các nguồn lệch nhẹ.
- ❌ = sai hoặc không tìm thấy. Không được dùng.
- `[tự tính]` = số do hệ thống tự tính, có ghi công thức.

---

## 5. Một lần chạy diễn ra như thế nào

Ví dụ: `/week-review`.

| Bước | Ai làm | Việc | File tạo ra trong `runs/<ngày>_W3_<tên>/` |
|---|---|---|---|
| **S1 Phạm vi** | Orchestrator | Đọc watchlist và báo cáo kỳ trước. Liệt kê sự kiện trong tuần, gán ưu tiên P1/P2/P3, ghi các mốc tham chiếu | `01_scope.md` |
| **S2 Thu thập** | 4 collector **song song** | Mỗi agent tìm phần của mình theo Data Card | `02_raw_vn.md`, `02_raw_global.md`, `02_raw_policy.md`, `02_raw_market.md` |
| **S2.5 Đối chiếu** | Orchestrator | Tìm số mâu thuẫn giữa 4 file | Cập nhật `01_scope.md` |
| **S3 Kiểm chứng** | fact-checker | Gắn nhãn ✅ ⚠️ ❌, xác định G2 đạt hay không. Không đạt thì gửi collector tìm lại 1 vòng | `03_verification.md` |
| **S4 Phân tích** | macro-analyst | Khung 11 mục và checklist G3 | `04_analysis.md` |
| **S5 Báo cáo** | Orchestrator | Viết theo mẫu | `report.md` → copy sang `reports/` |
| **S6 Lưu trữ** | Orchestrator | Ghi số ✅ vào `history.csv`, thay đổi chính sách vào `policy_log.csv` | |
| **S7 Theo dõi** | Orchestrator | Cập nhật `watchlist.md`, ghi "Bài học" | |

**Thời gian và chi phí tham khảo:** lần chạy thử W3 ngày 16/11/2026 mất khoảng 25–30 phút. Bốn collector chạy song song đã chạm giới hạn 5 giờ của gói.

---

## 6. Hướng dẫn sử dụng & thư viện prompt

### 6.1 Chuẩn bị (làm một lần)
1. **Mở repo bằng Claude Code:**
   - Trên claude.ai/code: tạo phiên mới và chọn repo `ngvantung/macro-monitor`.
   - Trên máy: `git clone`, rồi chạy `claude` trong thư mục repo.
2. **Mạng (cloud):** môi trường phải cho phép các host nguồn trong `knowledge/sources.md`. Môi trường Default đã được mở 20 host.
3. **Quyền:** ở chế độ auto, một số thao tác như `curl` hay khởi chạy agent có truy cập mạng có thể bị chặn. Nếu muốn chạy tự động trọn vẹn, thêm quy tắc cho phép trong `.claude/settings.json`. Có thể nhờ Claude: *"thêm quyền cho WebFetch tới các host trong sources.md"*.
4. **Kiểm tra nhanh:** gõ `/flash CPI Mỹ tháng gần nhất` và xem tỉ lệ ✅ trong báo cáo.

### 6.2 Lệnh workflow và prompt mẫu

**W1 – Báo cáo nhanh theo sự kiện**
```
/flash CPI Mỹ tháng 11/2026
```
```
/flash NHNN phát hành tín phiếu trở lại tuần này – đo phản ứng tỷ giá và liên ngân hàng
```
```
/flash FOMC 09/12/2026 – tập trung dot plot 2027 và tác động lên USD/VND
```

**W2 – Lịch tuần tới**
```
/week-ahead
```
```
/week-ahead – ưu tiên các sự kiện ảnh hưởng tỷ giá VND và nhóm cổ phiếu ngân hàng
```

**W3 – Tổng kết tuần**
```
/week-review
```
```
/week-review tuần 16–20/11/2026 – so với kỳ vọng trong báo cáo W2 cùng tuần, chấm điểm các giả thuyết trong watchlist
```

**W4 – Vĩ mô Việt Nam tháng**
```
/monthly-vn tháng 11/2026
```

**W5 – Triển vọng quý**
```
/outlook Q1/2027 – nhấn mạnh rủi ro tỷ giá và lộ trình lãi suất NHNN
```

### 6.3 Prompt hỏi tự do (không cần lệnh)
Nhờ CLAUDE.md và mô tả của các skill, Claude tự chọn workflow phù hợp:
```
CPI Việt Nam tháng 10 có gì đáng chú ý? So với mục tiêu năm và tác động tới NHNN.
```
```
Lợi suất UST 10Y vượt 5% thì tỷ giá USD/VND và VN-Index thường phản ứng thế nào? Dùng số liệu đã có trong data/history.csv.
```
```
Tóm tắt các công cụ NHNN đã dùng trong 3 tháng qua từ data/policy_log.csv và đánh giá NHNN đang ưu tiên mục tiêu nào.
```

### 6.4 Prompt khi bạn đã có số liệu
Dán số liệu vào để Claude không phải tìm lại. Nhanh hơn và tiết kiệm hạn mức:
```
Số liệu CPI VN tháng 11/2026 (Cục Thống kê, công bố 06/12): CPI +0,15% MoM, 4,95% YoY; lạm phát cơ bản 3,98% YoY.
Không tìm lại các số này. Hãy tìm thêm các mục "Tìm thêm" trong knowledge/indicators.md, chạy fact-checker cho phần tìm thêm, rồi phân tích theo /flash.
```

### 6.5 Prompt gọi trực tiếp một agent
Dùng khi chỉ cần một phần việc:
```
Dùng agent market-monitor: lấy UST 2Y/10Y, DXY, Brent, USD/VND trung tâm và VN-Index đóng cửa hôm qua. Ghi vào runs/<hôm nay>_adhoc/02_raw_market.md.
```
```
Dùng agent fact-checker để kiểm lại các số trong reports/2026-11-16_W3_tuan-9-15-11.md đang mang nhãn ⚠️.
```
```
Dùng agent macro-analyst: với giả định Fed tăng 50 bp ngày 09/12, dựng lại kịch bản cho tỷ giá, lãi suất liên ngân hàng và ngành ngân hàng VN.
```

### 6.6 Prompt bảo trì và cải tiến
```
Đọc mục "Ghi chú cho người thiết kế agent" trong các file runs/ gần nhất, đề xuất sửa .claude/agents và knowledge/collection-rules.md, rồi áp dụng.
```
```
Thêm chỉ số "Giá vàng SJC và chênh lệch với vàng thế giới" vào hệ thống: agent phụ trách, nguồn, mục Tìm thêm, kênh lan truyền.
```
```
Chấm điểm các giả thuyết trong data/watchlist.md đã đến hạn. Ghi kết quả đúng/sai và bài học vào báo cáo.
```
```
Đặt scheduled task: thứ Hai 07:15 chạy /week-ahead, thứ Bảy 14:10 chạy /week-review, ngày 6 hằng tháng 15:10 chạy /monthly-vn (giờ Việt Nam).
```

### 6.7 Viết prompt hiệu quả
| Nên | Không nên |
|---|---|
| Nêu **kỳ cụ thể**: "CPI Mỹ tháng 11/2026" | "CPI mới nhất" (dễ lẫn kỳ, lẫn năm) |
| Nêu **góc quan tâm**: "tác động lên USD/VND và ngân hàng" | Hỏi chung chung "phân tích CPI" |
| Dán số liệu bạn đã có và dặn "không tìm lại" | Bắt hệ thống tìm lại số bạn đã có |
| Dùng lệnh `/` cho báo cáo định kỳ | Tự mô tả lại cả quy trình trong prompt |
| Hỏi tiếp trong cùng phiên: "giải thích mục 6 kỹ hơn" | Mở phiên mới cho mỗi câu hỏi tiếp theo |
| Yêu cầu "chỉ dùng số ✅" khi cần cho báo cáo gửi người khác | Dùng số ⚠️ mà bỏ mất nhãn |

### 6.8 Mẹo vận hành
- **Chọn giờ chạy:** W3 nên chạy chiều thứ Bảy, khi công cụ tìm kiếm đã có số đóng cửa thứ Sáu. W1 nên chạy 30–60 phút sau giờ công bố.
- **Tiết kiệm hạn mức:** bỏ các collector không cần (W1 thường chỉ cần 2–3 agent). Dán sẵn số liệu nếu có. Tránh chạy nhiều workflow cùng lúc.
- **Đọc báo cáo:** đọc kết luận 1 câu, sau đó đến mục "Chưa xác minh được" để biết giới hạn của báo cáo, rồi mới vào chi tiết.
- **Sau mỗi lần chạy:** đẩy `reports/` và `data/` lên GitHub để phiên sau có bộ nhớ. Phiên cloud bị xóa khi hết.

---

## 7. Lộ trình triển khai

| Giai đoạn | Thời gian | Việc chính | Tiêu chí hoàn thành |
|---|---|---|---|
| **0. Thiết kế & chạy thử** ✅ | Đã xong (16/11/2026) | Xây 6 agent, 5 skill, knowledge, data. Chạy thử W3, chỉnh agent theo kết quả, đẩy lên GitHub | Báo cáo W3 đầu tiên. G3 đạt |
| **1. Ổn định dữ liệu** | Tuần 1–2 | Kiểm tra WebFetch tới 20 host (FRED CSV, NHNN, Cục Thống kê). Chạy `/flash Retail Sales Mỹ` (17/11) và `/flash` thêm 2–3 sự kiện. Tìm nguồn ổn định cho OMO, tín phiếu, liên ngân hàng. Bổ sung `policy_log` với các mức hiện hành | Tỉ lệ ✅ trên số chính ≥ 80%. Không còn số ❌ vì sai năm. W1 xong trong ≤ 30 phút |
| **2. Vận hành định kỳ** | Tuần 3–6 | Chạy tay đều đặn W2 (thứ Hai), W3 (thứ Bảy), W4 (ngày ~6). Mỗi lần ghi "Bài học" và chỉnh `collection-rules.md`. Chấm điểm giả thuyết sau ngày 13/12 | 4 tuần liên tiếp đủ W2 và W3. `history.csv` ≥ 150 dòng. Tỉ lệ dự báo đúng hướng được ghi lại |
| **3. Tự động hóa** | Tháng 2 | Đặt scheduled task cho W2, W3, W4. Thêm quyền để agent không bị chặn khi chạy không có người. Tự động commit `reports/` và `data/` lên GitHub sau mỗi lần chạy. Đặt thông báo khi báo cáo xong | Báo cáo tự xuất hiện đúng lịch 4 tuần liên tục, không cần can thiệp |
| **4. Mở rộng** | Tháng 3 trở đi | Dashboard từ `history.csv` (biểu đồ xu hướng, bảng 7 nhóm chỉ số). Đồng bộ báo cáo sang Notion hoặc Google Drive. Thêm chỉ số (vàng SJC, tồn kho BĐS, Nhật/BoJ). Chạy W5 Outlook hằng quý và chấm điểm dự báo quý trước. Cân nhắc Claude Agent SDK nếu cần chạy ổn định ở quy mô lớn | Dashboard cập nhật tự động. Báo cáo quý có bảng chấm điểm dự báo |

### Chỉ số đo hiệu quả hệ thống (KPI)
| KPI | Mục tiêu | Lần chạy thử |
|---|---|---|
| Tỉ lệ ✅ trên số chính | ≥ 80% | 57% |
| Số ❌ lọt vào bản thô | ≤ 2 | 5 (đã bị chặn ở G2) |
| Thời gian W1 | ≤ 30 phút | – |
| Thời gian W3 | ≤ 45 phút | ~30 phút |
| Tỉ lệ dự báo đúng hướng (kịch bản cơ sở) | ≥ 60% | Chấm lần đầu sau 13/12 |

---

## 8. Xử lý sự cố

| Triệu chứng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| Agent báo WebFetch `ENOTFOUND` / bị chặn | Host chưa có trong danh sách được phép của môi trường | Nhờ Claude thêm host vào môi trường (bạn duyệt trên thẻ hiện ra), hoặc sửa trong cài đặt môi trường cloud |
| Agent dừng giữa chừng, lỗi 429 "5-hour limit" | Hết hạn mức gói vì chạy nhiều agent song song | Chờ hạn mức reset rồi chạy tiếp. Giảm số collector, dán sẵn số liệu, chạy lần lượt |
| Số liệu sai năm (ví dụ tỷ giá 24.267 năm 2026) | Kết quả tìm kiếm trộn nhiều năm | Đã có quy tắc kiểm tra năm và biên hợp lý. Ưu tiên FRED CSV và trang gốc |
| Không có số đóng cửa thứ Sáu | Chạy quá sớm, chỉ mục tìm kiếm chưa cập nhật | Chạy W3 chiều thứ Bảy hoặc muộn hơn |
| "Permission denied by auto mode classifier" | Chế độ auto chặn thao tác có truy cập mạng | Cho phép thủ công khi được hỏi, hoặc thêm quy tắc quyền |
| G2 không đạt | Có số ❌ trong số chính | Orchestrator gửi lại collector 1 vòng. Vẫn không được thì chuyển vào "Chưa xác minh được" |
| Báo cáo thiếu so sánh kỳ trước | `history.csv` hoặc `reports/` trống ở phiên mới | Đảm bảo đã đẩy `data/` và `reports/` lên GitHub sau mỗi lần chạy |

---

## 9. Phụ lục

### 9.1 Thuật ngữ
| Thuật ngữ | Nghĩa |
|---|---|
| Actual / Consensus / Previous / Revision | Số thực tế / dự báo đồng thuận / kỳ trước / số điều chỉnh của kỳ trước |
| Surprise | Actual − Consensus |
| Data Card | Bảng chuẩn ghi một số liệu, có đủ các trường bắt buộc |
| Tier A/B/C | Mức tin cậy của nguồn: chính thức / tổng hợp có uy tín / báo chí |
| P1/P2/P3 | Mức ưu tiên thu thập |
| Run | Một lần chạy workflow, lưu trong `runs/<ngày>_<W#>_<tên>/` |

### 9.2 Cấu trúc thư mục
```
macro-monitor/
├── CLAUDE.md                 Quy tắc cứng, tự nạp vào mọi phiên
├── WORKFLOW.md               Quy chuẩn S1–S7, G1–G4, lịch vận hành
├── README.md                 Tổng quan và việc còn dở
├── docs/HUONG-DAN.md         Tài liệu này
├── .claude/
│   ├── agents/               6 subagent
│   └── skills/               5 workflow: flash, week-ahead, week-review, monthly-vn, outlook
├── knowledge/                sources, indicators, collection-rules, transmission, report-template
├── data/                     history.csv, policy_log.csv, watchlist.md
├── reports/                  Báo cáo cuối cùng
└── runs/                     File trung gian (không đưa lên repo)
```

### 9.3 Lịch vận hành mẫu (GMT+7)
| Thời điểm | Việc |
|---|---|
| Thứ Hai 07:15 | `/week-ahead` |
| Ngày làm việc đầu tháng 08:00 | `/flash PMI VN và PMI Trung Quốc` |
| Khoảng ngày 6, 15:10 | `/monthly-vn` |
| 30–60 phút sau CPI, NFP, PCE Mỹ (khoảng 20:30–21:30) | `/flash <sự kiện>` |
| Sáng sau FOMC | `/flash FOMC` |
| Thứ Bảy 14:10 | `/week-review` |
| Đầu quý | `/outlook` |
