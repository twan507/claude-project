# Claude Project — Hệ thống agent phân tích chứng khoán

Bộ 2 agent độc lập phục vụ phân tích cổ phiếu Việt Nam. Mỗi agent là 1 Claude Desktop Project riêng. File này là điểm vào cho con người và AI session sau hiểu toàn bộ kiến trúc, biết khi nào dùng agent nào, khi nào extend cái nào.

---

## 1. Tổng quan dự án

### 1.1. Hai agent

| Agent | Folder | Mục đích | Số file |
|---|---|---|---|
| **analysis_agent** | `analysis_agent/` | Phân tích cổ phiếu Việt Nam đa giai đoạn, output MD final structured (memo, báo cáo tuần, khuyến nghị mua, single-stock deep report) | 49 |
| **db_agent** | `agent_db/` | Phân tích single-shot nhanh: tra cứu, query MongoDB `agent_db`, đưa nhận định lẻ không qua workflow đa stage. v2 (fnx05): audience NĐT khách Finext, thêm tầng phase & danh mục | 7 |

### 1.2. Use case từng agent

- **analysis_agent** — khi cần một báo cáo deliverable hoàn chỉnh: viết memo deep-dive cho 1 mã, sinh báo cáo thị trường tuần 12 phần, soạn pitch khuyến nghị mua gửi khách hàng, lập portfolio plan, review cycle. Workflow có nhiều giai đoạn + checkpoint, output MD final structured (user copy/save thủ công).
- **db_agent** — khi chỉ cần tra cứu hoặc nhận định nhanh không cần qua workflow: "VNM giá bao nhiêu", "thị trường tuần qua dòng tiền thế nào", "ngành thép Q1 ra sao". Output dạng conversational, không cần file deliverable.

### 1.3. Quy tắc kiến trúc cốt lõi

**Mỗi agent độc lập 100%.** File trong agent này KHÔNG reference path / tên file của agent khác. Communication giữa 2 agent qua MD file mà user copy/paste thủ công, không có shared state hay cross-agent runtime call.

Hệ quả thực tế:
- Sửa 1 agent không phá agent khác
- Có thể swap / extend / tắt 1 agent độc lập
- Cùng 1 knowledge base có thể tồn tại ở 2 agent dưới dạng duplicate (hiện tại `K_agent_db_*` trong analysis_agent và `agent_db_*` trong db_agent có content gần identical — đây là chấp nhận được vì 2 agent độc lập, chứ không refactor về 1 nguồn).

---

## 2. Triển khai (Claude Desktop)

### 2.1. Mỗi agent = 1 Claude Desktop Project

Tạo 2 project riêng trong Claude Desktop app:

| Project name (đề xuất) | Source folder | Custom Instructions | Knowledge files |
|---|---|---|---|
| `Analysis Agent` | `analysis_agent/` | Paste nội dung `analysis_agent/system_prompt.md` | Upload toàn bộ file `.md` còn lại trong folder (48 file) |
| `DB Agent` | `agent_db/` | Paste nội dung `agent_db/system_prompt.md` | Upload `agent_db_01` đến `agent_db_06` (6 file) |

### 2.2. Khi cần update file

Sửa file ở folder gốc → re-upload vào project knowledge tương ứng. Claude Desktop project knowledge không tự sync với filesystem — phải update thủ công.

Khi sửa `system_prompt.md` của agent nào → paste lại vào ô Custom Instructions của project đó.

### 2.3. MongoDB connection

`analysis_agent` và `db_agent` đều giả định có quyền read MongoDB database tên `agent_db`. Tools để query DB không nằm trong project knowledge — phải được provide qua Claude Desktop integration / MCP server cấu hình bên ngoài. Schema 31 collection và query patterns được document trong `agent_db_01.md` (db agent) và `K_agent_db_01.md` (analysis_agent).

### 2.4. Behavioral guidelines (`CLAUDE.md`)

File `CLAUDE.md` ở root chứa behavioral guidelines chung cho mọi session AI làm việc trên project này (think before coding, simplicity first, surgical changes, goal-driven execution). File này tham khảo cho con người dev và AI khi maintain dự án — KHÔNG upload vào project knowledge của 3 agent (3 agent có system_prompt riêng).

---

## 3. `analysis_agent` — chi tiết

### 3.1. Kiến trúc 3 layer + 1 index

Pack vận hành theo kiến trúc module 3 layer:

- **K (Knowledge)** — schema, methodology, translation rules, query patterns, domain constraints. "Biết gì". Là thư viện, P và O re-queryable nhiều lần xuyên suốt session.
- **P (Process)** — workflow pipeline có thứ tự, checkpoint, audit. "Làm theo bước nào".
- **O (Output)** — structure rigid của deliverable (heading bắt buộc, độ dài, citation, K hygiene), tone, format, length, xưng hô. "Trình bày gì ở đâu". Output cuối là **MD final**.

**Index:** `KERNEL_SKELETON.md` ở gốc folder — liệt kê pack có sẵn + trigger activation. Đọc đầu session, mỗi session 1 lần.

### 3.2. Pack có sẵn

| Pack | Files | Mục đích |
|---|---|---|
| `K_agent_db` | 7 (`_00` master + `_01` đến `_06`) | Knowledge MongoDB `agent_db` chứng khoán VN (31 collection, gồm tầng phase & danh mục ở `_06`) |
| `K_sector_framework` | 1 | Khung phân tích ngành CFA institutional buy-side (DD/MP/SI/PM/ESG + per-sector quick-ref cho 18 ngành whitelist + Industry 4.0 lens) |
| `P_invest_memo` | 10 (`_00` master + `_01` đến `_09`) | Quy trình đầu tư cá nhân, horizon 1-6 tháng, long only, portfolio < 1 triệu USD |
| `P_weekly_overview` | 5 (`_00` master + `_01` đến `_04`) | Broadcast tổng quan thị trường tuần 12 phần fundamental-driven, audience nội bộ + KH |
| `P_vbse_strategy` | 10 (`_00` master + `_01` đến `_09`) | Chiến lược đầu tư VBSE deep nội bộ, 2 cycle (monthly parent + weekly child), 6 trục, 2-phase watchlist |
| `P_stock_report` | 5 (`_00` master + `_01` đến `_04`) | Báo cáo phân tích chuyên sâu 1 cổ phiếu (single hoặc pair 2-3 mã). Stage 1 16 sub-step (1a-1p) + 4 type framework (SXKD/NH/CK/BH; SXKD có mục 2.6 chuỗi giá trị áp dụng Porter + Smile Curve + GVC + Industry 4.0 + CFA Sector Analysis 2020) + 3 depth mode + audience flex. BCTC PDF mandatory |
| `O_invest_memo` | 7 (`_00` master + `_01` đến `_06`) | Render spec cho 6 deliverables của `P_invest_memo` |
| `O_weekly_overview` | 1 (`_00`) | Render spec broadcast tuần 12 phần rigid + 3 mode branding |
| `O_vbse_strategy` | 1 (`_00`) | Render spec chiến lược 2 mode (monthly + weekly) flex 6 trục |
| `O_stock_report` | 1 (`_00`) | Render spec báo cáo 1 cổ phiếu 6-7 phần rigid + 3 depth mode + audience flex (nội bộ/KH) |

### 3.3. `P_invest_memo` — workflow tóm tắt

5 giai đoạn + giai đoạn 6 song song:

```
Giai đoạn 1 (Tier 0)  — Gate vĩ mô + catalyst (file 01)        → CP1
Giai đoạn 2 (Tier 1)  — Chọn 3-5 ngành (file 02)               → CP2
Giai đoạn 3 (Tier 2)  — Screen 6-10 mã/ngành (file 03)         → CP3
Giai đoạn 4 (Tier 3)  — Chấm điểm top 3/ngành (file 04)        → CP4
Giai đoạn 5           — Memo deep-dive:
  Tier 5A (file 05)   — PDF forensic                            → CP5A per-stock
  Tier 5B (file 06)   — Valuation modeling                      → CP5B per-stock
  Tier 5C (file 07)   — Memo 7 phần                             → CP5C per-stock
Giai đoạn 6 (song song):
  Tier 6 (file 08)    — Portfolio construction
  Tier 7 (file 09)    — Monitoring + exit (4 review cycles: daily/weekly/monthly/quarterly)
```

**Note numbering:** Tier mapping không liên tục — bỏ qua Tier 4 do giai đoạn 5 (deep-dive) đã được tách thành 3 sub-tier 5A/5B/5C ở refactor lịch sử. File con vẫn dùng "Tier 5A/5B/5C" giữ nguyên để tránh phá hệ thống cross-reference.

**6 nguyên tắc Agent bất biến** (xem `P_invest_memo_00.md` mục 5):
1. Không skip variant perception (flex+downgrade nếu không có, không auto-reject)
2. Không vào position nếu chưa viết exit trigger measurable
3. Không size mỗi phiên giải ngân vượt 5% ADV 20 phiên (tổng vị thế = 5% × ADV × N với N=2-4 phiên)
4. Bear case steelman trước khi long (flex+downgrade size 30-70% nếu yếu, không auto-reject)
5. Dòng tiền dương + catalyst tiêu cực → loại
6. Mỗi giai đoạn kết bằng checkpoint, không tự chuyển tier

