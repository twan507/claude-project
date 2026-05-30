# Claude Project — Hệ thống agent phân tích chứng khoán & template báo cáo

Bộ 3 agent độc lập phục vụ phân tích cổ phiếu Việt Nam và render báo cáo branded. Mỗi agent là 1 Claude Desktop Project riêng. File này là điểm vào cho con người và AI session sau hiểu toàn bộ kiến trúc, biết khi nào dùng agent nào, khi nào extend cái nào.

---

## 1. Tổng quan dự án

### 1.1. Ba agent

| Agent | Folder | Mục đích | Số file |
|---|---|---|---|
| **analysis_agent** | `analysis_agent/` | Phân tích cổ phiếu Việt Nam đa giai đoạn, output MD final structured (memo, báo cáo tuần, khuyến nghị mua) | 29 |
| **template_agent** | `template_agent/` | Nhận MD/document bất kỳ → chuẩn hoá theo contract → render pptx branded (VBSE / Finext) | 8 |
| **db_agent** | `db_agent/` | Phân tích single-shot nhanh: tra cứu, query MongoDB `agent_db`, đưa nhận định lẻ không qua workflow đa stage | 7 |

### 1.2. Use case từng agent

- **analysis_agent** — khi cần một báo cáo deliverable hoàn chỉnh: viết memo deep-dive cho 1 mã, sinh báo cáo thị trường tuần 12 phần, soạn pitch khuyến nghị mua gửi khách hàng, lập portfolio plan, review cycle. Workflow có nhiều giai đoạn + checkpoint, output MD structured.
- **template_agent** — khi đã có content (do analysis_agent xuất ra, hoặc tài liệu sẵn từ nguồn khác) muốn trình bày dưới dạng pptx branded. Nhận input bất kỳ (PDF / DOCX / MD / paste), chuẩn hoá MD, hỏi pick brand, render binary.
- **db_agent** — khi chỉ cần tra cứu hoặc nhận định nhanh không cần qua workflow: "VNM giá bao nhiêu", "thị trường tuần qua dòng tiền thế nào", "ngành thép Q1 ra sao". Output dạng conversational, không cần file deliverable.

### 1.3. Quy tắc kiến trúc cốt lõi

**Mỗi agent độc lập 100%.** File trong agent này KHÔNG reference path / tên file của agent khác. Communication giữa các agent qua MD file mà user copy/paste thủ công, không có shared state hay cross-agent runtime call.

Hệ quả thực tế:
- Sửa 1 agent không phá agent khác
- Có thể swap / extend / tắt 1 agent độc lập
- Cùng 1 knowledge base có thể tồn tại ở 2 agent dưới dạng duplicate (hiện tại `K_agent_db_*` trong analysis_agent và `agent_db_*` trong db_agent có content gần identical — đây là chấp nhận được vì 2 agent độc lập, chứ không refactor về 1 nguồn).

---

## 2. Triển khai (Claude Desktop)

### 2.1. Mỗi agent = 1 Claude Desktop Project

Tạo 3 project riêng trong Claude Desktop app:

| Project name (đề xuất) | Source folder | Custom Instructions | Knowledge files |
|---|---|---|---|
| `Analysis Agent` | `analysis_agent/` | Paste nội dung `analysis_agent/system_prompt.md` | Upload toàn bộ file `.md` còn lại trong folder (28 file) |
| `Template Agent` | `template_agent/` | Paste nội dung `template_agent/system_prompt.md` | Upload `INDEX.md`, `FORMAT.md`, `WORKFLOW.md`, 2 `TEMPLATE_*.md` (5 file `.md`). **2 file `.pptx` KHÔNG upload được vào project knowledge** — user attach trong chat session khi cần render binary (xem mục 4.4 dưới) |
| `DB Agent` | `db_agent/` | Paste nội dung `db_agent/system_prompt.md` | Upload `agent_db_00` đến `agent_db_05` (6 file) |

### 2.2. Khi cần update file

Sửa file ở folder gốc → re-upload vào project knowledge tương ứng. Claude Desktop project knowledge không tự sync với filesystem — phải update thủ công.

Khi sửa `system_prompt.md` của agent nào → paste lại vào ô Custom Instructions của project đó.

### 2.3. MongoDB connection

`analysis_agent` và `db_agent` đều giả định có quyền read MongoDB database tên `agent_db`. Tools để query DB không nằm trong project knowledge — phải được provide qua Claude Desktop integration / MCP server cấu hình bên ngoài. Schema 25 collection và query patterns được document trong `agent_db_01.md` (db_agent) và `K_agent_db_01.md` (analysis_agent).

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
| `K_agent_db` | 6 (`_00` master + `_01` đến `_05`) | Knowledge MongoDB `agent_db` chứng khoán VN (25 collection) |
| `P_invest_memo` | 10 (`_00` master + `_01` đến `_09`) | Quy trình đầu tư cá nhân, horizon 1-6 tháng, long only, portfolio < 1 triệu USD |
| `P_weekly_overview` | 5 (`_00` master + `_01` đến `_04`) | Broadcast tổng quan thị trường tuần 12 phần fundamental-driven, audience nội bộ + KH |
| `P_vbse_strategy` | 10 (`_00` master + `_01` đến `_09`) | Chiến lược đầu tư VBSE deep nội bộ, 2 cycle (monthly parent + weekly child), 6 trục, 2-phase watchlist |
| `O_invest_memo` | 7 (`_00` master + `_01` đến `_06`) | Render spec cho 6 deliverables của `P_invest_memo` |
| `O_weekly_overview` | 1 (`_00`) | Render spec broadcast tuần 12 phần rigid + 3 mode branding |
| `O_vbse_strategy` | 1 (`_00`) | Render spec chiến lược 2 mode (monthly + weekly) flex 6 trục |

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

### 3.6. Triết lý flex+downgrade (xuyên suốt P_invest_memo + P_vbse_strategy)

Khi gate (Variant Perception, Bear Case, R/R) không pass strict, **agent KHÔNG tự reject** mã. Thay vào đó:
- Flag cảnh báo cụ thể (lý do gate yếu)
- Downgrade conviction / size theo mức độ yếu (ví dụ: bear yếu vừa → giảm size 30-50%, bear target dưới giá hiện tại → giảm size 50-70% coin-flip bet)
- User quyết định cuối: proceed với size nhỏ + audit log, hoặc loại mã

Triết lý: discipline ở dạng force user explicit aware về rủi ro, không che giấu. Chi tiết Gate 1 + Gate 2 ở `P_invest_memo_07.md` mục 4.

---

## 4. `template_agent` — chi tiết

### 4.1. Vai trò

Agent thuần **trình bày**. Nhận input bất kỳ định dạng (PDF / DOCX / MD / text / paste) → chuẩn hoá thành MD theo contract → render binary branded.

KHÔNG phân tích nội dung, KHÔNG sinh insight, KHÔNG query database.

### 4.2. File flat structure

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules vận hành agent (paste vào Custom Instructions) |
| `INDEX.md` | Manifest + workflow tổng quan |
| `FORMAT.md` | Spec MD chuẩn hoá — 9 report_types với section structure |
| `WORKFLOW.md` | Flow 7 stage + 3 checkpoint |
| `TEMPLATE_VBSE.md` / `.pptx` | Catalog 27 layout brand VBSE (navy + đỏ + tam giác vuông cân) |
| `TEMPLATE_FINEXT.md` / `.pptx` | Catalog 27 layout brand Finext (dark + violet + chevron `>>`) |

### 4.3. 9 report_type trong `FORMAT.md` mục 3