### 3.4. `P_weekly_overview` — 12 phần fundamental-driven

```
Pre-flight: hỏi file W-1 + context + branding info

Stage 1: Compose phần 2-9
  Phần 2  Review tuần trước (3 scorecard tables)
  Phần 3  Bối cảnh quốc tế
  Phần 4  Thị trường Việt Nam (aggregate 18 ngành whitelist)
  Phần 5  Vĩ mô & hàng hoá (institutional table 5 cột: Magnitude + Persistence)
  Phần 6  Biến động 18 ngành whitelist + earnings beat candidate
  Phần 7  Top dẫn dắt 2 góc nhìn + cảnh báo trap setup
  Phần 8  Tin tức & catalyst (+ conviction impact)
  Phần 9  Định vị VNINDEX + 3 kịch bản fundamental-driven + Risk map

CHECKPOINT 1: Regime + Sector bias (conviction + disconfirming bắt buộc)

Stage 2: Compose phần 10-12 + Phần 1
  Phần 10  Watchlist tách 2 hướng (cơ hội + cảnh báo)
  Phần 11  Lịch sự kiện tuần tới
  Phần 12  Tuyên bố miễn trừ trách nhiệm
  Phần 1   Tóm tắt điều hành (Key calls / Watch / Risk) — viết cuối
```

**Constraint chính:**
- Whitelist 18 ngành default (xem `K_agent_db_01` Section B); override khi user yêu cầu cụ thể
- 3 kịch bản phần 9 trigger primary là vĩ mô/cơ bản/chính sách/catalyst (technical chỉ confirmation phụ ≤30%)
- Cap technical toàn báo cáo ≤15%
- Mỗi call (regime, sector bias, watchlist mã) có conviction HIGH/MID/LOW + horizon 1-2 tuần / 2-4 tuần + 1-2 disconfirming signal
- KHÔNG dùng chỉ báo trend nội bộ
- Wording observation, không command
- Rank ngành tự tổng hợp theo `week_score` (DB không lưu industry_rank tĩnh)

### 3.5. `P_vbse_strategy` — Monthly parent + Weekly child, 6 trục flex

Pack chia 10 file:

```
_00  Master (philosophy fundamental supremacy + weight balance + 4 nguyên tắc)
_01  Trục 1 — Vĩ mô & tài chính
_02  Trục 2 — Định vị thị trường VN (fundamental-first)
_03  Trục 3 — Themes & narratives
_04  Trục 4 — Sector allocation (whitelist 18)
_05  Trục 5 — Risk scenarios (trigger macro/fundamental/policy ONLY)
_06  Trục 6 — Watchlist 2-phase (Phase 1 Screen cơ bản + Phase 2 Bucket entry PTKT)
_07  Workflow Monthly (Pre-flight + Stage 0 eval + CP0 + Stage 1-3 + CP1)
_08  Workflow Weekly (HARD GATE + Stage 0 + tracking với technical-as-noise rule + rebucket)
_09  User overlay + Self-audit + Edge cases + Output contract
```

**Constraint chính:**
- Fundamental supremacy: PTKT chỉ tồn tại ở Phase 2 Bucket entry Trục 6 (entry timing). Cap technical ≤15% toàn báo cáo (trừ Phase 2 Bucket)
- Trục 5 Risk: trigger macro/fundamental/policy ONLY, cấm technical primary
- Whitelist 18 ngành áp dụng default; user override được phép
- Weekly HARD GATE: không có monthly active → REFUSE
- Conviction + horizon + disconfirming bắt buộc mỗi theme/sector/mã
- Watchlist Phase 1: cơ bản + catalyst + thanh khoản (cấm PTKT filter). Phase 2: technical_zone đa khung → Bucket 1/2/3

### 3.6. `P_stock_report` — Single-stock deep analysis (ad-hoc / pair compare)

Pack chia 5 file (+ render spec `O_stock_report_00`):

```
_00  Master (mục đích, scope, differentiate với P_invest_memo Tier 5C, 6 nguyên tắc)
_01  Pre-flight 6 câu + Stage 1 Data Acquisition 16 sub-step (1a-1p)
       1a stock info + type SXKD/NH/CK/BH → 1b FA DB → 1c dòng tiền + tech zone
       → 1d khối ngoại + tự doanh → 1e major shareholders → 1f corporate actions
       → 1g news DB → 1h web search news → 1i BCTC PDF forensic 15-point
       → 1j sector context → 1k macro → 1l peer compare → 1m ADV
       → 1n earnings calendar → 1o ESG controversy
       → 1p Value chain data (top KH/NCC/channel/R&D/Industry 4.0 — SXKD mandatory)
_02  Type-specific framework cho 4 type (SXKD/NH/CK/BH)
       SXKD có mục 2.6 Chuỗi giá trị 10 sub-mục áp dụng 6 framework chuẩn quốc tế:
       Porter Value Chain (1985) + Porter 5 Forces (1979) + Smile Curve (Stan Shih 1992)
       + GVC governance (Gereffi 2005) + Industry 4.0 (CFA Sector Analysis 2020)
       + CFA chapter mapping (21 chapter ↔ VN whitelist)
_03  Stage 2 compose + 6-7 phần output rigid + 3 depth mode (Quick/Standard/Deep)
       + Variant Perception rule + Pair compare mode + Checkpoint 1+2
       (Phần 2 sub-section 3 Vị trí chuỗi giá trị MANDATORY SXKD với 6 sub-sub 3a-3f)
_04  Self-audit 47 điểm SXKD / 35 điểm NH/CK/BH + Edge cases + 10 failure modes
```

**Constraint chính:**
- **BCTC PDF mandatory** — REFUSE chạy nếu không upload (gate strict tuyệt đối)
- **Long-only** (Long / Watch / Avoid, không Short)
- **Web search VN cho equity, EN cho macro** (tài chính/dầu khí/kim loại)
- **Peer compare internet-first** + filter ADV ≥ 30 tỷ/ngày + market cap top 50
- **Strict reject Long pattern:** dòng tiền dương + catalyst tiêu cực material → auto Watch
- **Conviction CAP** at LOW cho penny (< 1.000 tỷ), at MID cho newly listed (< 2 năm)
- **Audience flex** (nội bộ analyst / KH) — wording + K hygiene khác nhau; KH KHÔNG nhận TP/SL số cụ thể
- **Value chain MANDATORY cho SXKD Standard+** — áp dụng đầy đủ 6 framework (Porter VC + 5 Forces + Smile Curve + GVC + Industry 4.0 + CFA). SKIP NH/CK/BH (đã có lens type-specific)

**Quan hệ với `P_invest_memo`:** Complement, không thay thế. P_stock_report dùng pre-screening / pitch nhanh / ad-hoc deep-dive 1 mã. P_invest_memo Tier 5C dùng full conviction memo cycle (sau Tier 0-3). KHÔNG auto-escalate sang Tier 5C — user phải explicit yêu cầu.

### 3.7. Triết lý flex+downgrade (xuyên suốt P_invest_memo + P_vbse_strategy)

Khi gate (Variant Perception, Bear Case, R/R) không pass strict, **agent KHÔNG tự reject** mã. Thay vào đó:
- Flag cảnh báo cụ thể (lý do gate yếu)
- Downgrade conviction / size theo mức độ yếu (ví dụ: bear yếu vừa → giảm size 30-50%, bear target dưới giá hiện tại → giảm size 50-70% coin-flip bet)
- User quyết định cuối: proceed với size nhỏ + audit log, hoặc loại mã

Triết lý: discipline ở dạng force user explicit aware về rủi ro, không che giấu. Chi tiết Gate 1 + Gate 2 ở `P_invest_memo_07.md` mục 4.

---

## 4. `K_sector_framework` — knowledge phụ trợ phân tích ngành

Pack K mới (1 file `K_sector_framework.md`) cung cấp khung phân tích ngành theo chuẩn institutional buy-side (chắt lọc từ CFA Sector Analysis Framework 2020), bao gồm:

- **Universal 5-dimension framework:** Demand Drivers / Market Position / Structural Influences / Performance Metrics / ESG — áp dụng cho mọi ngành
- **Per-sector quick-reference** cho 10-12 ngành trong whitelist 18 có direct CFA cover (NGANHANG, TIENICH, BDS, KCN, BANLE, VANTAI, CONGNGHE, XAYDUNG, THUCPHAM, NONGNGHIEP, CHUNGKHOAN, BAOHIEM override)
- **Guidance generic** cho 6-7 ngành whitelist không có direct CFA cover (DAUKHI, HOACHAT, KIMLOAI, DETMAY, KHOANGSAN, THUYSAN, CONGNGHIEP)
- **Industry 4.0 lens** — digital footprint, automation, AI/IoT disruption áp dụng cross-sector