| report_type | Section count | Length | Audience |
|---|---|---|---|
| `stock_pitch` | 13-16 (rigid + 4-7 luận điểm flex) | 12-18 trang | KH |
| `weekly_market` | 12 rigid | 9-11 trang | Nội bộ + KH |
| `market_scan` | 7 flex top-down | 8-15 trang | Nội bộ |
| `stock_memo` | 3-7 theo conviction tier | 3-15 trang | Nội bộ |
| `portfolio_plan` | 8 flex | 6-10 trang | Nội bộ |
| `portfolio_review_weekly` | 6 rigid | 0.5-2 trang | Nội bộ |
| `portfolio_review_monthly` | 8 rigid | 3-5 trang | Nội bộ |
| `portfolio_review_quarterly` | 9 rigid | 5-8 trang | Nội bộ |
| `custom` | flex 3-15 (quiz-driven) | tùy user | tùy user |

### 4.4. Workflow 7 stage

```
Stage 1   Ingest (đọc input, extract content thô)
Stage 1.5 Detect skip-normalize (≥4/6 signals match → skip Stage 2-5, đi thẳng Stage 6)
Stage 2   Parse (LLM analyze report_type + section + ambiguities)
Stage 3   Clarify (multi-choice questions, gom 3-5 câu/turn; nếu custom: quiz 7 câu trong 2 turn)
CP1       Clarification confirm
Stage 4   Normalize (LLM produce MD theo FORMAT contract)
CP2       MD draft review (user confirm/edit/fix)
Stage 5   Finalize MD
Stage 6   Brand pre-flight (VBSE / Finext / chỉ MD)
CP3       Brand confirm + pptx upload check
Stage 7   Render binary
```

**Brand whitelist strict:** chỉ VBSE và Finext. Brand khác → reject, không fallback render plain branded. Nếu cần brand mới → build TEMPLATE pack mới (xem mục 9 Hướng mở rộng).

**Pptx template upload runtime:** File `.pptx` không upload được vào Claude Desktop project knowledge (chỉ accept text-based file). Hệ quả:
- Catalog `.md` (`TEMPLATE_VBSE.md`, `TEMPLATE_FINEXT.md`) — trong project knowledge
- Binary `.pptx` (`TEMPLATE_VBSE.pptx`, `TEMPLATE_FINEXT.pptx`) — user **attach trong chat session** trước Stage 7 render

Workflow: agent ở CP3 sau khi user pick brand sẽ check session attachments. Nếu chưa có pptx tương ứng → agent request user upload, không vào Stage 7. Nếu user pick "(c) chỉ MD" → skip Stage 7, output MD final, không cần pptx. Chi tiết rule ở `template_agent/system_prompt.md` mục 5.7.

### 4.5. Custom quiz (FORMAT mục 3.9 + WORKFLOW mục 5.4-5.5)

Khi user pick `custom` (hoặc Stage 2 detect không match preset), agent chạy quiz 7 câu chia 2 turn:
- Turn 1 (4 câu): Mục đích / Audience / Length target / Tone
- Turn 2 (3 câu + bonus): Số section / Chart count / Citation style + (optional) section list user paste

Sau quiz, agent build spec runtime + lưu `custom_spec_id` (timestamp) trong frontmatter. Re-render cùng MD `custom` đã có spec → skip-normalize, đi thẳng brand pre-flight.

### 4.6. TEMPLATE pack độc lập về authoring

TEMPLATE pack runtime chỉ consume 2 nguồn data:
1. File pack của chính nó (catalog `.md` + binary `.pptx`)
2. MD final do upstream pipeline produce làm input

Reference đến `WORKFLOW.md` Stage 5/7 trong file TEMPLATE chỉ là pointer runtime (khi nào activate, đọc input từ đâu), không phải derive content. Layout/design tokens/render rule của TEMPLATE độc lập với spec của FORMAT/WORKFLOW.

### 4.7. Independence rule pragmatic (system_prompt mục 2)

Hiện tại rule đã loosen từ "tuyệt đối không reference" sang **"minimal cross-reference cho clarity runtime, cấm backward authoring dependency"**:

- FORMAT định nghĩa MD contract — không depend WORKFLOW/TEMPLATE để define spec
- WORKFLOW đọc FORMAT để biết target output — không depend TEMPLATE để define flow
- TEMPLATE runtime consume MD final — không depend FORMAT/WORKFLOW để define layout