**Khi nào active:** P pack tham chiếu khi cần deep-dive sector-level analysis. Cụ thể:
- `P_invest_memo_05/06/07` (Tier 5A/B/C deep-dive memo) — section "Business" trong memo 7 phần
- `P_vbse_strategy_04` (Trục 4 Sector allocation) — per-sector analytical lens
- `P_weekly_overview_02` (Phần 6 Biến động 18 ngành) — structural watch khi có chuyển động bất thường

**Không thay thế** `K_agent_db_04` (methodology diễn giải chỉ báo). 2 pack bổ trợ nhau: `K_agent_db_04` chuyên về **dòng tiền + PTCB 4 type doanh nghiệp** từ data DB, `K_sector_framework` chuyên về **industry structure + competitive dynamics + ESG** từ chuẩn CFA.

---

## 5. DB agent (`agent_db/`) — chi tiết

### 5.1. Vai trò

Agent phân tích **single-shot, conversational**. Query MongoDB `agent_db`, kết hợp web search, đưa nhận định chuyên môn có luận cứ. Không có workflow đa stage, không có deliverable file, không có checkpoint.

Use case điển hình:
- Tra cứu nhanh: "VNM giá hôm nay", "KLGD HPG tuần qua"
- Hỏi nhận định nhanh: "thị trường tuần này dòng tiền thế nào", "ngành thép Q1 2026 có gì đáng chú ý"
- Lookup methodology: "chỉ báo zone AAA nghĩa là gì"

### 5.2. File structure

| File | Vai trò |
|---|---|
| `system_prompt.md` | v2 — file resident duy nhất (paste vào Custom Instructions): vai trò, tone, bản đồ collection, đơn vị, phase (tín hiệu tham chiếu), khuyến nghị + hiệu suất 2 tầng, meta-rules, bảng dịch rút gọn, manifest. Gộp `agent_db_00` cũ (đã nghỉ hưu) |
| `agent_db_01.md` | Schema 31 collection (+ Section I phase & danh mục) + URL pattern finext.vn |
| `agent_db_02.md` | Query patterns 13 workflow A-M (M = phase & danh mục) |
| `agent_db_03.md` | Anti-patterns 10 case (case 9-10 mới: bối cảnh phase, hiệu suất 2 tầng) |
| `agent_db_04.md` | Methodology diễn giải chỉ báo + PTCB 4 type doanh nghiệp + bảng dịch taxonomy đầu file |
| `agent_db_05.md` | News methodology — 4 loại tin + framework chấm impact |
| `agent_db_06.md` | Phase & 3 danh mục hệ thống: 4 trạng thái, exposure, 7 chỉ số, bộ số FROZEN + disclaimer |

### 5.3. Quan hệ với `K_agent_db_*` của analysis_agent

Content của 6 file `agent_db_01..06` (db agent) gần **identical** với `K_agent_db_01..06` (analysis_agent) — port 2026-07-13. Khác biệt chỉ:
- File name prefix + internal cross-reference (`agent_db_XX` vs `K_agent_db_XX`)
- Reference đến system_prompt section number (db agent: mục 5/8.x/9; analysis: mục 5.x + `K_agent_db_00` mục 4.x/5.x/6)
- `K_agent_db_00` là file riêng của analysis_agent (db agent v2 gộp master vào system_prompt; analysis giữ `_00` theo rule master-first)
- Audience: db agent = NĐT khách Finext (clarify nới lỏng); analysis = analyst nội bộ (clarify 2 câu giữ nguyên)

**Đây là chấp nhận có chủ đích**: 2 agent độc lập về deployment (2 Claude Desktop Project riêng), nên knowledge base duplicate. Trade-off: maintenance burden khi update methodology phải apply 2 chỗ; lợi ích: 2 agent hoàn toàn không coupling, có thể swap/extend độc lập.

**Lưu ý:** `K_sector_framework` (analysis_agent) **không** có bản sao trong db_agent — đây là pack chuyên cho deep-dive analysis workflow, không phục vụ single-shot lookup.

**Convention khi update methodology:**
- Sửa 1 nguồn (ưu tiên `agent_db/agent_db_*` vì đơn giản hơn) → manual port sang nguồn còn lại
- Hoặc dùng git để track diff giữa 2 nguồn, đảm bảo content sync

---

## 6. Communication giữa 2 agent

2 agent **không call cross-agent runtime**. Communication qua **MD file / context user copy/paste thủ công**:

```
┌─────────────────────┐
│  analysis_agent     │  → Output: MD final (memo / weekly / pitch)
│  (3 layer K/P/O)    │     User copy/save thủ công, dùng tool render
└─────────────────────┘     bên ngoài nếu cần binary (pptx/docx).

┌─────────────────────┐
│  db_agent           │  ← Tra cứu lẻ, không gắn với pipeline
│  (single-shot)      │     Output: response inline trong chat
└─────────────────────┘
```

**Workflow điển hình end-to-end:**
1. User mở `analysis_agent` project, request "viết báo cáo tổng quan tuần [DD/MM]" hoặc "báo cáo chiến lược tháng [N]"
2. analysis_agent chạy pack tương ứng (`P_weekly_overview` 2 stage 1 checkpoint hoặc `P_vbse_strategy` 4 stage 2 checkpoint), xuất MD final structured trong chat
3. User copy MD ra ngoài, nếu cần render branded pptx/docx thì dùng tool render bên ngoài (out of scope project này)

**db_agent dùng song song khi cần tra cứu nhanh** trong quá trình:
- "VNM giá đóng cửa hôm qua bao nhiêu?" → db_agent trả lời inline 1 câu, không cần qua workflow

---

## 7. Convention chuẩn

### 7.1. Locale vi-VN

- Số: dấu chấm ngăn nghìn, dấu phẩy thập phân — `18.200 tỷ`, `15,5%`
- Phần trăm có dấu rõ: `+18,2%` / `-3,5%`
- Tiền VND: `tỷ VND` cho tổng, `đồng` cho giá per share, `nghìn`/`k` cho giá ngắn (`33.000 đ` hoặc `33k`)
- Date: `Q1/2026`, `tháng 4/2026`, `ngày 27/4/2026`. Không dùng `Q1 2026` hay `2026-04-15` trong prose (OK trong bảng)

### 7.2. Ticker

- UPPERCASE, không nháy: `VNM` (không `'VNM'`, `vnm`)

### 7.3. K hygiene — không lộ ký hiệu raw

3 nhóm cần dịch trước khi xuất output:
- **Nhóm 1 — DB raw:** `vsi`, `day_score`, `week_score`, `zone: A/AA/AAA`, `f382`, `poc`, `period: "2025_4"`, `*_pct`, `*_trend`, `rank_pct`...
- **Nhóm 2 — Taxonomy nội bộ:** "Kịch bản A-G/E1-E3", "Pitfall F1-F12", "HIGH/MID/LOW impact", "framework chấm điểm", tên section như "B5/B6/B7"
- **Nhóm 3 — Thuật ngữ EN chưa dịch:** "mean-reversion", "exhaustion", "Value Trap", "dead-cat bounce", "priced-in"...

Bảng dịch đầy đủ ở `K_agent_db_00` mục 5 (analysis_agent) hoặc system prompt mục 9 (db agent v2).

**Exception:** `article_slug` / `report_slug` khi ghép thành URL `https://finext.vn/news/{slug}` là output hợp lệ.

### 7.4. Citation — 4 nhóm

| Nhóm | Format | Ví dụ |
|---|---|---|
| Nhóm 1 — Dữ liệu agent_db nội bộ | `(nguồn: Tổng hợp)` | `Revenue VNM 2025 đạt 18.200 tỷ VND (nguồn: Tổng hợp)` |
| Nhóm 2 — Tin/báo cáo finext.vn | Markdown link | `[Finext, 18/4/2026](https://finext.vn/news/<slug>)` |
| Nhóm 3 — PDF user upload | Tên tài liệu + trang | `BCTC VNM Q4/2025 soát xét, mục 8` |
| Nhóm 4 — Web external | Markdown link | `[NYU Stern country risk](https://pages.stern.nyu.edu/...)` |

### 7.5. File naming output

**analysis_agent:**
- `tier{N}_<YYYYMMDD>_confirmed.md` (state files cycle)
- `tier5C_<TICKER>_<YYYYMMDD>_confirmed.md` (memo deep-dive per-stock)
- `tier6_portfolio_<YYYYMMDD>_confirmed.md`
- `tier7_weekly_<YYYYMMDD>.md` / `tier7_monthly_<YYYYMM>.md` / `tier7_quarterly_<YYYY_Q>.md`
- `weekly_overview_<YYYYMMDD>.md` (ngày cuối tuần — Chủ Nhật)
- `vbse_strategy_monthly_<YYYYMM>.md` (tháng báo cáo chiến lược)
- `vbse_strategy_weekly_<YYYYMMDD>.md` (ngày cuối tuần update chiến lược)

### 7.6. Constraint cốt lõi (audience cuối có thể là KH)

Pack `P_weekly_overview` và `P_vbse_strategy` (có mode branded gửi KH) tuân chặt:
- **Không dùng chỉ báo trend nội bộ** (`*.trend`, `*_recent.recent_trend`) khi render branded — audience cuối không hiểu methodology
- **Không command** (mua/bán/giảm tỷ trọng) — diễn đạt observation
- **Không xác suất % cho kịch bản** — dùng if-then trigger objective
- **Không level giá vào/ra/stop trong watchlist** (chỉ luận điểm + signal theo dõi + disconfirming)
- **Conviction + horizon + disconfirming** bắt buộc mỗi call (chuẩn institutional)
- **Whitelist 18 ngành default, override khi user yêu cầu** — rank ngành tự tổng hợp theo `week_score`

`P_invest_memo` (audience analyst nội bộ) được dùng trend, target giá modeling, scoring framework cụ thể.

---

## 8. Design decisions chính

### 8.1. Render binary out of scope của analysis_agent

`analysis_agent` xuất **MD final** là output cuối. Render pptx/docx/xlsx là concern downstream của tool render bên ngoài (out of scope project này). MD final đã đủ structured (heading hierarchy + chart annotation YAML + citation + locale) để tool render consume.

Lý do tách: render binary là concern khác với analysis quality. Tách giúp `analysis_agent` focus vào content depth, không bị phân tán bởi presentation/branding.

**Note:** Tại rev 6, 16/16 section "Guide render docx/pptx" trong các O pack đã được marked `[LEGACY]` (kèm note "Render binary out of scope, section giữ làm reference cho tool render bên ngoài"). Content section giữ nguyên — pass cleanup tiếp theo có thể xoá hẳn nếu cần thu gọn knowledge base. Không ảnh hưởng workflow runtime vì master rule rev 6 đã chốt MD final là output cuối.

### 8.2. Triết lý flex+downgrade thay strict reject (analysis_agent)

Khi gate methodology không pass strict (Variant Perception yếu, Bear Case rebuttal yếu, R/R thấp), agent **không tự reject** mã. Thay vào đó:
- Flag cảnh báo cụ thể
- Downgrade conviction / size theo mức độ
- User quyết định cuối: proceed với size nhỏ + audit log, hoặc loại mã

Lý do: discipline ở dạng force user explicit aware về rủi ro, không che giấu. User là người ra quyết định — agent đưa thông tin đầy đủ, không tự ý filter.

**Exception — Nguyên tắc 5 (P_invest_memo) vẫn strict reject:** "Dòng tiền dương + catalyst tiêu cực → loại" giữ behavior strict (không flex+downgrade). Khác với 5 nguyên tắc còn lại — Variant Perception / Bear Case / R/R là đánh giá **chủ quan** có thể debate, còn pattern "dòng tiền dương + catalyst tiêu cực" là **objective historical pattern** với base rate lỗi rất cao (retail trap kinh điển ở thị trường VN: dòng tiền vào muộn priced-in tin xấu chưa lộ). Đưa cho user "quyết" với pattern này là ép user override discipline về 1 loại lỗi đã có evidence rõ. Giữ strict reject ở đây là design decision có chủ đích.

### 8.3. Knowledge base duplicate (analysis_agent K + db_agent agent_db)

2 pack content gần identical, chấp nhận duplicate vì priority "agent độc lập 100%" cao hơn DRY. Trade-off đã document ở mục 5.3.

### 8.4. Conviction memo mới được vào position (analysis_agent P_invest_memo)

Trong workflow đầu tư (P_invest_memo), không vào position nếu chưa hoàn thành memo deep-dive (Tier 5C). Memo là gate cuối cùng — viết được memo 7 phần (Recommendation / Thesis / Variant / Business / Financial / Catalysts / Bear / Exit) đủ chuẩn mới được conviction để sizing.

---

## 9. Hướng mở rộng

### 9.1. Thêm pack mới trong `analysis_agent`

Pattern cũ (3 layer K/P/O):
1. Identify domain — pack mới là K, P, hay O?
2. Tạo file theo naming convention: `K_{domain}_{NN}.md` / `P_{flow_name}_{NN}.md` / `O_{format_or_style}_{NN}.md`
3. Pack có ≥3 file phải có file `_00` master (mục đích pack + manifest file con + flow + output contract)
4. Thêm entry vào `KERNEL_SKELETON.md` với trigger activation
5. Re-upload toàn bộ analysis_agent project knowledge

### 9.2. Thêm domain mới (vd thị trường ngoài VN)