**Cấm:** FORMAT đọc WORKFLOW spec để define MD; WORKFLOW đọc TEMPLATE catalog để define flow; TEMPLATE đọc FORMAT/WORKFLOW spec để define layout. Đây là backward authoring dependency.

**Cho phép:** mention runtime activation point (vd "WORKFLOW Stage 7", "TEMPLATE_VBSE / TEMPLATE_FINEXT brand whitelist") như pointer cho người đọc tài liệu / agent runtime — không dùng để build/derive content.

---

## 5. `db_agent` — chi tiết

### 5.1. Vai trò

Agent phân tích **single-shot, conversational**. Query MongoDB `agent_db`, kết hợp web search, đưa nhận định chuyên môn có luận cứ. Không có workflow đa stage, không có deliverable file, không có checkpoint.

Use case điển hình:
- Tra cứu nhanh: "VNM giá hôm nay", "KLGD HPG tuần qua"
- Hỏi nhận định nhanh: "thị trường tuần này dòng tiền thế nào", "ngành thép Q1 2026 có gì đáng chú ý"
- Lookup methodology: "chỉ báo zone AAA nghĩa là gì"

### 5.2. File structure

| File | Vai trò |
|---|---|
| `system_prompt.md` | Meta-rules (paste vào Custom Instructions) — 88 dòng, simpler analysis_agent system_prompt |
| `agent_db_00.md` | Master: mục đích, scope, manifest, domain rules, K hygiene, quy đổi đơn vị |
| `agent_db_01.md` | Schema 25 collection + URL pattern finext.vn |
| `agent_db_02.md` | Query patterns 12 workflow A-L |
| `agent_db_03.md` | Anti-patterns + case study lỗi quá khứ |
| `agent_db_04.md` | Methodology diễn giải chỉ báo + PTCB 4 type doanh nghiệp |
| `agent_db_05.md` | News methodology — 4 loại tin + framework chấm impact |

### 5.3. Quan hệ với `K_agent_db_*` của analysis_agent

Content của 6 file `agent_db_*` (db_agent) gần **identical 99%** với 6 file `K_agent_db_*` (analysis_agent). Khác biệt chỉ:
- File name prefix
- Internal cross-reference (`agent_db_XX` vs `K_agent_db_XX`)
- Reference đến system_prompt section number (2 system_prompt khác nhau)
- 1 vài legacy comment ("Rule 6") trong K_agent_db_04 (analysis_agent)

**Đây là chấp nhận có chủ đích**: 2 agent độc lập về deployment (3 Claude Desktop Project riêng), nên knowledge base duplicate. Trade-off: maintenance burden khi update methodology phải apply 2 chỗ; lợi ích: 2 agent hoàn toàn không coupling, có thể swap/extend độc lập.

**Convention khi update methodology:**
- Sửa 1 nguồn (ưu tiên `db_agent/agent_db_*` vì đơn giản hơn) → manual port sang nguồn còn lại
- Hoặc dùng git để track diff giữa 2 nguồn, đảm bảo content sync

---

## 6. Communication giữa 3 agent

3 agent **không call cross-agent runtime**. Communication qua **MD file mà user copy/paste thủ công**:

```
┌─────────────────────┐
│  analysis_agent     │  → Output: MD final (memo / weekly / pitch)
│  (3 layer K/P/O)    │
└──────────┬──────────┘
           │ (user copy MD)
           ▼
┌─────────────────────┐
│  template_agent     │  → Output: pptx branded (VBSE / Finext)
│  (7 stage)          │
└─────────────────────┘

┌─────────────────────┐
│  db_agent           │  ← Tra cứu lẻ, không gắn với pipeline
│  (single-shot)      │     Output: response inline trong chat
└─────────────────────┘
```