Hiện tại 2 agent đều scope cho thị trường VN (giả định MongoDB `agent_db` chứa data VN). Để extend ra thị trường khác:
- Build pack K mới (`K_us_market_*` chẳng hạn) cho schema/data nguồn US
- Build pack P mới phù hợp methodology US (DCF, peer multiples — khác VN ở P/E benchmark, dynamics ngành)
- Build pack O mới cho format US (USD, MM-DD-YYYY, etc.)
- KHÔNG mix VN + US trong cùng pack — methodology + locale + audience khác nhau

### 9.3. Thêm audience mới (vd retail, intern)

`analysis_agent` hiện assume audience analyst/broker nội bộ (được phép nhận khuyến nghị cụ thể). Để serve audience khác:
- Build agent riêng (project Claude Desktop riêng) với system_prompt + K pack restricted
- Không nên sửa K_agent_db cũ — break audience analyst hiện tại

---

## 10. Note về deployment legacy

**Lịch sử:** Một số file P/O packs trong `analysis_agent` từng có reference đến:
- Path `/mnt/user-data/outputs/` (Linux)
- Tool `present_files`
- Skill `/mnt/skills/public/docx`, `/mnt/skills/public/pptx`

Đây là legacy từ deployment Claude Code / Claude skill (môi trường có filesystem + skill tool). Trên **Claude Desktop** (môi trường hiện tại) các path/tool này không tồn tại.

**Trạng thái hiện tại (rev 6):** Các reference active đã được clean — thay bằng wording "xuất nội dung MD trong message (Claude Desktop), user copy/save thủ công". Các section render binary đã được marked `[LEGACY]`. Reference legacy còn lại chỉ trong các block historical note có chủ đích.

Output flow đúng trên Claude Desktop: Claude trả về MD final trong message, user copy/save thủ công, hoặc Claude tạo artifact trong chat.

---

## 11. Behavioral guidelines (`CLAUDE.md`)

File `CLAUDE.md` ở root chứa 4 nguyên tắc behavioral chung cho mọi session AI làm việc trên project này:

1. **Think Before Coding** — Don't assume, surface tradeoffs, ask before silently picking interpretation
2. **Simplicity First** — minimum code that solves the problem, no speculative features
3. **Surgical Changes** — touch only what must, clean up only own mess, match existing style
4. **Goal-Driven Execution** — define success criteria, loop until verified

Áp dụng khi maintain project: thêm pack, sửa methodology, refactor structure. Không upload vào project knowledge của 2 agent (2 agent có system_prompt riêng) — đây là guideline cho dev / AI dev assistant.

---

## 12. Khi gặp sự cố

### 12.1. Agent không hiểu request

- Verify đã upload đúng files vào project knowledge (analysis_agent: 50 file knowledge + system_prompt paste, db agent: 6 file `agent_db_01..06` + system_prompt paste)
- Verify Custom Instructions đã paste đúng `system_prompt.md` của agent đó
- Re-upload knowledge nếu vừa sửa file gốc

### 12.2. Output format không đúng

- Check đã match convention vi-VN ở mục 7.1 chưa
- Check K hygiene — có lộ ký hiệu DB raw không
- Check citation 4 nhóm có đầy đủ không

### 12.3. Workflow stuck ở checkpoint

- analysis_agent có checkpoint discipline strict: agent KHÔNG tự chuyển stage qua CP. User phải explicit confirm/override
- Nếu agent skip CP, có thể system_prompt chưa load đúng → re-paste Custom Instructions

### 12.4. K methodology không sync giữa analysis_agent và db_agent

- Hiện tại 2 nguồn duplicate, sync manual qua git (xem mục 5.3)
- Khi sửa methodology ở 1 nguồn → port sang nguồn còn lại bằng cách diff file tương ứng

---

## 13. Khi mở session Claude Desktop mới

Project knowledge đã đủ context cho AI hoạt động. Nhưng nếu cần dev/maintenance (sửa pack, thêm pack, audit), AI session sau nên đọc:

1. **`CLAUDE.md`** ở root — behavioral guidelines
2. **`README.md`** ở root (file này) — kiến trúc tổng thể
3. Tuỳ task:
   - Sửa analysis_agent → đọc `analysis_agent/system_prompt.md` + `analysis_agent/KERNEL_SKELETON.md` + master file của pack liên quan (`*_00.md`)
   - Sửa db agent → đọc `agent_db/system_prompt.md` (v2 — master gộp vào đây)

---

**Cập nhật convention/methodology:** thay đổi nội dung file → commit git → re-upload vào Claude Desktop project tương ứng. Không có auto-sync giữa filesystem và Claude Desktop project knowledge.