**Workflow điển hình end-to-end:**
1. User mở `analysis_agent` project, request "viết báo cáo tổng quan tuần [DD/MM]" hoặc "báo cáo chiến lược tháng [N]"
2. analysis_agent chạy pack tương ứng (`P_weekly_overview` 2 stage 1 checkpoint hoặc `P_vbse_strategy` 4 stage 2 checkpoint), xuất MD final structured
3. User copy MD → mở `template_agent` project → paste vào chat
4. template_agent detect skip-normalize (input đã match contract) → hỏi pick brand
5. User pick VBSE → template_agent render pptx/docx branded
6. User download file, gửi audience

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

Bảng dịch đầy đủ ở `K_agent_db_00` mục 5 (analysis_agent) hoặc `agent_db_00` mục 5 (db_agent).

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

**template_agent:**
- MD chuẩn hoá: theo `report_type` + ngày/ticker (xem `WORKFLOW.md` mục 9)
- Binary: `<report_type>_<id>_<YYYYMMDD>_<brand>.pptx` với `<brand>` = `vbse` hoặc `finext`

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

`analysis_agent` xuất **MD final** là output cuối. Render pptx/docx/xlsx là concern downstream của `template_agent` (hoặc tool render bên ngoài). MD final đã đủ structured (heading hierarchy + chart annotation YAML + citation + locale) để consume.

Lý do tách: render binary là concern khác với analysis quality. Tách giúp `analysis_agent` focus vào content depth, `template_agent` focus vào visual presentation.

**Note:** Tại rev 6, 16/16 section "Guide render docx/pptx" trong các O pack đã được marked `[LEGACY]` (kèm note "Render binary out of scope, section giữ làm reference cho tool render bên ngoài"). Content section giữ nguyên — pass cleanup tiếp theo có thể xoá hẳn nếu cần thu gọn knowledge base. Không ảnh hưởng workflow runtime vì master rule rev 6 đã chốt MD final là output cuối.

### 8.2. Triết lý flex+downgrade thay strict reject (analysis_agent)

Khi gate methodology không pass strict (Variant Perception yếu, Bear Case rebuttal yếu, R/R thấp), agent **không tự reject** mã. Thay vào đó:
- Flag cảnh báo cụ thể
- Downgrade conviction / size theo mức độ
- User quyết định cuối: proceed với size nhỏ + audit log, hoặc loại mã

Lý do: discipline ở dạng force user explicit aware về rủi ro, không che giấu. User là người ra quyết định — agent đưa thông tin đầy đủ, không tự ý filter.

**Exception — Nguyên tắc 5 (P_invest_memo) vẫn strict reject:** "Dòng tiền dương + catalyst tiêu cực → loại" giữ behavior strict (không flex+downgrade). Khác với 5 nguyên tắc còn lại — Variant Perception / Bear Case / R/R là đánh giá **chủ quan** có thể debate, còn pattern "dòng tiền dương + catalyst tiêu cực" là **objective historical pattern** với base rate lỗi rất cao (retail trap kinh điển ở thị trường VN: dòng tiền vào muộn priced-in tin xấu chưa lộ). Đưa cho user "quyết" với pattern này là ép user override discipline về 1 loại lỗi đã có evidence rõ. Giữ strict reject ở đây là design decision có chủ đích.

### 8.3. Brand whitelist strict (template_agent)

Chỉ render được 2 brand: VBSE và Finext. Brand khác → reject, không fallback render plain.

Lý do: bảo đảm output luôn match 1 trong 2 brand chuẩn hoặc không có output. Tránh sản phẩm half-baked có thể bị gửi nhầm cho KH dưới brand không official.

### 8.4. Skip-normalize fast path (template_agent)

Khi input MD đã match `FORMAT.md` contract (≥4/6 signals: frontmatter + heading + section count + chart YAML + citation + locale), template_agent skip Stage 2-5 (parse, clarify, normalize, finalize) → đi thẳng Stage 6 brand pre-flight.

Lý do: MD từ analysis_agent đã chuẩn rồi không cần re-normalize. Tiết kiệm thời gian + tránh LLM overwrite content có chủ đích.

### 8.5. Knowledge base duplicate (analysis_agent K + db_agent agent_db)

2 pack content gần identical, chấp nhận duplicate vì priority "agent độc lập 100%" cao hơn DRY. Trade-off đã document ở mục 5.3.

### 8.6. Independence rule pragmatic (template_agent system_prompt mục 2)

Loosen rule "tuyệt đối không reference" → "minimal cross-reference cho clarity runtime, cấm backward authoring dependency". Lý do: rule strict tuyệt đối làm vô nghĩa pointer runtime cần thiết (FORMAT mention WORKFLOW Stage, TEMPLATE mention WORKFLOW Stage 7, etc.).

### 8.7. Conviction memo mới được vào position (analysis_agent P_invest_memo)

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

### 9.2. Thêm `report_type` mới trong `template_agent`

1. Thêm row trong bảng `FORMAT.md` mục 2.1 (frontmatter whitelist) + spec section structure ở mục 3.X
2. Update `INDEX.md` bảng "9 loại báo cáo" → "10 loại"
3. Update `WORKFLOW.md` mục 4.1 detection list + mục 9 naming convention output + mục 12.1 binary naming
4. Re-upload template_agent project knowledge

### 9.3. Thêm brand mới trong `template_agent`

1. Build template pptx mới: 27 layout với cấu trúc 1-1 mapping với TEMPLATE_VBSE/TEMPLATE_FINEXT (để runtime fit cùng MD content)
2. Tạo file `TEMPLATE_<BRAND>.md` catalog: design tokens, layout list, render rules
3. Update `INDEX.md` brand whitelist từ 2 → 3
4. Update `system_prompt.md` mục 5.4 brand whitelist
5. Update `WORKFLOW.md` mục 10 brand pre-flight question + mục 11 CP3 routing
6. Re-upload template_agent project knowledge

### 9.4. Thêm domain mới (vd thị trường ngoài VN)

Hiện tại 3 agent đều scope cho thị trường VN (giả định MongoDB `agent_db` chứa data VN). Để extend ra thị trường khác:
- Build pack K mới (`K_us_market_*` chẳng hạn) cho schema/data nguồn US
- Build pack P mới phù hợp methodology US (DCF, peer multiples — khác VN ở P/E benchmark, dynamics ngành)
- Build pack O mới cho format US (USD, MM-DD-YYYY, etc.)
- KHÔNG mix VN + US trong cùng pack — methodology + locale + audience khác nhau

### 9.5. Thêm audience mới (vd retail, intern)

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

Áp dụng khi maintain project: thêm pack, sửa methodology, refactor structure. Không upload vào project knowledge của 3 agent (3 agent có system_prompt riêng) — đây là guideline cho dev / AI dev assistant.

---

## 12. Khi gặp sự cố

### 12.1. Agent không hiểu request

- Verify đã upload đúng files vào project knowledge (analysis_agent: 25 file, template_agent: 7 file, db_agent: 6 file)
- Verify Custom Instructions đã paste đúng `system_prompt.md` của agent đó
- Re-upload knowledge nếu vừa sửa file gốc

### 12.2. Output format không đúng

- Check đã match convention vi-VN ở mục 7.1 chưa
- Check K hygiene — có lộ ký hiệu DB raw không
- Check citation 4 nhóm có đầy đủ không

### 12.3. Workflow stuck ở checkpoint

- analysis_agent / template_agent có checkpoint discipline strict: agent KHÔNG tự chuyển stage qua CP. User phải explicit confirm/override
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
   - Sửa template_agent → đọc `template_agent/system_prompt.md` + `template_agent/INDEX.md` + `FORMAT.md` + `WORKFLOW.md`
   - Sửa db_agent → đọc `db_agent/system_prompt.md` + `db_agent/agent_db_00.md`

---

**Cập nhật convention/methodology:** thay đổi nội dung file → commit git → re-upload vào Claude Desktop project tương ứng. Không có auto-sync giữa filesystem và Claude Desktop project knowledge.
